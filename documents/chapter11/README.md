# 대형 모델을 이용한 실습 학습: RLHF

> 본 실습 매뉴얼은 온라인 자료 [블로그](https://newfacade.github.io/notes-on-reinforcement-learning/17-ppo-trl.html) 및 [trl 예제](https://github.com/huggingface/trl/blob/main/examples/notebooks/gpt2-sentiment.ipynb)를 번역하고 통합한 것입니다.

재현 실험 구성: 단일 카드 NVDIA A800-SXM4-80GB는 10097MiB를 차지하고 학습에는 35분 19초가 소요됩니다.

튜토리얼 읽기: [슬라이드](./RLHF.pdf)

노트북:[노트북](./RLHF.ipynb)

## PPO 작동 방식
1. 롤아웃: 언어 모델은 쿼리를 기반으로 응답을 생성합니다.
2. 평가: 쿼리와 응답은 기능, 모델, 인간 피드백 또는 이들의 조합을 사용하여 평가됩니다. 이 프로세스는 각 쿼리/응답 쌍에 대해 **스칼라 값**을 생성해야 합니다.
3. 최적화: 최적화 단계에서는 쿼리/응답 쌍을 사용하여 시퀀스에 있는 토큰의 로그 확률을 계산합니다. 이는 훈련된 모델과 참조 모델을 통해 수행됩니다. 두 출력 간의 KL 차이는 생성된 응답이 참조 언어 모델에서 너무 멀리 벗어나지 않도록 하기 위한 추가 보상 신호로 사용됩니다. 그런 다음 활성 언어 모델은 PPO를 사용하여 훈련됩니다.
<div style="text-align: center">
<img src='figs/trl1.png' width='600'>
<p style="text-align: center;"> <b> 그림: </b> PPO 흐름 차트 </p>
</div>

# GPT-2를 미세 조정하여 긍정적인 리뷰 생성
> BERT 감정 분류기를 보상 기능으로 사용하여 긍정적인 IMDB 영화 리뷰를 생성하도록 GPT-2를 최적화합니다.

<div style="text-align: center">
<img src='figs/gpt2_bert_training.png' width='600'>
<p style="text-align: center;"> <b> 그림: </b> GPT-2에 대한 실험 설정 미세 조정 </p>
</div>

우리는 IMDB 데이터 세트를 기반으로 긍정적인 영화 리뷰를 생성하기 위해 GPT-2를 미세 조정합니다. 모델은 실제 검토의 시작을 받고 긍정적인 후속 콘텐츠를 생성해야 합니다. 긍정적인 후속 콘텐츠를 보상하기 위해 BERT 분류기를 사용하여 생성된 문장의 감정을 분석하고 분류기의 출력을 PPO 훈련에 대한 보상 신호로 사용합니다.

## 실험적 설정

### 모델 및 데이터 다운로드
데이터세트
```bash
export HF_ENDPOINT=https://hf-mirror.com; huggingface-cli download --resume-download stanfordnlp/imdb --local-dir dataset/imdb --repo-type dataset
```
참조 모델
```bash
export HF_ENDPOINT=https://hf-mirror.com; huggingface-cli download --resume-download lvwerra/gpt2-imdb --local-dir model/gpt2-imdb
```
보상 모델
```bash
export HF_ENDPOINT=https://hf-mirror.com; huggingface-cli download --resume-download lvwerra/distilbert-imdb --local-dir model/distilbert-imdb
```
### 종속성 가져오기
```python
# %pip install -r requirements.txt
# import os
# os.environ['CUDA_VISIBLE_DEVICES'] = '7'
```


```python
import torch
from tqdm import tqdm
import pandas as pd

tqdm.pandas()

from transformers import pipeline, AutoTokenizer
from datasets import load_dataset

from trl import PPOTrainer, PPOConfig, AutoModelForCausalLMWithValueHead
from trl.core import LengthSampler
```
### 구성
```python
config = PPOConfig(
    model_name="model/gpt2-imdb",
    learning_rate=1.41e-5,
    log_with="wandb",
)

sent_kwargs = {"top_k": None, "function_to_apply": "none", "batch_size": 16}
```


```python
import wandb

wandb.init()
```
`gpt2_imdb`이라는 GPT-2 모델을 로드한 것을 볼 수 있습니다. 이 모델은 Hugging Face의 [스크립트](https://github.com/huggingface/transformers/blob/main/examples/legacy/run_language_modeling.py)(특별 설정 없음)를 사용하여 IMDB 데이터 세트의 추가 1개 시대에 대해 미세 조정되었습니다. 나머지 매개변수는 주로 원본 논문 "[Fine-Tuning Language Models from Human Preferences](https://huggingface.co/papers/1909.08593)"에서 가져왔습니다. 이 모델과 BERT 모델은 모두 Hugging Face의 모델 라이브러리에서 사용할 수 있으며, 구체적인 링크는 [여기](https://huggingface.co/models)입니다.

## 데이터 및 모델 로드

### IMDB 데이터 세트 로드
IMDB 데이터 세트에는 "긍정적"/"부정적" 감정 피드백이 표시된 50,000개의 영화 리뷰가 포함되어 있습니다. IMDB 데이터 세트를 DataFrame에 로드하고 길이가 200자 이상인 리뷰를 필터링합니다. 그런 다음 각 텍스트 조각을 토큰화하고 `LengthSampler`를 사용하여 지정된 길이로 무작위로 자릅니다.
```python
def build_dataset(
    config,
    dataset_name="dataset/imdb",
    input_min_text_length=2,
    input_max_text_length=8,
):
    """
    Build dataset for training. This builds the dataset from `load_dataset`, one should
    customize this function to train the model on its own dataset.

    Args:
        dataset_name (`str`):
            The name of the dataset to be loaded.

    Returns:
        dataloader (`torch.utils.data.DataLoader`):
            The dataloader for the dataset.
    """
    tokenizer = AutoTokenizer.from_pretrained(config.model_name)
    tokenizer.pad_token = tokenizer.eos_token
    # load imdb with datasets
    ds = load_dataset(dataset_name, split="train")
    ds = ds.rename_columns({"text": "review"})
    ds = ds.filter(lambda x: len(x["review"]) > 200, batched=False)

    input_size = LengthSampler(input_min_text_length, input_max_text_length)

    def tokenize(sample):
        sample["input_ids"] = tokenizer.encode(sample["review"])[: input_size()]
        sample["query"] = tokenizer.decode(sample["input_ids"])
        return sample

    ds = ds.map(tokenize, batched=False)
    ds.set_format(type="torch")
    return ds
```


```python
dataset = build_dataset(config)


def collator(data):
    return dict((key, [d[key] for d in data]) for key in data[0])
```
### 사전 훈련된 GPT2 언어 모델 로드
우리는 값 헤드를 사용하여 GPT2 모델과 토크나이저를 로드합니다. 모델을 두 번 로드했습니다. 첫 번째 모델은 최적화에 사용되었고, 두 번째 모델은 초기점으로부터 KL-divergence를 계산하기 위한 참조로 사용되었습니다. 이는 최적화된 모델이 원래 언어 모델에서 너무 멀리 벗어나지 않도록 PPO 교육 중에 추가 보너스 신호 역할을 합니다.
```python
model = AutoModelForCausalLMWithValueHead.from_pretrained(config.model_name)
ref_model = AutoModelForCausalLMWithValueHead.from_pretrained(config.model_name)
tokenizer = AutoTokenizer.from_pretrained(config.model_name)

tokenizer.pad_token = tokenizer.eos_token
```
### PPOTrainer 초기화
`PPOTrainer`은 후속 장비 할당 및 최적화를 담당합니다.
```python
ppo_trainer = PPOTrainer(
    config, model, ref_model, tokenizer, dataset=dataset, data_collator=collator
)
```
### BERT 분류기 로드
IMDB 데이터세트에 미세 조정된 BERT 분류기를 로드했습니다.
```python
device = ppo_trainer.accelerator.device
if ppo_trainer.accelerator.num_processes == 1:
    device = 0 if torch.cuda.is_available() else "cpu"  # to avoid a `pipeline` bug
sentiment_pipe = pipeline(
    "sentiment-analysis", model="model/distilbert-imdb", device=device
)
```
cuda:0을 사용하도록 설정된 장치


모델은 음수 및 양수 클래스의 로짓을 출력합니다. 우리는 포지티브 클래스의 로짓을 언어 모델의 보상 신호로 사용할 것입니다.
```python
text = "this movie was really bad!!"
sentiment_pipe(text, **sent_kwargs)
```




    [{'label': 'NEGATIVE', 'score': 2.3350484371185303},
     {'label': 'POSITIVE', 'score': -2.726576089859009}]




```python
text = "this movie was really good!!"
sentiment_pipe(text, **sent_kwargs)
```
[{'라벨': '양성', '점수': 2.557040214538574},
     {'라벨': 'NEGATIVE', '점수': -2.294790267944336}]



### 빌드 설정
응답 생성을 위해 샘플링 방법만 사용하고 top-k 및 커널 샘플링을 끄고 최소 길이를 설정합니다.
```python
gen_kwargs = {
    "min_length": -1,
    "top_k": 0.0,
    "top_p": 1.0,
    "do_sample": True,
    "pad_token_id": tokenizer.eos_token_id,
}
```
## 모델 최적화

### 훈련 루프

훈련 루프는 다음과 같은 주요 단계로 구성됩니다.
1. 정책 네트워크(GPT-2)에서 쿼리 응답 받기
2. BERT로부터 질의/응답 감정 얻기
3. (쿼리, 응답, 보상) 삼중항을 활용한 PPO 최적화 전략을 사용합니다.
```python
output_min_length = 4
output_max_length = 16
output_length_sampler = LengthSampler(output_min_length, output_max_length)


generation_kwargs = {
    "min_length": -1,
    "top_k": 0.0,
    "top_p": 1.0,
    "do_sample": True,
    "pad_token_id": tokenizer.eos_token_id,
}


for epoch, batch in enumerate(tqdm(ppo_trainer.dataloader)):
    query_tensors = batch["input_ids"]

    #### Get response from gpt2
    response_tensors = []
    for query in query_tensors:
        gen_len = output_length_sampler()
        generation_kwargs["max_new_tokens"] = gen_len
        query_response = ppo_trainer.generate(query, **generation_kwargs).squeeze()
        response_len = len(query_response) - len(query)
        response_tensors.append(query_response[-response_len:])
    batch["response"] = [tokenizer.decode(r.squeeze()) for r in response_tensors]

    #### Compute sentiment score
    texts = [q + r for q, r in zip(batch["query"], batch["response"])]
    pipe_outputs = sentiment_pipe(texts, **sent_kwargs)
    positive_scores = [
        item["score"]
        for output in pipe_outputs
        for item in output
        if item["label"] == "POSITIVE"
    ]
    rewards = [torch.tensor(score) for score in positive_scores]

    #### Run PPO step
    stats = ppo_trainer.step(query_tensors, response_tensors, rewards)
    ppo_trainer.log_stats(stats, batch, rewards)
```
0%| | 0/194 [00:00<?, ?it/s]The attention mask is not set and cannot be inferred from input because pad token is same as eos token. As a consequence, you may observe unexpected behavior. Please pass your input's `attention_mask` to obtain reliable results.
      4%|▍         | 8/194 [01:23<32:18, 10.42s/it]You seem to be using the pipelines sequentially on GPU. In order to maximize efficiency please use a dataset
    100%|██████████| 194/194 [35:19<00:00, 10.92s/it]


### 훈련 진행
훈련 진행 상황을 추적하기 위해 가중치 및 편향을 사용하는 경우 아래와 유사한 곡선이 표시됩니다. wandb.ai에서 대화형 예시 보고서를 확인하세요: [링크](https://wandb.ai/huggingface/trl/runs/w9l3110g).
<div style="text-align: center">
<img src='figs/gpt2_tuning_progress.png' width='800'>
<p style="text-align: center;"> <b> 그림: </b> 훈련 중 보상 평균의 진화 </p>
</div>
여러 최적화 단계 후에 모델이 더 많은 긍정적인 결과를 생성하기 시작하는 것을 볼 수 있습니다.  

## 모델 확인
IMDB 데이터세트의 몇 가지 예를 살펴보겠습니다. `ref_model`를 사용하여 최적화된 모델 `model`을 사전 최적화된 모델과 비교할 수 있습니다.
```python
#### get a batch from the dataset
bs = 16
game_data = dict()
dataset.set_format("pandas")
df_batch = dataset[:].sample(bs)
game_data["query"] = df_batch["query"].tolist()
query_tensors = df_batch["input_ids"].tolist()

response_tensors_ref, response_tensors = [], []

#### get response from gpt2 and gpt2_ref
for i in range(bs):
    query = torch.tensor(query_tensors[i]).to(device)

    gen_len = output_length_sampler()
    query_response = ref_model.generate(
        query.unsqueeze(0), max_new_tokens=gen_len, **gen_kwargs
    ).squeeze()
    response_len = len(query_response) - len(query)
    response_tensors_ref.append(query_response[-response_len:])

    query_response = model.generate(
        query.unsqueeze(0), max_new_tokens=gen_len, **gen_kwargs
    ).squeeze()
    response_len = len(query_response) - len(query)
    response_tensors.append(query_response[-response_len:])

#### decode responses
game_data["response (before)"] = [
    tokenizer.decode(response_tensors_ref[i]) for i in range(bs)
]
game_data["response (after)"] = [
    tokenizer.decode(response_tensors[i]) for i in range(bs)
]

#### sentiment analysis of query/response pairs before/after
texts = [q + r for q, r in zip(game_data["query"], game_data["response (before)"])]
pipe_outputs = sentiment_pipe(texts, **sent_kwargs)
positive_scores = [
    item["score"]
    for output in pipe_outputs
    for item in output
    if item["label"] == "POSITIVE"
]
game_data["rewards (before)"] = positive_scores

texts = [q + r for q, r in zip(game_data["query"], game_data["response (after)"])]
pipe_outputs = sentiment_pipe(texts, **sent_kwargs)
positive_scores = [
    item["score"]
    for output in pipe_outputs
    for item in output
    if item["label"] == "POSITIVE"
]
game_data["rewards (after)"] = positive_scores

# store results in a dataframe
df_results = pd.DataFrame(game_data)
df_results
```
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>쿼리</th>
      <th>응답(이전)</th>
      <th>응답(이후)</th>
      <th>보상(이전)</th>
      <th>보상(이후)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>글쎄 나도 알아요</td>
      <td>Cantor는</td>일 수 있습니다.
      <td>..하지만 정말 좋았어요</td>
      <td>0.230196</td>
      <td>2.281557</td>
    </tr>
    <tr>
      <th>1</th>
      <td>정말 훌륭해요,</td>
      <td>일반적인 방식의 직접 비디오 필름</td>
      <td>즐거운 영화입니다.&lt;|텍스트 끝|></td>
      <td>2.846593</td>
      <td>2.840860</td>
    </tr>
    <tr>
      <th>2</th>
      <td>이제, I</td>
      <td> 제임스와 함께할 기회가 없었어요</td>
      <td>은 성장하는 에피소드를 좋아했으며 </td>은(는)
      <td>0.656194</td>
      <td>2.525894</td>
    </tr>
    <tr>
      <th>3</th>
      <td>우리는 경향이 있습니다</td>
      <td>아서를 만나지 않으려면</td>
      <td>이것을 매우 좋아합니다</td>
      <td>-0.280880</td>
      <td>2.183822</td>
    </tr>
    <tr>
      <th>4</th>
      <td>속담 "책을 판단하지 말라</td>
      <td>by the 표지"가 인기를 끌었습니다. 유약 처리 후...</td>
      <td>칭찬이 많지만 추천합니다...</td>
      <td>0.274649</td>
      <td>2.065951</td>
    </tr>
    <tr>
      <th>5</th>
      <td>이해한 적이 없습니다</td>
      <td>예술가가 왜 그렇게 많은지,</td>
      <td>이 영화는 재미있지만</td>
      <td>0.835574</td>
      <td>2.782384</td>
    </tr>
    <tr>
      <th>6</th>
      <td>휴(에드 해리스)는</td>입니다
      <td>칭찬받는 "영웅"과 그의 약혼자</td>
      <td>적응이 좋은 멋진 배우</td>
      <td>1.580167</td>
      <td>2.602940</td>
    </tr>
    <tr>
      <th>7</th>
      <td>이 특별한 Joe McDoakes</td>
      <td>' 에피소드에서 모든 잘못된 부분이 발생했고 </td>
      <td>movie는 정말 훌륭한 영화입니다. 그것</td>
      <td>0.870956</td>
      <td>2.795245</td>
    </tr>
    <tr>
      <th>8</th>
      <td>자매들</td>
      <td>Vrooms 8.23, </td> 전체에 가입했습니다.
      <td>우주1: 써니는 귀엽고, 귀엽기도 하고...</td>
      <td>1.175259</td>
      <td>2.062330</td>
    </tr>
    <tr>
      <th>9</th>
      <td>나는 이것을 매우 좋아했습니다</td>
      <td>필름, 처음에는 분명히 나쁜 생각이었습니다...</td>
      <td>쇼, 내가 여러 번 본 것을 알아두세요</td>
      <td>1.058164</td>
      <td>2.511273</td>
    </tr>
    <tr>
      <th>10</th>
      <td>그가 되고 싶다면</td>
      <td>재밌네요, 그는 </td> 할 수 있습니다
      <td>결국 천재,</td>
      <td>-0.388943</td>
      <td>0.405888</td>
    </tr>
    <tr>
      <th>11</th>
      <td>그건 내꺼야</td>
      <td>등급...&lt;br /&gt;&lt;br /&gt;</td>에도 불구하고
      <td>Way는 제가 본 최고의 영화였습니다.</td>
      <td>-0.151680</td>
      <td>2.473050</td>
    </tr>
    <tr>
      <th>12</th>
      <td>이것은 아마도 가장 좋은 단편일 것입니다</td>
      <td>약 2년만에 만난 영화</td>
      <td>영화가 작성된 적이 있습니다. 매우 기억에 남는 내용이 있습니다...</td>
      <td>2.511835</td>
      <td>2.775994</td>
    </tr>
    <tr>
      <th>13</th>
      <td>어떤 사람들은 이것이</td>라고 말합니다.
      <td>할리우드에서 정확히 무슨 일이 일어나는지; 어디로...</td>
      <td>청취할 만한 강력한 영화입니다. 정말 </td>을 포착합니다.
      <td>0.637631</td>
      <td>2.821085</td>
    </tr>
    <tr>
      <th>14</th>
      <td></td>의 리메이크
      <td>"오즈의 마법사</td>
      <td>전설적인 킹간 오일</td>
      <td>0.292409</td>
      <td>0.434021</td>
    </tr>
    <tr>
      <th>15</th>
      <td>정말 끔찍하네요</td>
      <td>영화!<|텍스트 끝|></td>
      <td>다지는 소리가 너무 좋아서 너무 좋아요! 우리는 </td>을 가지고 있습니다
      <td>-2.681461</td>
      <td>2.340650</td>
    </tr>
  </tbody>
</table>


생성된 시퀀스의 평균/중간 보상을 살펴보면서 중요한 차이점을 발견했습니다.
```python
print("mean:")
display(df_results[["rewards (before)", "rewards (after)"]].mean())
print()
print("median:")
display(df_results[["rewards (before)", "rewards (after)"]].median())
```
의미:

    보상 (이전) 0.591666
    보상 (이후) 2.243934
    dtype: float64


​
    중앙값:

    보상 (이전) 0.646912
    보상 (이후) 2.492161
    dtype: float64


## 모델 저장
마지막으로 나중에 사용할 수 있도록 모델을 저장합니다.
```python
model.save_pretrained("model/gpt2-imdb-pos-v2")
tokenizer.save_pretrained("model/gpt2-imdb-pos-v2")
```




    ('model/gpt2-imdb-pos-v2/tokenizer_config.json',
     'model/gpt2-imdb-pos-v2/special_tokens_map.json',
     'model/gpt2-imdb-pos-v2/vocab.json',
     'model/gpt2-imdb-pos-v2/merges.txt',
     'model/gpt2-imdb-pos-v2/added_tokens.json',
     'model/gpt2-imdb-pos-v2/tokenizer.json')

