## 부록 C. 실습 코드·설정 모음

이 부록은 본문에서 "부록 C의 파일"로 넘긴 것들의 지도입니다. 코드는 책의 저장소 `work/` 아래에 장별 폴더로 들어 있고, 폴더마다 `verify.py`가 있습니다. 전체를 복사해 두고 `python3 verify.py`를 돌리면 본문의 실행 결과가 그대로 재현됩니다.

이 부록을 읽는 순서는 두 가지입니다. 실습 중이라면 C.1과 C.2만 보고 해당 폴더로 가십시오. 책을 덮은 뒤 다시 열었다면 C.4와 C.5에서 설정 파일을 복사해 쓰십시오.

### C.1 실행 환경

| 항목 | 값 | 비고 |
|---|---|---|
| Python | 3.12.16 | 3.12 이상이면 됨 |
| anthropic | 2.12.0 | 실제 호출을 할 때만 필요 |
| claude-agent-sdk | 0.2.187 | 5장의 SDK 확인용 |
| mcp | 2.3.0 | 15·32장의 서버와 클라이언트 |
| API 키 | 불필요 | 모든 `verify.py`는 대본으로 돈다 |

모든 검증은 2026년 11월 24일에 이 환경에서 실행했습니다. 라이브러리 판이 다르면 `verify.py`가 먼저 멈추고 판을 알려 줍니다. 그때는 본문의 코드를 고치지 말고, 설치본의 문서에서 바뀐 이름을 확인하십시오.

### C.2 폴더 지도

| 장 | 폴더 | 핵심 파일 | 실행 |
|---|---|---|---|
| 4 | `ch04-messages/` | `round_trip.py` | `python3 round_trip.py` |
| 5 | `ch05-agent-loop/` | `loop_v1.py`, `transport.py`, `tools.py` | `python3 verify.py` |
| 6 | `ch06-lifecycle/` | `loop_v2.py` | `python3 verify.py` |
| 7 | `ch07-coordinator/` | `hub.py` | `python3 verify.py` |
| 8 | `ch08-context/` | `packets.py` | `python3 verify.py` |
| 9 | `ch09-enforcement/` | `gates.py` | `python3 verify.py` |
| 10 | `ch10-decomposition/` | `pipeline.py` | `python3 verify.py` |
| 11 | `ch11-session/` | `session.py` | `python3 verify.py` |
| 12 | `ch12-tool-interface/` | `interface.py` | `python3 verify.py` |
| 13 | `ch13-errors/` | `errors.py`, `agent.py` | `python3 verify.py` |
| 14 | `ch14-tool-choice/` | `roster.py` | `python3 verify.py` |
| 15 | `ch15-mcp/` | `mini_server.py`, `client.py`, `config.py` | `python3 verify.py` |
| 16 | `ch16-builtin/` | `filetools.py` | `python3 verify.py` |
| 17 | `ch17-claude-md/` | `memory.py`, `shared/` | `python3 verify.py` |
| 18 | `ch18-commands-skills/` | `skills.py`, `project/` | `python3 verify.py` |
| 19 | `ch19-path-rules/` | `pathrules.py` | `python3 verify.py` |
| 20 | `ch20-plan-mode/` | `planmode.py` | `python3 verify.py` |
| 21 | `ch21-iterative-ci/` | `ci.py` | `python3 verify.py` |
| 22 | `ch22-explicit-criteria/` | `criteria.py`, `documents.py` | `python3 verify.py` |
| 23 | `ch23-few-shot/` | `fewshot.py` | `python3 verify.py` |
| 24 | `ch24-json-schema/` | `schema.py` | `python3 verify.py` |
| 25 | `ch25-validation-batch/` | `pipeline.py`, `schema.py` | `python3 verify.py` |
| 26 | `ch26-multi-pass-review/` | `review.py` | `python3 verify.py` |
| 27 | `ch27-conversation-context/` | `case.py` | `python3 verify.py` |
| 28 | `ch28-escalation/` | `escalate.py` | `python3 verify.py` |
| 29 | `ch29-error-propagation/` | `propagate.py` | `python3 verify.py` |
| 30 | `ch30-large-codebase/` | `repo.py`, `state/` | `python3 verify.py` |
| 31 | `ch31-review-provenance/` | `review.py` | `python3 verify.py` |
| 32 | `docs-mcp/` | `src/docs_mcp/server.py`, `.mcp.json` | 아래 C.4 |
| 33 | `ch33-eval-observability/` | `cases.py`, `judge.py`, `pipeline.py` | `python3 verify.py` |
| 34 | `ch34-internal-tools/` | `policy.json`, `policy_gate.py`, `scaffold.py` | `python3 verify.py` |
| 35 | `ch35-cost-security-plugins/` | `pricing.py`, `latency.py`, `redact.py`, `plugin.py` | `python3 verify.py` |

5~31장의 폴더 중 여러 곳에는 5장의 대본 전송 코드를 복사한 `ch05.py`가 함께 들어 있습니다. 장마다 앞 장을 다시 받지 않고 바로 실행하기 위한 것입니다. 실제로는 앞 폴더의 파일을 그대로 쓰는 편이 낫습니다. 두 벌이 있으면 어느 쪽이 최신인지 헷갈립니다.

### C.3 실행과 확인

1. `work/`로 이동해 장 폴더로 들어갑니다. 확인: `verify.py`와 `verify.log`가 보입니다.

2. `python3 verify.py`를 실행합니다. 확인: 본문에 나온 실행 결과가 순서대로 출력되고, 마지막 줄에 "검증 통과"가 나옵니다.

3. `verify.log`를 엽니다. 확인: 방금 출력과 같은 내용이 파일로 남아 있습니다. 본문의 숫자와 대조할 때 이 파일을 쓰십시오.

4. 일부러 하나를 망가뜨립니다. 예를 들어 33장의 `cases.py`에서 한 케이스의 정답을 지우고 다시 돌립니다. 확인: 게이트 판정이 바뀝니다. 이 확인이 "숫자가 어디서 나오는가"를 몸으로 익히게 합니다.

### C.4 docs-mcp 서버

32장의 서버는 폴더 하나로 들어 있습니다. 구조는 다음과 같습니다.

```text
docs-mcp/
├── .mcp.json               # Claude Code 등록용
├── pyproject.toml          # 패키지 메타데이터
├── data/
│   ├── docs.json           # 문서 12건
│   └── tickets.json        # 티켓 6건
├── src/docs_mcp/
│   ├── server.py           # 도구 4개와 리소스 1개
│   ├── store.py            # 검색·조회
│   ├── errors.py           # 구조화 오류
│   └── http_entry.py       # 인증이 붙은 HTTP 진입점
└── tests/
    ├── test_contract.py    # stdio 계약 시험
    ├── description_check.py
    ├── http_check.py
    ├── sdk_options.py
    └── vague_server.py     # 일부러 모호한 설명
```

실행 명령은 세 가지입니다. 로컬 서버를 띄우려면 `python3 -m docs_mcp.server`, 계약을 시험하려면 `python3 tests/test_contract.py`, 모호한 설명의 오선택을 재현하려면 `python3 tests/vague_server.py`입니다. `PYTHONPATH=src`를 붙여야 모듈을 찾습니다.

`.mcp.json`은 프로젝트 스코프 등록의 예입니다. 경로를 `${CLAUDE_PROJECT_DIR:-.}`로 잡아, 저장소를 어디에 복제해도 동작하게 했습니다. 토큰은 `${DOCS_HUB_TOKEN:-local-dev}`로 확장합니다. 비밀 값을 파일에 적지 않고 환경변수로 넘기는 방식이 이 두 줄에 있습니다.

```json
{
  "mcpServers": {
    "docs-hub": {
      "command": "python3",
      "args": ["-m", "docs_mcp.server"],
      "env": {
        "PYTHONPATH": "${CLAUDE_PROJECT_DIR:-.}/src",
        "DOCS_HUB_TOKEN": "${DOCS_HUB_TOKEN:-local-dev}"
      },
      "timeout": 60000
    }
  }
}
```

### C.5 복사해 쓰는 설정

스킬의 머리말은 네 줄이면 충분합니다. 설명이 호출 조건을 정하고, 허용 도구가 권한을 좁힙니다.

```text
---
name: triage-escalate
description: 고객 문의를 분류해 에스컬레이션 패킷을 만들고 사람에게 넘긴다. 정책 공백, 규정에 없는 예외 요청, 법적 조치 언급, 담당자 판단이 필요한 건에 사용한다.
allowed-tools: Read, Grep, mcp__docs-hub__search_docs, mcp__docs-hub__get_doc, escalate_to_human
---
```

경로 규칙은 앞부분의 `paths`에 범위를 적고, 본문에 그 범위에서만 적용할 관례를 씁니다. 루트 문서에 조건문으로 적으면 모든 작업에서 읽히므로, 조건부 로딩의 이점이 사라집니다.

```text
---
paths:
  - "src/**/*.py"
  - "tests/**/*.py"
---
src에서는 예외를 던지지 않고 결과 객체를 돌려준다.
tests에는 실제 고객의 문장을 넣지 않는다.
```

CI는 비대화형 실행 두 줄과 검사 한 줄이면 시작합니다. 결과를 JSON으로 받아야 다음 단계가 지적을 셀 수 있습니다.

```text
claude -p "$(cat .claude/prompts/review.md)" --output-format json > review.json
python3 .claude/ci/gate.py review.json
```

### C.6 남는 산출물

34장을 실행하면 `ch34-internal-tools/build/triage-tool/`이 만들어집니다. 이 폴더에는 규칙 문서, 스킬, 정책표, 훅, 경로 규칙, 이슈 폼, CI 워크플로, 상태 파일 양식이 들어 있습니다. 35장을 실행하면 `ch35-cost-security-plugins/build/plugin/`에 플러그인 폴더와 마켓플레이스 목록이 생깁니다.

두 폴더는 그대로 사내 저장소에 올릴 수 있습니다. 올리기 전에 고칠 곳은 세 군데입니다. `policy.json`의 소유자, 규칙 문서의 빌드 명령, 그리고 플러그인 매니페스트의 이름과 버전입니다. 나머지는 조직 이름과 무관하게 동작합니다.

### C.7 검증 범위

이 책의 코드는 모두 실행해 확인했지만, 세 가지는 실제 값이 아닙니다. 첫째, 모델의 판정과 토큰 수치는 대본과 가정값입니다. 둘째, 지연과 비용은 3장과 35장의 단가표를 전제로 계산한 값입니다. 셋째, Claude Code의 실행 파일이 없는 환경이라 훅과 CI는 생성된 파일을 직접 실행해 확인했습니다.

이 구분을 알고 읽으면 숫자를 그대로 옮기지 않게 됩니다. 숫자는 구조를 보여 주는 표본이고, 독자의 값은 독자의 로그에서 나옵니다.
