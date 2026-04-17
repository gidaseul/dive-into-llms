# 사전 학습 언어 모델 미세 조정 및 배포

> 소개: 사전 훈련된 모델의 미세 조정을 소개하는 부분입니다.
특정 작업에 대해 사전 훈련된 모델의 성능을 향상시키고 싶으십니까? 적합한 사전 훈련된 모델을 선택하고, 특정 작업에 맞게 미세 조정한 후, 미세 조정된 모델을 편리한 데모에 배포해 보세요!
## 이 튜토리얼의 목표:
1. Transformers 툴킷 사용에 익숙함
2. 사전 학습된 모델의 미세 조정 및 추론을 마스터합니다(분리된 사용자 정의 버전 및 기본 통합 버전).
3. 데모 배포를 위한 Gradio Spaces 사용법 익히기
4. 다양한 유형의 사전 훈련 모델의 선택 및 적용 시나리오를 이해합니다.
## 이 튜토리얼의 내용:
### 1. 작업 준비:
#### 1.1 툴킷 이해하기: Transformers
https://github.com/huggingface/transformers

> 🤗 Transformers는 사전 훈련된 고급 모델을 쉽게 다운로드하고 교육할 수 있는 API와 도구를 제공합니다. 사전 훈련된 모델을 사용하면 컴퓨팅 소비와 탄소 배출량을 줄이고 처음부터 훈련하는 데 필요한 시간과 리소스를 절약할 수 있습니다. 이러한 모델은 다음과 같은 다양한 양식의 공통 작업을 지원합니다.
📝 자연어 처리: 텍스트 분류, 개체명 인식, 질문 답변, 언어 모델링, 요약, 번역, 객관식 및 텍스트 생성.
🖼️ 머신 비전: 이미지 분류, 객체 감지 및 의미론적 분할.
🗣️ 오디오: 자동 음성 인식 및 오디오 분류.
🐙 다중 모드: 양식 ​​질문 응답, 광학 문자 인식, 스캔한 문서에서 정보 추출, 비디오 분류 및 시각적 질문 응답.

자세한 중국어 문서: https://huggingface.co/docs/transformers/main/zh/index

![껴안는 얼굴](./assets/huggingface.PNG)

#### 1.2 설치 환경: 텍스트 분류(예: 가짜 뉴스 탐지)를 예로 들어 보겠습니다.
1. 텍스트 분류의 사례 라이브러리에 들어가서 Readme를 참조하여 주요 매개변수를 이해하고 요구사항.txt 및 run_classification.py를 다운로드합니다.

https://github.com/huggingface/transformers/tree/main/examples/pytorch/text-classification

2. 설치 환경:
- 1. conda를 통해 새 환경 만들기: conda create -n llm python=3.9
- 2. 가상 환경으로 진입: conda activate llm
- 3. pip 설치 변압기
- 4. 요구 사항.txt, pip install -r 요구 사항.txt에서 자동으로 설치된 토치를 삭제합니다.

> 다운로드 속도가 느리다면 국내 소스를 이용하시면 됩니다: pip [Packages] -i https://pypi.tuna.tsinghua.edu.cn/simple

> 국내 소스를 사용하여 파이토치를 설치하는 경우 파이토치의 CPU 버전이 자동으로 선택되어 GPU를 실행할 수 없으므로——

- 5. conda 설치 pytorch

> 다운로드 속도가 느린 경우 다음 블로그에 따라 콘다 이미지를 구성할 수 있습니다: https://blog.csdn.net/weixin_42797483/article/details/132048218

3. 데이터 준비: Kaggle에 설정된 가짜 트윗 데이터를 예로 들어 보겠습니다: https://www.kaggle.com/c/nlp-getting-started/data

#### 1.3 처리된 프로젝트 패키지(데모 코드 및 데이터)
(1) 분리 및 사용자 정의 버전(핵심 모듈을 분리하여 이해하기 쉽도록 하고, 데이터 로딩, 모델 구조, 평가지표 등을 사용자 정의 가능)

[TextClassification사용자 정의 다운로드 링크](https://pan.quark.cn/s/00dae5c2b128)

(2) 기본 통합 버전(코드가 더 작음)
풍부하고 복잡하기 때문에 하이퍼파라미터는 일반적으로 직접 호출되며 약간의 개발 임계값이 있습니다)

[텍스트 분류 다운로드 링크](https://pan.quark.cn/s/9d0510f1c98d)


### 2. 분리 버전 기반 맞춤형 개발(최소 실행 가능한 제품 MVP)
세 가지 주요 파일이 있습니다: main.py 메인 프로그램, utils_data.py 데이터 로딩 및 처리 파일,modeling_bert.py 모델 구조 파일

![프로젝트 구조](./assets/0.png)

#### 2.1 핵심 모듈 이해
1. 데이터 로드 및 처리(utils_data.py)
![utils_data.py](./assets/1.png)

2. 모델 로드(modeling_bert.py)

![modeling_bert_1.py](./assets/2.png)

![modeling_bert_2.py](./assets/3.png)


3. 훈련/검증/예측(main.py)

![main.py](./assets/4.png)

#### 2.2 학습/검증/예측 원스톱 프로세스 실행
```bash
python main.py
```
### 3. 통합 버전 기반 미세 조정 (선택 사항, run_classification.py 기반)

#### 3.1 주요 모듈을 이해합니다.
1. 데이터 로드(csv 또는 json 형식)
![데이터 로드](./assets/5.png)

2. 공정 데이터
![프로세스 데이터](./assets/6.png)

3. 모델 로드
![모델 로드](./assets/7.png)

4. 훈련/검증/예측
![기차 개발자 예측](./assets/8.png)

#### 3.2 훈련 모델
동시에 개발 세트를 확인하고, 테스트 세트를 예측하고, 다음 스크립트를 실행합니다.
```bash
python run_classification.py \
    --model_name_or_path  bert-base-uncased \
    --train_file data/train.csv \
    --validation_file data/val.csv \
    --test_file data/test.csv \
    --shuffle_train_dataset \
    --metric_name accuracy \
    --text_column_name "text" \
    --text_column_delimiter "\n" \
    --label_column_name "target" \
    --do_train \
    --do_eval \
    --do_predict \
    --max_seq_length 512 \
    --per_device_train_batch_size 32 \
    --learning_rate 2e-5 \
    --num_train_epochs 1 \
    --output_dir experiments/
```
오류나 중단이 발생하는 경우 일반적으로 네트워크 문제입니다.

1. <u>가 모델을 다운로드하면 "네트워크에 연결할 수 없습니다"가 표시됩니다. 모델을 로컬에 수동으로 다운로드: https://huggingface.co/google-bert/bert-base-uncased</u>
2. 데이터를 추가한 후 <u>이 중단되고 CTRL+C를 종료한 후 디스플레이가 "연결" 상태에서 중단되는 경우 이는 평가 패키지가 평가 표시기를 로드할 때 네트워크 연결 실패로 인해 발생합니다. </u>

![버그](./assets/9.png)

이 경우 [evaluate GitHub 저장소](https://github.com/huggingface/evaluate/tree/main)에서 전체 패키지를 내려받은 뒤, 하이퍼파라미터의 `--metric_name` 경로를 로컬 지표 경로로 바꿔 실행할 수 있습니다.
```bash
python run_classification.py \
    --model_name_or_path  bert-base-uncased \
    --train_file data/train.csv \
    --validation_file data/val.csv \
    --test_file data/test.csv \
    --shuffle_train_dataset \
    --metric_name evaluate/metrics/accuracy/accuracy.py \
    --text_column_name "text" \
    --text_column_delimiter "\n" \
    --label_column_name "target" \
    --do_train \
    --do_eval \
    --do_predict \
    --max_seq_length 512 \
    --per_device_train_batch_size 32 \
    --learning_rate 2e-5 \
    --num_train_epochs 1 \
    --output_dir experiments/
```
![참조 결과](./assets/10.png)


### 4. 모델 배포: 모델 훈련이 완료된 후 Gradio Spaces에서 온라인 데모를 구축할 수 있습니다.
#### 4.1 그라디오 스페이스 튜토리얼
https://huggingface.co/docs/hub/en/spaces-sdks-gradio
#### 4.2 스페이스 만들기
1. https://huggingface.co/new-space?sdk=gradio
2. 참고: 열리지 않으면 과학적으로 인터넷 서핑을 시도해 보십시오.

![그라디오 스페이스](./assets/gradio.png)

#### 4.3 핵심 추론 코드
자세한 내용은 프로젝트 패키지의 app.py를 참조하세요.

![app.py](./assets/11.png)

#### 4.4 Gradio Spaces에 app.py, 환경 구성 파일 및 모델 업로드
1. 구성 파일(requirements.txt)
```
transformers==4.30.2
torch==2.0.0
```
2. 문서개요

![파일 개요](./assets/12.png)


3. 데모 효과
참고용 배포 성공 사례: https://huggingface.co/spaces/cooelf/text-classification
소스 코드는 오른쪽 상단의 "파일" 열에서 볼 수 있습니다.

![파일](./assets/13.png)


4. 이스터 에그: Spaces 플랫폼에서 매주 핫한 데모를 볼 수 있으며, 관심 있는 대형 모델과 데모를 검색하여 시도해 볼 수 있습니다.
![공백](./assets/14.png)


### 5. 고급 연습
1. 감성 분류, 뉴스 분류, 취약점 분류 등 다른 분류/회귀 작업을 시도해 보세요.
2. T5, ELECTRA 등 다른 유형의 모델을 사용해 보세요.
### 6. 기타 일반적으로 사용되는 모델
1. 질문과 답변 모델: https://github.com/huggingface/transformers/tree/main/examples/pytorch/question-answering
2. 텍스트 요약: https://github.com/huggingface/transformers/tree/main/examples/pytorch/summarization
3. 추론을 위해 Llama2를 호출합니다: https://huggingface.co/docs/transformers/en/model_doc/llama2
4. Llama2의 경량 미세 조정(LoRA): https://github.com/peremartra/Large-Language-Model-Notebooks-Course/blob/main/5-Fine%20Tuning/LoRA_Tuning_PEFT.ipynb


### 7. 추가 자료
1. 대규모 언어 모델(LLM)(https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247484539&idx=1&sn=6ee42baab4ad792e74ac6a89d7dd87d9&chksm=c2ce3e0af5b9b71c578cb6836e09cce5f60b4a1cfffadc1c4c98211fc0af621bfaaaa8fe0aff&mpshare=1&scene=24&srcid=0331FO6iqVqs5Vx3iHrtqouQ&sharer_shareinfo=45c3fb78d9cfb9627908da44dd7f5559&sharer_shareinfo_first=45c3fb78d9cfb9627908da44dd7f5559&key=a2847c972f830c4143e00e0430f657d8ab5acae4ad24ca628213273021453a3a9e17984627e2ab8506d1dcf6e1fabd9ec3123a5f71d2a65295ad6f6f56da2224d4a6e3228237c237447bbf48a6eff1f53e1971503f26c3fb5b9d99d27eca7266529a5f86d75bada7ec10bf314687bfcbf9fa3b09b8e36e73f6d6a154a5ce5ff0&ascene=14&uin=NzE3NzkyOTQx&devicetype=iMac20%2C1+OSX+OSX+14.2.1+build(23C71)&version=13080610&nettype=WIFI&lang=zh_CN&countrycode=CN&fontScale=100&exportkey=n_ChQIAhIQM7tgcovlhXp1y5J%2BRMnhfhL3AQIE에 대한 포괄적이고 심층적인 분석을 제공하는 43페이지 [리뷰] 97dBBAEAAAAAAFsTBwTm4FMAAAAOpnltbLcz9gKNyK89dVj0MxFZswDc%2Fk646vJPW2S3JFh8H2JhyZXi PbIl%2Bh23CsewrmIoZ4j0D2zMNylC3pLbhu9FIARUKYn%2F0r0OIdHnxesVFpw1qLo6uBJ3zmbsKBVXM05 %2B0MiOBIfShfpiIfraK7THzak94U0RdS1flC%2BIDjTb5SmZs9Z4XTyTsN0QXR6NWjAXFeuxnMB4SENMJ 8dUR8n08b3DGKtz9rfefn0JRlsX4mGcLvOFsFwg4nk35nl4C3Wgcs4OYociKm5UHabdhWT7%2FWbNNToZLD 39eD%2FL4Xo%3D&acctmode=0&pass_ticket=8xG4QKOnJeEFUtXkScVmMqb3omdWJbPuc%2BhN5%2BA7 %2FOXj7ex757M2ABNO4GVVnv2T3VtqrC9gZH%2FrU09rrUxJcA%3D%3D&wx_header=0) (Word2Vec 작성자가 제작)
   
    논문 제목: 대규모 언어 모델: 설문조사

    논문 링크: https://arxiv.org/pdf/2402.06196.pdf

2. [GPT, GPT-2, GPT-3 논문 집중 읽기 [논문 집중 읽기]] (https://www.bilibili.com/video/BV1AF411b7xQ?t=0.0)

3. [GPT 지필집중독서지도하기【지필집중독서·48】](https://www.bilibili.com/video/BV1hd4y187CR?t=0.4)
