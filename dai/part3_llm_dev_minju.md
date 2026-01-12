# Part 3 | LLM을 이용한 개발기

## 1. RAG(Retrieval-Augmented Generation, 검색 증강 생성)란?

LLM이 외부 지식 베이스에서 관련 정보를 검색해 답변을 생성하는 기술.
- 2020년 Facebook AI Research에서 처음 제안
- LLM의 환각(Hallucination) 현상 완화
- 모델 재학습 없이 최신 정보 활용 가능

### 기존 LLM vs RAG

```plaintext
// 기존 LLM
질문 → LLM(학습된 지식만 사용) → 답변 (outdated 가능성)

// RAG
질문 → 벡터 DB 검색 → 관련 문서 추출 → LLM + 검색 결과 → 정확한 답변
```

### RAG 파이프라인 구성

```plaintext
1. 데이터 준비 (Indexing)
   문서 수집 → 청킹(Chunking) → 임베딩 변환 → 벡터 DB 저장

2. 검색 및 생성 (Retrieval & Generation)
   사용자 질문 → 질문 임베딩 → 유사 문서 검색 → LLM에 컨텍스트 전달 → 답변 생성
```

## 2. 용어 정리

### 1. Few-shot 프롬프팅

LLM에게 몇 가지 예시를 보여주고 패턴을 학습시키는 기법.

```plaintext
Zero-shot: 예시 없이 바로 질문
One-shot: 예시 1개 제공
Few-shot: 예시 2~5개 제공

예시가 많을수록 정확도 향상, 단 토큰 비용 증가
```

### 2. BigQuery

Google Cloud의 서버리스 데이터 웨어하우스. LLM 서비스에서 로그, 프롬프트, 응답 데이터를 분석하기 위해 사용함.

```plaintext
특징:
- SQL로 페타바이트급 데이터 분석
- 서버 관리 불필요
- 사용한 만큼만 비용 지불
```

### 3. 시맨틱 캐싱 (Semantic Caching)

의미적으로 유사한 질문에 대해 캐시된 답변을 반환하는 기법.

```plaintext
작동 원리:
1. 질문 → 임베딩 벡터 변환
2. 캐시된 질문들과 유사도 계산
3. 임계값(0.85~0.95) 이상이면 캐시 히트
4. 미만이면 LLM 호출 후 캐시 저장

장점:
- API 비용 대폭 절감
- 응답 시간 단축
- 동일 의도 질문 일관된 답변
```

### 4. chromem-go

Go 언어용 임베딩 가능한 벡터 데이터베이스.

```plaintext
특징:
- 외부 의존성 제로
- Chroma와 유사한 인터페이스
- 인메모리 + 영구 저장 옵션
- RAG 구현에 최적화

사용 예시:
- Go 백엔드에서 직접 벡터 검색
- 별도 DB 서버 없이 임베딩
```

### 5. 주성분 분석 (PCA)

고차원 데이터의 핵심 정보를 유지하면서 차원을 줄이는 기법.

```plaintext
사용 이유:
- 임베딩 벡터: 보통 768~1536차원
- 고차원 = 저장공간 ↑, 계산비용 ↑

작동 방식:
1. 데이터 분산이 가장 큰 방향(주성분) 찾기
2. 상위 k개 주성분만 선택
3. 원본 데이터를 k차원으로 투영

효과:
- 1536차원 → 256차원 (83% 압축)
- 검색 속도 향상
- 핵심 의미 정보는 보존
```

### 6. DBSCAN 알고리즘

밀도 기반 클러스터링 알고리즘.

```plaintext
- Epsilon(ε): 이웃 판단 반경
- MinPts: 군집 형성 최소 점 개수
- Core Point: ε 내 MinPts 이상 있는 점
- Border Point: Core Point 이웃이지만 자신은 아닌 점
- Noise Point: 어디에도 속하지 않는 점

RAG 사용 예시:
- 유사 질문 그룹화
- 캐시 최적화
- 중복 문서 제거
```

| 항목 | K-Means | DBSCAN |
|---|---|---|
| 클러스터 수 | 사전 지정 필요 | 자동 결정 |
| 클러스터 형태 | 원형 형태만 가능 | 다양한 형태 지원 |
| 이상치 처리 | 클러스터에 포함됨 | 노이즈로 분리 |

## 참고 자료

1. [당근 테크 블로그, "당근의 GenAI 플랫폼"](https://medium.com/daangn/당근의-genai-플랫폼-ee2ac8953046)
2. [AWS, "RAG란?"](https://aws.amazon.com/ko/what-is/retrieval-augmented-generation)