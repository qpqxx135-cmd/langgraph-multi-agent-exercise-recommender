# LangGraph Multi-Agent Exercise Recommender

## 프로젝트 개요

본 프로젝트는 사용자의 자연어 질문을 입력받아 현재 상태를 분석하고, 적절한 운동 후보를 추천한 뒤 최종 답변을 생성하는 **LangGraph 기반 멀티 에이전트 운동 추천 시스템**입니다.

예시 질문은 다음과 같습니다.

> 체력이 안좋고, 살이 계속 찌는데 어떤 운동을 할까?

본 프로젝트의 핵심은 하나의 LLM이 모든 작업을 한 번에 처리하는 방식이 아니라, 역할이 분리된 여러 에이전트가 순차적으로 협업하도록 구성했다는 점입니다.

사용자의 질문은 다음 3단계 에이전트 흐름을 거쳐 처리됩니다.

1. 증상 추출 에이전트
2. 운동 후보 에이전트
3. 답변 생성 에이전트

---

## 프로젝트 목적

이 프로젝트의 목적은 LangGraph를 활용하여 멀티 에이전트 워크플로우를 직접 구현하고, 각 에이전트가 독립적인 역할을 수행하면서 하나의 최종 답변을 생성하는 과정을 실습하는 것입니다.

기존의 단일 LLM 호출 방식은 입력 질문에 대해 바로 답변을 생성하지만, 본 프로젝트에서는 작업을 단계별로 분리했습니다.

이를 통해 다음과 같은 장점을 확인할 수 있습니다.

- 에이전트별 역할 분리
- 중간 처리 결과 확인 가능
- 답변 생성 과정의 해석 가능성 향상
- 프롬프트 수정 및 디버깅 용이
- LangGraph 기반 워크플로우 구조 이해

---

## 멀티 에이전트 구조

### 1. 증상 추출 에이전트

사용자의 질문에서 현재 겪고 있는 문제나 상태를 추출합니다.

예시 입력:

```txt
체력이 안좋고, 살이 계속 찌는데 어떤 운동을 할까?
```

예시 출력:

```txt
['체력 저하', '체중 증가']
```

이 에이전트는 사용자의 질문에서 운동 추천을 바로 생성하지 않고, 사용자가 현재 겪고 있는 상태만 추출하는 역할을 합니다.

---

### 2. 운동 후보 에이전트

증상 추출 에이전트가 추출한 사용자의 상태를 바탕으로 적절한 운동 후보를 생성합니다.

예시 입력:

```txt
체력 저하, 체중 증가
```

예시 출력:

```txt
['걷기', '수영', '가벼운 요가', '저강도 근력운동', '팔굽혀펴기', '스쿼트 기본 형태', '스트레칭']
```

이 에이전트는 체력이 낮은 사용자가 무리하지 않고 시작할 수 있는 운동을 중심으로 후보를 생성합니다.  
특히 고강도 운동보다는 낮은 강도의 유산소 운동, 기초 근력운동, 스트레칭 위주의 운동을 추천하도록 구성했습니다.

---

### 3. 답변 생성 에이전트

증상 추출 에이전트와 운동 후보 에이전트의 결과를 바탕으로 최종 사용자 답변을 생성합니다.

예시 입력:

```txt
사용자 상태: 체력 저하, 체중 증가
운동 후보: 걷기, 수영, 가벼운 요가, 저강도 근력운동, 팔굽혀펴기, 스쿼트 기본 형태, 스트레칭
```

예시 출력 형식:

```txt
[사용자 상태 요약]
- 체력 저하와 체중 증가가 함께 나타나는 상태입니다.
- 처음부터 고강도 운동보다 낮은 강도의 운동부터 시작하는 것이 좋습니다.

[추천 운동 리스트]
- 걷기: 낮은 강도의 운동으로 시작하기 좋고 체력 향상에 도움이 됩니다.
- 수영: 관절 부담이 적고 전신 운동 효과가 있습니다.
- 가벼운 요가: 유연성과 기초 체력 향상에 도움이 됩니다.
- 저강도 근력운동: 근육량 유지와 기초대사량 관리에 도움이 됩니다.

[주의사항]
- 운동 전 스트레칭을 통해 부상을 예방하는 것이 좋습니다.
- 통증이나 어지러움이 있으면 즉시 운동을 중단해야 합니다.
```

이 에이전트는 단순히 운동 후보를 나열하는 것이 아니라, 사용자의 상태를 요약하고 운동별 추천 이유와 주의사항까지 포함한 개조식 답변을 생성합니다.

---

## 전체 시스템 흐름

본 프로젝트의 전체 처리 과정은 다음과 같습니다.

```txt
사용자 질문 입력
        ↓
증상 추출 에이전트
        ↓
운동 후보 에이전트
        ↓
답변 생성 에이전트
        ↓
최종 답변 출력
```

---

## LangGraph 그래프 구조

LangGraph에서는 각 에이전트를 하나의 노드로 구성하고, 노드 간 연결을 통해 실행 흐름을 정의합니다.

본 프로젝트의 그래프 구조는 다음과 같습니다.

```txt
__start__
    ↓
상태_추출
    ↓
운동_추천
    ↓
답변_작성
    ↓
__end__
```

아래 이미지는 LangGraph로 구성한 멀티 에이전트 워크플로우 구조입니다.

<!-- 그래프 구조 사진 넣는 부분 -->
<!-- 파일 경로: images/graph.png -->

<img width="168" height="808" alt="image" src="https://github.com/user-attachments/assets/ffbd35c3-261c-4501-9e94-8b787dec3d11" />


---

## 실행 결과

아래 이미지는 실제 사용자 질문을 입력했을 때 각 에이전트가 순차적으로 실행된 결과입니다.

<!-- 실행 결과 사진 넣는 부분 -->
<!-- 파일 경로: images/result.png -->

<img width="1106" height="810" alt="image" src="https://github.com/user-attachments/assets/14072b70-b7b4-496b-ac0a-15f2a36bd9c0" />



실행 결과에서는 다음 내용을 확인할 수 있습니다.

- 사용자 질문
- 증상 추출 에이전트 결과
- 운동 후보 에이전트 결과
- 답변 생성 에이전트 결과
- 최종 답변

---

## 기술 스택

| 구분 | 기술 |
|---|---|
| Language | Python |
| Workflow Framework | LangGraph |
| LLM Framework | LangChain |
| Local LLM Runtime | Ollama |
| LLM Model | EXAONE 3.5 2.4B |
| Prompt Template | ChatPromptTemplate |
| Output Parser | StrOutputParser |
| Graph Visualization | Mermaid / ASCII Graph |
| Development Environment | Jupyter Notebook |

---

## 주요 라이브러리

```python
from typing import TypedDict, List

from langgraph.graph import StateGraph, START, END
from langchain_ollama import ChatOllama
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

from IPython.display import Image, display
```

---

## 설치 방법

### 1. Python 라이브러리 설치

```bash
pip install langgraph langchain langchain-core langchain-ollama grandalf ipython
```

### 2. Ollama 모델 다운로드

본 프로젝트에서는 Ollama 기반 로컬 LLM으로 `exaone3.5:2.4b` 모델을 사용했습니다.

```bash
ollama pull exaone3.5:2.4b
```

### 3. Ollama 서버 실행

```bash
ollama serve
```

---

## 실행 방법

Jupyter Notebook 환경에서 아래 파일을 실행합니다.

```txt
langgraph.ipynb
```

실행 순서는 다음과 같습니다.

```txt
1. 라이브러리 설치
2. Ollama 모델 실행 확인
3. LLM 설정
4. State 정의
5. 에이전트 함수 정의
6. LangGraph 그래프 구성
7. 그래프 시각화
8. 사용자 질문 입력 및 실행
```

---

## 핵심 코드 설명

### 1. State 정의

LangGraph에서는 각 노드가 공유하는 데이터를 `State`로 관리합니다.

```python
class ExerciseState(TypedDict):
    user_question: str
    symptoms: List[str]
    exercise_candidates: List[str]
    final_answer: str
```

각 State 값의 의미는 다음과 같습니다.

| State Key | 설명 |
|---|---|
| user_question | 사용자가 입력한 원본 질문 |
| symptoms | 증상 추출 에이전트가 추출한 사용자 상태 |
| exercise_candidates | 운동 후보 에이전트가 생성한 추천 운동 후보 |
| final_answer | 답변 생성 에이전트가 생성한 최종 답변 |

---

### 2. LLM 설정

Ollama에서 실행 중인 로컬 LLM을 LangChain의 `ChatOllama`로 연결합니다.

```python
llm = ChatOllama(
    model="exaone3.5:2.4b",
    temperature=0.2
)
```

`temperature`를 낮게 설정하여 답변의 일관성을 높였습니다.

---

### 3. 그래프 노드 등록

각 에이전트 함수는 LangGraph의 노드로 등록됩니다.

```python
graph_builder.add_node("상태_추출", symptom_extractor_agent)
graph_builder.add_node("운동_추천", exercise_candidate_agent)
graph_builder.add_node("답변_작성", answer_generator_agent)
```

---

### 4. 그래프 흐름 연결

에이전트 실행 순서는 `add_edge()`를 통해 정의합니다.

```python
graph_builder.add_edge(START, "상태_추출")
graph_builder.add_edge("상태_추출", "운동_추천")
graph_builder.add_edge("운동_추천", "답변_작성")
graph_builder.add_edge("답변_작성", END)
```

이 구조를 통해 사용자 질문은 다음 순서로 처리됩니다.

```txt
START → 상태_추출 → 운동_추천 → 답변_작성 → END
```

---

### 5. 그래프 컴파일

정의한 그래프는 실행 전에 `compile()`을 통해 컴파일합니다.

```python
app = graph_builder.compile()
```

---

### 6. 그래프 시각화

그래프 구조는 다음 코드로 시각화할 수 있습니다.

```python
app.get_graph().print_ascii()
```

또는 Mermaid PNG 형태로 출력할 수 있습니다.

```python
display(Image(app.get_graph().draw_mermaid_png()))
```

---

## 실행 예시

### 입력 질문

```txt
체력이 안좋고, 살이 계속 찌는데 어떤 운동을 할까?
```

### 중간 결과 예시

```txt
[1] 증상/상태 추출

추출 결과: ['체력 저하', '체중 증가']

[2] 운동 후보 생성

추천 후보: ['걷기', '수영', '가벼운 요가', '저강도 근력운동', '팔굽혀펴기', '스쿼트 기본 형태', '스트레칭']
```

### 최종 답변 예시

```txt
[사용자 상태 요약]
- 체력 저하 및 체중 증가 상태로 운동을 통해 건강 개선이 필요합니다.
- 현재 활동 수준이 낮아 점진적인 운동 시작이 권장됩니다.

[추천 운동 리스트]
- 걷기: 낮은 강도의 운동으로 시작해 체력 향상에 도움이 됩니다.
- 수영: 관절 부담이 적고 전신 운동 효과가 있습니다.
- 가벼운 요가: 유연성과 기초 체력 향상에 도움이 됩니다.
- 저강도 근력운동: 근육량 유지와 기초대사량 관리에 도움이 됩니다.

[주의사항]
- 모든 운동 전 스트레칭을 통해 부상을 예방해야 합니다.
- 개인의 건강 상태에 맞는 운동 계획을 세우는 것이 좋습니다.
```

---

## 프로젝트 특징

### 1. 단일 LLM 호출이 아닌 멀티 에이전트 구조

본 프로젝트는 하나의 프롬프트로 바로 최종 답변을 생성하지 않고, 작업을 여러 단계로 나누었습니다.

```txt
질문 분석 → 상태 추출 → 운동 후보 생성 → 답변 작성
```

이를 통해 각 단계의 역할을 명확히 분리하고, 중간 결과를 확인할 수 있습니다.

---

### 2. State 기반 데이터 전달

LangGraph의 `State`를 사용하여 각 에이전트의 결과를 다음 에이전트로 전달했습니다.

예를 들어, 상태 추출 에이전트가 생성한 `symptoms`는 운동 후보 에이전트의 입력으로 사용됩니다.  
운동 후보 에이전트가 생성한 `exercise_candidates`는 답변 생성 에이전트의 입력으로 사용됩니다.

---

### 3. 로컬 LLM 기반 실행

Ollama를 사용하여 로컬 환경에서 LLM을 실행했습니다.  
이를 통해 외부 API 키 없이도 LangGraph 기반 멀티 에이전트 실습이 가능합니다.

---

### 4. 중간 결과 확인 가능

각 에이전트의 결과를 출력하도록 구성하여 워크플로우의 처리 과정을 확인할 수 있습니다.

```txt
[1] 증상/상태 추출
[2] 운동 후보 생성
[3] 답변 생성
```

이 출력은 에이전트별 역할이 실제로 분리되어 작동하고 있음을 보여줍니다.

---

## 한계점 및 개선 방향

### 한계점

- LLM 응답 특성상 실행할 때마다 추천 운동이나 답변 표현이 조금씩 달라질 수 있습니다.
- 본 시스템은 의학적 진단 시스템이 아니라 일반적인 운동 추천 예시입니다.
- 현재는 단일 질문에 대해 순차적인 에이전트 흐름만 구현했습니다.
- 사용자의 나이, 성별, 운동 경험, 질환 여부 등 세부 정보를 반영하지는 않았습니다.
