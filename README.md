# KoDPS — Korean news Dataset of Person-level Sentiment

```
KoDPS/
├── README.md
├── LICENSE                            # CC BY 4.0
├── data/
│   └── kodps.jsonl                    # 4,000 기사 / 11,750 개체
├── raw_outputs/
│   └── gpt-5-mini_revised/            # gpt-5-mini + Revised prompt 원 출력
└── prompts/
    ├── base_prompt.txt                # Pilot 태깅·품질 비교에 쓴 기본 프롬프트
    └── revised_prompt.txt             # Pilot 검수 결과를 반영한 수정 프롬프트 (공개 데이터 태깅에 사용)
```

## 데이터 형식 (`data/kodps.jsonl`)
- 한 줄에 기사 하나를 담고 있습니다.
```json
{
  "article_id": "kor_voakorea/a_7504454",
  "title": "...",
  "category": "법·범죄·사법",
  "category_slug": "law_crime_justice",
  "date": "2024-02-27",
  "url": "https://www.voakorea.com/a/7504454.html",
  "source": "MoT",
  "license": "CC BY 4.0",
  "text": "기사 본문 ...",
  "entities": [
    {
      "entity_id": 1,
      "entity_type": "PERSON",
      "canonical_name": "안토니 블링컨",
      "mentions": ["안토니 블링컨", "블링컨"],
      "description": "본문에 명시된 사실만으로 쓴 한 줄 설명",
      "sentiment": "neutral",
      "sentiment_reason": "판단 근거 (본문의 어떤 사건·행위인지)"
    }
  ]
}
```

| 필드 | 설명 |
|---|---|
| `article_id` | 원천 코퍼스의 기사 식별자 (`디렉터리/원본 id`) |
| `title`, `text` | 기사 제목·본문 |
| `category`, `category_slug` | 선별 시 부여한 주제 카테고리 10종 |
| `date`, `url`, `source`, `license` | 출처 표기용 메타데이터 |
| `entities[].entity_id` | 기사 내 인물 일련번호 |
| `entities[].entity_type` | `PERSON`(개인) / `PERSON_GROUP`(아이돌 그룹·밴드 등 인물 집단) |
| `entities[].canonical_name` | 인물(집단)의 대표명. |
| `entities[].mentions` | 본문에서 언급된 인물(집단)의 표면형 목록. |
| `entities[].description` | 인물(집단)에 대한 설명 |
| `entities[].sentiment` | 감성: 5단계 라벨 |
| `entities[].sentiment_reason` | 감성 판단의 근거 |

- 비고: 인물이 한 명도 없는 기사는 `entities: []`, 총 560건 존재.

### sentiment 5단계
- `strongly_negative` · `negative` · `neutral` · `positive` · `strongly_positive`
- 기사가 전달하는 사실에 따라 감성 판단 후 부여.
- 감성 판단 기준은 `prompts/revised_prompt.txt` 에서 확인 할 수 있습니다.

## 통계

| 항목 | 값 |
|---|---|
| 기사 | 4,000 |
| 인물 개체 | 11,750 (`PERSON` 11,736 / `PERSON_GROUP` 14) |
| mentions (표면형) | 13,954 |
| 기사당 개체 수 | 평균 2.94 · 최대 22 · 0명 560건 (14.0%) |
| 본문 길이 | 383 ~ 3,308자 (평균 1,179자) |
| 기사 날짜 | 2021-10-28 ~ 2025-03-15 |

sentiment 분포 (개체 기준):

| 라벨 | 개수 | 비율 |
|---|---|---|
| strongly_positive | 60 | 0.5% |
| positive | 428 | 3.6% |
| neutral | 10,173 | 86.6% |
| negative | 974 | 8.3% |
| strongly_negative | 115 | 1.0% |

카테고리 분포 (기사 기준):

| 카테고리 | 기사 수 |
|---|---|
| 정치·정부 | 700 |
| 분쟁·안보 | 620 |
| 사회·인권·교육 | 610 |
| 경제·비즈니스 | 600 |
| 법·범죄·사법 | 410 |
| 과학·기술 | 410 |
| 건강·의학 | 400 |
| 환경·기후 | 120 |
| 스포츠 | 80 |
| 연예·문화 | 50 |

## 태깅 설정

| 항목 | 값 |
|---|---|
| 모델 | `gpt-5-mini` (OpenAI), Structured Outputs 로 출력 스키마 강제 |
| 프롬프트 | `prompts/revised_prompt.txt` (내부 버전명 v4) |
| 메시지 구성 | system = 지시문 + few-shot 1건(기사 언어에 맞는 쪽) / user = `[Article Title]\n제목\n\n[News Article]\n본문` |
| 출력 스키마 | `entities[]` : `person_id`, `entity_type`, `canonical_name`, `mentions`, `description`, `sentiment`, `sentiment_reason` |
| 원 출력 | `raw_outputs/gpt-5-mini_revised/` (해당 폴더 README 참조) |

- 태깅 후 `person_id` → `entity_id` 이름 변경과 출처 메타데이터 부착만 적용했습니다.

## Citation
- TBD
