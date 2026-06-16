---
tags:
  - cs/meta
  - type/index
---

# CS YAML Tag Rules

## 기본 원칙

- 태그는 본문 중간보다 YAML frontmatter에 둔다.
- 한 노트에는 보통 4~8개만 붙인다.
- 시험 시점 태그는 쓰지 않는다.
- 폴더로 이미 구분되는 내용은 태그로 반복하지 않고, 나중에 다시 찾을 검색 축만 남긴다.

## 추천 축

### 큰 분야

- `cs/network`
- `cs/database`
- `cs/security`
- `cs/crypto`
- `cs/os`
- `cs/algorithm`
- `cs/ai`

### 수업/자료 묶음

- `course/iot-network`
- `course/database-security`
- `course/digital-forensics`

### 노트 성격

- `type/moc`
- `type/index`
- `type/lecture-note`
- `type/problem-solving`
- `type/assignment`
- `type/checklist`

### 개념 태그 예시

- `network/network-layer`
- `network/data-plane`
- `network/control-plane`
- `network/link-layer`
- `network/routing`
- `network/ip`
- `network/dhcp`
- `network/wifi`
- `db/access-control`
- `db/encryption`
- `db/tde`
- `db/big-data`
- `security/access-control`
- `security/security-models`
- `security/key-management`
- `crypto/symmetric`
- `crypto/asymmetric`
- `privacy/data-lifecycle`
- `privacy/de-identification`
- `algorithm/dijkstra`

## 템플릿

```yaml
---
tags:
  - cs/network
  - course/iot-network
  - type/lecture-note
  - network/routing
---
```
