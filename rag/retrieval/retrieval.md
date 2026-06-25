# 리트리벌(Retrieval) 개요

질의에 맞는 문서를 골라오는 단계 정리, 벡터 스토어의 기본 검색 위에서 검색 전략을 다룸
> 이전 단계(벡터 스토어)는 `../vector-store/vector-store.md` 참고
> 도구별 상세는 각 하위 폴더(`chroma/` 등) 참고

## 1. Retriever란

- 질의(query)를 받아 관련 문서 리스트를 반환하는 공통 인터페이스 → `retriever.invoke(query)`
- 벡터 스토어를 `as_retriever()`로 변환해 만들거나, 벡터와 무관한 retriever(BM25 등)도 동일 인터페이스로 사용
- 체인·에이전트에 그대로 꽂을 수 있음 (다음 단계 generation으로 연결)

## 2. 벡터 스토어 → Retriever

```python
retriever = vector_store.as_retriever(search_kwargs={'k': 3})
retriever.invoke('거주자의 정의는?')
```

- `search_type`으로 검색 방식 선택: `similarity`(기본), `mmr`, `similarity_score_threshold`
- `search_kwargs`로 세부 조정: `k`(개수), `filter`(메타데이터), `score_threshold` 등

## 3. 검색 전략 갈래

| 전략 | 핵심 | 언제 |
|------|------|------|
| Similarity | 질의와 가까운 벡터 k개 | 기본 |
| MMR (Maximal Marginal Relevance) | 유사도 + 다양성 → 중복 줄임 | 비슷한 청크가 많을 때 |
| Multi-Query | 질의를 여러 변형으로 확장해 검색 | 질의 표현이 모호할 때 |
| Contextual Compression | 검색 후 관련 부분만 추려 압축 | 노이즈 많은 긴 청크 |
| Ensemble | 여러 retriever 결과 결합 (예: BM25 + 벡터) | 키워드+의미 둘 다 필요 |
| Parent Document | 작은 청크로 검색, 큰 원본 반환 | 검색 정확도와 맥락 둘 다 |

## 4. 폴더 구조

```
retrieval/
├── retrieval.md              ← (이 문서) 전체 개요·인덱스
└── chroma/                   Chroma 기반 retriever 실습
    └── retrieval.md          chroma 전용 상세
```

## 5. 참고 문서

- LangChain Retrieval: https://docs.langchain.com/oss/python/langchain/retrieval
- Retriever 통합 목록: https://docs.langchain.com/oss/python/integrations/retrievers
