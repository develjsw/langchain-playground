# Chroma 벡터 스토어 정리

`Chroma`로 임베딩 저장·검색하는 방법 정리
> 전체 벡터 스토어 선택지 개요는 상위 `../vector-store.md` 참고
> 이전 단계(임베딩) 전제는 `../../embedding/embedding.md` 참고

## 1. 전제 (구성)

- 설치: `pip install -qU "langchain-chroma>=0.1.2"`
- 로컬 파일로 영속화(persist) 가능 → 재실행해도 임베딩(=비용) 다시 안 함
- 임베딩 객체(`OpenAIEmbeddings` 등)를 넘기면 Chroma가 내부에서 `embed_documents`/`embed_query`를 호출

## 2. 생성 두 방법

### 방법1: 생성 → 추가 (표준 인터페이스)

```python
from langchain_chroma import Chroma
from uuid import uuid4

vector_store = Chroma(
    collection_name='tax_docx',
    embedding_function=embedding,          # ⚠️ 생성자는 embedding_function
    persist_directory='./chroma-db',       # 영속화 폴더
)

uuids = [str(uuid4()) for _ in range(len(docs))]   # id 직접 부여 → update/delete 관리 쉬움
vector_store.add_documents(documents=docs, ids=uuids)
```

### 방법2: 한 번에 (from_documents)

```python
vector_store = Chroma.from_documents(
    documents=docs,
    embedding=embedding,                   # ⚠️ from_documents는 embedding
    collection_name='tax_docx',
    persist_directory='./chroma-db',
)
```

- 두 방법은 결과 동일 (`from_documents`가 내부에서 `add_documents` 호출)
- ⚠️ **같은 `collection_name`+`persist_directory`로 둘 다 실행하면 문서 중복 저장됨** → 하나만 실행하거나 컬렉션 이름 분리

### 저장한 DB 다시 불러오기 (재임베딩 방지)

```python
vector_store = Chroma(
    collection_name='tax_docx',
    embedding_function=embedding,
    persist_directory='./chroma-db',
)
```

## 3. CRUD (id 기반)

```python
# 수정 (단수): document_id 하나
vector_store.update_document(document_id=uuids[0], document=updated_doc)
# 수정 (복수): ids 리스트로 여러 개 한 번에
vector_store.update_documents(ids=[uuids[0]], documents=[updated_doc])
# 삭제
vector_store.delete(ids=[uuids[-1]])
```

- 여기서 `document_id`/`ids`는 **add 시점에 부여된 그 문서의 id** → 방법1에서 직접 만든 `uuids`가 그 값
- ⚠️ `add_documents`에 `ids`를 안 넘기면 Chroma가 id를 자동 생성 → 나중에 `get`으로 조회하지 않으면 알 수 없음 → `uuids`로 직접 부여하면 update/delete 관리가 쉬움 (LangChain `Chroma.add_texts` 소스 기준: `ids` 미지정 시 `uuid4` 자동 생성)

### 저장된 값 조회 (`get`)

```python
# 특정 id만
vector_store.get(ids=[uuids[0]])

# 앞 3개 미리보기
vector_store.get(limit=3)
```

- `get`은 기본적으로 `ids`/`documents`/`metadatas`를 반환 (`embeddings`만 무거워서 제외) → `include`로 따로 지정할 필요 없음, `embeddings`까지 보려면 `include=['embeddings']`
- 조회는 LangChain `Chroma`의 public 메서드 `get` 사용 (안정적인 공식 인터페이스)

## 4. 검색

```python
# 기본 검색
results = vector_store.similarity_search('거주자의 정의는?', k=3)

# 메타데이터 필터
results = vector_store.similarity_search('거주자의 정의는?', k=3, filter={'source': 'tax.docx'})

# 점수까지 (거리 기반: 작을수록 유사)
for res, score in vector_store.similarity_search_with_score('거주자의 정의는?', k=3):
    print(f'[SIM={score:3f}] {res.page_content[:80]}')
```

## 5. Retriever로 변환 (다음 단계 경계)

```python
retriever = vector_store.as_retriever(search_kwargs={'k': 3})
retriever.invoke('거주자의 정의는?')
```

- `as_retriever`부터는 **retrieval 영역** → 상세는 다음 단계에서 다룸

## 6. 주의

- `./chroma-db` 퍼시스트 폴더는 git에 올리지 말 것 → `.gitignore`에 `chroma-db/` 추가
- 생성자 `embedding_function` vs `from_documents` `embedding` 인자명 차이 주의

## 7. 참고

- Chroma 통합: https://docs.langchain.com/oss/python/integrations/vectorstores/chroma
- 벡터 스토어 공통 인터페이스: https://docs.langchain.com/oss/python/integrations/vectorstores