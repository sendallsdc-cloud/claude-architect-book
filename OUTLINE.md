# 집필용 장별 한 줄 설명

아직 본문을 쓰기 전 단계라 `pages/`에 장 파일이 없습니다. 이 파일은 장 제목과 한 줄 설명만 담은 집필용 메모이며, 위키독스 동기화는 `README.md`·`TOC.md`·`pages/`만 읽습니다.

## Part 0. 자격과 시험의 지도

- 01. Claude 자격 제도, 정확히 알고 시작하기 — 5개 시험의 지도, CCAR-F의 60문항·120분·720점·$125, 파트너 기업 소속 제한, 2026-12-07 개정 대상 3과목과 이 시험의 관계

## Part I. 기초 — Claude를 시스템으로 보는 눈

- 02. 앱·API·Agent SDK·Claude Code·MCP·Managed Agents — 여섯 표면을 "누가 루프를 돌리는가" 하나의 축으로 구분
- 03. 모델 선택과 손잡이 — 모델 티어, effort, thinking, 컨텍스트 한도가 비용과 지연을 어떻게 바꾸는가
- 04. Messages API와 tool use의 최소 원리 — messages·system·tool_use·tool_result·stop_reason만으로 읽는 루프
- 05. 첫 에이전트 루프 — 30줄짜리 루프를 만들고 일부러 실패시킨 뒤 고친다

## Part II. 에이전트 아키텍처 (시험 비중 27%)

- 06. 에이전틱 루프의 수명주기 — stop_reason 기반 종료 판단과 텍스트 판별·반복 상한 안티패턴
- 07. 코디네이터-서브에이전트 — 허브-스포크 구조, 격리된 컨텍스트, 위임 여부를 가르는 기준
- 08. 컨텍스트 전달과 병렬 실행 — Task 도구, allowedTools, 구조화된 전달, 한 응답에서의 다중 Task
- 09. 강제와 핸드오프 — 프로그램적 게이트와 프롬프트 지시의 차이, PostToolUse 정규화, 에스컬레이션 요약
- 10. 태스크 분해 — 프롬프트 체이닝과 동적 분해, 파일별 작업과 통합 패스
- 11. 세션·재개·포크 — --resume, fork_session, 오래된 도구 결과를 다시 넣는 방법

## Part III. 도구 설계와 MCP (시험 비중 18%)

- 12. 도구 인터페이스 설계 — 도구 설명이 선택을 결정한다, 경계·예시·입출력, 겹치는 도구 분리
- 13. 구조화된 에러 응답 — isError·errorCategory·isRetryable·부분 결과의 네 축
- 14. 도구 배분과 tool_choice — 도구 수가 늘 때의 실패, 역할별 스코프, auto/any/forced
- 15. MCP 서버 통합 — 프로젝트 .mcp.json과 사용자 설정의 차이, 환경변수 확장, resources, 원격·터널
- 16. 내장 도구 Read·Write·Edit·Bash·Grep·Glob — 탐색 전략, Edit 실패 시 Read+Write로 전환

## Part IV. Claude Code (시험 비중 20%)

- 17. CLAUDE.md 계층과 모듈화 — user·project·directory 계층, @import, .claude/rules/, /memory
- 18. 커맨드와 스킬 — commands 디렉터리와 SKILL.md, context: fork, allowed-tools, argument-hint
- 19. 경로별 규칙 — YAML paths 글롭으로 조건부 로딩하기, 디렉터리 CLAUDE.md와의 차이
- 20. plan mode vs 직접 실행 — 판단 기준과 Explore 서브에이전트, 조사에서 실행으로 넘기는 조합
- 21. 반복 개선과 CI/CD — 입출력 예시, 테스트 우선, -p·--output-format json·--json-schema, 독립 리뷰

## Part V. 프롬프트와 구조화 출력 (시험 비중 20%)

- 22. 명시적 기준 — "보수적으로 판단하라"가 왜 실패하는가, 카테고리와 심각도 정의
- 23. few-shot — 2~4개 예시로 모호한 사례의 추론을 고정하고 환각을 줄이기
- 24. tool_use + JSON 스키마 — tool_choice, required/nullable, enum+other, 의미 오류와 형식 오류의 분리
- 25. 검증·재시도·배치 — 오류 피드백 재시도, 재시도가 무효한 조건, Batch 50%·24시간·custom_id
- 26. 다중 인스턴스 리뷰 — 자기 리뷰의 한계, 파일별과 통합 리뷰의 역할, 신뢰도 라우팅

## Part VI. 컨텍스트와 신뢰성 (시험 비중 15%)

- 27. 대화 컨텍스트 보존 — 점진 요약의 위험, 중간 손실, 케이스 팩트 블록, 도구 출력 다이어트
- 28. 에스컬레이션과 모호성 해소 — 명시적 기준, 고객 요청 존중, 정책 공백, 다중 매칭
- 29. 다중 에이전트 에러 전파 — 접근 실패와 정상적인 빈 결과의 구분, 부분 결과 전달
- 30. 대규모 코드베이스 컨텍스트 — 컨텍스트 열화, 스크래치패드, 서브에이전트 위임, manifest 복구
- 31. 휴먼 리뷰·신뢰도·출처 — 층화 샘플링, 필드별 신뢰도, claim-source 매핑, 상충 주석

## Part VII. 실무 본편 — 시험을 넘어 프로덕션으로

- 32. MCP 서버를 직접 만들어 Claude Code·Agent SDK에 붙이기 — 도구 3개와 resource, 구조화 에러, 인증, 테스트, 배포까지 (대표 프로젝트)
- 33. 평가·관측 파이프라인 — 골든셋, LLM 심사 편향, 회귀 테스트, 로그·트레이스, 대시보드
- 34. Claude Code로 사내 도구 만들기 — 요구사항에서 CLAUDE.md·스킬·훅·CI·배포까지, 권한과 보안
- 35. 비용·지연·보안·플러그인 — 프롬프트 캐싱, 배치, 모델 라우팅, 샌드박스, 플러그인과 Managed Agents 선택 기준

## Part VIII. 시험 전략

- 36. 12주 학습 계획 — 도메인 가중치 순 배분, 주별 산출물, 파트너사 스터디 운영안
- 37. 자체 모의고사 60문항 — 전부 자체 제작, 시나리오별 배분과 해설·오답 진단표
- 38. 시험 당일과 그 이후 — 신분증·NDA·금지물품, 시간 배분, 점수 리포트, 재응시·갱신

## 부록

- A. 영한 용어 대조표 — 시험에 나오는 용어 120개
- B. 도메인별 치트시트 — 다섯 도메인 요약 카드
- C. 실습 코드·설정 모음 — 복사해 쓰는 .mcp.json, SKILL.md, rules, CI 설정
- D. 공식 자료 지도 — URL과 확인일
- E. 시험 범위 밖 실무 도구 — Vision·Streaming·Files·Citations
