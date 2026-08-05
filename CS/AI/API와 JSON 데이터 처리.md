---
title: API와 JSON 데이터 처리
aliases:
  - HTTP 요청과 응답
  - JSON 파싱
tags:
  - cs/ai
  - type/concept
  - ai/api
  - data/json
---

# API와 JSON 데이터 처리

## 핵심 정리

> [!definition]
> API는 프로그램끼리 기능과 데이터를 주고받는 접점이다. 클라이언트가 HTTP 요청을 보내면 서버는 상태 코드와 응답 본문을 반환한다. JSON 응답은 Python에서 주로 `list`와 `dict`로 변환되며, 필요한 필드만 추출해 표나 LLM 컨텍스트로 사용할 수 있다.

## 왜 필요한가

AI 애플리케이션은 모델 API, 검색 API, 사내 데이터 API 등 외부 시스템과 통신한다. 응답 구조와 실패 가능성을 이해해야 데이터를 안전하게 받아 후속 처리로 연결할 수 있다.

## 동작 흐름

HTTP 요청 → 상태 코드 확인 → JSON 변환 → 중첩 데이터 접근 → 필요한 필드 추출 → DataFrame 또는 LLM 컨텍스트 구성

## 반드시 알아야 할 문법

- `requests.get(url, timeout=5)`: GET 요청
- `response.raise_for_status()`: HTTP 오류 확인
- `response.json()`: JSON을 Python 객체로 변환
- `dict.get(key, default)`: 선택적인 키를 안전하게 조회
- `"\n".join(lines)`: 여러 줄을 컨텍스트 문자열로 구성
- `json.dumps(data, ensure_ascii=False, indent=2)`: Python 객체를 읽기 좋은 JSON 문자열로 변환

## 예외 처리

네트워크 요청은 항상 실패 가능성이 있다. 시간 초과, 연결 실패, HTTP 오류, JSON 변환 실패를 구분해 처리해야 한다. 대체 데이터를 사용할 때는 실제 응답과 같은 구조를 유지해야 후속 코드가 그대로 동작한다.

## LLM 연결

LLM 요청에는 모델, 메시지와 컨텍스트, 생성 옵션 등이 포함될 수 있다. 응답에서는 답변뿐 아니라 토큰 사용량과 종료 이유 같은 메타데이터도 확인한다. API 키는 코드에 직접 적지 않고 환경 변수나 안전한 비밀 관리 방식으로 주입한다.

## 자주 하는 실수

- `timeout` 없이 요청한다.
- 상태 코드를 확인하지 않고 응답을 사용한다.
- 모든 응답이 정상 JSON이라고 가정한다.
- 중첩된 `list`와 `dict`의 접근 순서를 혼동한다.
- 질문과 관계없는 원본 전체를 LLM 컨텍스트에 포함한다.
- API 키를 노트북, 로그, 캡처 화면에 노출한다.

## 관련 개념

- [[CS/AI/Python 핵심 문법]]
- [[CS/AI/Pandas 데이터 처리]]
- [[CS/AI/RAG는 무엇이고 왜 파인튜닝만으로는 부족한가]]

## 관련 강의

- [[00 Python에서 API·NumPy·Pandas로 이어지는 데이터 처리 기초]]
