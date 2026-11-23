## 32.1 이 장에서 만드는 것

이 장을 마치면 사내 문서와 지원 티켓을 조회하는 MCP 서버 `docs-hub` 하나를 직접 갖게 됩니다. 도구 4개와 리소스 1개를 갖추고, 실패를 모델이 스스로 고칠 수 있는 형태로 돌려주며, 로컬에서는 stdio로, 원격에서는 인증이 붙은 Streamable HTTP로 뜹니다. 같은 서버를 Claude Code의 `.mcp.json`에 등록하고 Agent SDK의 `mcpServers` 옵션에 넘겨, 두 클라이언트가 같은 도구 목록을 보는 것까지 확인합니다.

15장에서 MCP를 소비자 입장에서 다뤘다면 이 장은 생산자 입장입니다. 15장은 이미 만들어진 서버를 설정 파일로 붙이는 일이었고, 이 장은 스키마를 정하고 오류를 설계하고 인증과 타임아웃을 정해 배포하는 일입니다. 12장의 "설명이 선택을 결정한다"와 13장의 구조화 에러가 여기서 실제 코드가 됩니다.

완성 후 디렉터리는 다음과 같습니다.

```text
docs-mcp/
├── .mcp.json                  # Claude Code 프로젝트 스코프 등록
├── pyproject.toml
├── data/
│   ├── docs.json              # 문서 5건
│   ├── tickets.json           # 티켓 3건
│   └── reports/               # create_report가 쓰는 곳
├── src/docs_mcp/
│   ├── server.py              # 도구 4개 + 리소스 1개
│   ├── errors.py              # 구조화 오류
│   ├── store.py               # 데이터 접근
│   └── http_entry.py          # 원격 배포 진입점
└── tests/
    ├── test_contract.py       # 계약·오류 복구 6건
    ├── description_check.py   # 설명 판별력 측정
    ├── vague_server.py        # 일부러 모호하게 만든 서버
    └── sdk_options.py         # Agent SDK 옵션 확인
```

의존성은 `mcp>=2.3,<3` 하나와 HTTP 배포용 `uvicorn`뿐입니다. 이 장의 코드는 2026-11-23에 Python 3.12.16, MCP Python SDK 2.3.0, Agent SDK 0.2.187, MCP 명세 2026-07-28 기준으로 실행해 확인했고, 본문의 실행 로그는 모두 그 환경의 출력입니다. 모델과 SDK는 빠르게 바뀌므로 버전을 먼저 확인하고, 코드가 맞지 않으면 문서보다 설치본을 보십시오. 이 장에서 확인한 API는 `mcp` 2.x 기준이며 1.x와 다릅니다.

검증 범위는 분명히 밝혀 둡니다. 서버 코드와 두 전송, 도구 계약과 오류 복구, Agent SDK 옵션 구성은 직접 실행했습니다. Claude Code CLI에 등록해 대화를 돌리는 것과 모델이 도구를 고르는 장면은 이 환경에 Claude Code와 API 키가 없어 실행하지 못했고, 공식 문서로 확인한 내용입니다. 오선택은 32.7에서 독자가 재현할 절차로 남깁니다.

## 32.2 왜 서버를 직접 만드는가

공개된 MCP 서버만으로 되는 일이 적지 않습니다. 파일 시스템, GitHub, 데이터베이스, 이슈 트래커에는 이미 서버가 있습니다. 그래서 "직접 만들어야 하는가"는 취향이 아니라 데이터 위치와 업무 규칙의 문제입니다. 다음 셋 중 하나라도 해당하면 직접 만드는 편이 낫습니다.

첫째, 데이터가 사내에만 있습니다. 팀 위키의 운영 문서, 사내 티켓 시스템, 권한이 걸린 저장소는 외부 SaaS용 서버로 닿지 않습니다. 둘째, 도구의 경계가 우리 업무 규칙입니다. 이 장의 문서는 반드시 "검색 결과의 `doc_id`로 본문을 읽는" 2단계로 접근하고, 보고서는 정해진 인용만으로 만듭니다. 이런 규칙은 데이터베이스 위에 SQL 도구 하나를 얹는 방식으로는 지켜지지 않습니다. 셋째, 모델이 실수했을 때 되돌릴 정보를 우리가 압니다. 어떤 오류는 다시 시도하면 되고, 어떤 오류는 인자를 고쳐야 하며, 어떤 오류는 사람을 불러야 합니다. 이 구분은 데이터를 가진 쪽만 정확히 합니다.

반대로 필요한 기능이 이미 공개 서버에 있고, 사내 규칙이 얇고, 읽기만 하면 되는 경우에는 만들지 않는 편이 좋습니다. 서버 하나의 비용은 코드보다 운영입니다. 인증, 타임아웃, 로그, 배포, 스키마 변경 관리가 따라붙습니다. 도구 하나를 만드는 데 반나절이 들고, 그 도구를 안전하게 운영하는 데 일주일이 드는 일이 흔합니다.

직접 만들기로 정했다면 첫 질문은 "도구 몇 개인가"가 아니라 "모델이 이 일을 어디서 멈춰야 하는가"입니다. 이 장의 예에서 `search_docs`는 후보를 보여 주고 멈추며, `get_doc`은 한 건을 읽고 멈춥니다. 보고서 작성은 사용자가 요청할 때만 일어납니다. 도구마다 멈추는 지점이 정해져 있으면 스키마와 오류도 자연스럽게 따라옵니다.

이 장의 데이터는 문서 5건과 티켓 3건짜리입니다. 크기는 중요하지 않습니다. 스키마와 오류를 먼저 고정하고 그 위에 실제 시스템을 갈아 끼우는 순서가 핵심입니다. 5건에서 통과한 계약은 5만 건에서도 같은 모양으로 동작하며, 데이터가 커질 때 바뀌는 것은 `store.py` 안쪽뿐입니다.

## 32.3 설계 — 도구 4개를 표로 먼저 확정한다

12장의 원칙은 "설명이 선택을 결정한다"였습니다. 그 원칙을 지키는 가장 확실한 방법은 코드보다 표를 먼저 채우는 것입니다. 표를 채우다 보면 "이 도구는 언제 쓰면 안 되는가"에 답하지 못하는 도구가 드러납니다. 그런 도구는 모델도 구분하지 못합니다.

### 32.3.1 도구 목록

| 이름 | 확정한 설명(모델이 보는 문장) | 쓰지 말아야 할 때 |
|---|---|---|
| `search_docs` | 사내 문서를 키워드로 찾아 doc_id 목록을 돌려준다. 본문은 돌려주지 않으므로 본문이 필요하면 이 결과의 doc_id로 get_doc을 호출한다. space를 지정하면 그 공간(infra, support, security)의 문서만 검색한다 | 이미 doc_id를 알고 있을 때 |
| `get_doc` | doc_id 하나로 문서 본문 전체를 읽는다. doc_id는 search_docs 결과에서 얻는다. 제목만 알고 doc_id가 없으면 이 도구를 쓰지 말고 search_docs를 먼저 호출한다 | doc_id 없이 제목만 알 때. 이때는 search_docs가 먼저다 |
| `get_ticket` | 지원 티켓 번호로 상태·요약·연관 문서를 조회한다. 번호는 'T-'와 네 자리 숫자다. 문서 검색에는 쓰지 않는다 | 문서를 찾을 때. 티켓 번호는 T- 형식이다 |
| `create_report` | 문서·티켓을 인용한 마크다운 보고서 파일을 새로 만든다. 본문을 인자로 받지 않고 doc_id·ticket_id만 받으므로 인용 내용은 서버가 읽어 채운다. 파일을 쓰는 도구이므로 사용자가 보고서 작성을 요청했을 때만 호출한다 | 사용자가 보고서 작성을 요청하지 않았을 때 |

이름은 동사와 목적어 순으로 짓고, 명세가 허용하는 문자(영문·숫자·`_`·`-`·`.`)만 씁니다. 128자를 넘지 않아야 하고 한 서버 안에서 유일해야 합니다.

### 32.3.2 입출력 스키마

| 도구 | 입력(필수 / 선택) | 출력 |
|---|---|---|
| `search_docs` | `query` / `space`(`infra`,`support`,`security`), `top_k` 1~20 | `query`, `total`, `hits[]`(`doc_id`,`title`,`space`,`updated_at`,`score`), `note` |
| `get_doc` | `doc_id` | `doc_id`, `title`, `space`, `updated_at`, `body` |
| `get_ticket` | `ticket_id` | `ticket_id`, `subject`, `status`, `priority`, `created_at`, `related_docs[]`, `summary` |
| `create_report` | `title`, `doc_ids[]` / `ticket_ids[]` | `path`, `bytes_written`, `doc_ids[]`, `ticket_ids[]` |

출력 스키마를 정하는 이유는 셋입니다. 모델이 결과의 어느 필드를 인용해야 하는지 알게 되고, 클라이언트가 결과를 검증할 수 있고, 필드 이름이 곧 문서가 됩니다. `search_docs`의 `total`은 `top_k`보다 클 수 있다는 점을 설명에 적어 둡니다. 이 한 줄이 없으면 모델은 돌려받은 3건이 전부라고 믿습니다.

입력 쪽에서도 같은 원칙이 적용됩니다. `doc_id`에는 "제목을 넣지 않는다"를, `ticket_id`에는 "`T-`로 시작하는 네 자리"를 적습니다. 스키마가 `string` 하나뿐인 도구는 모델에게 무엇을 넣어야 하는지 알려 주지 못하므로, 설명이 사실상 타입의 일부가 됩니다. 반대로 선택 인자의 기본값을 코드에 두면 스키마에도 `default`가 실려 모델이 생략해도 된다는 것을 압니다.

### 32.3.3 실패의 분류

| `errorCategory` | 뜻 | `isRetryable` | 모델의 다음 행동 |
|---|---|---|---|
| `invalid_argument` | 인자 형식·범위 오류 | false | `hint`를 읽고 인자를 고쳐 다시 호출 |
| `not_found` | 식별자가 없다 | false | `partialResults`나 `search_docs`로 다시 찾기 |
| `unavailable` | 백엔드 일시 장애 | true | 같은 인자로 한 번 더 호출 |
| `permission` | 권한 밖 | false | 사람에게 넘기기 |

`isRetryable`을 오류마다 따로 두는 이유는 재시도가 만능이 아니기 때문입니다. 없는 `doc_id`를 세 번 다시 물어도 결과는 같습니다. 반대로 파일 잠금이나 일시적인 연결 실패는 한 번 더 부르면 풀립니다. 이 둘을 한 오류로 뭉뚱그리면 모델은 둘 중 하나를 잘못 처리합니다.

### 32.3.4 도구가 아니라 리소스로 두는 것

문서 카탈로그는 도구가 아니라 리소스 `docs://catalog`로 노출합니다. 리소스는 부작용이 없고 사용자가 고르는 데이터이며, 클라이언트가 대화 전에 컨텍스트로 넣을 수 있습니다. 카탈로그를 도구로 만들면 모델이 "무엇이 있는지" 알기 위해 매번 호출하고 그 출력이 대화에 쌓입니다. 리소스로 두면 그 호출이 사라집니다.

## 32.4 구현 — Python과 MCP SDK 2.x

### 32.4.1 전송을 먼저 고른다

2026-07-28 명세는 stdio와 Streamable HTTP 두 표준 전송을 정의하고, 두 전송의 메시지 의미가 같다고 못박습니다. 전송은 배달 방식일 뿐입니다. stdio는 클라이언트가 서버를 자식 프로세스로 띄워 표준 입출력으로 줄 단위 JSON-RPC를 주고받고, Streamable HTTP는 하나의 엔드포인트에 POST하고 응답을 JSON 또는 요청 단위 SSE로 받습니다.

이 장은 stdio를 기본으로 잡고 배포 단계에서 Streamable HTTP를 더합니다. 개발 루프가 짧고(프로세스를 띄우고 끄는 것으로 끝납니다), 자격 증명이 프로세스 환경변수로 들어오며(명세도 HTTP가 아닌 전송은 환경변수에서 자격 증명을 얻으라고 규정합니다), Claude Code가 로컬 도구용으로 stdio를 안내하기 때문입니다. SSE 전송은 Claude Code 문서에서 폐기로 표시되어 있어 원격이 필요해지면 SSE가 아니라 Streamable HTTP로 갑니다.

주의할 점이 하나 있습니다. 2026-07-28 명세는 상태를 연결에 두지 않습니다. 모든 요청이 `_meta`에 프로토콜 버전과 클라이언트 능력을 실어 보내고, 서버는 이전 요청에서 문맥을 추론하지 않습니다. 이 장의 서버도 "연결됐으니 로그인 상태" 같은 것을 기억하지 않습니다. 세션 비슷한 상태가 필요하면 핸들을 결과로 돌려주고 다음 호출의 인자로 받아야 합니다.

이 규칙은 편의보다 안정성을 위한 것입니다. 연결 하나에 상태를 매달면 그 연결이 끊기는 순간 상태가 사라지고, 로드밸런서 뒤에서 여러 대가 뜨면 요청마다 다른 서버가 받습니다. 요청 하나에 필요한 정보가 모두 들어 있으면 서버를 몇 대 띄우든 같은 답이 나옵니다. 이 장의 `create_report`가 파일 경로를 결과로 돌려주는 것도 같은 이유입니다. 경로를 서버가 기억하지 않아도, 클라이언트가 그 값을 다음 호출에 넘길 수 있습니다.

### 32.4.2 SDK 2.x에서 이름이 바뀐 것

인터넷의 MCP 예제 대부분은 1.x 기준입니다. 설치본 2.3.0에서 `mcp.server.fastmcp`를 import하면 다음 메시지와 함께 실패합니다.

```text
ModuleNotFoundError: No module named 'mcp.server.fastmcp'. This is mcp 2.x, where FastMCP was
renamed to MCPServer (from mcp.server.mcpserver import MCPServer) ...
```

이 장의 코드는 `MCPServer`를 씁니다. 1.x 예제를 복사했다면 import 두 줄을 바꾸는 것으로 대부분 해결됩니다. 클라이언트를 직접 쓰는 쪽도 조심하십시오. 2.x에서는 결과 객체의 필드가 snake_case입니다. 명세의 `isError`는 `result.is_error`, `structuredContent`는 `result.structured_content`입니다.

### 32.4.3 데이터

`data/docs.json`은 다음 형식이고 5건이 같은 모양입니다. `data/tickets.json`은 `ticket_id`·`subject`·`status`·`priority`·`created_at`·`related_docs`·`summary`를 가집니다.

```json
{
  "SUPPORT-002": {
    "doc_id": "SUPPORT-002", "title": "고객 환불 처리 규정", "space": "support",
    "updated_at": "2026-11-03",
    "body": "결제 후 7일 이내 요청은 전액 환불한다.\n7일이 지난 요청은 사용 이력이 없을 때만 전액 환불한다."
  }
}
```

### 32.4.4 구조화 오류

도구가 던질 오류 하나와, 백엔드 장애를 그 오류로 바꾸는 도구를 만듭니다.

```python
"""도구 실패를 모델이 읽고 스스로 고칠 수 있는 형태로 만든다."""

import json
from contextlib import contextmanager
from typing import Literal

from mcp.server.mcpserver.exceptions import ToolError

from .store import BackendUnavailable

ErrorCategory = Literal["invalid_argument", "not_found", "unavailable", "permission"]


class DocsError(ToolError):
    """분류·재시도 가능 여부·부분 결과를 JSON 한 덩어리로 실어 보낸다."""

    def __init__(self, category: ErrorCategory, message: str, *, hint: str,
                 retryable: bool = False, partial: list[dict] | None = None) -> None:
        self.payload = {
            "errorCategory": category,
            "isRetryable": retryable,
            "message": message,
            "hint": hint,
            "partialResults": partial or [],
        }
        super().__init__(json.dumps(self.payload, ensure_ascii=False))


def fail(category: ErrorCategory, message: str, **kwargs) -> DocsError:
    """도구 안에서 한 줄로 구조화 오류를 던진다."""
    return DocsError(category, message, **kwargs)


@contextmanager
def backend_errors():
    """백엔드 장애를 '다시 보내면 되는 오류'로 바꾼다."""
    try:
        yield
    except BackendUnavailable as exc:
        raise DocsError("unavailable", str(exc), retryable=True,
                        hint="같은 인자로 한 번 더 호출한다. 두 번 연속 실패하면 사람에게 알린다.") from exc
```

`ToolError`를 상속한 것이 핵심입니다. SDK는 이 예외를 받으면 도구 결과를 `isError: true`로 만들고 메시지를 모델에게 전달합니다. 13장에서 정한 형태가 프로토콜 수준으로 올라간 셈입니다. 명세도 도구 오류를 두 갈래로 나눕니다. 요청 구조 자체가 틀린 경우(없는 도구 이름, 스키마 위반)는 JSON-RPC 오류로, 모델이 스스로 고칠 수 있는 실패는 `isError: true` 결과로 돌려줍니다. 후자에 우리의 분류를 실으면 됩니다.

`fail()`을 따로 둔 이유는 호출부를 짧게 만들기 위해서만이 아닙니다. 오류를 만드는 지점을 한 곳으로 모으면 `errorCategory`와 `isRetryable`의 조합이 흔들리지 않습니다. 같은 성격의 실패가 어떤 도구에서는 재시도 가능으로, 다른 도구에서는 불가능으로 나가는 것이 가장 흔한 실수입니다.

### 32.4.5 데이터 접근

`store.py`는 파일 두 개를 읽고, 읽기 실패를 `BackendUnavailable`로 바꿉니다. 위의 `backend_errors()`가 그것을 재시도 가능한 오류로 변환합니다.

```python
"""샘플 백엔드. 실제 사내 시스템이면 DB·검색엔진·티켓 API 호출로 바뀐다."""

import json
from pathlib import Path

DATA_DIR = Path(__file__).resolve().parents[2] / "data"


class BackendUnavailable(Exception):
    """일시적으로 데이터를 읽지 못했다. 같은 요청을 다시 보내면 성공할 수 있다."""


def _load(name: str) -> dict[str, dict]:
    try:
        return json.loads((DATA_DIR / name).read_text(encoding="utf-8"))
    except OSError as exc:
        raise BackendUnavailable(f"{name} 읽기 실패: {exc.strerror}") from exc


def load_docs() -> dict[str, dict]:
    return _load("docs.json")


def load_tickets() -> dict[str, dict]:
    return _load("tickets.json")


def search_docs(query: str, space: str | None, top_k: int) -> tuple[int, list[dict]]:
    terms = [t for t in query.lower().split() if t]
    scored = []
    for doc in load_docs().values():
        if space and doc["space"] != space:
            continue
        text = f"{doc['title']} {doc['body']}".lower()
        if score := sum(text.count(t) for t in terms):
            scored.append((score, doc))
    scored.sort(key=lambda pair: (-pair[0], pair[1]["doc_id"]))
    keys = ("doc_id", "title", "space", "updated_at")
    hits = [{k: d[k] for k in keys} | {"score": s} for s, d in scored[:top_k]]
    return len(scored), hits


def write_report(name: str, body: str) -> str:
    out = DATA_DIR / "reports"
    out.mkdir(parents=True, exist_ok=True)
    safe = "".join(ch for ch in name if ch not in '/\\:*?"<>|').strip() or "report"
    try:
        (out / f"{safe}.md").write_text(body, encoding="utf-8")
    except OSError as exc:
        raise BackendUnavailable(f"보고서 쓰기 실패: {exc.strerror}") from exc
    return f"reports/{safe}.md"
```

### 32.4.6 서버 본체

출력 모델은 pydantic으로 정의합니다. SDK가 이 모델에서 `outputSchema`를 만들고 반환값을 `structuredContent`에 담아 줍니다. 파라미터 설명은 `Annotated`와 `Field`로 달고, 이 설명이 `inputSchema`의 `description`으로 들어갑니다.

```python
"""docs-mcp: 사내 문서·지원 티켓을 Claude에 연결하는 MCP 서버."""

import json
from typing import Annotated

from pydantic import BaseModel, Field

from mcp.server.mcpserver import MCPServer

from . import store
from .errors import backend_errors, fail

server = MCPServer(name="docs-hub", version="0.3.0",
                   instructions="본문이 필요하면 search_docs로 doc_id를 찾아 get_doc으로 읽는다.")


class DocHit(BaseModel):
    doc_id: str = Field(description="get_doc의 입력으로 그대로 쓰는 식별자")
    title: str
    space: str = Field(description="infra, support, security 중 하나")
    updated_at: str
    score: int = Field(description="검색어 일치 횟수")


class SearchResult(BaseModel):
    query: str
    total: int = Field(description="조건에 맞는 전체 문서 수. top_k보다 클 수 있다")
    hits: list[DocHit]
    note: str | None = Field(default=None, description="0건일 때 다음에 시도할 조건")


class DocContent(BaseModel):
    doc_id: str
    title: str
    space: str
    updated_at: str
    body: str = Field(description="줄바꿈을 보존한 원문")


class Ticket(BaseModel):
    ticket_id: str
    subject: str
    status: str = Field(description="open 또는 closed")
    priority: str = Field(description="normal, high, critical")
    created_at: str
    related_docs: list[str]
    summary: str


class ReportResult(BaseModel):
    path: str = Field(description="저장된 보고서의 상대 경로")
    bytes_written: int
    doc_ids: list[str]
    ticket_ids: list[str]


@server.tool(
    name="search_docs",
    description=(
        "사내 문서를 키워드로 찾아 doc_id 목록을 돌려준다. 본문은 돌려주지 않으므로 "
        "본문이 필요하면 이 결과의 doc_id로 get_doc을 호출한다. space를 지정하면 그 "
        "공간(infra, support, security)의 문서만 검색한다."
    ),
)
def search_docs(
    query: Annotated[str, Field(description="찾을 키워드. 공백으로 나눈 단어를 모두 포함하는 문서를 찾는다")],
    space: Annotated[str | None, Field(description="공간 한정: infra, support, security. 생략하면 전체")] = None,
    top_k: Annotated[int, Field(description="돌려줄 최대 건수. 1~20, 기본 3")] = 3,
) -> SearchResult:
    """팀 위키·운영 문서에서 키워드가 맞는 문서를 찾는다."""
    if not query.strip():
        raise fail("invalid_argument", "query가 비어 있다.",
                   hint="주제를 두 단어 이상으로 적어 다시 호출한다. 예: '환불 기간'")
    if not 1 <= top_k <= 20:
        raise fail("invalid_argument", f"top_k={top_k}는 허용 범위 밖이다.",
                   hint="top_k는 1~20 사이 정수로 다시 호출한다.")
    with backend_errors():
        total, hits = store.search_docs(query, space, top_k)
    return SearchResult(
        query=query, total=total, hits=[DocHit(**hit) for hit in hits],
        note=None if hits else "일치 0건. space 지정을 빼거나 더 짧은 키워드로 다시 검색한다.",
    )


@server.tool(
    name="get_doc",
    description=(
        "doc_id 하나로 문서 본문 전체를 읽는다. doc_id는 search_docs 결과에서 얻는다. "
        "제목만 알고 doc_id가 없으면 이 도구를 쓰지 말고 search_docs를 먼저 호출한다."
    ),
)
def get_doc(
    doc_id: Annotated[str, Field(description="search_docs가 돌려준 doc_id. 제목을 넣지 않는다")],
) -> DocContent:
    """문서 한 건의 본문을 읽는다."""
    with backend_errors():
        docs = store.load_docs()
    if (doc := docs.get(doc_id)) is None:
        scored = sorted(
            ((len(set(doc_id) & set(d["title"] + d["doc_id"])), d["doc_id"], d["title"])
             for d in docs.values()),
            reverse=True,
        )
        raise fail(
            "not_found", f"doc_id={doc_id}인 문서가 없다.",
            hint="partialResults의 doc_id로 다시 호출하거나 search_docs로 검색한다.",
            partial=[{"doc_id": i, "title": t} for score, i, t in scored if score][:3],
        )
    return DocContent(**doc)


@server.tool(
    name="get_ticket",
    description=(
        "지원 티켓 번호로 상태·요약·연관 문서를 조회한다. 번호는 'T-'와 네 자리 숫자다. "
        "문서 검색에는 쓰지 않는다."
    ),
)
def get_ticket(
    ticket_id: Annotated[str, Field(description="'T-'로 시작하는 티켓 번호. 예: T-1041")],
) -> Ticket:
    """지원 티켓 한 건을 조회한다."""
    with backend_errors():
        ticket = store.load_tickets().get(ticket_id)
    if ticket is None:
        raise fail("not_found", f"ticket_id={ticket_id}인 티켓이 없다.",
                   hint="번호는 'T-' 뒤에 네 자리 숫자다. 예: T-1041")
    return Ticket(**ticket)


@server.tool(
    name="create_report",
    description=(
        "문서·티켓을 인용한 마크다운 보고서 파일을 새로 만든다. 본문을 인자로 받지 않고 "
        "doc_id·ticket_id만 받으므로 인용 내용은 서버가 읽어 채운다. 파일을 쓰는 도구이므로 "
        "사용자가 보고서 작성을 요청했을 때만 호출한다."
    ),
)
def create_report(
    title: Annotated[str, Field(description="보고서 제목. 파일명으로도 쓰인다")],
    doc_ids: Annotated[list[str], Field(description="인용할 문서의 doc_id 목록. 최소 1건")],
    ticket_ids: Annotated[list[str] | None, Field(description="인용할 티켓 번호 목록. 선택")] = None,
) -> ReportResult:
    """문서·티켓 인용을 모아 보고서 파일을 만든다."""
    ticket_ids = ticket_ids or []
    if not doc_ids:
        raise fail("invalid_argument", "doc_ids가 비어 있다.",
                   hint="search_docs로 찾은 doc_id를 하나 이상 넣어 다시 호출한다.")
    with backend_errors():
        docs, tickets = store.load_docs(), store.load_tickets()

    unknown = [d for d in doc_ids if d not in docs] + [t for t in ticket_ids if t not in tickets]
    if unknown:
        raise fail(
            "not_found", f"찾을 수 없다: {', '.join(unknown)}",
            hint="존재하는 항목만 남기고 다시 호출한다. 나머지로도 보고서는 만들 수 있다.",
            partial=[{"doc_id": d, "title": docs[d]["title"]} for d in doc_ids if d in docs],
        )

    lines = [f"# {title}", ""]
    for doc_id in doc_ids:
        doc = docs[doc_id]
        lines += [f"## {doc['title']} ({doc_id})", "",
                  f"출처: {doc['space']} · 갱신 {doc['updated_at']}", "", doc["body"], ""]
    for ticket_id in ticket_ids:
        lines += [f"## 티켓 {ticket_id}: {tickets[ticket_id]['subject']}", "",
                  tickets[ticket_id]["summary"], ""]
    body = "\n".join(lines)

    with backend_errors():
        path = store.write_report(title.replace(" ", "-")[:40], body)
    return ReportResult(path=path, bytes_written=len(body.encode()), doc_ids=doc_ids,
                        ticket_ids=ticket_ids)


@server.resource(
    "docs://catalog",
    name="문서 카탈로그",
    description="문서 5건의 doc_id·제목·공간·갱신일. 검색 없이 전체 지도를 볼 때 읽는다.",
    mime_type="application/json",
)
def catalog() -> str:
    with backend_errors():
        docs = store.load_docs()
    keys = ("doc_id", "title", "space", "updated_at")
    return json.dumps([{k: d[k] for k in keys} for d in docs.values()],
                      ensure_ascii=False, indent=2)


def main() -> None:
    server.run()


if __name__ == "__main__":
    main()
```

### 32.4.7 실행

`src`를 경로에 넣고 모듈로 띄웁니다. stdio 서버는 화면에 아무것도 출력하지 않는 것이 정상입니다. 표준 출력은 MCP 메시지 전용이고 로그는 표준 오류로 보냅니다.

```bash
PYTHONPATH=src python3 -m docs_mcp.server
```

붙었는지는 클라이언트 코드로 확인합니다. 다음은 `tests/test_contract.py`의 실행 결과입니다.

```text
1) 도구 4개·스키마 확인: OK
2) 검색→본문: total=2 top=SUPPORT-002 OK
3) not_found → 부분 결과 SUPPORT-002로 재시도 성공: OK
4a) 백엔드 장애: unavailable retryable=True OK
4b) 같은 호출 재시도 성공: OK
5) 보고서 생성 reports/환불-기준-정리.md (906 bytes): OK
6) 없는 티켓 포함 시 부분 결과와 함께 거부: OK
통과 6건
```

이 테스트는 배포 전 점검이 아니라 개발 중에 계속 돌리는 검사입니다. 도구 설명을 고치는 날에는 32.7의 설명 비교를 함께 돌리고, 백엔드를 바꾸는 날에는 오류 복구 항목을 먼저 봅니다. 실패가 나면 어느 도구의 계약이 깨졌는지 바로 드러납니다.

## 32.5 Claude Code에 등록하기

Claude Code는 서버 설정을 세 스코프로 나눠 보관합니다. 이 차이를 모르면 "내 컴퓨터에서는 되는데 팀원 것은 안 되는" 상황이 생깁니다.

| 스코프 | 저장 위치 | 범위 | 쓰는 경우 |
|---|---|---|---|
| `local`(기본) | `~/.claude.json`의 해당 프로젝트 항목 | 나만, 이 프로젝트에서만 | 자격 증명이 들어간 실험 |
| `project` | 프로젝트 루트의 `.mcp.json` | 팀 전체, 저장소와 함께 배포 | 팀이 같은 도구를 써야 할 때 |
| `user` | `~/.claude.json` | 나만, 모든 프로젝트 | 여러 프로젝트에서 쓰는 개인 도구 |

같은 이름의 서버가 여러 스코프에 있으면 Claude Code는 하나만 연결하고, 우선순위는 local, project, user, 플러그인, claude.ai 커넥터 순입니다. 스코프끼리 필드를 합치지 않고 한쪽 정의를 통째로 씁니다.

이 장의 서버는 프로젝트 스코프로 등록합니다. 루트에 `.mcp.json`을 두고 저장소에 커밋하면 팀원이 같은 도구를 갖습니다.

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

읽을 때 주의할 점이 다섯 가지 있습니다.

1. `type`을 생략하면 stdio 서버로 읽습니다. 반대로 `url`이 있는데 `type`이 없으면 설정 오류입니다. 그때는 `"type": "http"`를 넣습니다. `streamable-http`도 같은 전송의 별칭으로 받습니다.
2. `command`, `args`, `env`, `url`, `headers` 안에서 `${VAR}`와 `${VAR:-기본값}`을 쓸 수 있습니다. 위 예의 `PYTHONPATH`는 프로젝트 루트 기준 `src`를 가리킵니다. `${CLAUDE_PROJECT_DIR}`는 서버 프로세스 환경에 설정되는 값이라 기본값을 붙여야 합니다.
3. 변수가 없고 기본값도 없으면 경고만 남기고 `${VAR}` 문자열이 그대로 서버에 갑니다. 기동은 되지만 의도한 값이 아닙니다.
4. 빈 문자열로 설정한 변수는 "설정됨"으로 취급됩니다. 기본값을 쓰려면 빈 값이 아니라 변수를 지워야 합니다.
5. 원격 서버의 `url`과 `headers`에서는 Claude Code 자신의 자격 증명 변수가 빈 값으로 읽힙니다. 프로젝트의 `.mcp.json`이 사용자의 클라우드 자격 증명을 서버로 보내지 못하게 막는 장치입니다. 서버에 토큰을 주려면 `DOCS_HUB_TOKEN`처럼 이름을 따로 만드십시오.

명령줄로 등록할 때는 `--` 뒤에 실행 명령을 둡니다. `--`가 없으면 서버의 인자를 Claude Code가 자기 옵션으로 읽습니다.

```bash
claude mcp add --transport stdio docshub --scope project \
  --env PYTHONPATH=./src -- python3 -m docs_mcp.server
claude mcp list
```

`claude mcp add`가 `Added ...`를 출력하면 설정이 기록된 것입니다. 연결 여부는 `claude mcp list`의 상태 표시와 대화 중 `/mcp`에서 확인합니다. `.mcp.json`으로 등록한 서버는 처음 대화를 열 때 승인을 한 번 요구하고, 승인 기록은 `claude mcp reset-project-choices`로 지웁니다.

stdio 서버는 Claude Code가 띄우고 종료도 관리합니다. 세션이 끝나면 SIGINT를 보내고 100밀리초 뒤에도 살아 있으면 SIGTERM, 그 뒤 400밀리초에 SIGKILL로 정리합니다. 종료 시 파일을 닫거나 잠금을 푸는 일이 있다면 이 시간 안에 끝내야 합니다.

스코프를 고르는 기준은 "이 설정이 누구의 것인가"입니다. 팀이 같은 도구를 써야 하고 자격 증명이 개인별로 다르지 않다면 `project`가 맞습니다. 반대로 개인 토큰이 들어가거나 실험 중인 서버라면 `local`에 두고 저장소에는 넣지 않습니다. `user`는 여러 저장소에서 같은 서버를 쓰고 싶을 때만 씁니다. 같은 이름을 여러 스코프에 두면 Claude Code가 경고하고 하나만 쓰므로, 팀 저장소에 `project`로 넣은 서버를 개인 `local`에도 넣어 두는 습관은 피하십시오.

## 32.6 Agent SDK에서 붙이기

같은 서버를 Agent SDK에서도 씁니다. 서버를 코드로 넘기거나 `.mcp.json`을 그대로 읽게 할 수 있습니다.

```python
from claude_agent_sdk import ClaudeAgentOptions

options = ClaudeAgentOptions(
    mcp_servers={
        "docs-hub": {
            "type": "stdio",
            "command": "python3",
            "args": ["-m", "docs_mcp.server"],
            "env": {"PYTHONPATH": "src"},
        }
    },
    allowed_tools=["mcp__docs-hub__search_docs", "mcp__docs-hub__get_doc"],
    strict_mcp_config=True,
)
```

`allowed_tools`의 이름 규칙은 `mcp__<서버 이름>__<도구 이름>`입니다. 서버 단위로 열려면 `mcp__docs-hub__*`를 씁니다. 위 예는 읽기 도구 둘만 열고 쓰기 도구인 `create_report`는 뺐습니다. 파일을 만드는 도구는 사용자가 요청했을 때만 쓰이도록 스코프를 좁히는 편이 안전합니다.

`strict_mcp_config=True`는 코드로 넘긴 서버만 쓰고 `.mcp.json`, 사용자 설정, 플러그인이 제공하는 서버는 무시하라는 뜻입니다. 배포용 코드에서는 이 조합으로 "어떤 도구가 열려 있는지"를 한 곳에서 정합니다. 반대로 Claude Code와 같은 `.mcp.json`을 그대로 쓰고 싶으면 `mcp_servers`를 비우고 `setting_sources=["project"]`를 넣습니다. 이때는 파일이 진실이고, 코드는 도구 목록만 좁힙니다. 두 방식을 섞으면 strict 쪽이 이겨 `.mcp.json`이 무시됩니다.

옵션 구성은 실제로 확인했습니다. 필드 이름과 값이 받아들여지는지 출력으로 봅니다.

```text
allowed_tools: ['mcp__docs-hub__search_docs', 'mcp__docs-hub__get_doc']
mcp_servers keys: ['docs-hub']
strict_mcp_config: True
```

도구 이름 전체 길이는 128자를 넘지 않아야 합니다. 서버 이름을 길게 지으면 도구 이름과 합쳐져 제한을 넘고, Claude Code는 그 도구를 목록에서 뺍니다. 서버 이름은 짧게 짓는 편이 좋습니다.

권한을 좁히는 것은 보안만의 문제가 아닙니다. 모델이 볼 수 있는 도구가 줄면 선택지도 줄어 오선택이 함께 줄어듭니다. 읽기만 하는 에이전트에 쓰기 도구를 열어 두면, 사용자가 "환불 규정 정리해 줘"라고 했을 때 파일이 생기는 일이 생깁니다. 기본값을 최소 권한으로 두고 필요한 곳에서만 넓히십시오.

## 32.7 일부러 망가뜨려 보기

### 32.7.1 설명을 모호하게 만들면 무엇이 무너지는가

12장의 주장을 이 서버로 시험합니다. 도구는 그대로 두고 설명만 "문서를 검색한다.", "문서를 가져온다.", "티켓을 가져온다.", "보고서를 만든다."로 바꾼 서버를 하나 더 띄웁니다.

```python
"""같은 도구를 모호한 설명으로 등록한 비교용 서버."""

import sys
from pathlib import Path

sys.path.insert(0, str(Path(__file__).resolve().parents[1] / "src"))

from mcp.server.mcpserver import MCPServer  # noqa: E402

from docs_mcp.server import create_report, get_doc, get_ticket, search_docs  # noqa: E402

server = MCPServer(name="docs-hub", version="0.3.0-vague")
for fn, desc in [(search_docs, "문서를 검색한다."), (get_doc, "문서를 가져온다."),
                 (get_ticket, "티켓을 가져온다."), (create_report, "보고서를 만든다.")]:
    server.add_tool(fn, description=desc)

if __name__ == "__main__":
    server.run()
```

두 서버가 실제로 내보내는 `tools/list`를 받아 비교했습니다. 경계 문장은 "~지 않는다", "~말고", "~때만"처럼 쓰면 안 되는 경우를 밝히는 문장을 셉니다. 유사도는 설명의 한글 두 글자 묶음을 비교한 자카드 값입니다.

| 항목 | 좋은 설명 | 모호한 설명 |
|---|---|---|
| 경계 문장이 있는 도구 | 4개 중 4개 | 4개 중 0개 |
| 설명 길이 | 64~137자 | 9자 |
| 가장 비슷한 두 도구의 유사도 | 0.10 (`get_doc`~`search_docs`) | 0.40 (`get_doc`~`get_ticket`) |

모델이 보는 것은 이 네 줄이 전부입니다. 좋은 설명에서 `get_doc`은 "doc_id는 search_docs 결과에서 얻는다", "doc_id가 없으면 이 도구를 쓰지 말고 search_docs를 먼저 호출한다"를 말합니다. 모호한 설명에는 그 판단 근거가 없고, `search_docs`와 `get_doc`의 설명이 27% 겹칩니다. 이 상태에서 "환불 규정 좀 찾아줘"라는 요청이 오면 `get_doc`을 부르고 실패하는 경로가 열립니다.

오선택의 비용은 한 번의 실패로 끝나지 않습니다. 잘못 고른 도구가 실패하면 모델은 같은 도구를 인자만 바꿔 다시 부르고, 그것도 실패하면 세 번째 선택지를 고릅니다. 사용자는 그동안 같은 요청을 세 번 반복한 셈이고, 대화 기록에는 실패한 호출이 쌓입니다. 설명 한 문장을 고치는 일이 호출 세 번을 아끼는 일입니다.

여기서 정직하게 밝힙니다. 어떤 도구를 실제로 고르는지는 모델이 정하는 일이라, 위 표는 선택 결과가 아니라 선택에 쓰이는 입력의 차이입니다. 오선택 자체는 다음 순서로 직접 재현하십시오.

1. `vague_server.py`를 Claude Code에 다른 이름으로 등록합니다. 예: `claude mcp add --transport stdio vague -- python3 tests/vague_server.py`
2. "환불 규정 좀 찾아줘"라고 요청하고 어떤 도구가 불렸는지 확인합니다.
3. 같은 요청을 `docs-hub`로 바꿔 반복하고 두 기록을 비교합니다.

### 32.7.2 오류를 구조화하면 복구가 보인다

13장의 형태를 프로토콜에 실으면 어떻게 보이는지 봅니다. `get_doc`에 `doc_id` 대신 제목을 넣은 경우입니다.

```text
Error executing tool get_doc: {"errorCategory": "not_found", "isRetryable": false,
"message": "doc_id=환불 규정인 문서가 없다.",
"hint": "partialResults의 doc_id로 다시 호출하거나 search_docs로 검색한다.",
"partialResults": [{"doc_id": "SUPPORT-002", "title": "고객 환불 처리 규정"}, ...]}
```

모델은 여기서 세 가지를 얻습니다. 다시 시도해도 소용없다는 것(`isRetryable: false`), 인자를 고쳐야 한다는 것(`hint`), 그리고 고칠 후보(`partialResults`)입니다. `SUPPORT-002`로 다시 호출하면 성공합니다. 이 왕복이 `tests/test_contract.py`의 3번 항목입니다.

일시 장애는 다르게 나옵니다. 데이터 파일을 잠시 옮겨 장애를 만들면 다음 응답이 옵니다.

```text
{"errorCategory": "unavailable", "isRetryable": true,
"message": "docs.json 읽기 실패: No such file or directory",
"hint": "같은 인자로 한 번 더 호출한다. 두 번 연속 실패하면 사람에게 알린다.",
"partialResults": []}
```

파일을 되돌린 뒤 같은 인자로 다시 호출하면 성공합니다. 두 경우의 차이는 `isRetryable` 한 필드에 담겨 있고, 모델은 그 필드로 "같은 인자로 다시"와 "다른 인자로 다시"를 구분합니다.

읽는 쪽에서 주의할 점이 둘 있습니다. 첫째, SDK가 오류 텍스트 앞에 `Error executing tool <이름>: `를 붙입니다. JSON만 파싱하려면 첫 `{`부터 잘라 읽으십시오. 둘째, 이 접두사 뒤의 내용은 우리가 만든 `message`이며, 예상하지 못한 예외는 여기에 오지 않습니다.

### 32.7.3 예상 못 한 예외는 메시지가 사라진다

같은 서버에서 정렬 키를 잘못 만들어(비교할 수 없는 값을 키로 넣어) `TypeError`가 나게 했을 때, 클라이언트가 받은 결과는 이것이었습니다.

```text
Error executing tool get_doc
```

`ToolError`가 아닌 예외는 SDK가 충돌로 처리하고, 서버 로그에만 트레이스백을 남기고 모델에게는 도구 이름만 보냅니다. 모델 입장에서는 원인을 알 수 없는 실패이고 재시도해도 결과가 같습니다. 그래서 도구 안에서 예상 가능한 실패는 반드시 `ToolError` 계열로 바꿔야 합니다. `backend_errors()`로 감싼 이유가 그것입니다.

## 32.8 배포와 운영

### 32.8.1 원격 진입점

원격으로 열 때는 Streamable HTTP로 띄우고 그 앞에 인증을 둡니다. `pip install 'mcp>=2.3,<3' uvicorn`을 먼저 실행하십시오. 다음 파일은 정적 베어러 토큰을 검사하는 ASGI 미들웨어를 서버 앱에 씌웁니다.

```python
"""원격 배포용 진입점: Streamable HTTP + 정적 베어러 토큰."""

import os

import uvicorn
from starlette.responses import PlainTextResponse

from .server import server


class BearerGuard:
    """모든 HTTP 요청에 Authorization: Bearer <token>을 요구한다."""

    def __init__(self, app, token: str) -> None:
        self.app, self.token = app, token

    async def __call__(self, scope, receive, send) -> None:
        if scope["type"] == "http":
            given = dict(scope["headers"]).get(b"authorization", b"").decode()
            if given != f"Bearer {self.token}":
                await PlainTextResponse("unauthorized", status_code=401)(scope, receive, send)
                return
        await self.app(scope, receive, send)


def build_app():
    token = os.environ["DOCS_HUB_TOKEN"]  # 없으면 기동 실패가 맞다
    return BearerGuard(server.streamable_http_app(), token)


if __name__ == "__main__":
    uvicorn.run(build_app(), host="0.0.0.0", port=int(os.environ.get("PORT", "8765")))
```

미들웨어를 순수 ASGI 함수로 쓴 이유가 있습니다. 응답 본문을 모아서 처리하는 방식의 미들웨어는 스트리밍 응답을 막을 수 있습니다. 위 코드는 HTTP 요청의 헤더만 보고 그대로 넘깁니다. 실행과 확인 결과입니다.

```text
no token -> 401 unauthorized
token -> 200 OK, get_ticket is_error=False
```

### 32.8.2 인증을 어디까지 직접 할 것인가

정적 토큰은 사내 단기 수단입니다. 서버가 사내망 안에 있고 앞단에 리버스 프록시나 터널이 있으며 사용자가 소수일 때는 충분합니다. 이때도 토큰은 저장소에 넣지 말고 환경변수로 주입하고 `.mcp.json`에서는 `${DOCS_HUB_TOKEN}`으로 참조합니다.

외부에 공개하거나 사용자가 여럿이면 OAuth 2.0으로 갑니다. 명세는 HTTP 전송에 인증 프레임워크를 따로 규정하고, Claude Code도 원격 서버가 401이나 403을 돌려주면 `/mcp`에서 로그인 흐름을 엽니다. 정적 토큰은 이 흐름을 대신하지 못합니다. OAuth로 넘어갈 시점은 35장에서 비용·보안과 함께 다룹니다.

사내망 서버를 밖으로 내보내야 한다면 MCP 터널을 검토합니다. 터널은 `cloudflared`로 아웃바운드 연결만 열어 인바운드 포트를 열지 않고 사내 서버에 닿게 합니다. 2026-11 기준 research preview이고 Managed Agents와 Messages API 경로에서 씁니다. 터널은 서버 인증을 대신하지 않으므로 서버 쪽 OAuth나 베어러 토큰은 그대로 두어야 하고, 요청이 터널 호스트명으로 들어오므로 서버의 허용 호스트 설정도 함께 맞춰야 합니다.

### 32.8.3 타임아웃

타임아웃은 세 층으로 나뉩니다.

| 층 | 기본값 | 조정 방법 |
|---|---|---|
| 서버 연결 | 30초 | `MCP_TIMEOUT` 환경변수 |
| 도구 호출 | 거의 무제한(`MCP_TOOL_TIMEOUT` 미설정 시 약 28시간) | 서버 항목의 `timeout`(밀리초, 1000 미만은 무시), 또는 `MCP_TOOL_TIMEOUT` |
| 응답 없는 호출 | HTTP 5분, stdio 30분 | `CLAUDE_CODE_MCP_TOOL_IDLE_TIMEOUT` |

여기에 하나 더 있습니다. 대화 중 도구 호출이 2분을 넘으면 Claude Code는 그 호출을 백그라운드 작업으로 옮기고 모델에게는 작업 ID를 줍니다. 보고서 생성처럼 오래 걸리는 도구를 만들 때는 이 동작을 알고 있어야 합니다. 결과는 작업이 끝날 때 알림으로 도착합니다.

이 장의 서버는 문서 5건짜리라 호출이 밀리초 단위로 끝납니다. 실제 백엔드로 바꾸면 서버별 `timeout`을 먼저 정하고, 그보다 오래 걸리는 작업은 도구를 쪼개거나 백그라운드로 넘기는 편이 낫습니다. 타임아웃을 길게 잡는 대신 작업을 나누면 사용자는 진행 상황을 보고 중간에 멈출 수 있습니다.

### 32.8.4 로그와 비용

stdio 서버는 표준 출력에 MCP 메시지 외의 것을 쓰면 안 됩니다. 로그는 표준 오류로 보냅니다. 남길 항목은 요청 ID, 도구 이름, 소요 시간, 오류 범주, 반환 건수입니다. 응답 본문 전체를 남기면 문서 내용이 로그에 쌓이므로 길이와 식별자만 남깁니다.

비용은 도구 출력의 토큰에서 나옵니다. 같은 질문에 문서 3건 전문을 돌려주는 도구와 `doc_id` 3건만 돌려주는 도구는 모델 비용이 몇 배 차이 납니다. `search_docs`가 본문을 돌려주지 않고 `top_k` 상한을 20으로 둔 이유입니다. 카탈로그를 리소스로 뺀 것도 탐색 호출을 줄이는 장치입니다.

배포 전에는 다음 항목을 확인합니다. 코드가 도는 것과 운영이 되는 것은 다른 문제입니다.

| 점검 | 확인 방법 | 실패했을 때 |
|---|---|---|
| 도구 계약 | `tools/list`로 도구 4개와 `outputSchema`를 확인 | 모델이 결과의 필드를 읽지 못함 |
| 오류 분류 | 없는 `doc_id`로 호출해 `errorCategory`를 확인 | 재시도와 중단을 모델이 판단하지 못함 |
| 인증 | 토큰 없이 호출해 401을 확인 | 서버가 공개됨 |
| 타임아웃 | 느린 백엔드로 호출해 중단 지점 확인 | 대화가 멈추거나 백그라운드로 밀림 |
| 로그 | 표준 출력을 비우고 표준 오류만 쓰는지 확인 | 메시지가 오염되어 연결이 끊김 |
| 비용 | 도구 한 번의 출력 크기 상한 확인 | 대화 토큰이 급증함 |

## 32.9 이 장의 체크리스트

- [ ] 도구 4개의 이름·설명·입출력 스키마를 코드보다 먼저 표로 확정했다.
- [ ] 모든 도구 설명에 "언제 쓰지 않는가"를 한 문장 이상 넣었다.
- [ ] 출력 모델을 정의해 `outputSchema`가 생성되는 것을 확인했다.
- [ ] 예상 가능한 실패를 `ToolError` 계열로 바꿨고 오류마다 `isRetryable`을 정했다.
- [ ] 식별자를 못 찾았을 때 후보(`partialResults`)를 돌려준다.
- [ ] `tests/test_contract.py`가 통과한다. 특히 백엔드 장애 뒤 같은 호출이 성공하는 항목.
- [ ] `.mcp.json`에 비밀 값을 직접 쓰지 않고 `${VAR}`로 참조한다.
- [ ] 쓰기 도구는 `allowed_tools`에서 기본으로 빼 두었다.
- [ ] 원격 배포 시 토큰 검사와 허용 호스트를 확인했고 401 응답을 실제로 봤다.
- [ ] 서버별 `timeout`을 정하고 그보다 오래 걸리는 작업의 처리 방법을 적어 두었다.

## 확인한 문서

- MCP 명세 2026-07-28(기본 프로토콜, stdio·Streamable HTTP 전송, 도구): https://modelcontextprotocol.io/specification/2026-07-28
- Claude Code MCP 설정(스코프, `.mcp.json`, 환경변수 확장, 타임아웃): https://code.claude.com/docs/en/mcp
- Agent SDK MCP 연결(`mcpServers`, `allowedTools`): https://code.claude.com/docs/en/agent-sdk/mcp
- MCP Python SDK 2.x 마이그레이션: https://py.sdk.modelcontextprotocol.io/v2/migration/
- MCP 터널(research preview, 아웃바운드 연결): https://platform.claude.com/docs/en/agents-and-tools/mcp-tunnels/overview

위 문서와 코드 실행 결과는 2026-11-23에 확인했습니다. 문의: sendallsdc@gmail.com
