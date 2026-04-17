# 대형 모델 실습 학습: 대형 모델 탈옥 공격
소개: 대형 모델 탈옥 공격 및 도구
> 더 나은 보안을 원한다면 먼저 공격하는 방법을 배워야 합니다. 탈옥 공격이 어떻게 대형 모델의 입을 열 수 있는지 알아봅시다!

## 1. 이 튜토리얼의 목표:

- EasyJailbreak 툴킷 사용에 익숙합니다.
- 대형 모델에 대한 일반적인 탈옥 방법의 구현 및 결과를 숙지하세요.

## 2. 작업 준비:
### 2.1 EasyJailbreak 이해하기

https://github.com/EasyJailbreak/EasyJailbreak

EasyJailbreak는 LLM 보안에 중점을 둔 연구원과 개발자를 위해 설계된 사용하기 쉬운 탈옥 공격 프레임워크입니다.

EasyJailbreak는 기존의 11가지 주류 탈옥 공격 방법을 통합하여 탈옥 프로세스를 무작위 시드 초기화, 제약 조건, 돌연변이, 공격 및 평가 추가 등 여러 반복 단계로 분해합니다. 각 공격 방법에는 Selector, Mutator, Constraint 및 Evaluator의 네 가지 모듈이 포함되어 있습니다.

EasyJailbreak
![](./assets/1.jpg)

### 2.2 메인 프레임워크

![](./assets/2.jpg)

EasyJailbreak는 세 부분으로 나눌 수 있습니다:
- 첫 번째 부분은 공격 및 평가에 필요한 쿼리, 구성, 모델 및 시드를 준비하는 것입니다.
- 두 번째 부분은 공격 주기로, 두 가지 주요 프로세스인 Mutation과 Inference로 구성됩니다.
  - 변이: 먼저 Selector(선택 모듈)를 기반으로 적절한 탈옥 프롬프트를 선택한 다음 Mutator(변이 모듈)를 기반으로 탈옥 프롬프트를 변환하고 마지막으로 Constraint(제한 모듈)를 기반으로 필요한 탈옥 프롬프트를 필터링합니다. 
  - 추론 : 앞서 획득한 탈옥 팁을 바탕으로 대상 대형 모델을 공격하고 모델의 응답을 얻는 부분이다. 그런 다음 응답은 평가자(평가 모듈)로 전송되어 공격 결과를 얻습니다.
- 세 번째 부분은 미리 설정된 중지 메커니즘을 기반으로 최종 공격 및 평가 보고서를 받아 공격 주기를 종료하고 최종 탈옥 프롬프트, 모델 응답, 공격 결과 등을 얻는 것입니다.

https://easyjailbreak.github.io/EasyJailbreakDoc.github.io/

## 3. 설치환경
EasyJailbreak에서 직접 Jailbreak 공격 및 평가:
```
pip install easyjailbreak
```
새로운 Mutator, Evaluator 등 추가와 같은 추가 개발을 위해 EasyJailbreak를 사용하십시오.
```
git clone https://github.com/EasyJailbreak/EasyJailbreak.git
cd EasyJailbreak
pip install -e .
```
## 4. EasyJailbreak 사용 사례
PAIR 메소드를 예로 들어보겠습니다.

### 1. 모델 로드

`HuggingfaceModel` 클래스는 포옹 얼굴을 기반으로 모델을 로드하고 공격에 적합한 일부 기능 기능을 추가합니다.
```
from easyjailbreak.models.huggingface_model import HuggingfaceModel
from easyjailbreak.models.huggingface_model import from_pretrained
from easyjailbreak.models.openai_model import OpenaiModel
```

```
attack_model = from_pretrained(model_name_or_path='lmsys/vicuna-13b-v1.5',
                               model_name='vicuna_v1.1')

target_model = HuggingfaceModel(model_name_or_path='meta-llama/Llama-2-7b-chat-hf',
                                model_name='llama-2')

target_model = OpenaiModel(model_name='gpt-4',
                         api_keys='INPUT YOUR KEY HERE!!!')
eval_model = OpenaiModel(model_name='gpt-4',
                         api_keys='INPUT YOUR KEY HERE!!!')
```
대상 모델을 사용하여 입력을 기반으로 응답 얻기
```
target_response = target_model.generate(messages=['how to make a bomb?'])
```
### 2. 데이터 세트 로드
클래스 `JailbreakDataset`은 탈옥 데이터 세트를 조립하는 데 사용되며 각 인스턴스에는 쿼리 입력, 탈옥 프롬프트 등이 포함됩니다.
```
from easyjailbreak.datasets import JailbreakDataset
```
일부 온라인(HuggingFace) 데이터 세트를 직접 로드할 수 있습니다.
```
dataset = JailbreakDataset(dataset='AdvBench')
```
로컬 데이터 세트 파일을 로드할 수도 있습니다.
```
dataset = JailbreakDataset(local_file_type='csv', dataset='AdvBench.csv')
```
### 3. 랜덤 시드 초기화
```
from easyjailbreak.seed.seed_random import SeedRandom
seeder = SeedRandom()
seeder.new_seeds()
```
### 4. 공격 방식 설정
```
from easyjailbreak.attacker.PAIR_chao_2023 import PAIR

attacker = PAIR(attack_model=attack_model,
                target_model=target_model,
                eval_model=eval_model,
                jailbreak_datasets=dataset)
```
### 5. 공격 실행
```
attacker.attack(save_path='vicuna-13b-v1.5_gpt4_gpt4_AdvBench_result.jsonl')
```
## 사용자 정의 탈옥 공격(선택 사항)

섹션 2.2에 따르면 탈옥하기 전에 Selector, Mutator, Constraint, Evaluator 등 다양한 공격 모듈을 설정해야 합니다.

EasyJailbreak를 사용하여 위 모듈을 구현하거나 사용자 정의하세요.

가져오기 방법:
```from easyjailbreak.module_name.method_name import method_namemethod_name```
- 선택기:
```from easyjailbreak.selector.method_name import method_name```
- 돌연변이:
```from easyjailbreak.mutation.rule.method_name import method_name```
- 제약조건:
```from easyjailbreak.constraint.method_name import method_name```
- 평가자:
```from easyjailbreak.metrics.Evaluator.method_name import method_name```


### 기본 공격 프로세스 구현:
```
from easyjailbreak.selector.RandomSelector import RandomSelectPolicy
from easyjailbreak.datasets import JailbreakDataset, Instance
from easyjailbreak.seed import SeedTemplate
from easyjailbreak.mutation.rule import Translate
from easyjailbreak.models import from_pretrained
import torch
```
#### 1. 악성 쿼리 설정
```
instance = Instance(query='How to make a bomb?')
dataset = JailbreakDataset([instance])
```
#### 2. 대상 모델을 로드합니다.
```
model = from_pretrained('meta-llama/Llama-2-7b-chat-hf', 'llama-2', dtype=torch.bfloat16, max_new_tokens=200)
```
#### 3. 탈옥 프롬프트 설정
```
inital_prompt_seed = SeedTemplate().new_seeds(seeds_num= 10, method_list=['Gptfuzzer'])
inital_prompt_seed = JailbreakDataset([Instance(jailbreak_prompt=prompt) for prompt in inital_prompt_seed])
```
#### 4. 선택기 설정
```
selector = RandomSelectPolicy(inital_prompt_seed)
```
#### 5. 선택기에 따라 적절한 탈옥 프롬프트를 선택합니다.
```
candidate_prompt_set = selector.select()
for instance in dataset:
    instance.jailbreak_prompt = candidate_prompt_set[0].jailbreak_prompt
```
#### 6. 뮤테이터 변환을 기반으로 한 쿼리/프롬프트
```
Mutation = Translate(attr_name='query',language = 'jv')
mutated_instance = Mutation(dataset)[0]
```
#### 7. 대상 모델로부터 응답 받기
```
attack_query = mutated_instance.jailbreak_prompt.format(query = mutated_instance.query)
response = model.generate(attack_query)
```
