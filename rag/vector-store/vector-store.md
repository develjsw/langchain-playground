# 벡터 스토어(Vector Store) 개요

임베딩된 벡터를 저장하고 유사도로 검색하는 단계 정리, LangChain은 여러 DB를 동일 인터페이스로 제공
> 이전 단계(임베딩)는 `../embedding/embedding.md` 참고
> 다음 단계(검색·리트리버)는 `../retrieval/` (예정)
> 도구별 상세는 각 하위 폴더(`chroma/` 등) 참고

## 1. 인터페이스

모든 벡터 스토어는 아래 메서드로 통일됨 → DB만 갈아끼우면 됨

| 분류 | 메서드 | 동작 |
|------|--------|------|
| 추가 | `add_documents(documents, ids)` | 문서를 임베딩해서 저장 (내부에서 `embed_documents` 호출) |
| 수정 | `update_documents(ids, documents)` | id 기준으로 덮어쓰기 |
| 삭제 | `delete(ids)` | id 기준으로 제거 |
| 검색 | `similarity_search(query, k)` | 가까운 청크 k개 반환 (내부에서 `embed_query` 호출) |
| 검색 | `similarity_search_with_score(query, k)` | 유사도 점수까지 함께 반환 |
| 변환 | `as_retriever()` | 체인에 꽂을 Retriever로 변환 (→ 여기서 retrieval 단계로 경계) |

- 임베딩을 직접 호출할 필요 없음 → 벡터 스토어가 내부에서 대신 호출함 (`../embedding/embedding.md` 18번 줄 참고)
- `from_documents(documents, embedding, ...)`는 "생성 + add_documents"를 한 줄로 합친 편의 메서드 (deprecated 아님)

## 2. 두 가지 생성 방식

| 방식 | 코드 | 언제 |
|------|------|------|
| **생성 → 추가** | `vector_store = Chroma(...)` → `vector_store.add_documents(docs, ids)` | 표준 인터페이스, 나중에 추가·삭제·id 관리할 때 |
| **한 번에** | `vector_store = Chroma.from_documents(docs, embedding, ...)` | 한 방에 다 넣고 시작할 때 (간단) |

- 공식 개념 문서는 표준 인터페이스(`add_documents`)를 우선 설명, 통합 문서는 `from_documents`도 함께 제공
- ⚠️ 생성자 인자명은 `embedding_function`, `from_documents` 인자명은 `embedding` (헷갈리기 쉬움)

## 3. providers 선택지

| providers | 클래스 | 패키지 | 특징 |
|--------|--------|--------|------|
| InMemory | `InMemoryVectorStore` | `langchain-core` | 메모리 전용, 학습·테스트용 (재실행 시 사라짐) |
| Chroma | `Chroma` | `langchain-chroma` | 로컬 퍼시스트(파일 저장), 프로토타입 표준 |
| FAISS | `FAISS` | `langchain-community` | Meta 라이브러리, 로컬·고속 (※ community 아카이브 주의) |
| Pinecone·Qdrant·Milvus·PGVector | 각 클래스 | 각 패키지 | 서버형, 운영·대규모 |

- 전체 providers 목록: https://docs.langchain.com/oss/python/integrations/vectorstores
- Chroma 상세 → `chroma/vector-store.md`

## 4. 유사도 점수 읽는 법

- `similarity_search_with_score`가 주는 값은 **거리(distance)** → **작을수록 더 유사**
- Chroma 기본 거리 척도는 `l2`로 알려져 있고, `collection_metadata`로 `cosine` 등 변경 가능

## 5. 폴더 구조

```
vector-store/
├── vector-store.md            ← (이 문서) 전체 개요·인덱스
└── chroma/                    Chroma 벡터 스토어
    ├── vector-store.md        chroma 전용 상세
    └── 01-docx-chroma.ipynb
```

## 6. 참고 문서

- LangChain 벡터 스토어 통합: https://docs.langchain.com/oss/python/integrations/vectorstores
- Chroma 통합: https://docs.langchain.com/oss/python/integrations/vectorstores/chroma