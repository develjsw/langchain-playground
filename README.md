# LangChain PlayGround

> 정해진 규칙 없이 자유롭게 추가하며 실험하고 기록하는 공간

## Deprecated
LangChain의 패키지 통합 및 분리 작업으로 인해 Deprecated 되거나 Archive 되는 경우가 빈번히 발생하므로 최신 패키지 및 클래스/메서드 사용을 위해 기재

- `langchain-community` 패키지는 아카이브됨 ( 2026년 5월 26일 )
  - 공식저장소: https://github.com/langchain-ai/langchain-community
  - 대체할 패키지들로 변경해서 사용
    - EX) `langchain-community`에서 사용하던 Docx2txtLoader 부분을 `langchain-unstructured`에서 UnstructuredLoader로 대체
- `langchain-core`패키지의 `load_and_split` 메서드 Deprecated됨  
  - 공식문서: https://reference.langchain.com/python/langchain-core/document_loaders/base/BaseLoader/load_and_split

## 학습 기록
- RAG: (Loader → Chunking → Embedding → Vector Store → Retrieval → Generation)
  - Loader: [loading/loader.md](rag/loading/loader.md)
  - Chunking: [chunking/chunking.md](rag/chunking/chunking.md)
  - Embedding: [embedding/embedding.md](rag/embedding/embedding.md)
  - Vector Store: [vector-store/vector-store.md](rag/vector-store/vector-store.md)
 

- Graph RAG: ---- Next task ----