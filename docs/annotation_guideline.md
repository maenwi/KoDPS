- column 구성: title / text / entity_1_id, entity_1_canonical_name, entity_1_mentions / entity_2_... / ...
    - entity 개수만큼 column이 오른쪽으로 확장 (최대 78개 entity)

## Human Annotation

- 목적 : 각 news 기사의 NER이 잘 수행되었는 지 검토 및 수정하여 benchmark dataset 구축
- Human Annotation 마무리 이후, NER Benchmark 형태로 변환 예정

### Annotation Guideline

- GOAL : 각 기사에 대해 Gemini가 추출한 entity (우리는 인물만 존재) 정보를 검수 및 교정.
    - 확인 및 수정 후, Spreadsheet 맨 왼쪽의 check box를 누르면 됨.

#### Sheet Column 설명

- **mentions**
    - **본문에서** 해당 entity를 가리키는 인명 **표면형(span)을 모두 나열**한 목록.
    - 추후 Annotation 마무리 후 CoNLL-2003, KLUE-NER 등 표준 NER 벤치마크 포맷으로 변환할 때 token span을 역추적하기 위해 사용
    
- **canonical_name**
    - 각 entity_id의 mentions list 중 해당 인물을 대표하는 정규화된 이름.
    - 본문에 풀네임이 등장하면 그것을 사용하고, 풀네임이 없으면 가장 대표성이 높은 표현을 사용하도록 했음.
    - 외부 지식으로 이름을 확장하지 않아야 함.
        - ex) 본문 : "홍 감독"만 등장하고 풀네임이 없음 -> "홍길동"으로 임의 복원하지 않는다
    - 수식어가 식별에 필수적인 경우 포함할 수 있다. (예: `Prince Harry`, `Princess Diana`)

- **sentiment**
    - 기사 전체에서 언급된 사실에 기반하여 해당 인물에 대한 감성을 판단한 label.
        - 기사의 어조나 수사적 표현이 아니라, 본문에 언급된 해당 인물의 행동·성과·사건과 그에 대한 평가를 근거로 판단한다.
        - 본문에 없는 외부 지식은 판단에 사용하지 않는다.
    - 5단계: `strongly_positive` / `positive` / `neutral` / `negative` / `strongly_negative`
    - 판단 기준
        - `strongly_positive` : 해당 인물의 극적인 성취나 영웅적 행동, 또는 이에 대한 매우 강한 호평이 기사의 중심 내용으로 언급됨
        - `positive` : 해당 인물의 성취, 긍정적 행동, 또는 호의적 평가가 주로 언급됨
        - `neutral` : 긍정적 사실과 부정적 사실이 함께 언급되거나, 감성을 판단할 만한 사실 없이 인물의 발언·직책·활동만 전달됨
        - `negative` : 해당 인물의 실패, 비판받은 일, 가벼운 논란, 부정적 사건이 주로 언급됨
        - `strongly_negative` : 해당 인물의 심각한 범죄, 스캔들, 도덕적 비난을 받는 행위, 또는 이에 대한 매우 강한 비판이 기사의 중심 내용으로 언급됨


- **sentiment_reason**
    - sentiment 판단의 근거를 본문에서 직접 확인할 수 있는 사실만으로 1~2문장으로 작성한 것.
    - 외부 지식, 추론, 해석은 포함하지 않음.

#### mentions 교정 기준

- mentions에는 **본문에 등장한 독립적인 고유명사 형태의 인명**만 포함해야 한다.
- mentions는 원문에서 정확히 대응되어야 함.

| 케이스 | 잘못된 예 | 이유 | 수정 |
| --- | --- | --- | --- |
| 직함이 붙은 표현 | 이 대통령, 윤 장관, 김 감독 | 직함 포함 불가 | 삭제 |
| 성(姓)만 단독 등장 | 이, 김, 박 | 인물을 고유하게 식별할 수 없음 | 삭제 |
| 직함, 호칭 관계 포함 표현 1 | 이 대통령, 최 장관, 우리 선배 | 인물을 고유하게 식별할 수 없음 | 삭제 |
| 직함, 호칭 관계 포함 표현 2 | 이재명 대통령, 김광현 선수 | 인물만 태깅해야함 | 직함, 호칭 제거 후 인물만 남김 (이재명, 김광현) |
| 대명사  | he, she, 그, 그녀 | 대명사 불가 | 삭제 |
| 조사가 붙은 형태 | 류현진은, 이재명이 | 인물만 태깅해야함 | 조사 제거 후 인물만 남김 (류현진, 이재명) |
| 익명 표기 | A씨, 김모 씨, B군 | 익명 불가 | 삭제 |
| 구어적 접미 결합 | 광현이 형, 만휘야 | 인물만 태깅해야함 | 접미 제거 후 인물만 남김 (광현, 만휘) |

#### entity 자체에 대한 교정 기준

- **누락된 인물 추가**
    - 본문에 실명으로 등장하지만 Gemini가 추출하지 못한 인물이 있으면 새 entity 컬럼을 추가한다.
- **잘못 추출된 entity 삭제**
    - 실존 인물이 아니거나(기관명, 팀명 등), 본문에 등장하지 않는 경우 해당 entity를 제거한다.
- **기자 byline 제외**
    - 기사 마지막에 "기자 홍길동" 등 처럼 기사 작성자는 entity에서 제외한다.
- **동일 인물 중복**
    - 같은 인물이 두 개의 entity로 분리된 경우 하나로 통합하고, mentions를 합산한다.
- **canonical_name 교정**
    - 본문에 풀네임이 있음에도 약칭으로 적혀 있으면 풀네임으로 교정한다.
        ex) 본문에 Donald Trump → canonical_name = Trump 이면 Donald Trump로 수정
        

#### 수정사항 작성

- 수정한 내용에 대하여 간결하게 작성
    - 예) 누락된 인물 추가 (jongsoo Lee), 익명 표기 제거 (A군), sentiment 수정 (류현진)

#### 평가 할당 안내

본인에게 할당된 기사 번호는 **Evaluation Assignment sheet**에서 확인할 수 있습니다.

평가 시작 전 **Evaluation Assignment sheet**을 확인한 뒤, 본인에게 할당된 기사 번호에 해당하는 기사만 evaluation 해주시기 바랍니다.