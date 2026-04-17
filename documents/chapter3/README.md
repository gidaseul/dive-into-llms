# 대형 모델 실습 학습: 대형 모델 지식 편집
소개: 언어 모델 편집 방법 및 도구
> 특정 지식에 대한 언어 모델의 메모리를 제어하고 싶으십니까? 적절한 편집 방법을 선택하고, 특정 지식을 편집하고, 편집된 모델을 검증해보자!

## 1. 이 튜토리얼의 목표:

- EasyEdit 툴킷 사용에 익숙하신 분
- 언어 모델의 편집 방법을 마스터하세요(가장 간단함)
- 다양한 유형의 편집 방법에 대한 선택 및 적용 시나리오를 이해합니다.

## 2. 작업 준비:
### 2.1 EasyEdit 이해하기

https://github.com/zjunlp/EasyEdit

EasyEdit은 GPT-J, Llama, GPT-NEO, GPT2, T5 등과 같은 언어 모델을 편집하기 위한 Python 패키지입니다. 이 패키지의 목표는 사용하기 쉽고 확장하기 쉬운 동시에 다른 입력의 성능에 부정적인 영향을 주지 않고 특정 지식에 대한 언어 모델의 동작을 효과적으로 변경하는 것입니다.

EasyEdit은 기존의 널리 사용되는 편집 방법을 통합합니다.
![](./assets/1.png)

### 2.2 메인 프레임워크

![](./assets/2.png)
EasyEdit에는 각각 편집 시나리오, 편집 기술 및 평가 방법을 나타내는 통합 편집기, 방법 및 평가 프레임워크가 포함되어 있습니다.
- Editor: 편집할 모델, 편집할 지식, 기타 필요한 하이퍼파라미터를 포함한 작업 시나리오를 설명합니다.
- 방법: 사용된 특정 지식 편집 방법(예: ROME, MEND 등)입니다.
- 평가: 신뢰성, 다양성, 지역성, 이식성을 포함한 지식 편집 성능을 평가하는 지표입니다.
- 트레이너: 일부 편집 방법에는 트레이너 모듈에 의해 구현되는 특정 트레이닝 프로세스가 필요합니다.
## 3. 설치 환경:
```
git clone https://github.com/zjunlp/EasyEdit.git
(선택 사항) conda create -n EasyEdit python=3.9.7
cd EasyEdit
pip install -r requirements.txt
```
## 4. 사례 편집
> 목표: GPT-2-XL의 지식 메모리를 변경하고 리오넬 메시의 경력을 축구에서 농구(축구->농구)로 변경합니다.
단계:
- 편집방법 선택 및 Parameter 준비
- 지식 편집을 위한 데이터 준비
- 인스턴스화 편집기
-달려!
다음은 ROME 방법을 예로 들어 자세히 소개합니다.
### 4.1 로마
목성 노트북: [https://colab.research.google.com/drive/1KkyWqyV3BjXCWfdrrgbR-QS3AAokVZbr?usp=sharing#scrollTo=zWfGkNb9FBJQ]
- 편집방법 선택 및 Parameter 준비
  - 편집 방법으로 ROME을 선택하고 ROME 및 GPT2-XL에 필요한 매개변수를 준비합니다.
  - 예: alg_name: "ROME", model_name: "./hugging_cache/gpt2-xl" 또는 모델의 로컬 경로, "device": 사용된 GPU 일련번호
  - 나머지 매개변수는 기본값으로 유지될 수 있습니다.
![](./assets/3.png)
- 지식 편집을 위한 데이터 준비
```
    prompts = ['Question:What sport does Lionel Messi play? Answer:'] # x_e
    ground_truth = ['football'] # y
    target_new = ['basketball'] # y_e
    subject = ['Lionel Messi'] 
    ```
- 편집기를 인스턴스화하고 인스턴스화를 위해 준비된 매개변수를 BaseEditor 클래스에 전달하고 사용자 정의된 편집기 인스턴스를 가져옵니다.
```
    hparams = ROMEHyperParams.from_hparams('./hparams/ROME/gpt2-xl.yaml')
    editor=BaseEditor.from_hparams(hparams)
    ```
- 달리다! 편집기의 편집 메서드를 호출합니다.
```
    metrics, edited_model, _ = editor.edit(
        prompts=prompts,
        ground_truth=ground_truth,
        target_new=target_new,
        subject=subject,
        keep_original_weight=False
    )
    ```
![](./assets/4.png)
> 참고: 모델을 처음 편집할 때 Wiki 코퍼스가 다운로드되고 각 레이어의 상태가 모델에 대해 계산 및 저장되며(stats_dir: "./data/stats") 각 후속 편집에 재사용됩니다. 따라서 최초 편집에는 다소 시간이 소요될 수 있으니, 네트워크가 원활할 경우 인내심을 갖고 기다려 주시기 바랍니다.
### 4.2 검증 및 평가
editor.edit는 메트릭(EasyEdit의 평가 모듈에 의해 계산됨)을 반환합니다. 형태는 다음과 같습니다:
![](./assets/5.png)
보편성, 지역성, 이식성의 가치를 얻으려면 edit 메소드에서 평가할 데이터를 전달해야 합니다.

지역성을 예로 들면 편집 메소드가 지역성 지표, 즉 locality_inputs에 대한 모델 응답의 정확성을 계산하게 됩니다.
```
locality_inputs = {
    'neighborhood':{
        'prompt': ['Joseph Fischhof, the', 'Larry Bird is a professional', 'In Forssa, they understand'],
        'ground_truth': ['piano', 'basketball', 'Finnish']
    }
}
metrics, edited_model, _ = editor.edit(
    prompts=prompts,
    ground_truth=ground_truth,
    target_new=target_new,
    locality_inputs=locality_inputs,
    keep_original_weight=False
)
```
또는 이전 모델과 이후 모델의 생성된 동작을 직접 비교합니다.
```
generation_prompts = [
    "Lionel Messi, the",
    "The law in Ikaalinen declares the language"
]

model = GPT2LMHeadModel.from_pretrained('./hugging_cache/gpt2').to('cuda')
batch = tokenizer(generation_prompts, return_tensors='pt', padding=True, max_length=30)

pre_edit_outputs = model.generate(
    input_ids=batch['input_ids'].to('cuda'),
    attention_mask=batch['attention_mask'].to('cuda'),
    max_new_tokens=3
)
post_edit_outputs = edited_model.generate(
    input_ids=batch['input_ids'].to('cuda'),
    attention_mask=batch['attention_mask'].to('cuda'),
    max_new_tokens=3)
```
## 5. 크기 조정 편집(선택 사항)
### 5.1 일괄 편집
여러 데이터 조각이 병렬 목록을 형성하고 일괄 편집을 위해 편집 메소드로 전달될 수 있습니다. 이 경우 MEMIT가 가장 좋은 방법입니다. (https://colab.research.google.com/drive/1P1lVklP8bTyh8uxxSuHnHwB91i-1LW6Z）
```
prompts = ['Question:What sport does Lionel Messi play? Answer:',
            'The law in Ikaalinen declares the language']
ground_truth = ['football', 'Finnish']
target_new = ['basketball', 'Swedish']
subject = ['Lionel Messi', 'Ikaalinen']
```
### 5.2 벤치마크에서 테스트됨
- 반대말
-ZsRE
```
{
    "case_id": 4402,
    "pararel_idx": 11185,
    "requested_rewrite": {
      "prompt": "{} debuted on",
      "relation_id": "P449",
      "target_new": {
        "str": "CBS",
        "id": "Q43380"
      },
      "target_true": {
        "str": "MTV",
        "id": "Q43359"
      },
      "subject": "Singled Out"
    },
    "paraphrase_prompts": [
      "No one on the ground was injured.  v",
      "The sex ratio was 1063. Singled Out is to debut on"
    ],
    "neighborhood_prompts": [
      "Daria premieres on",
      "Teen Wolf was originally aired on",
      "Spider-Man: The New Animated Series was originally aired on",
      "Celebrity Deathmatch premiered on",
      "\u00c6on Flux premiered on",
      "My Super Psycho Sweet 16 premieres on",
      "Daria was released on",
      "Jersey Shore premiered on",
      "Skins was originally aired on",
      "All You've Got premiered on"
    ]
  }
  ```
https://github.com/zjunlp/EasyEdit/blob/main/examples/run_zsre_llama2.py 
