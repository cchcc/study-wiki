개발자에게 배우는 Cursor AI 고급 노하우 : 컨텍스트 엔지니어링 기반 Agent 실무 도입

https://fastcampus.co.kr/classroom/255142


# Vide Coding
- 의도, 맥락을 전달하고 파악하는 것이 중요하다
- 이런것들을 관리하는 기법이 필요하다
  - 코드가 작성되는 시점읜 상황은?
  - 코드가 미치는 영향?
  - 내가 전달한 지시사항에 모든 의도가 들어가 있는가?


# 맥락 전달
## 맥락이란
- 목표: 뭘해야 하는지
- 지식: 목표 달성을 위해 알아야 되는것?
- 제약조건: 어떤 규칙으로?
- ai 가 기억할수 있는 맥락의 크기는 한계가 있음을 주의

## 맥락을 전달하는 설계 방법(Context Engineering)
- Selection: 할것의 범위 좁히기
  - 파일 함수등을 지정 하기
- Isolation: 하지 말아야 할것을 지정해주기
- Write: 참고해야되는거(기록되어있는거) 알려주기
  - Cursor Rules 지정
- Compress: 결과를 이렇게 이렇게 요약해줘
  - 긴 작업에서 중간중간 햇던일을 압축한다
  - ai 가 기억할수 잇는 컨텍스트의 크기가 제한이 있어서 그러함

## 명령 과정
- 명확한 임무 부여
- 관찰(틀렸는지)
- 피드백 루프

# 협업, 위임: ai 를 협업과 위임의 관점으로 바라보기
협업: 알려줘
  - 내가 알아야 되는거를 확장
  - 문제 해결 방법을 도출
위임: 해줘
  - 역할을 명확하게하고 명령과 행동을 통해 업무를 수행하게 하는것
  - 리더십 역량중 하나

## 개발 단계. 그리고 workflow
- 요구사항 분석: 협업
  - 불확실한 영역을 좁히기
  - workflow 1: 문제정의 설계
  - workflow 2: task 분리 및 역할 분배
- 구현 설계: 위임
  - 구체적인 스펙을 잘 전달
  - workflow 3: task 개발 계획 수림
  - workflow 4: 코드 장성
- 리뷰 & 개선: 협업
  - 구체적인 스펙에서 뭔가 빠트린건 없나
  - workflow 5: 검토 및 피드백

## 언제 협업 -> 위임 넘어가나
- 문제의 풀이방법이 명확해 졋을때
- 반복 가능한 어떤 패턴을 찾앗을때
- 원하는 결과를 정확하게 지시 가능할때

## 언제 위임 -> 협업 넘어가나
- 결과물이 예상과 다를때
- 더 나은 방향을 고민해야될때
- 새로운 요건이 추가되었을때

## ai 에게 어떻게 맥락을 잘 전달할수 있을까
- long/short term memory
- retrieved information
- state, history
- available tools
- instructions, system prompt
- user prompt

# 보안
- outbound: 내부 정보를 외부로 유출함
  - 최소한의 권한부여
  - 민감한 파일에는 명시적으로 접근 금지
  - 데이터 마스킹
- inbound: 취약점을 내부에 적용시켜버림
  - 보안 코딩 가이드 주입
  - 외부 라이브러리를 아무거나 가져오는게 아니라 승인된 것들만 사용하도록
  - 보안 검증 절차를 주입(소나큐브 통과)

# Cursor

개발할때 전달하는 맥락 6가지
- 요구사항
- 수정해야할 소스 코드
- 연관된 소스코드
- 라이브러리 & 프레임웤의 문서
- 동작 과정, 절차
- 코드 작성 규칙, 컨벤션

맥락에도 생명주기가 있음
실시간 동작을 눈으로 확인 가능
context 구조화가 용이 하다

# mcp
## 구조
mcp server <-> mcp <-> mcp client(cursor)

## mcp 가 하는일
- Collet: 맥락을 가져오기
- Action: 결과를 실행

## mcp type
local mcp: 로컬에 띄워놓고 연결해서 사용. 주로 node, python 사용 
remote mcp

https://smithery.ai/
- mcp 모음

https://cursor.directory/
- rules 모음


# rules
- insruction 의 일부로 적용함
- 작은 크기로 지시사항을 지정하기 위해(요금제와 관련이)

## user rules
- 계정에 설저되는 룰
- 개인적인 워크 플로우

## project rules
- 플젝별 아키텍쳐, 개발철학, 컨벤션, 기술스택

system prompt + user rules + user promt: 이렇게 3가지가 하나로 묶임
project rulse: 상황에 따라 선택, 적용

