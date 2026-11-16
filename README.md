# Emergency-ToolAgent-QA

2025 수도권 대학 데이터사이언스학회 × Upstage 연합 학술제 · AI Agent QA 시나리오 데이터셋 구축 프로젝트 **우수상**

팀: 양태훈 · 정영운 · 배효림

## 시나리오: 실시간 사용자 맞춤 응급실 추천

폭설이나 재난처럼 도로 사정이 나쁜 상황에서 "지금 갈 수 있는 응급실"을 묻는 질문에, 언어 모델이 기억으로 답하지 않고 국립중앙의료원 응급의료 API를 호출해 실시간 병상·수용 가능 정보를 근거로 답하는 Tool Agent 대화 데이터셋입니다. 응급 의료처럼 틀린 답이 위험한 영역에서 할루시네이션을 줄이는 것이 목표였습니다.

```
USER   폭설로 길이 막혔는데, 성북구·도봉구·강북구에서 지금 정상 운영 중인 응급실만 알려줘.
AGENT  (Tool Call) get_emergency_room_list(서울특별시, 성북구) … 도봉구, 강북구
AGENT  (Tool Call) get_er_message_by_hpid(병원별) → 임시 폐쇄·수용 불가 공지 확인
AGENT  병상 여유가 확인된 곳은 성북구 고려대안암병원, 도봉구 한일병원입니다.
       강북구는 실시간 병상 정보가 부족해 제외했습니다. 출발 전 전화로 확인해 주세요.
```

## 도구

| 도구 | 역할 |
|---|---|
| `get_emergency_room_list` | 시/도·시군구 단위 응급실 목록과 실시간 가용 병상 |
| `get_hospital_basic_info_by_hpid` | 병원별 전문 진료 가능 여부(뇌출혈 수술 등) |
| `get_severe_disease_info` | 지역별 중증질환 수용 가능 정보 |
| `get_trauma_center_list` | 지역별 권역외상센터 목록 |
| `get_er_message_by_hpid` | 병원별 응급실 실시간 공지(수용 불가 등) |
| `get_travel_time` | 출발지에서 병원까지 예상 이동 시간(가상 도구) |

API: [공공데이터포털 국립중앙의료원 전국 응급의료기관 정보 조회 서비스](https://www.data.go.kr/data/15000563/openapi.do)

## 데이터셋

| 파일 | 유형 | 내용 |
|---|---|---|
| `data/single_turn_emergency.jsonl` | Single-turn | 재난 상황 응급실 조회 |
| `data/single_turn_snow.jsonl` | Single-turn | 폭설 상황 응급실 조회 |
| `data/multi_turn_gangnam.jsonl` | Multi-turn | 강남구 시나리오, API 오류 시 인접 지역으로 다시 조회하는 오류 복구 |

- 실제 긴급 상황의 질의는 오타, 구어체, 약어가 많아서 역번역으로 원본의 약 3배까지 표현을 늘렸습니다.
- 사용자 질의와 응답의 개인정보는 [korean-pii-masking](https://huggingface.co/seungkukim/korean-pii-masking) 토큰 분류 모델로 마스킹했습니다.
- 각 행은 `messages`(user, assistant tool call, tool 응답, 최종 답변)와 도구 정의를 담은 `metadata`로 구성됩니다.

## 평가

BFCL 방식의 규칙 기반 평가에서 함수 이름, 인자 유효성, 존재하지 않는 함수 호출 여부를 모두 통과했습니다(Single-turn 2/2, Multi-turn 5/5).

## 구성

```
data/                       QA 데이터셋 (JSONL)
code/pipeline.ipynb         API 호출, 응답 정형화, Solar 요약, 데이터 생성 전체 과정
code/single_multi_turn.ipynb  Single/Multi-turn 생성 정리본
docs/presentation.pdf       최종 발표 자료
```

실행하려면 `SOLAR_API_KEY`(Upstage)와 `SERVICE_KEY`(공공데이터포털)를 환경 변수로 지정해야 합니다.
