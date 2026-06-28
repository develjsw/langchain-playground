# Pinecone 벡터 스토어 정리

`Pinecone`(서버형 벡터 DB)으로 임베딩 저장·검색하는 방법 정리
> 전체 벡터 스토어 선택지 개요는 상위 `../vector-store.md` 참고
> 이전 단계(임베딩) 전제는 `../../embedding/embedding.md` 참고

## 1. 전제 (구성)

- 설치: `pip install -qU langchain-pinecone langchain-openai`
- 서버형 → 로컬 파일이 아니라 Pinecone 클라우드 인덱스에 저장 (퍼시스트 폴더 없음)
- `PINECONE_API_KEY` 환경변수 필요 (`.env` 권장)
- ⚠️ 이름이 겹치는 `Pinecone` 두개 → 패키지로 구분

| import | 패키지 | 용도 |
|--------|--------|------|
| `from pinecone import Pinecone` | `pinecone` (Pinecone SDK) | 인증·인덱스 생성/관리 클라이언트 |
| `from langchain_pinecone import PineconeVectorStore` | `langchain-pinecone` | 벡터 스토어 (이걸 사용) |

- `langchain_pinecone`의 구 `Pinecone`은 `PineconeVectorStore`의 **deprecated 별칭** → `PineconeVectorStore` 사용

## 2. 인덱스 준비 (Pinecone SDK)

```python
from pinecone import Pinecone, ServerlessSpec

pc = Pinecone()                    # PINECONE_API_KEY 환경변수 자동 사용

index_name = 'tax-docx-index'

if not pc.has_index(index_name):
    pc.create_index(
        name=index_name,
        dimension=3072,            # ⚠️ 임베딩 모델 차원과 일치시켜야 함
        metric='cosine',
        spec=ServerlessSpec(cloud='aws', region='us-east-1'),
    )

index = pc.Index(index_name)
```

- ⚠️ `dimension`은 임베딩 모델 출력 차원과 반드시 일치 → `text-embedding-3-large`는 3072, `text-embedding-3-small`은 1536
- `metric`은 보통 `cosine` (유클리드 `euclidean`, 내적 `dotproduct`도 가능)
- `ServerlessSpec`의 `cloud`/`region`은 사용 가능한 값으로 지정

## 3. 벡터 스토어 생성 + 추가

```python
from langchain_pinecone import PineconeVectorStore
from uuid import uuid4

vector_store = PineconeVectorStore(index=index, embedding=embedding)

uuids = [str(uuid4()) for _ in range(len(docs))]   # id 직접 부여 → delete/덮어쓰기 관리 쉬움
vector_store.add_documents(documents=docs, ids=uuids)
```

- 생성자 인자명은 `embedding` (Chroma 생성자의 `embedding_function`과 다름 → 헷갈리기 쉬움)
- 미리 만든 `index` 객체를 넘김 → 벡터 스토어가 내부에서 `embed_documents`/`embed_query` 호출

## 4. 검색

```python
# 기본 검색
results = vector_store.similarity_search('거주자의 정의는?', k=3)

# 점수까지 (cosine 유사도: 클수록 유사)
for res, score in vector_store.similarity_search_with_score('거주자의 정의는?', k=3):
    print(f'[SIM={score:3f}] {res.page_content[:80]}')
```

- ⚠️ 점수 해석은 `metric`에 따라 다름 → Chroma 기본(`l2`, 제곱 유클리드)은 거리(작을수록 유사), Pinecone `cosine`은 Pinecone 기준 높을수록 유사

## 5. 삭제 (delete)

```python
vector_store.delete(ids=[uuids[-1]])
```

- `ids`는 add 시점에 부여한 그 문서의 id → 직접 만든 `uuids`가 그 값

## 6. 다음 단계 (Retrieval)

- `as_retriever`로 retriever 변환부터는 **retrieval 영역** → `../../retrieval/pinecone/retrieval.md` 참고

## 7. 참고

- Pinecone 통합: https://docs.langchain.com/oss/python/integrations/vectorstores/pinecone
- 벡터 스토어 공통 인터페이스: https://docs.langchain.com/oss/python/integrations/vectorstores
