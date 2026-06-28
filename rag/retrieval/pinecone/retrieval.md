# Pinecone Retriever 정리

`Pinecone` 벡터 스토어를 retriever로 변환해 검색하는 방법 정리
> 검색 전략 전체 개요는 상위 `../retrieval.md` 참고
> 이전 단계(벡터 스토어) 전제는 `../../vector-store/pinecone/vector-store.md` 참고

## 1. 전제 (구성)

- 설치: `pip install -qU langchain-pinecone langchain-openai`
- retriever는 벡터 스토어가 있어야 만들 수 있음 → 벡터 스토어 단계에서 만든 `vector_store` 사용
- 서버형 → 이미 만든 Pinecone 인덱스에 `PineconeVectorStore(index=index, embedding=embedding)`로 붙어 변환 가능

## 2. Retriever로 변환

```python
retriever = vector_store.as_retriever(
    search_type='similarity',          # 기본값
    search_kwargs={'k': 3},
)
retriever.invoke('거주자의 정의는?')
```

- `as_retriever`는 `VectorStoreRetriever`를 반환 → `invoke`/`batch` 같은 공통 Runnable 인터페이스로 사용
- `search_type`은 `'similarity'`(기본)·`'mmr'`·`'similarity_score_threshold'` 지원

## 3. 검색 방식

### 방식1: similarity (유사도 top-k)

```python
retriever = vector_store.as_retriever(
    search_type='similarity',
    search_kwargs={'k': 3},
)
```

- 질의 벡터와 가장 가까운 `k`개를 그대로 반환 → 기본 전략

### 방식2: mmr (유사도 + 다양성)

```python
retriever = vector_store.as_retriever(
    search_type='mmr',
    search_kwargs={'k': 3, 'fetch_k': 10, 'lambda_mult': 0.3},
)
```

- `fetch_k`: MMR 후보로 먼저 가져올 개수 (보통 `k`의 3~5배) → 이 중에서 다양성 고려해 `k`개 선별
- `lambda_mult`: 다양성 조절값 → `0.0` 다양성 최대, `1.0` 유사도 최대 (기본 `0.5`)

### 방식3: similarity_score_threshold (점수 컷)

```python
retriever = vector_store.as_retriever(
    search_type='similarity_score_threshold',
    search_kwargs={'k': 3, 'score_threshold': 0.5},
)
```

- `score_threshold` 이상인 문서만 반환 → 관련 없는 결과 컷
- ⚠️ Pinecone 인덱스 `metric='cosine'`은 높을수록 유사 → threshold 기준도 그에 맞춤

## 4. 배치 검색

```python
retriever.batch([
    '거주자 정의는?',
    '배당소득 계산법',
])
```

- 여러 질의를 한 번에 처리 → 각 질의별 문서 리스트의 리스트로 반환
- Runnable 인터페이스라 동기/비동기(`ainvoke`/`abatch`) 모두 지원

## 5. 다음 단계 경계

- retriever가 반환한 문서를 프롬프트에 넣어 LLM이 답하면 generation 단계
- retriever는 체인·에이전트에 그대로 연결 가능

## 6. 참고

- Retrieval 개요: https://docs.langchain.com/oss/python/langchain/retrieval
- 시맨틱 검색(as_retriever 예시): https://docs.langchain.com/oss/python/langchain/knowledge-base
- Pinecone 통합: https://docs.langchain.com/oss/python/integrations/vectorstores/pinecone
