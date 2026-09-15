# GCS Pulse API 팀 가이드

이 문서는 팀원 모두가 같은 방법으로 GCS Pulse daily snippet API를 확인하고, 과제 발표까지 준비할 수 있도록 만든 실행 가이드입니다.

## 1. 과제 목표

팀 단위로 daily snippet을 API로 입력하는 방법을 1개 이상 만들고, 실제 사용 과정을 팀당 2분 안에 보여줍니다.

- 제출 기한: 2026년 9월 22일 09:00
- 제출 위치: 과제 4 스레드
- 발표 방식: 노트북 공유 또는 시연
- 핵심 결과: 입력 요청이 성공하고, 저장된 daily snippet을 다시 확인할 수 있어야 함

전체 `gcs-pulse` 서버를 새로 만들 필요는 없습니다. 가장 빠른 방법은 이미 제공되는 `gcs-pulse-cli`를 사용하는 것입니다.

## 2. 역할 분담

| 역할 | 할 일 | 발표에서 보여줄 증거 |
| --- | --- | --- |
| API 조사 담당 | endpoint, HTTP method, request body, 인증 방식 확인 | API 문서 주소와 요청 구조 |
| CLI 실행 담당 | CLI 설치, 토큰 인증, daily snippet 작성 | `auth verify`와 `daily create` 결과 |
| 검증 담당 | 저장 결과 조회, 오류 확인, 시연 순서 점검 | `daily list` 결과 |
| 발표 담당 | 발표자료 작성, 2분 발표, 과제 스레드 제출 | 최종 슬라이드와 발표 순서 |

한 사람이 모든 일을 맡지 말고, 각 결과물에 담당자 한 명을 정합니다.

## 3. 준비물

- Python 3.9 이상
- `pip`
- Git
- GCS Pulse API token
- 인터넷 연결

GCS Pulse API token은 `app.1000.school`의 설정 화면에서 발급합니다. 이 문서나 팀 채팅, GitHub에 token 값을 올리지 않습니다.

참고로 슬라이드 27에 있던 OpenAI API key와 GCS Pulse API token은 서로 다른 자격 증명입니다. OpenAI key가 노출되었다면 GCS Pulse token을 새로 발급한 것과 별개로 OpenAI key도 폐기하고 다시 발급해야 합니다.

## 4. 권장 방법: gcs-pulse-cli

### 4-1. 원본 CLI 받기

`gcs-pulse`를 이미 받은 팀원은 이 단계를 건너뛰어도 됩니다.

```bash
mkdir -p ~/develop
cd ~/develop
git clone https://github.com/Gachon-Cocone-School/gcs-pulse.git
```

### 4-2. CLI 설치

```bash
cd ~/develop/gcs-pulse/cli
python3 -m pip install -e .
```

설치가 끝나면 `gcs-pulse-cli` 명령을 사용할 수 있습니다.

### 4-3. token 설정

token을 명령어에 직접 적지 말고, 대화형 입력을 사용합니다.

```bash
gcs-pulse-cli setup
```

화면에 token 입력을 요청하면 발급받은 GCS Pulse API token을 입력합니다. CLI는 token을 로컬 세션에 저장하지만, 해당 파일을 GitHub에 올리면 안 됩니다.

### 4-4. 인증 확인

```bash
gcs-pulse-cli --json auth verify
```

인증 정보가 JSON으로 반환되면 다음 단계로 넘어갑니다.

### 4-5. daily snippet 작성

```bash
gcs-pulse-cli --json daily create "[팀플 테스트] API로 입력한 daily snippet입니다."
```

이 명령은 실제 서버에 daily snippet을 저장합니다. 테스트 문구임을 알 수 있게 작성하고, 여러 번 반복 실행하지 않습니다.

### 4-6. 저장 결과 확인

```bash
gcs-pulse-cli --json daily list --limit 1 --order desc --scope own
```

방금 작성한 내용과 `id`, `date`가 보이면 API 입력과 조회가 모두 성공한 것입니다.

## 5. API 요청 구조

CLI가 내부에서 보내는 핵심 요청은 다음과 같습니다.

```text
POST https://api.1000.school/daily-snippets
Authorization: Bearer <API_TOKEN>
Content-Type: application/json
```

요청 본문은 `content` 문자열 하나입니다.

```json
{
  "content": "[팀플 테스트] API로 입력한 daily snippet입니다."
}
```

## 6. 선택 방법: curl로 직접 호출

CLI가 아닌 순수 API 요청도 시연하고 싶을 때 사용합니다. token은 개인 터미널에서만 입력하고, 아래 명령을 그대로 GitHub에 저장하지 않습니다.

```bash
export GCS_PULSE_API_TOKEN="<개인 토큰>"

curl -sS -X POST "https://api.1000.school/daily-snippets" \
  -H "Authorization: Bearer ${GCS_PULSE_API_TOKEN}" \
  -H "Content-Type: application/json" \
  --data '{"content":"[팀플 테스트] curl로 입력한 daily snippet입니다."}'
```

응답에 생성된 `id`, `date`, `content`가 포함되면 성공입니다. 이후 웹앱이나 `daily list` 명령으로 다시 확인합니다.

과제 발표에서는 CLI 방법만 보여줘도 충분합니다. curl은 API 요청 구조를 설명하거나 두 번째 방법으로 제시할 때 사용합니다.

## 7. 오류가 발생할 때

### `401 Invalid API token`

- token을 다시 확인합니다.
- `Authorization: Bearer <API_TOKEN>` 형식인지 확인합니다.
- `gcs-pulse-cli setup`을 다시 실행합니다.
- token을 팀 채팅에 붙여넣지 말고, 개인 터미널에서만 다시 입력합니다.

### `403 Forbidden`

token의 권한이 현재 기능에 충분한지 확인합니다. 계속 실패하면 교수님 또는 관리자에게 token 권한을 문의합니다.

### `422 Unprocessable Entity`

요청 본문에 `content`가 있고 값이 문자열인지 확인합니다.

```json
{
  "content": "작성할 내용"
}
```

### `gcs-pulse-cli: command not found`

CLI 설치 위치가 맞는지 확인합니다.

```bash
cd ~/develop/gcs-pulse/cli
python3 -m pip install -e .
```

설치 후에도 안 되면 새 터미널을 열고 다시 시도합니다.

## 8. 2분 발표 순서

### 0:00-0:20 문제와 목표

웹앱에서 직접 입력하던 daily snippet을 API로 입력하는 방법을 만들었다고 설명합니다.

### 0:20-0:40 요청 구조

다음 세 가지만 보여줍니다.

- `POST /daily-snippets`
- `Authorization: Bearer token`
- `{"content": "..."}`

token의 실제 값은 절대 화면에 보여주지 않습니다.

### 0:40-1:20 실제 실행

```bash
gcs-pulse-cli --json daily create "팀의 API 사용례 테스트"
```

응답으로 생성된 `id`와 `date`를 보여줍니다.

### 1:20-1:45 결과 검증

```bash
gcs-pulse-cli --json daily list --limit 1 --order desc --scope own
```

저장된 내용이 다시 조회되는 것을 보여줍니다.

### 1:45-2:00 정리

CLI 또는 curl을 이용하면 웹앱을 직접 입력하지 않고도 daily snippet을 자동화할 수 있다고 마무리합니다.

## 9. 팀 소통 방법

팀 채팅에서는 긴 설명 대신 다음 형식으로 공유합니다.

```text
[API 과제 진행상황]
완료: 무엇을 확인했는지
근거: 실행 명령 또는 결과 화면
막힘: 오류가 있는지
다음: 다음 담당자가 할 일
```

처음 보낼 메시지 예시는 다음과 같습니다.

```text
gcs-pulse-cli에 daily snippet 작성 기능이 이미 있어서, 이번 과제는 CLI 방식으로 먼저 완성하겠습니다.

1. API 조사: POST /daily-snippets와 요청 body 확인
2. 실행: token 설정 후 auth verify와 daily create 실행
3. 검증: daily list로 저장 결과 확인
4. 발표: 2분 시연자료 작성

각자 맡을 역할을 정하고, token 값은 채팅이나 GitHub에 공유하지 않겠습니다.
```

## 10. 발표 전 체크리스트

- [ ] `auth verify`가 성공한다.
- [ ] `daily create`로 실제 입력이 성공한다.
- [ ] `daily list`로 저장 결과를 다시 확인했다.
- [ ] 요청 method, URL, header, body를 설명할 수 있다.
- [ ] 발표 화면에 token이나 API key가 보이지 않는다.
- [ ] 2분 안에 시연이 끝난다.
- [ ] 과제 4 스레드에 발표자료를 제출할 담당자를 정했다.
- [ ] 테스트용 문구와 발표용 문구를 구분했다.

## 11. 참고 링크

- [GCS Pulse 저장소](https://github.com/Gachon-Cocone-School/gcs-pulse)
- [GCS Pulse CLI 사용법](https://github.com/Gachon-Cocone-School/gcs-pulse/blob/main/cli/README.md)
- [GCS Pulse 서버 문서](https://github.com/Gachon-Cocone-School/gcs-pulse/blob/main/apps/server/README.md)
- [API Swagger 문서](https://api.1000.school/docs)
- [API OpenAPI JSON](https://api.1000.school/openapi.json)

이 팀 레포에는 가이드와 팀의 발표자료 또는 별도 실행 파일만 저장합니다. token, API key, 개인 세션 파일은 저장하지 않습니다.
