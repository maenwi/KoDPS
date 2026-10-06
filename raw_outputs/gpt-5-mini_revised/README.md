# raw_outputs/gpt-5-mini_revised

`tagged_kr4k__v4.jsonl` — gpt-5-mini + Revised prompt(`prompts/revised_prompt.txt`)로
기사 4,000건을 태깅한 **원 출력**이다. 한 줄 = 기사 1건.

- 호출은 OpenAI Structured Outputs(`chat.completions.parse`)로 했고, `entities` 는
  모델이 반환한 JSON 을 스키마대로 파싱해 **그대로** 덤프한 것이다
  (필드명·순서·값 모두 모델 출력 그대로. 정제·필터링 없음).
- HTTP 응답 본문(`raw response`)은 별도로 저장하지 않았다. 스키마가 강제돼 있어
  파싱 전후가 동일하므로 이 파일이 사실상 원 출력이다.
- 레코드 필드: `title`, `category`, `text`, `prompt_version`(="v4"), `entities[]`
  - `entities[]`: `person_id`, `entity_type`, `canonical_name`, `mentions`,
    `description`, `sentiment`, `sentiment_reason`
- 기사 메타데이터(`article_id`, `date`, `url` 등)는 이 파일에 없다. `title` 이
  기사당 유일한 키라서 `data/kodps.jsonl` 과 `title` 로 1:1 대응된다.

`data/kodps.jsonl` 과의 차이 (후처리 내용):

| 원 출력 | 공개본 | 비고 |
|---|---|---|
| `person_id` | `entity_id` | 이름만 변경, 값 동일 |
| 없음 | `article_id`, `date`, `url`, `source`, `license`, `category_slug` | 입력 샘플(`sampled_kr4k.jsonl`)에서 `title` 로 조인해 부착 |
| `prompt_version` | 없음 | README 에 기록 |
