Claude를 그냥 단순 프롬포트에서 질문 대답형태로 사용하면 100% 효율로 사용할 수 없다. 


### CLAUDE.md
- 클로드한테 우리 프로젝트가 뭔지 알려주는 설명서. 가볍게 만들 것. 
- 목차만 주고 필요할 때 찾아서 쓰게 해야함.


### 시스템 
MCP나 플러그인을 최소화해서 사용할 것. 
MCP를 너무 많이 켜두지 말고 필요한 것만 활성화 


### statusline 
- 토큰 사용량 실시간 모니터링 


### 컨텍스트는 우유다 
- `/compact` 큰 기능을 완성했을 때나, 작업 맥락이 바뀔 때 사용해야함.

### 모델 고르기 
- 모든 모델을 최상위 모델인 opus를 사용할 필요가 없다.
- sonnet (default) 가장 기본적임. 
- 

### 레퍼런스 코드 같이 주기 

### 서브 에이전트
- planner : 계획을 담당하는 에이전트 
- developer : 개발을 담당하는 에이전트 
- reviewer : 리뷰를 담당하는 에이전트
- tester : 테스트를 담당하는 에이전트 
- documenter : 다큐먼트 문서 관리 및 기록을 담당하는 에이전트 

### skills.md
- 직원(서브에이전트)들이 따라야하는 룰 

```
# 비전공자도 이해할 수 있게, 코드마다 한국어 주석을 달아줘.
중요도를 추가하는 기능을 skills 폴더에 있는 md파일을 참고해서 만들어줘 
```

### commands 폴더
develop.md 
/develop 이라고 입력하면 서브에이전트를 이용해서 기능을 구현해주는 command를 만들어줘 그리고 항상 skills도 참고하게 해줘.

/develop 마감일 기능을 추가해줘 

### Git Worktree
각 폴더에서 클로드를 따로 사용하면 병렬 실행이 가능함.


### HOOK 
특정 순간에 알람이 오는 기능.

SessionStart - 기록 로드
PreCompact - 중요 내용 저장
Stop - 학습 기록 


### 에이전트 스킬 마켓 플레이스
- react-development (리액트 관련 개발시 사용)
- bkit 
```
/plugin marketplace add popup-studio-ai/bkit-claude-code
/plugin install bkit
```
PDCA 를 에이전트로 담은 것 (Plan, Do, Check, Act(개선-조치))
ex) /pdca plan todo-app 을 만들어줘 