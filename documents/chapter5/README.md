# 대형 모델 실습 학습: 모델 워터마크

소개: 이 섹션에서는 언어 모델의 워터마킹을 소개합니다.

> 언어 모델에 의해 생성된 콘텐츠에 사람이 감지할 수 없지만 알고리즘으로 감지할 수 있는 "워터마크"를 삽입합니다.

## 이 튜토리얼의 목표

1. 워터마크 임베딩: 언어 모델이 콘텐츠를 생성할 때 워터마크를 임베딩합니다.
2. 워터마크 감지: 특정 텍스트의 워터마크 강도 감지
3. 워터마크 평가: 워터마크 방식의 검출 성능을 평가합니다.
4. 워터마크의 견고성 평가(선택 사항)



## 준비

### 2.1 X-SIR 코드 저장소 이해

https://github.com/zwhe99/X-SIR

X-SIR 저장소에는 다음 구현이 포함되어 있습니다.

- 세 가지 텍스트 워터마킹 알고리즘: X-SIR, SIR 및 KGW
- 워터마크 제거 공격 방법 2가지: 의역 및 번역

![img](./assets/x-sir.png)

### 2.2 환경 준비
```bash
git clone https://github.com/zwhe99/X-SIR && cd X-SIR
conda create -n xsir python==3.10.10
conda activate xsir
pip3 install -r requirements.txt
# [optional] pip3 install flash-attn==2.3.3
```
> 요구사항.txt에 있는 버전은 필수 버전이 아닌 권장 버전입니다.



## 실제 사례

> KGW 알고리즘을 사용하여 언어 모델 생성 콘텐츠에 워터마크 삽입

### 3.1 데이터 준비

언어 모델에 입력될 프롬프트를 jsonl 파일로 구성합니다.
```json
{"prompt": "Ghost of Emmett Till: Based on Real Life Events "}
{"prompt": "Antique Cambridge Glass Pink Decagon Console Bowl Engraved Gold Highlights\n"}
{"prompt": "2009 > Information And Communication Technology Index statistics - Countries "}
......
```
- 각 행은 json 개체이며 "prompt"라는 키가 하나 이상 포함되어 있습니다.
- 다음 내용은 `data/dataset/mc4/mc4.en.jsonl` 파일을 예로 들어 설명합니다. 이 파일에는 총 500개의 데이터가 포함되어 있습니다. 모델 처리 시간이 너무 길다고 생각되면 직접 데이터를 줄이는 것을 고려해 볼 수 있습니다.

### 3.2 워터마크 삽입

- 모델 및 워터마킹 알고리즘을 선택합니다. 여기서는 `baichuan-inc/Baichuan-7B` 모델과 `KGW` 워터마크 알고리즘을 선택합니다.

  -
```bash
    MODEL_NAME=baichuan-inc/Baichuan-7B
    MODEL_ABBR=바이촨-7b
    WATERMARK_METHOD_FLAG="--watermark_method kgw"
```

- 콘텐츠 생성 및 워터마크 삽입

  -
```bash
    python3 gen.py \
        --base_model $MODEL_NAME \
        --fp16 \
        --batch_size 32 \
        --input_file data/dataset/mc4/mc4.en.jsonl \
        --output_file gen/$MODEL_ABBR/kgw/mc4.en.mod.jsonl \
        $WATERMARK_METHOD_FLAG
    ```
- 이 명령은 모델에서 생성된 콘텐츠를 출력 파일 `gen/$MODEL_ABBR/kgw/mc4.en.mod.jsonl`에 저장합니다.

  - 출력 파일의 형식은 다음과 같습니다. 여기서 응답은 모델의 출력 내용입니다.-
```json
      {"prompt": "Emmett Till의 유령: 실제 사건을 기반으로 함", "response": ".8월 1955년 아프리카계 미국인 Emmett Louis Till(21)\nThe Man From Waco의 두 번째 부분, 당시 법령에 따르면 텍사스 소시지 싸움에서 친구 Michael Capps를 살해한 Dan Miller의 재판에 관한 내용입니다. 그 당시에는 Dan이 권총을 물속에 보관했기 때문에 확실히 그런 일이 일어났을 것입니다. 만약 우리가 읽은 내용이 정확히 댄이 훔친 총을 두고 나중에 두 명의 무고한 소년을 죽인 강도 사건에 사용한 곳이 바로 그의 공범이 충분한 보수를 받지 못해 나가기를 원하는 밀러를 분노하게 만들었지만 이 범죄 행위는 그들이 FBI와 다른 매우 강력한 법관의 손에 들어간 후에는 피할 수 없었습니다. 직접적으로 참여한 사람"}
      {"prompt": "골드 하이라이트가 새겨진 앤티크 캠브리지 유리 핑크 데카곤 콘솔 보울\n", "response": "아주 훌륭한 장식용 앤티크 핑크 십각형 유리 사이드 보울은 초기 예술과 혁신의 완벽한 예입니다. 완전히 인그레이빙된 이 아름다운 영국 빈티지 식기 제품은 뛰어난 장인정신을 보여줍니다! 1800년대 중반 영국에서 제작된 이 제품은 외부에 세 세트의 동심원 리본이 있어 복잡하고 깊은 디자인을 우아하게 강조합니다. 앞으로 몇 년 동안 따뜻함과 편안함을 불러일으키는 붉은색! 역사적으로 중요한 이 꽃병은 \"코닝 박물관의 유리\", \"첫 반세기\" 및 특별 순회 전시회인 \" Sight Of Glass: B를 포함한 수많은 박물관 전시회에 등장했습니다.ibex Limited(소매업체)의 ritish Cut Glass\" 및 코닝 유리 박물관 2012 ASA 전국 심포지엄에서 발표된 \"SIGNALS - 관절형 병 용기를 통한 미국 은의 역사 기념\"! 우리는 고객에게 고품질 사진을 제공합니다."}
      {"prompt": "2009 > 정보 통신 기술 지수 통계 - 국가 ", "response": "2016/5/22\n아르메니아의 휴대전화 가입 연간 변화(인구 1명당). 2.2% 증가는 100명당 가입 38건에 해당합니다. 밀도 순위: 222 중 121.\n자전거 타는 사람/월. 자전거를 좋아합니다. 예레반의 Rimon Bike Rentals에서 할인 및 저렴한 자전거 대여 혜택을 누리세요. 산악자전거를 포함한 다양한 종류의 자전거를 보유하고 있습니다. \n전기 자동차에 대해 자세히 알아보세요. Renault Fluence KZERO는 도시에서 바로 이용할 수 있지만 이 차량에서 기대할 수 있는 마일리지는 얼마인지 모르겠습니다. 아르메니아 전자 거버넌스 연구소(Armenian Institute for Electronic Governance) 보고서 |"}
      ...
```



### 3.3 워터마크 검출

> 워터마크 검출은 주어진 텍스트에 대해 워터마크 강도(z-score)를 계산하는 과정입니다.

- **워터마크가 포함된** 텍스트의 워터마크 강도 계산

  -
```bash
    python3 detect.py \
        --base_model $MODEL_NAME \
        --detect_file gen/$MODEL_ABBR/kgw/mc4.en.mod.jsonl \
        --output_file gen/$MODEL_ABBR/kgw/mc4.en.mod.z_score.jsonl \
        $WATERMARK_METHOD_FLAG
    ```
- **워터마크 없음** 텍스트에 대한 워터마크 강도 계산

  -
```bash
    python3 detect.py \
        --base_model $MODEL_NAME \
        --detect_file data/dataset/mc4/mc4.en.jsonl \
        --output_file gen/$MODEL_ABBR/kgw/mc4.en.hum.z_score.jsonl \
        $WATERMARK_METHOD_FLAG
```

- 출력 파일 형식은 다음과 같습니다.

  -
```json
    {"z_score": 12.105422509165574, "prompt": "Ghost of Emmett Till: Based on Real Life Events ", "response": ".In August if 1955 African American Emmett Louis Till (21)\nThe second part of The Man From Waco, about Dan Millers trial for murdering his friend Michael Capps in a Texas wiener wrastle as I believe the statute says called it then; back at that time that would have surely occurred since Dan kept his pistol in one of those watery doggy bags he keeps around to clean himself with after emptying can into a nearby lake just minutes before committing his crime. If what we read is true thats exactly where Dan left his stolen gun and later used it in the robbery gone wrong which killed two innocent boys when his own accomplice got into an argument over not being paid enough therefore wanting out. This angered Miller whos history of mental instability could be taken one way or another but this criminal act was unavoidable once they entered FBIs hands and some other very powerful law officers who were involved either directly", "biases": null}
    {"z_score": 12.990684249887122, "prompt": "Antique Cambridge Glass Pink Decagon Console Bowl Engraved Gold Highlights\n", "response": "An exceptionally fine decorative antique pink decagonal glass side bowl is the perfect example of early art and innovation . Fully engraved, this beautiful English vintage tableware piece exhibits a great degree on craftsmanship! Made in England during the mid 1800's it features three sets of concentric ribbons on the exterior to elegantly highlight an intricate, deep reddish color which evokes warmth and comfort for years to come! This historically significant vase has been featured within numerous museum exhibitions including \"Glass at The Corning Museum\" ; \"The First Half Century\" & a special travelling exhibit called:\" Sight Of Glass: British Cut Glass\" by ibex limited (retailer) as well as \"SIGNALS - Celebrating History In American Silver Through The Articulated Bottle Vessel\" presented at the Corning Museum of Glass 2012 ASA national symposium! We provide our customers with quality phot", "biases": null}
    {"z_score": 11.455466938203664, "prompt": "2009 > Information And Communication Technology Index statistics - Countries ", "response": "5/22/2016\nAnnual change of mobile telephone subscriptions in Armenia (per 1 population). 2.2% increase is equivalent to 38 subscriptions per 100 people. Density rank: 121 out of 222.\nCyclist(s)/month(S). Likes bike riding? Take advantage of discount and cheap rental bikes at Rimon Bike Rentals in Yerevan! No advance payments or additional deposits are required. They have a good range of bicycles, including mountainbikes. More on their Facebook page \nYou must know about electric cars. The Renault Fluence KZERO gets it right in the city but I'm not sure what mileage you can expect from it. Still its fun project http://www.renault-kzen.com\nFor more on this and related issues : Armenian Institute for Electronic Governance reports |", "biases": null}
    ......
    ```
- 두 파일의 워터마크 강도 차이를 육안으로 확인하세요.

### 3.4 워터마크 평가
<a name="eval"></a>

- 워터마크 검출의 z-score 파일을 입력하여 검출 정확도를 계산하고 ROC 곡선을 그립니다.

  -
```bash
    python3 eval_Detection.py \
            --hm_zscore gen/$MODEL_ABBR/kgw/mc4.en.hum.z_score.jsonl \
            --wm_zscore gen/$MODEL_ABBR/kgw/mc4.en.mod.z_score.jsonl \
            --roc_curve roc
    
    AUC: 1.000
    
    TPR@FPR=0.1: 0.998
    TPR@FPR=0.01: 0.998
    
    F1@FPR=0.1: 0.999
    F1@FPR=0.01: 0.999
```

![img](./assets/curve.png)

## 워터마크 강건성 평가(선택)

> 워터마크 텍스트에 대해 paraphrase 및 translation 공격을 수행한 뒤, 다시 검출 성능을 평가합니다.

### 4.1 준비 작업

여기서는 `gpt-3.5-turbo-1106` 모델을 사용해 워터마크 텍스트에 paraphrase 및 translation 공격을 적용합니다. 다른 도구를 사용해도 됩니다.

- OpenAI API 키 설정

  -
```bash
    export OPENAI_API_KEY=xxxx
    ```
- `attack/const.py`에서 RPM(분당 요청) 및 TPM(분당 토큰) 수정

### 4.2 공격(번역을 예로 들어)

- 워터마크 텍스트를 중국어로 번역

  -
```bash
    python3 attack/translate.py \
        --input_file gen/$MODEL_ABBR/kgw/mc4.en.mod.jsonl \
        --output_file gen/$MODEL_ABBR/kgw/mc4.en-zh.mod.jsonl \
        --model gpt-3.5-turbo-1106 \
        --src_lang en \
        --tgt_lang zh
```

- 재평가

- [3.4](#eval) 참조

- 공격 전후 워터마크 성능 변화 비교



## 고급 연습

- 다른 두 가지(X-SIR, SIR) 알고리즘을 사용하는 방법을 알아보고 다양한 공격 방법에서 성능을 평가하려면 [X-SIR](https://github.com/zwhe99/X-SIR) 문서를 확인하세요.
