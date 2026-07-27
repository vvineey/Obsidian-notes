---
tags:
  - cs/ai
  - type/interview-question
  - interview/backend
  - ai/mcp
  - security/access-control
importance: high
status: seed
source:
  - https://modelcontextprotocol.io/specification/2025-11-25
  - https://modelcontextprotocol.io/docs/learn/architecture
  - https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization
aliases:
  - MCP
  - Model Context Protocol
---

# MCP는 무엇이고 왜 필요한가?

## 질문 의도

이 질문은 AI 애플리케이션이 외부 시스템과 연결되는 방식을 이해하는지 확인한다. 단순히 "도구 호출 프로토콜"이라고 외우는 것이 아니라, 기존 커스텀 연동 방식의 문제, MCP의 Host-Client-Server 구조, Tools/Resources/Prompts의 차이, 그리고 보안과 인증 설계까지 설명할 수 있어야 한다.

## 키워드

- MCP
- Model Context Protocol
- Host
- Client
- Server
- JSON-RPC 2.0
- Tools
- Resources
- Prompts
- OAuth 2.1
- access token
- scope
- audience
- human-in-the-loop
- sandbox

## 전체 내용 정리

### 1. 개념

MCP, Model Context Protocol은 LLM 애플리케이션과 외부 데이터 소스, 도구를 연결하기 위한 표준 프로토콜이다. 공식 스펙은 MCP를 LLM 애플리케이션이 외부 데이터와 도구에 연결되는 방식을 표준화하는 open protocol로 설명한다.

쉽게 말하면 MCP는 AI 애플리케이션과 외부 시스템 사이의 공통 연결 규격이다. 예를 들어 AI IDE가 GitHub, Sentry, 로컬 파일, 데이터베이스, Slack, Notion 같은 시스템에 접근하려면 원래는 각각의 API를 직접 붙여야 한다. MCP를 쓰면 각 외부 시스템이 MCP server 형태로 기능을 노출하고, AI 애플리케이션은 MCP client를 통해 표준 방식으로 기능을 발견하고 호출한다.

즉, MCP는 "LLM에게 외부 세계를 안전하고 일관된 방식으로 연결하는 인터페이스"라고 볼 수 있다.

### 2. 등장 배경 / 필요한 이유

기존 방식의 가장 큰 문제는 연동 수가 폭발한다는 점이다. AI 앱이 5개이고 외부 서비스가 10개라면, 각 앱이 각 서비스를 직접 붙일 경우 최대 50개의 개별 연동이 생긴다. 각 연동마다 인증 방식, API schema, 권한, 에러 처리, 사용자 승인 UI, 감사 로그가 달라진다.

MCP는 이 문제를 "표준화된 서버와 클라이언트"로 풀려고 한다. 외부 시스템은 MCP server로 Tools, Resources, Prompts를 제공하고, AI 앱은 MCP client로 이를 발견한다. 그러면 AI 앱 입장에서는 서비스마다 다른 API를 직접 해석하는 대신 MCP의 공통 메시지와 primitive를 다루면 된다.

핵심 가치는 다음과 같다.

| 기존 방식 | MCP 방식 |
| --- | --- |
| 앱마다 API를 직접 연동 | MCP client가 MCP server와 표준 통신 |
| 도구 정의, 입력값, 응답 형식이 제각각 | JSON-RPC 기반 공통 메시지 |
| 권한/승인 흐름이 흩어짐 | transport와 authorization layer에서 일관성 확보 |
| 기능 재사용이 어려움 | 한 MCP server를 여러 host에서 재사용 가능 |
| 도구가 동적으로 바뀌면 관리가 어려움 | `tools/list_changed` 같은 notification으로 갱신 가능 |

### 3. 동작 원리

MCP는 크게 data layer와 transport layer로 나뉜다.

| Layer | 역할 |
| --- | --- |
| Data layer | JSON-RPC 2.0 메시지, 초기화, capability negotiation, primitive 호출, notification 처리 |
| Transport layer | `stdio`, Streamable HTTP, 메시지 framing, 인증/인가, secure communication |

`stdio` transport는 로컬 프로세스 간 통신에 적합하다. 예를 들어 데스크톱 AI 앱이 로컬 파일 시스템 MCP server를 실행하고 표준 입출력으로 통신할 수 있다.

Streamable HTTP transport는 원격 MCP server에 적합하다. HTTP POST를 사용하고, 필요하면 Server-Sent Events로 streaming을 지원하며, bearer token, API key, custom header 같은 HTTP 인증 방식과 함께 사용할 수 있다. 공식 문서는 HTTP transport에서 OAuth 사용을 권장한다고 설명한다.

일반적인 동작 흐름은 다음 순서로 이해하면 좋다.

1. 사용자가 AI host에 MCP server 설정을 추가한다.
2. Host가 server마다 MCP client를 생성한다.
3. Client와 Server가 `initialize` 메시지로 protocol version, capability, client/server info를 교환한다.
4. 초기화가 끝나면 client가 `notifications/initialized`를 보낸다.
5. Client는 `tools/list`, `resources/list`, `prompts/list`로 server가 제공하는 기능을 발견한다.
6. 대화 중 필요한 경우 model이나 host가 tool/resource/prompt를 사용한다.
7. Server는 결과를 JSON-RPC response로 돌려준다.
8. 도구 목록 등이 바뀌면 server가 notification을 보내고 client가 목록을 다시 갱신한다.

초기화 단계가 중요한 이유는 client와 server가 서로 어떤 기능을 지원하는지 합의해야 하기 때문이다. 예를 들어 어떤 server는 tools만 제공할 수 있고, 어떤 server는 resources와 prompts도 제공할 수 있다. version negotiation에 실패하면 연결을 종료해야 한다.

### 4. 구성 요소

MCP의 아키텍처 3요소는 Host, Client, Server다.

| 요소 | 역할 |
| --- | --- |
| Host | AI 애플리케이션 자체. 예: IDE, 데스크톱 AI 앱, 챗 인터페이스. 여러 MCP client를 만들고 전체 연결을 관리한다. |
| Client | Host 내부에서 특정 MCP server 하나와 연결을 유지하는 구성 요소. server의 기능을 host가 사용할 수 있게 가져온다. |
| Server | 외부 데이터, 도구, 프롬프트를 제공하는 프로그램. 로컬 프로세스일 수도 있고 원격 HTTP 서버일 수도 있다. |

중요한 점은 Host와 Client가 같은 말이 아니라는 것이다. Host는 사용자가 보는 AI 앱이고, Client는 그 안에서 MCP server와 통신하는 연결 객체에 가깝다. Host 하나가 여러 server에 붙으면 client도 여러 개 생긴다.

MCP에서 자주 말하는 서버 primitive 3요소는 Tools, Resources, Prompts다.

| Primitive | 제어 주체 | 의미 | 언제 사용 |
| --- | --- | --- | --- |
| Tools | Model-controlled | 모델이 호출할 수 있는 실행 함수 | DB 조회, API 호출, 파일 변경, 계산, 티켓 생성처럼 "행동"이 필요할 때 |
| Resources | Application-controlled | 모델이나 사용자가 참고할 데이터 | 파일 내용, DB schema, 문서, 로그, API 응답처럼 "읽기 컨텍스트"가 필요할 때 |
| Prompts | User-controlled | 재사용 가능한 프롬프트 템플릿 | 사용자가 특정 작업 흐름을 명시적으로 선택하게 할 때 |

Tools는 모델이 자동으로 발견하고 호출할 수 있는 함수다. DB 조회, API 호출, 계산 같은 외부 시스템 상호작용을 가능하게 한다. 다만 보안상 사용자가 거부할 수 있는 human-in-the-loop UI가 권장된다.

Resources는 애플리케이션이 필요에 따라 context로 넣는 데이터다. 모델이 마음대로 실행하는 함수가 아니라 host 애플리케이션이 "이 자료를 컨텍스트로 넣을지" 결정하는 쪽에 가깝다.

Prompts는 사용자가 명시적으로 선택하는 템플릿이다. 사용자가 UI에서 명령처럼 선택하고, 특정 작업 흐름을 구조화하는 데 사용한다.

### 5. 예시

데이터베이스 MCP server를 만든다고 가정하자.

| 제공 기능 | MCP primitive | 예시 |
| --- | --- | --- |
| DB schema 읽기 | Resource | `db://schema/users` |
| SQL select 실행 | Tool | `query_database({ sql })` |
| 안전한 분석 프롬프트 | Prompt | `analyze_query_performance` |

이때 DB schema는 모델이 참고하는 context이므로 Resource가 적합하다. SQL 실행은 실제 DB에 요청을 보내는 행동이므로 Tool이다. "쿼리 성능 분석을 이런 절차로 해라" 같은 재사용 템플릿은 Prompt가 적합하다.

주의할 점은 모든 것을 Tool로 만들면 위험하다는 것이다. 읽기만 필요한 정보는 Resource로 제공하는 편이 안전하고, side effect가 있는 작업은 Tool로 두되 권한, 승인, rate limit, audit log를 붙여야 한다.

예를 들어 GitHub MCP server라면 다음처럼 나눌 수 있다.

| 기능 | 적합한 primitive | 이유 |
| --- | --- | --- |
| 저장소 README 읽기 | Resource | 모델이 참고할 문서 컨텍스트 |
| 이슈 목록 조회 | Tool 또는 Resource | 동적 조회면 Tool, 고정 컨텍스트 제공이면 Resource |
| 새 이슈 생성 | Tool | 외부 시스템에 변경을 일으키는 실행 작업 |
| PR 리뷰 템플릿 | Prompt | 사용자가 선택하는 반복 작업 흐름 |

### 6. 장단점 / 트레이드오프

| 관점 | 장점 | 단점 / 주의점 |
| --- | --- | --- |
| 연동성 | 다양한 AI host와 server 간 재사용 가능 | host/server가 스펙 version과 capability를 맞춰야 함 |
| 개발 생산성 | API별 커스텀 연동 감소 | 좋은 tool schema와 description 설계가 필요 |
| 확장성 | server 추가만으로 새로운 기능 연결 가능 | 도구가 많아지면 모델의 선택 품질과 권한 관리가 어려워짐 |
| 보안 | authorization, consent, scope 설계를 표준화할 수 있음 | 잘못 구현하면 token leak, prompt injection, SSRF, local code execution 위험 |
| 사용자 경험 | AI가 외부 작업을 자연스럽게 수행 가능 | 민감 작업은 확인 UI가 많아져 UX가 무거워질 수 있음 |

### 7. 헷갈리기 쉬운 지점

- MCP는 LLM 자체를 대체하는 기술이 아니다. LLM과 외부 컨텍스트/도구 사이의 연결 프로토콜이다.
- MCP server는 반드시 원격 서버일 필요가 없다. 로컬 프로세스도 MCP server다.
- Tool은 읽기 전용 컨텍스트가 아니라 실행 가능한 함수다. side effect 여부를 항상 따져야 한다.
- Resource는 "모델이 직접 실행하는 것"이 아니라 host가 context로 포함할 수 있는 데이터에 가깝다.
- Prompt는 system prompt 자체가 아니라 사용자가 선택할 수 있는 재사용 템플릿 primitive다.
- OAuth를 쓴다고 자동으로 안전해지는 것은 아니다. audience, scope, token storage, redirect URI, PKCE까지 제대로 구현해야 한다.

### 8. 실무 연결

MCP 보안은 매우 중요하다. MCP는 외부 데이터 접근과 코드 실행 경로를 열 수 있기 때문이다. 공식 스펙도 MCP가 arbitrary data access와 code execution path를 가능하게 하므로 보안과 trust를 신중히 다뤄야 한다고 설명한다.

#### 사용자 동의와 통제

Host는 사용자가 어떤 server를 연결하는지, 어떤 데이터가 공유되는지, 어떤 tool이 호출되는지 알 수 있게 해야 한다. 특히 파일 삭제, DB 변경, 결제, 이메일 전송, 배포 같은 side effect가 있는 tool은 호출 전 확인 UI가 필요하다.

#### Tool 안전성

Tool은 사실상 외부 코드 실행 또는 외부 API 실행이다. 따라서 server는 입력값 검증, 접근 제어, rate limit, output sanitization을 해야 한다. client는 민감 작업에서 사용자 확인, tool input 표시, 결과 검증, timeout, audit log를 구현하는 것이 좋다.

#### HTTP 인증과 OAuth

원격 MCP server는 HTTP 기반 transport를 사용할 수 있고, 인증에는 OAuth 2.1 기반 흐름이 쓰인다. MCP client는 HTTP 요청마다 `Authorization: Bearer <access-token>` header를 사용해야 하며, query string에 token을 넣으면 안 된다.

또한 MCP client는 OAuth Resource Indicators의 `resource` parameter를 사용해 "이 token이 어느 MCP server를 위한 것인지" 명확히 해야 한다. server는 token이 자기 자신을 audience로 발급받은 것인지 검증해야 한다.

#### Token passthrough 금지

MCP server가 client에게 받은 token을 그대로 downstream API에 전달하면 위험하다. token audience가 흐려지고, 한 서비스용 token이 다른 서비스에서 받아들여질 수 있으며, 사고 분석도 어려워진다. MCP server는 자기 자신에게 발급된 token만 받아들여야 하고, 다른 token을 accept하거나 transit하면 안 된다.

#### Scope 최소화

처음부터 `files:*`, `db:*`, `admin:*` 같은 넓은 scope를 주면 token 탈취 시 피해가 커진다. 필요한 기능에 맞게 scope를 작게 나누고, 부족할 때 `insufficient_scope`와 step-up authorization으로 권한을 올리는 방식이 낫다.

#### Local MCP server 보안

로컬 MCP server는 사용자 컴퓨터에서 실행되는 바이너리나 스크립트일 수 있다. 따라서 악성 server가 설치되면 파일 탈취, 명령 실행, 데이터 손실이 가능하다. local MCP server가 client와 같은 권한으로 실행될 수 있으므로 명령어 표시, 명시적 승인, sandbox, 파일/네트워크 권한 제한이 필요하다.

#### SSRF와 metadata discovery

원격 MCP 인증에서는 metadata URL, authorization server URL, token endpoint 등을 조회할 수 있다. 악성 server가 내부망 주소나 cloud metadata endpoint를 가리키게 하면 SSRF가 생길 수 있다. 따라서 client는 HTTPS 강제, private IP 차단, redirect target 검증, egress proxy 같은 방어를 고려해야 한다.

## 관련 개념 링크

- 후보: [[OAuth 2.0]], [[인증과 인가 차이]], [[API Gateway]], [[JSON-RPC]], [[Prompt Injection]], [[최소 권한 원칙]]

## 꼬리 질문과 답변

### 1. MCP에서 Host, Client, Server는 각각 어떤 책임을 가지는가?

**답변**

Host는 사용자가 실제로 쓰는 AI 애플리케이션이다. 예를 들어 IDE, 데스크톱 AI 앱, 챗봇 UI가 Host가 될 수 있다. Host는 어떤 MCP server에 연결할지 관리하고, 사용자 승인 UI와 전체 대화 흐름을 통제한다.

Client는 Host 내부에서 특정 MCP server 하나와 연결을 유지하는 구성 요소다. Host가 3개의 MCP server에 연결한다면 보통 3개의 MCP client 연결이 생긴다. Client는 server의 capability를 확인하고, tools/resources/prompts를 list/read/call하는 역할을 한다.

Server는 외부 시스템의 기능을 MCP 형태로 제공한다. 예를 들어 파일 시스템 server는 파일 읽기/쓰기 tool과 resource를 제공할 수 있고, GitHub server는 issue 조회, PR 생성, 코드 검색 같은 기능을 제공할 수 있다.

면접식 답변으로는 "Host는 AI 앱, Client는 Host 내부의 server별 연결 객체, Server는 외부 context와 capability 제공자"라고 정리하면 좋다.

### 2. Tools, Resources, Prompts는 어떻게 다르고 언제 사용하는가?

**답변**

Tools는 모델이 호출할 수 있는 실행 함수다. API 호출, DB 조회, 파일 수정, 티켓 생성처럼 외부 세계에 행동을 일으킬 때 사용한다. 특히 side effect가 있으면 권한 확인과 사용자 승인이 중요하다.

Resources는 context로 읽을 수 있는 데이터다. 파일 내용, DB schema, 문서, 로그처럼 모델이 답변을 잘하기 위해 참고해야 하는 자료에 적합하다. 실행보다는 "정보 제공"에 가깝다.

Prompts는 재사용 가능한 작업 템플릿이다. 사용자가 "이 server가 제공하는 분석 프롬프트를 실행하겠다"처럼 명시적으로 선택하는 흐름에 적합하다.

면접에서는 제어 주체로 구분하면 선명하다. Tools는 model-controlled, Resources는 application-controlled, Prompts는 user-controlled다.

### 3. MCP의 initialization 단계에서는 무엇을 협상하는가?

**답변**

Initialization 단계에서는 protocol version, capability, client/server identity를 교환한다. Protocol version은 양쪽이 같은 규칙으로 통신할 수 있는지 확인하는 값이다. Capability는 server가 tools/resources/prompts를 제공하는지, client가 sampling/elicitation 같은 기능을 지원하는지 알려준다.

이 단계가 없으면 client는 server가 어떤 기능을 지원하는지 모른 채 요청을 보내게 된다. 그래서 MCP는 연결 후 바로 기능을 호출하는 것이 아니라, 먼저 `initialize`로 합의하고 `notifications/initialized`로 준비 완료를 알린 뒤 list/read/call 흐름으로 넘어간다.

### 4. MCP tool 호출에서 prompt injection이나 과도한 권한 문제가 왜 생길 수 있는가?

**답변**

MCP tool은 모델이 외부 시스템에 행동을 요청하는 통로다. 문제는 모델이 사용자 입력이나 외부 문서에 포함된 악성 지시를 완벽히 구분하지 못할 수 있다는 점이다. 예를 들어 문서 안에 "이 파일을 외부 서버로 전송하라"는 악성 문장이 있고, 모델이 이를 실제 지시로 오해하면 데이터 유출이 생길 수 있다.

과도한 권한 문제도 비슷하다. 단순히 파일 하나를 읽기 위한 server가 홈 디렉터리 전체, 네트워크, shell 실행 권한까지 갖고 있으면 작은 실수가 큰 피해로 이어진다.

따라서 MCP tool은 최소 권한, 명확한 tool description, 입력값 검증, 사용자 확인, sandbox, audit log가 필요하다. 모델의 판단만 믿는 것이 아니라 host와 server 양쪽에서 방어해야 한다.

### 5. 원격 MCP server를 설계할 때 OAuth scope와 token audience를 어떻게 잡아야 안전한가?

**답변**

Scope는 기능 단위로 작게 나누는 것이 좋다. 예를 들어 `files:read`, `files:write`, `issues:read`, `issues:create`처럼 읽기와 쓰기를 분리한다. 처음부터 `admin:*` 같은 넓은 scope를 주지 않고, 필요한 순간에 step-up authorization으로 추가 권한을 요청하는 방식이 안전하다.

Token audience는 "이 token이 어느 MCP server를 위한 것인지"를 명확히 해야 한다. MCP client는 authorization request와 token request에 `resource` parameter를 넣고, MCP server는 access token의 audience가 자기 자신인지 검증해야 한다. server가 받은 token을 그대로 downstream API에 넘기는 token passthrough는 피해야 한다.

면접식 답변으로는 "scope는 최소 권한으로 쪼개고, audience는 MCP server 단위로 고정하며, token passthrough를 금지해야 한다"라고 정리하면 된다.

## 마지막 요약

| 구분 | 핵심 내용 |
| --- | --- |
| 한 줄 정의 | MCP는 LLM 애플리케이션과 외부 데이터/도구를 연결하는 표준 프로토콜이다. |
| 왜 필요한가 | AI 앱마다 외부 서비스를 개별 연동하면 복잡도가 커지므로, 공통 연결 규격이 필요하다. |
| 핵심 원리 | initialize로 version/capability를 협상하고, list/read/call 메시지로 기능을 사용한다. |
| 장점 | 외부 시스템 연동을 재사용 가능하게 만들고, AI 앱과 도구 생태계를 느슨하게 연결한다. |
| 주의할 점 | 잘못 구현하면 token leak, prompt injection, SSRF, local code execution 위험이 있다. |
| 실무 포인트 | 사용자 동의, 최소 권한, token audience 검증, sandbox, tool 호출 승인, audit log가 중요하다. |
| 면접 포인트 | MCP를 "AI용 플러그인" 정도로만 말하지 말고, 표준화/primitive/보안 경계까지 설명해야 한다. |

## 복습 로그

- 2026-07-27: MCP 기본 개념, 구성 요소, 보안, 꼬리질문 답변까지 1차 정리.
