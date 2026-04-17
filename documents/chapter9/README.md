# 대형 모델 실습: GUI 에이전트 구축
## 이 튜토리얼의 목표
1. GUI Agent 분야의 기술경로 및 연구현황을 이해한다.

2. 오픈 소스 모델 Qwen2-VL-7B를 기반으로 하고 OS-Kairos 데이터 세트를 사용하여 적응형 인간-컴퓨터 상호 작용 GUI 에이전트를 구축해 보십시오.

3. 구축된 GUI 에이전트를 기반으로 추론해 보세요.
## 1. 준비
### 1.1 GUI 에이전트의 기술 경로 및 연구 현황을 이해합니다.
튜토리얼 읽기: [[슬라이드](https://github.com/gidaseul/dive-into-llms/blob/main/documents/chapter8/GUIagent.pdf)]

### 1.2 적응형 인간-컴퓨터 상호작용 GUI 에이전트가 무엇인지 이해하기
참고 논문: [[논문](https://arxiv.org/abs/2503.16465)]

### 1.3 데이터 세트 준비
이 튜토리얼에서는 OS-Kairos를 예로 들어 이 작업의 오픈 소스 데이터 세트를 기반으로 간단한 GUI 에이전트를 구축하고 추론합니다.

[[Data](https://github.com/Wuzheng02/OS-Kairos)]의 README.md 파일에 있는 다운로드 링크에서 데이터 세트를 다운로드하고 환경에 추출합니다.

데이터 형식의 예는 다음과 같습니다.
```json
    {
"task": "NetEase Cloud Music을 열고 "Shape of You"를 검색한 후 이 노래를 재생하세요.",
        "image_path": "/data1/wuzh/cloud_music/images/1736614680.6518524_1.png",
        "list": [
"NetEase 클라우드 음악 열기",
"홈페이지 상단의 검색창을 클릭하세요",
"입력:당신의 모양",
"올바른 검색결과를 선택하세요",
"재생하려면 노래 제목을 클릭하세요."
        ],
        "now_step": 1,
        "previous_actions": [
            "CLICK <point>[[381,367]]</point>"
        ],
        "score": 5,
        "osatlas_action": "CLICK <point>[[454,87]]</point>",
        "teacher_action": "CLICK <point>[[500,100]]</point>",
        "success": false
    },
```
### 1.4 모델 준비
[[모델](https://huggingface.co/Qwen/Qwen2-VL-7B-Instruct)]에서 Qwen2-VL-7B 모델 가중치를 다운로드합니다.

## 1.5 감독된 미세 조정 코드 준비
이 튜토리얼에서는 LLaMa-Factory를 사용하여 Qwen2-VL-7B의 감독된 미세 조정을 수행하므로 먼저 [[Code](https://github.com/hiyouga/LLaMA-Factory/)]에서 소스 코드를 다운로드합니다.

## 2.GUI 에이전트 구축
### 2.1 데이터 전처리
다음 코드를 get_sharpgpt.py로 저장하고, Kairos_train.json과 처리된 학습 세트 json이 저장될 경로를 입력한 후 파일을 실행합니다.
```python
import json


with open('', 'r', encoding='utf-8') as f:
    data = json.load(f)


preprocessed_data = []
for item in data:
    task = item.get("task", "")
    previous_actions = item.get("previous_actions", "") 
    image_path = item.get("image_path","")
    prompt_text = f"""
    You are now operating in Executable Language Grounding mode. Your goal is to help users accomplish tasks by suggesting executable actions that best fit their needs and give a score. Your skill set includes both basic and custom actions:

    1. Basic Actions
    Basic actions are standardized and available across all platforms. They provide essential functionality and are defined with a specific format, ensuring consistency and reliability. 
    Basic Action 1: CLICK 
        - purpose: Click at the specified position.
        - format: CLICK <point>[[x-axis, y-axis]]</point>
        - example usage: CLICK <point>[[101, 872]]</point>

    Basic Action 2: TYPE
        - purpose: Enter specified text at the designated location.
        - format: TYPE [input text]
        - example usage: TYPE [Shanghai shopping mall]

    Basic Action 3: SCROLL
        - Purpose: SCROLL in the specified direction.
        - Format: SCROLL [direction (UP/DOWN/LEFT/RIGHT)]
        - Example Usage: SCROLL [UP]

    2. Custom Actions
    Custom actions are unique to each user's platform and environment. They allow for flexibility and adaptability, enabling the model to support new and unseen actions defined by users. These actions extend the functionality of the basic set, making the model more versatile and capable of handling specific tasks.

    Custom Action 1: PRESS_BACK
        - purpose: Press a back button to navigate to the previous screen.
        - format: PRESS_BACK
        - example usage: PRESS_BACK

    Custom Action 2: PRESS_HOME
        - purpose: Press a home button to navigate to the home page.
        - format: PRESS_HOME
        - example usage: PRESS_HOME

    Custom Action 3: ENTER
        - purpose: Press the enter button.
        - format: ENTER
        - example usage: ENTER

    Custom Action 4: IMPOSSIBLE
        - purpose: Indicate the task is impossible.
        - format: IMPOSSIBLE
        - example usage: IMPOSSIBLE

    In most cases, task instructions are high-level and abstract. Carefully read the instruction and action history, then perform reasoning to determine the most appropriate next action.

    And I hope you evaluate your action to be scored, giving it a score from 1 to 5. 
    A higher score indicates that you believe this action is more likely to accomplish the current goal for the given screenshot.
    1 means you believe this action definitely cannot achieve the goal.
    2 means you believe this action is very unlikely to achieve the goal.
    3 means you believe this action has a certain chance of achieving the goal.
    4 means you believe this action is very likely to achieve the goal.
    5 means you believe this action will definitely achieve the goal.

    And your final goal, previous actions, and associated screenshot are as follows:
    Final goal: {task}
    previous actions: {previous_actions}
    screenshot:<image>
    Your output must strictly follow the format below, and especially avoid using unnecessary quotation marks or other punctuation marks.(where osatlas action must be one of the action formats I provided and score must be 1 to 5):
    action:
    score:
    """

    action = item.get("osatlas_action", "")
    score = item.get("score", "")
    preprocessed_item = {
        "messages": [
            {
                "role": "user",
                "content": prompt_text,
            },
            {
                "role": "assistant",
                "content": f"action: {action}\nscore:{score}",
            }
        ],
        "images":[image_path]
    }
    preprocessed_data.append(preprocessed_item)


with open('', 'w', encoding='utf-8') as f:
    json.dump(preprocessed_data, f, ensure_ascii=False, indent=4)
```
이 단계 후에 우리는 OS-Kairos 훈련 세트를 다음 훈련 단계에 편리한 Sharpgpt 형식을 따르는 데이터로 변환하고 데이터 전처리가 완료되었습니다.
## 2.2 감독된 미세 조정
이전 단계에서는 LLaMa-Factory 교육에 적합한 형식으로 Kairos 데이터 세트를 얻었습니다. 그런 다음 LLaMa-Factory를 수정하여 데이터 세트를 등록하고 훈련 정보를 구성해야 합니다.
먼저 data/dataset_info.json을 수정하고 다음 내용을 추가하여 데이터 세트를 등록합니다.
```json
"Karios" :{
  "file_name": "Karios_qwenscore.json",
  "formatting": "sharegpt",
  "columns": {
    "messages": "messages",
    "images": "images"
  },
  "tags": {
    "role_tag": "role",
    "content_tag": "content",
    "user_tag": "user",
    "assistant_tag": "assistant"
  }
},   
```
교육 정보를 구성하려면 /examples/train_full/qwen2vl_full_sft.yaml을 계속 수정하세요.
```yaml
### model
model_name_or_path: 
stage: sft
do_train: true
finetuning_type: full
deepspeed: examples/deepspeed/ds_z3_config.json

### dataset
dataset: Karios
template: qwen2_vl
cutoff_len: 4096
max_samples: 9999999
#max_samples: 999
overwrite_cache: true
preprocessing_num_workers: 4

### output
output_dir: 
logging_steps: 10
save_steps: 60000
plot_loss: true
overwrite_output_dir: true

### train
per_device_train_batch_size: 2
gradient_accumulation_steps: 2
learning_rate: 1.0e-5
num_train_epochs: 5.0
lr_scheduler_type: cosine
warmup_ratio: 0.1
bf16: true
ddp_timeout: 180000000

### eval
val_size: 0.1
per_device_eval_batch_size: 1
eval_strategy: steps
eval_steps: 20000
```
위의 구성은 예시입니다. model_name_or_path는 Qwen2-VL-7B의 경로이고, output_dir은 중단점과 로그가 저장되는 경로입니다.
구성한 후 추론을 구성할 수 있습니다. 명령줄 예시:
```
CUDA_VISIBLE_DEVICES=0,1,2,3,4,5,6,7 FORCE_TORCHRUN=1 llamafactory-cli train examples/train_full/qwen2vl_full_sft.yaml
```
여기에서는 최소 3개의 80GB A100 컴퓨팅 리소스를 사용해야 합니다.

## 3.OS-카이로스 추론 검증
훈련 후에는 추론을 수행할 수 있습니다. 선택할 수 있는 방법은 다양합니다. 변환기 라이브러리 및 토치 라이브러리를 통해 직접 추론 코드를 구축하거나 원클릭 추론을 위해 LLaMa-Factory를 사용할 수 있습니다. 다음은 원클릭 추론을 위해 LLaMa-Factory를 사용하는 방법을 보여줍니다.
먼저 /examples/inference/qwen2_vl.yaml을 수정해야 합니다. 예는 다음과 같습니다
```yaml
model_name_or_path: 
template: qwen2_vl
```
여기서 model_name_or_path는 훈련 후 중단점 경로입니다.
그러면 합격할 수 있어요
```
CUDA_VISIBLE_DEVICES=0 FORCE_TORCHRUN=1 llamafactory-cli webchat examples/inference/qwen2_vl.yaml
```
원클릭 추론을 위해서는 OS-Kairos 테스트 세트의 사진이나 개인 휴대폰의 스크린샷을 꺼내서 get_sharpgpt.py 형식으로 텍스트 프롬프트를 기반으로 에이전트에 입력하면 됩니다. OS-Karios 추론 후 현재 수행되어야 한다고 생각하는 작업과 이 작업에 대한 신뢰도 점수를 제공합니다. 신뢰도 점수가 낮을수록 현재 명령이 OS-Kairos의 기능 경계를 초과하고 인간과 컴퓨터 상호 작용을 위해 인간의 개입이 필요하다는 의미입니다.