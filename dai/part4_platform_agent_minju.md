# Part 4 | AI 플랫폼과 AI 에이전트 개발기

## 1. AI 에이전트란?

자율적으로 판단하고 행동하며, 도구를 활용해 목표를 달성하는 AI 시스템.
- 단순 챗봇과 달리 **스스로 계획**하고 **실행**
- 외부 도구(API, 검색, DB 등)와 연동 가능
- Function Calling, 플러그인 등을 통해 실제 업무 수행

### 챗봇 vs AI 에이전트

```plaintext
// 챗봇
사용자 질문 → 답변 생성 → 끝

// AI 에이전트
목표 설정 → 계획 수립 → 도구 선택 → 실행 → 결과 검증 → 반복
```

## 2. 멀티 에이전트 시스템

### 사용하는 이유

하나의 에이전트가 모든 일을 처리하면 한계 발생
- 복잡한 작업에서 정확도 저하
- 역할 분리가 안 되어 디버깅 어려움
- 특정 도메인 전문성 부족

### 멀티 에이전트 구조

<img width="1113" height="531" alt="image" src="https://github.com/user-attachments/assets/5455f9a8-8a7e-4954-924e-340080d3db44" />

### 핵심 개념

**1) 역할 기반 설계**
```plaintext
각 에이전트에게 명확한 역할 부여
- 연구원: 정보 수집 및 분석
- 작성자: 콘텐츠 생성
- 검토자: 품질 검증 및 피드백
```

**2) 오케스트레이션**
```plaintext
에이전트 간 협업을 조율하는 방식
- 순차적(Sequential): A → B → C
- 병렬(Parallel): A, B, C 동시 실행
- 계층적(Hierarchical): 관리자가 하위 에이전트 지휘
```

**3) 핸드오프(Handoff)**
```plaintext
작업 완료 후 다음 에이전트에게 전달
- 공유 메모리를 통한 컨텍스트 유지
- 이전 작업 결과를 입력으로 활용
```

## 3. 오케스트레이션 패턴

<img width="666" height="536" alt="image" src="https://github.com/user-attachments/assets/531866b0-bd25-4a35-a9ea-600e4c431488" />

*출처: [Saga 오케스트레이션 패턴 - AWS 권장 가이드](https://docs.aws.amazon.com/ko_kr/prescriptive-guidance/latest/cloud-design-patterns/saga-orchestration.html)*

### LLM 기반 오케스트레이션

```plaintext
장점:
- 유연한 의사결정
- 복잡한 상황 대응 가능
- 새로운 작업에 적응력 높음

단점:
- 비용 증가
- 예측 어려움
- 디버깅 복잡
```

### 코드 기반 오케스트레이션

```plaintext
장점:
- 결정론적, 예측 가능
- 비용 절감
- 디버깅 용이

단점:
- 유연성 부족
- 사전 정의된 흐름만 가능
```

### 하이브리드 접근

```plaintext
실무에서 권장되는 방식:
- 핵심 흐름: 코드로 제어
- 동적 판단 필요 시: LLM 활용
- 예: 분류는 LLM, 실행은 코드
```

## 4. 멀티 에이전트 프레임워크

### 주요 도구 비교

| 프레임워크 | 특징 | 적합한 상황 |
|-----------|------|------------|
| **CrewAI** | 역할 기반, 로우코드 | 빠른 프로토타입, 팀 협업 시뮬레이션 |
| **LangGraph** | 상태 그래프, 세밀한 제어 | 복잡한 조건 분기, 프로덕션 |
| **AutoGen** | 그룹 채팅 방식 | 토론/협의 필요한 작업 |
| **OpenAI Agents SDK** | 공식 SDK, 안정성 | OpenAI 생태계 활용 |

### CrewAI 기본 구조

```python
from crewai import Agent, Task, Crew, Process

# 에이전트 정의
researcher = Agent(
    role="시장 조사 전문가",
    goal="최신 트렌드와 경쟁사 분석",
    backstory="10년 경력의 시장 분석가"
)

writer = Agent(
    role="콘텐츠 작성자",
    goal="명확하고 설득력 있는 문서 작성",
    backstory="테크 블로거 출신 작가"
)

# 작업 정의
research_task = Task(
    description="AI 에이전트 시장 동향 조사",
    expected_output="마크다운 형식의 분석 리포트",
    agent=researcher
)

write_task = Task(
    description="조사 결과를 블로그 글로 변환",
    expected_output="1000자 내외의 블로그 포스트",
    agent=writer
)

# 크루 구성 및 실행
crew = Crew(
    agents=[researcher, writer],
    tasks=[research_task, write_task],
    process=Process.sequential
)

result = crew.kickoff()
```

## 5. 주의사항

### 멀티 에이전트의 함정

**1) 에이전트 과잉**
```plaintext
❌ 단순 작업에 5개 에이전트 투입
⭕ 복잡도에 맞는 최소한의 에이전트
```

**2) 무한 루프**
```plaintext
❌ 종료 조건 없이 반복
⭕ 최대 반복 횟수, 타임아웃 설정
```

**3) 비용 폭발**
```plaintext
❌ 에이전트마다 고가 모델 사용
⭕ 역할에 따라 모델 차등 적용
   - 판단: GPT-4
   - 단순 작업: GPT-3.5 또는 로컬 모델
```

## CoT(Chain of Thought)

### CoT란?

**"단계별로 생각하세요"** 라고 AI에게 요청하는 프롬프트 기법.

```plaintext
일반 프롬프트:
"사과 10개를 샀고, 4개를 나눠주고, 5개를 더 샀다. 몇 개?"
→ AI: "11개" (틀림)

CoT 프롬프트:
"단계별로 생각하세요. 사과 10개를 샀고..."
→ AI: 
  1. 처음 10개
  2. 4개 나눠줌 → 6개 남음
  3. 5개 추가 → 11개
  답: 11개 (맞음!)
```

### 사용하는 이유

```plaintext
사람이 문제를 푸는 것처럼 복잡한 문제를 작은 단계로 나눠서 해결

효과:
- 수학 문제 정확도 57% (기존 18%)
- 논리적 추론 능력 향상
- 답변의 근거 추적 가능
```

## 참고 자료

1. [IBM, "AI 에이전트 오케스트레이션이란?"](https://www.ibm.com/kr-ko/think/topics/ai-agent-orchestration)
2. [Prompting Guide, "Chain-of-Thought"](https://www.promptingguide.ai/kr/techniques/cot)