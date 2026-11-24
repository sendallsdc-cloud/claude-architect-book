이 부록은 본문에서 쓴 용어를 영문과 한글로 나란히 둔 표입니다. 시험은 영어와 일본어로 출제되므로, 한글 뜻을 알면서도 영문 표기를 모르면 문항의 지시를 놓칩니다. 표의 마지막 칸은 그 용어가 처음 나온 장입니다. 복습할 때는 한글 칸을 가리고 영문만 보고 뜻을 말해 보십시오.

식별자는 원형을 유지했습니다. 코드에 그대로 쓰이는 이름(`stop_reason`, `tool_choice`)은 번역하지 않고, 문서에서 쓰는 개념어만 한글을 붙였습니다.

용어를 외우는 방법은 하나입니다. 영문을 보고 한글 뜻을 말한 뒤, 그 용어가 등장하는 상황을 한 문장으로 만들어 보십시오. 예를 들어 `isRetryable`이라면 "같은 호출을 다시 해도 되는지를 표시하는 칸"이라는 뜻과 함께 "오류 응답에 이 칸이 없으면 모델이 무한히 재시도한다"는 상황이 따라붙어야 합니다. 뜻만 외운 용어는 선택지에서 알아보지 못합니다.

표에 없는 용어를 만나면 제품 문서의 정의를 먼저 보십시오. 이 책이 쓴 한글이 문서의 표준 번역과 다를 수 있습니다. 다만 `tool_choice`, `stop_reason`처럼 코드에 박혀 있는 이름은 어떤 문서에서도 번역하지 않으므로, 그대로 익히는 편이 좋습니다.

### A.1 자격·제도

| 영문 | 한국어 | 한 줄 설명 | 장 |
|---|---|---|---|
| Claude Certification Program | 클로드 자격 프로그램 | Anthropic이 운영하는 자격 제도 | 1 |
| Claude Partner Network | 클로드 파트너 네트워크 | 응시 자격의 전제가 되는 기업 프로그램 | 1 |
| eligibility | 응시 자격 | 기업 소속·회사 도메인 이메일·연령·신분증 조건 | 1 |
| company domain email | 회사 도메인 이메일 | 개인 주소로는 등록·결제 불가 | 1 |
| Pearson VUE | 피어슨 VUE | 시험 전달·감독 기관 | 1 |
| OnVUE | 온라인 감독 시험 | 집이나 사무실에서 보는 감독 시험 | 1 |
| test center | 테스트센터 | 지정 장소에서 보는 시험 | 1 |
| scaled score | 척도 점수 | 원점수 퍼센트가 아닌 환산 점수(합격 720) | 1 |
| criterion-referenced | 준거참조 | 다른 응시자가 아니라 기준과 비교하는 채점 | 1 |
| NDA | 비밀유지 동의 | 거부하면 시험이 종료되고 환불되지 않음 | 38 |
| certification terms | 자격 약관 | 법적 조건 문서 | 38 |
| exam policy | 시험 정책 | 감독·부정행위·제재 규정 | 38 |
| retake | 재응시 | 14일·30일·90일 간격, 12개월 4회 | 38 |
| recertification | 갱신 | 12개월마다 무료 비감독 온라인 평가 | 38 |
| Credly badge | 크레들리 배지 | 취득을 증명하는 디지털 배지 | 1 |

### A.2 표면과 모델

| 영문 | 한국어 | 한 줄 설명 | 장 |
|---|---|---|---|
| surface | 표면 | 제품이 제공되는 접점(앱·API·SDK·CLI) | 2 |
| Messages API | 메시지 API | 요청과 응답의 기본 형식 | 2 |
| Agent SDK | 에이전트 SDK | 루프·도구·세션·훅을 제공하는 라이브러리 | 2 |
| Claude Code | 클로드 코드 | 터미널·IDE에서 도는 에이전틱 코딩 도구 | 2 |
| MCP | 모델 컨텍스트 프로토콜 | 도구·자원을 주고받는 개방 규약 | 2 |
| Managed Agents | 매니지드 에이전트 | Anthropic이 호스팅하는 에이전트 런타임 | 2 |
| model family | 모델 계열 | Fable·Opus·Sonnet·Haiku 네 단계 | 3 |
| snapshot ID | (고정) 스냅샷 아이디 | 날짜 없는 ID 자체가 고정본 | 3 |
| effort | 추론 노력 | 같은 모델의 출력량·깊이를 조절하는 손잡이 | 3 |
| thinking | 사고 | 현재 모델의 기본 동작, 깊이는 effort로 조절 | 3 |
| context window | 컨텍스트 창 | 한 요청에 담을 수 있는 토큰 한도 | 3 |
| context rot | 컨텍스트 부패 | 토큰이 늘수록 정확도가 떨어지는 현상 | 3 |
| prompt caching | 프롬프트 캐시 | 반복되는 접두를 재사용하는 기능 | 35 |
| batch processing | 배치 처리 | 모아서 보내고 할인·지연을 맞바꾸는 방식 | 25 |
| rate limit | 호출 한도 | 단위 시간당 허용되는 요청 수 | 3 |

### A.3 요청과 응답

| 영문 | 한국어 | 한 줄 설명 | 장 |
|---|---|---|---|
| message | 메시지 | 역할(role)과 내용(content)의 묶음 | 4 |
| system prompt | 시스템 프롬프트 | 대화 전체에 적용되는 지시 | 4 |
| user / assistant | 사용자 / 어시스턴트 | 메시지의 두 역할 | 4 |
| content block | 콘텐츠 블록 | 텍스트·도구 호출·결과의 단위 | 4 |
| tool_use | 도구 호출 블록 | 모델이 도구를 부르겠다는 출력 | 4 |
| tool_result | 도구 결과 블록 | 실행 결과를 돌려주는 입력 | 4 |
| tool_use_id | 호출 아이디 | 호출과 결과를 짝짓는 식별자 | 4 |
| stop_reason | 종료 사유 | `tool_use`·`end_turn`·`max_tokens` 등 | 4 |
| max_tokens | 출력 상한 | 사고와 답변을 합친 길이의 상한 | 3 |
| usage | 사용량 | 입력·출력·캐시 토큰 수 | 3 |
| output_config | 출력 설정 | effort 값을 넣는 자리 | 3 |
| streaming | 스트리밍 | 토큰을 실시간으로 받는 방식(범위 밖) | E |
| SSE | 서버 전송 이벤트 | 스트리밍의 전송 형식(범위 밖) | E |
| citations | 인용 | 근거를 붙여 출력하는 기능 | 31 |
| Files API | 파일 API | 파일을 올려 참조하는 기능(범위 밖) | E |

### A.4 에이전트와 오케스트레이션

| 영문 | 한국어 | 한 줄 설명 | 장 |
|---|---|---|---|
| agentic loop | 에이전틱 루프 | 모델 호출과 도구 실행을 반복하는 구조 | 5 |
| termination condition | 종료 조건 | 루프를 끝내는 판정 규칙 | 6 |
| iteration cap | 반복 상한 | 무한 루프를 막는 횟수 제한 | 6 |
| coordinator | 코디네이터 | 일을 나누고 결과를 모으는 중심 에이전트 | 7 |
| subagent | 서브에이전트 | 격리된 컨텍스트에서 맡은 일만 하는 에이전트 | 7 |
| hub-and-spoke | 허브-스포크 | 중심과 바깥으로 나뉜 위임 구조 | 7 |
| delegation | 위임 | 일과 권한을 다른 에이전트에 넘기는 판단 | 7 |
| context passing | 컨텍스트 전달 | 넘길 정보를 골라 구조화하는 일 | 8 |
| allowedTools | 허용 도구 목록 | 서브에이전트가 쓸 수 있는 도구의 범위 | 8 |
| parallel execution | 병렬 실행 | 한 응답에서 여러 작업을 동시에 띄우기 | 8 |
| handoff | 핸드오프 | 사람이나 다른 단계로 넘기는 일 | 9 |
| enforcement | 강제 | 프롬프트가 아니라 코드로 막는 일 | 9 |
| programmatic gate | 프로그램적 게이트 | 실행 전에 코드가 판정하는 관문 | 9 |
| hook | 훅 | 생명주기 지점에서 실행되는 우리 코드 | 9 |
| PreToolUse | 도구 호출 전 훅 | 호출을 막거나 변환하는 지점 | 9 |
| PostToolUse | 도구 호출 후 훅 | 결과를 정규화하는 지점 | 9 |
| task decomposition | 태스크 분해 | 큰 일을 처리 가능한 단위로 나누기 | 10 |
| prompt chaining | 프롬프트 체이닝 | 단계를 미리 고정한 연결 | 10 |
| dynamic decomposition | 동적 분해 | 중간 결과를 보고 다음을 정하는 방식 | 10 |
| session | 세션 | 대화와 상태의 단위 | 11 |
| resume | 재개 | 이전 세션을 이어 가는 일 | 11 |
| fork | 포크 | 같은 지점에서 갈라 두 갈래로 실험하는 일 | 11 |

### A.5 도구와 MCP

| 영문 | 한국어 | 한 줄 설명 | 장 |
|---|---|---|---|
| tool definition | 도구 정의 | 이름·설명·입출력 스키마의 묶음 | 12 |
| tool description | 도구 설명 | 모델의 선택을 결정하는 문장 | 12 |
| boundary | 경계 | 이 도구가 하고 하지 않는 일의 선 | 12 |
| input schema | 입력 스키마 | 인자의 형식과 제약 | 12 |
| output schema | 출력 스키마 | 결과의 필드와 형식 | 12 |
| structured error | 구조화 오류 | 범주·재시도 여부·힌트를 담은 오류 | 13 |
| errorCategory | 오류 범주 | 오류의 종류를 나타내는 값 | 13 |
| isRetryable | 재시도 가능 여부 | 같은 호출을 다시 해도 되는지 | 13 |
| partialResults | 부분 결과 | 일부만 성공했을 때의 결과 | 13 |
| tool_choice | 도구 선택 방식 | auto·any·특정 도구 강제 | 14 |
| tool sprawl | 도구 폭증 | 도구가 너무 많아 선택이 흐려지는 상태 | 14 |
| namespacing | 이름 공간 | 도구 이름을 묶어 구분하는 방식 | 14 |
| tool search | 도구 검색 | 필요한 도구를 찾아 쓰는 방식 | 14 |
| .mcp.json | 프로젝트 MCP 설정 | 저장소 단위 서버 등록 파일 | 15 |
| user scope | 사용자 범위 | 내 컴퓨터 전체에 적용되는 설정 | 15 |
| env expansion | 환경변수 확장 | 설정에서 환경변수를 참조하는 문법 | 15 |
| resource | 리소스 | 도구 호출 없이 읽는 자료 | 15 |
| elicitation | 유도 질문 | 서버가 사용자에게 묻는 규약 | 15 |
| remote MCP | 원격 MCP | 인증이 붙은 네트워크 너머의 서버 | 15 |
| MCP tunnel | MCP 터널 | 사내망 서버를 안전하게 연결하는 통로 | 15 |
| built-in tools | 내장 도구 | Read·Write·Edit·Bash·Grep·Glob | 16 |
| JSON-RPC | JSON-RPC | MCP의 요청·응답 형식 | 15 |
| transport | 전송 | stdio와 Streamable HTTP | 15 |
| contract test | 계약 시험 | 서버의 약속이 지켜지는지 보는 시험 | 32 |

### A.6 Claude Code

| 영문 | 한국어 | 한 줄 설명 | 장 |
|---|---|---|---|
| CLAUDE.md | 클로드 규칙 문서 | 프로젝트·사용자·디렉터리별 지시 파일 | 17 |
| memory hierarchy | 메모리 계층 | user·project·directory 세 층 | 17 |
| @import | 임포트 | 다른 문서를 끌어오는 문법 | 17 |
| rules | 규칙 파일 | `.claude/rules/`의 조건부 규칙 | 17 |
| path-specific rule | 경로별 규칙 | glob으로 범위를 정한 규칙 | 19 |
| slash command | 슬래시 커맨드 | 사용자가 이름으로 부르는 절차 | 18 |
| skill | 스킬 | 설명을 읽은 모델이 로드하는 절차 | 18 |
| SKILL.md | 스킬 정의 파일 | 머리말과 본문으로 된 스킬 | 18 |
| description | 설명 | 스킬·서브에이전트의 호출 조건 | 18 |
| allowed-tools | 허용 도구 | 스킬이 쓸 수 있는 도구의 범위 | 18 |
| context: fork | 컨텍스트 포크 | 별도 컨텍스트에서 실행하는 설정 | 18 |
| argument-hint | 인자 안내 | 커맨드 입력 형식의 힌트 | 18 |
| plan mode | 계획 모드 | 실행 전에 조사와 계획을 만드는 모드 | 20 |
| Explore subagent | 탐색 서브에이전트 | 조사만 하는 읽기 전용 에이전트 | 20 |
| custom subagent | 사용자 정의 서브에이전트 | `.claude/agents/`에 정의된 보조 에이전트 | 7 |
| headless mode | 비대화형 모드 | `-p`로 한 번에 실행하는 방식 | 21 |
| output-format json | JSON 출력 | 결과를 기계가 읽는 형식으로 받기 | 21 |
| json-schema | JSON 스키마 | 출력 구조를 강제하는 옵션 | 21 |
| CI/CD | 지속적 통합·배포 | 자동 검사와 배포의 흐름 | 21 |
| permission mode | 권한 모드 | 무엇을 묻지 않고 실행할지 정하는 설정 | 35 |
| sandbox | 샌드박스 | 파일·네트워크를 격리하는 실행 환경 | 35 |
| plugin | 플러그인 | 스킬·훅·서버를 묶은 배포 단위 | 35 |
| marketplace | 마켓플레이스 | 플러그인 목록을 제공하는 저장소 | 35 |
| manifest | 매니페스트 | 플러그인의 이름·버전·구성 선언 | 35 |

### A.7 프롬프트와 구조화 출력

| 영문 | 한국어 | 한 줄 설명 | 장 |
|---|---|---|---|
| explicit criteria | 명시적 기준 | 판단의 조건을 문장으로 정하는 일 | 22 |
| false positive | 오탐 | 아니라고 해야 할 것을 맞다고 판정 | 22 |
| severity | 심각도 | 문제의 등급과 그 기준 | 22 |
| few-shot | 소수 예시 | 예시 몇 개로 판단 방식을 보여 주기 | 23 |
| boundary case | 경계 사례 | 규칙이 갈리는 모호한 사례 | 23 |
| structured output | 구조화 출력 | 스키마에 맞춘 기계가 읽는 출력 | 24 |
| strict tool use | 엄격한 도구 사용 | 스키마를 강제하는 모드 | 24 |
| nullable | 널 허용 | 값이 없음을 표현하는 형식 | 24 |
| enum | 열거형 | 정해진 값의 목록 | 24 |
| other | 기타 | 목록 밖의 값을 담는 칸 | 24 |
| validation retry | 검증 재시도 | 오류를 알려 주고 다시 시도하는 일 | 25 |
| custom_id | 항목 아이디 | 배치에서 요청과 결과를 짝짓는 식별자 | 25 |
| multi-pass review | 다중 패스 리뷰 | 여러 번 나눠 검토하는 설계 | 26 |
| LLM-as-judge | 모델 판정자 | 모델에게 채점하게 하는 방식 | 33 |
| confidence routing | 신뢰도 라우팅 | 확신 정도에 따라 처리 경로를 나누기 | 31 |

### A.8 컨텍스트·신뢰성·평가

| 영문 | 한국어 | 한 줄 설명 | 장 |
|---|---|---|---|
| progressive summarization | 점진 요약 | 대화를 요약해 문맥을 줄이는 일 | 27 |
| fact block | 팩트 블록 | 조건과 사실을 고정해 두는 문서 | 27 |
| lost in the middle | 중간 소실 | 긴 문맥 가운데의 정보를 놓치는 현상 | 27 |
| context editing | 컨텍스트 편집 | 오래된 도구 결과를 걷어내는 일 | 27 |
| compaction | 압축 | 긴 세션의 이력을 줄여 이어 가기 | 27 |
| escalation | 에스컬레이션 | 사람에게 넘기는 판단과 절차 | 28 |
| ambiguity | 모호성 | 해석이 갈리는 요청 | 28 |
| policy gap | 정책 공백 | 규정에 답이 없는 상태 | 28 |
| error propagation | 오류 전파 | 실패를 구조화해 상위로 전달하는 일 | 29 |
| partial result | 부분 결과 | 일부만 성공한 결과 | 29 |
| scratchpad | 스크래치패드 | 진행과 결정을 적는 작업 노트 | 30 |
| file manifest | 자료 목록 | 읽은 자료와 남은 자료의 관리표 | 30 |
| stratified sampling | 층화 표본 | 세그먼트별로 나눠 뽑는 표본 | 31 |
| provenance | 출처 | 정보의 근거와 이력 | 31 |
| golden set | 골든셋 | 정답을 붙인 시험지 | 33 |
| regression eval | 회귀 평가 | 하던 것을 아직 하는지 보는 검사 | 33 |
| capability eval | 역량 평가 | 무엇을 할 수 있는지 보는 검사 | 33 |
| gate | 게이트 | 배포를 막거나 여는 판정 | 33 |
| observability | 관측 | 로그·지표로 상태를 보는 일 | 33 |
| p95 latency | 95백분위 지연 | 느린 쪽 끝의 지연 | 33 |
| provenance chain | 출처 사슬 | 값에서 원문까지 이어지는 연결 | 31 |
