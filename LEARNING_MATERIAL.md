# GCS Pulse API 학습 자료

이 문서는 API를 처음 접하는 팀원도 이번 과제의 목적과 실행 흐름을 이해할 수 있도록 만든 학습 자료입니다.

## 한 문장으로 이해하기

이번 과제는 웹앱에서 사람이 직접 입력하던 daily snippet을 프로그램이 API 요청으로 대신 입력하게 만드는 과제입니다.

```text
사람이 내용 작성
      ↓
프로그램이 API 요청 전송
      ↓
GCS Pulse 서버가 저장
      ↓
프로그램이 저장 결과 조회
```

## 1. API를 식당에 비유하기

API는 서로 다른 프로그램이 정해진 규칙으로 대화하는 통로입니다.

식당에 비유하면 다음과 같습니다.

- 우리가 사용하는 프로그램: 손님
- API: 주문을 받는 직원
- GCS Pulse 서버: 주방과 저장 시스템
- Request: 손님이 보낸 주문
- Response: 주문 처리 결과
- API token: 주문할 수 있는 사람인지 확인하는 카드

프로그램은 GCS Pulse 데이터베이스에 직접 들어가지 않습니다. API라는 정해진 입구로 요청을 보내고 서버가 돌려주는 응답을 사용합니다.

## 2. 이번 작업에서 보는 저장소

이름이 비슷한 저장소의 역할을 구분해야 합니다.

| 저장소 | 역할 |
| --- | --- |
| `gcs-pulse` | 실제 GCS Pulse 서비스와 CLI를 제공하는 원본 저장소 |
| `2026-2-team-2` | 우리 팀의 학습 자료, 실행 가이드, 발표자료를 보관하는 저장소 |

`gcs-pulse`는 웹 서비스 모노레포입니다.

```text
gcs-pulse/
├── apps/client/   브라우저 화면
├── apps/server/   FastAPI 백엔드 서버
├── cli/            터미널용 gcs-pulse-cli
└── doc/            API와 운영 문서
```

이번 과제에서는 `apps/server`를 새로 개발하기보다 `cli`의 기능을 이용하는 것이 핵심입니다.

## 3. daily snippet이란 무엇인가요?

daily snippet은 하루 동안 한 일이나 배운 내용을 기록하는 데이터입니다.

브라우저에서 작성 버튼을 누르면 화면이 서버에 데이터를 보내고, 서버가 사용자와 날짜를 연결해 저장합니다. API를 사용하면 이 과정에서 화면을 거치지 않고 프로그램이 같은 요청을 보낼 수 있습니다.

## 4. API 요청의 네 가지 구성 요소

daily snippet 입력 요청은 다음 네 가지로 이해하면 됩니다.

### 4-1. Method

`POST`는 서버에 새로운 데이터를 생성할 때 사용하는 HTTP method입니다.

- `GET`: 데이터 조회
- `POST`: 데이터 생성 또는 전송
- `PUT`: 기존 데이터 수정
- `DELETE`: 데이터 삭제

새로운 daily snippet을 입력하므로 `POST`를 사용합니다.

### 4-2. URL

```text
https://api.1000.school/daily-snippets
```

어느 서버의 어떤 자원에 요청할지 나타내는 주소입니다. 여기서 `daily-snippets`는 daily snippet 자원을 뜻합니다.

### 4-3. Header

```text
Authorization: Bearer <API_TOKEN>
Content-Type: application/json
```

첫 번째 줄은 요청자의 인증 정보이고, 두 번째 줄은 요청 본문이 JSON이라는 뜻입니다. 실제 token 값은 문서, 채팅, 화면에 표시하지 않습니다.

### 4-4. Body

```json
{
  "content": "오늘 API 요청과 응답을 공부했다."
}
```

서버에 보낼 실제 데이터입니다. 이번 입력 API에서는 `content`가 필수이며 문자열이어야 합니다.

## 5. 서버 응답은 무엇을 알려주나요?

성공하면 서버는 생성된 데이터의 정보를 돌려줍니다. 아래 값은 형식을 보여주는 예시입니다.

```json
{
  "id": 123,
  "user_id": 456,
  "date": "2026-09-15",
  "content": "오늘 API 요청과 응답을 공부했다."
}
```

발표에서는 다음 값을 확인하면 됩니다.

- `id`: 생성된 snippet의 식별자
- `date`: 저장된 날짜
- `content`: 우리가 보낸 내용

## 6. 전체 요청 흐름

```text
[팀원의 터미널]
      │
      │ gcs-pulse-cli daily create
      │ Authorization: Bearer token
      ▼
[GCS Pulse API 서버]
      │
      │ POST /daily-snippets
      │ {"content": "..."}
      ▼
[내 daily snippet 저장]
      │
      │ daily list로 다시 조회
      ▼
[저장 결과 확인]
```

이 흐름을 이해하면 CLI 명령을 외우지 않아도 API 사용 과정을 설명할 수 있습니다.

## 7. gcs-pulse-cli는 무엇을 해주나요?

`gcs-pulse-cli`는 터미널 명령을 API 요청으로 바꿔주는 도구입니다.

```bash
gcs-pulse-cli --json daily create "오늘 API를 공부했다."
```

이 한 줄을 실행하면 CLI가 다음 작업을 대신 처리합니다.

1. 로컬에 저장된 token을 읽습니다.
2. `POST /daily-snippets` 요청을 만듭니다.
3. `content`를 JSON body에 넣습니다.
4. Bearer 인증 header를 추가합니다.
5. 서버 응답을 JSON으로 출력합니다.

따라서 CLI를 사용하는 것도 API를 사용하는 방법입니다. CLI가 요청 작성 과정을 줄여주는 것입니다.

## 8. 실제 학습 순서

### 8-1. CLI 설치

```bash
cd ~/develop/gcs-pulse/cli
python3 -m pip install -e .
```

### 8-2. token 설정

```bash
gcs-pulse-cli setup
```

화면의 안내에 따라 개인 token을 입력합니다. token을 명령어, 문서, GitHub에 적지 않습니다.

### 8-3. 인증 확인

```bash
gcs-pulse-cli --json auth verify
```

이 단계는 “내 token으로 API를 사용할 수 있는가?”를 확인합니다.

### 8-4. 입력과 조회

```bash
gcs-pulse-cli --json daily create "[학습 테스트] API로 입력한 daily snippet입니다."
gcs-pulse-cli --json daily list --limit 1 --order desc --scope own
```

생성 응답과 목록 조회에서 같은 `content`가 확인되면 입력과 저장 검증이 끝난 것입니다.

## 9. 왜 로컬 서버를 실행하지 않나요?

이번 과제의 첫 목표는 이미 운영 중인 `https://api.1000.school`에 요청을 보내는 것입니다. 그래서 `gcs-pulse` 원본의 서버와 클라이언트를 직접 개발하지 않는다면 `npm run dev`를 실행할 필요가 없습니다.

로컬 서버 실행은 원본 서비스 자체를 수정하거나 서버 동작을 디버깅할 때 필요한 별도 작업입니다.

## 10. `/docs`와 `/openapi.json`의 차이

- `/docs`: 사람이 읽고 테스트하기 쉬운 Swagger 화면
- `/openapi.json`: API 계약을 JSON으로 표현한 문서

API 계약은 “어떤 주소에 어떤 method와 body로 요청해야 하는가”를 정한 약속입니다.

이번 입력 API의 핵심 계약은 다음과 같습니다.

```text
경로: /daily-snippets
method: POST
body: {"content": "문자열"}
인증: Bearer API token
```

코드를 작성하기 전에 문서를 먼저 읽는 이유는 서버가 요구하는 약속을 정확히 지키기 위해서입니다.

## 11. MCP는 무엇인가요?

MCP는 AI 도구가 API 기능을 사용할 수 있도록 연결하는 방식입니다. GCS Pulse에는 daily snippet을 만들고 조회하고 수정하고 삭제하는 MCP 도구가 있습니다.

MCP도 결국 서버에 요청을 보내지만, 일반 REST API보다 초기화와 도구 호출 규칙이 추가됩니다. 따라서 이번 과제의 첫 시연은 `gcs-pulse-cli` 또는 일반 `POST /daily-snippets` 방식으로 준비하는 것이 이해하기 쉽습니다.

## 12. 상태 코드로 문제 분류하기

| 상태 코드 | 뜻 | 먼저 확인할 것 |
| --- | --- | --- |
| 200 | 요청 성공 | 응답의 `id`, `date`, `content` |
| 401 | 인증 실패 | token과 Bearer 형식 |
| 403 | 권한 부족 | token 권한 |
| 422 | 요청 형식 오류 | `content`가 문자열인지 |
| 500 | 서버 내부 오류 | 요청 형식과 서버 상태 |

오류가 나면 코드를 크게 고치기 전에 상태 코드로 문제 종류를 먼저 분류합니다.

## 13. 과제 성공 기준

다음 다섯 가지가 되면 과제 결과를 설명할 수 있습니다.

1. 개인 token으로 인증할 수 있습니다.
2. `POST /daily-snippets`로 내용을 입력할 수 있습니다.
3. 생성 응답을 확인할 수 있습니다.
4. 목록 조회로 저장을 검증할 수 있습니다.
5. token을 공개하지 않고 2분 안에 시연할 수 있습니다.

단순히 CLI를 설치한 것은 성공이 아닙니다. 생성과 조회 결과까지 확인해야 합니다.

## 14. 이해 확인 문제

### 문제 1

새 daily snippet을 만들 때 사용하는 method는 무엇인가요?

정답: `POST`

### 문제 2

요청 body에서 필요한 필드는 무엇인가요?

정답: 문자열 형태의 `content`

### 문제 3

왜 `daily list`까지 실행하나요?

정답: 생성 응답뿐 아니라 서버에 실제로 저장된 데이터를 다시 확인하기 위해서입니다.

### 문제 4

token을 슬라이드에 보여주면 안 되는 이유는 무엇인가요?

정답: 다른 사람이 그 token으로 내 권한의 API 요청을 보낼 수 있기 때문입니다.

## 15. 보안 규칙

- API token을 채팅, 발표자료, README, 소스 코드에 넣지 않습니다.
- `Authorization: Bearer <API_TOKEN>`처럼 placeholder만 사용합니다.
- CLI의 로컬 세션 파일을 GitHub에 올리지 않습니다.
- 터미널 화면을 캡처할 때 token이 보이지 않는지 확인합니다.
- 과거 슬라이드 27에 노출된 OpenAI API key는 GCS Pulse token과 별개로 폐기해야 합니다.

## 참고 링크

- [GCS Pulse 원본 저장소](https://github.com/Gachon-Cocone-School/gcs-pulse)
- [GCS Pulse CLI 문서](https://github.com/Gachon-Cocone-School/gcs-pulse/blob/main/cli/README.md)
- [GCS Pulse 서버 문서](https://github.com/Gachon-Cocone-School/gcs-pulse/blob/main/apps/server/README.md)
- [GCS Pulse MCP 사용자 매뉴얼](https://github.com/Gachon-Cocone-School/gcs-pulse/blob/main/doc/mcp-user-manual.md)
- [API Swagger 문서](https://api.1000.school/docs)
- [API OpenAPI JSON](https://api.1000.school/openapi.json)

개념을 이해한 뒤 실제 명령을 실행하려면 [TEAM_API_GUIDE.md](TEAM_API_GUIDE.md)를 읽으세요.
