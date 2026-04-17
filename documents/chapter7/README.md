# LLM 텍스트 스테가노그래피

이 프로젝트는 생성된 텍스트의 정보를 숨기기 위해 GPT-2 모델을 사용하여 LLM(대형 언어 모델) 기반의 텍스트 스테가노그래피를 구현합니다. 이 프로젝트에는 허프만 코딩(Huffman Coding)과 고정 길이 코딩(FLC)이라는 두 가지 코딩 방법이 포함되어 있습니다.

## 기능

- GPT-2 텍스트 생성에 스테가노그래피 구현
- 두 가지 인코딩 방법을 지원합니다:
  - 허프만 코딩
  -고정 길이 코딩(FLC)
- 비교를 위해 스테가노그래픽 텍스트와 일반 텍스트를 동시에 생성

## 환경 요구사항

-파이썬 3.8+
-파이토치
- 트랜스포머
- CUDA(선택 사항, GPU 가속용)

## 설치 단계

종속성 패키지를 설치합니다.
```bash
pip install torch # GPU인 경우 토치는 지정된 GPU 버전이라는 점에 유의하세요.
pip install transformers
pip install jupyter
```
## 사용방법

1. Jupyter 노트북을 엽니다.
```bash
jupyter notebook llm_stega.ipynb
```
2. 노트북은 세 가지 주요 부분으로 구성됩니다.
   - 허프만 코딩 구현
   - 고정 길이 코딩(FLC) 구현
   - 텍스트 생성 및 스테가노그래피

3. 스테가노그래피 텍스트 생성:
   - 인코딩 방법 선택(허프만 코딩 또는 FLC)
   - 매개변수 설정(인코딩된 k 값)
   - 사용자 정의 프롬프트 단어로 기본 기능 실행

샘플 코드:
```python
# 스테가노그래픽 프로세서를 초기화합니다.
k = 2
handle = Huffman(k=2**k, bits=bits)
# 아니면 FLC를 사용하세요
# handle = FLC(k=k, bits=bits)

# 스테가노그래피로 텍스트 생성
prompt = "Hello，I'm"
stega_output, normal_output = generate_text_with_steganography(
    model, tokenizer, prompt, handle
)
```
## 출력 파일

프로그램은 두 개의 텍스트 파일을 생성합니다.
- `outputs-gpt2-stega.txt`: 스테가노그래피 텍스트
- `outputs-gpt2-normal.txt`: Text originally generated normally without steganography constraints

## 메모

- 최초 실행 시 GPT-2 모델이 자동으로 다운로드됩니다. 중국에서는 먼저 https://hf-mirror.com 미러를 통해 GPT-2 모델을 다운로드하거나 다른 대형 모델을 사용할 수도 있습니다.
- 디스크 공간이 충분한지 확인하세요. (모델은 약 500MB)
- 더 나은 성능을 위해 CUDA 지원 GPU를 사용하는 것이 좋습니다
- 스테가노그래피 정보의 길이는 생성된 텍스트의 길이에 영향을 미칩니다.

## 작동 원리

1. 스테가노그래피 프로세스:
   - 비밀 정보를 바이너리 시퀀스로 변환
   - GPT-2 모델을 사용하여 텍스트 생성
   - 인코딩 방법을 통해 생성된 텍스트에 정보 비트를 삽입합니다.

2. 디코딩 프로세스:
   - 동일한 GPT-2 모델 및 컨텍스트를 사용합니다.
   - 디코딩 방법을 통해 텍스트에서 숨겨진 정보를 추출합니다.
   - 이진 시퀀스를 원래 정보로 다시 변환