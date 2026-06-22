# 임베딩(Embedding) 개요

텍스트를 벡터로 변환하는 방법 정리, LangChain은 여러 제공자를 동일 인터페이스로 제공
> 이전 단계(청킹)는 `../chunking/chunking.md` 참고
> 다음 단계(벡터 저장·검색)는 `../vector-store/vector-store.md` 참고
> 도구별 상세는 각 하위 폴더(`openai/` 등) 참고

## 1. 인터페이스 (공식)

모든 임베딩 제공자는 두 메서드로 통일됨 → 제공자만 갈아끼우면 됨

| 메서드 | 동작 |
|--------|------|
| `embed_documents(texts)` | 문서 **여러 개**를 벡터 리스트로 (저장용) |
| `embed_query(text)` | 쿼리 **1개**를 벡터로 (검색 입력용) |

- 일부 제공자는 문서용/쿼리용 처리를 다르게 함 → 그래서 메서드가 둘로 나뉨
- 벡터스토어·리트리버가 내부에서 이 두 메서드를 호출함 (→ 04~05 단계)

## 2. 모델 선택 기준 (공식)

- **품질**: [MTEB 리더보드](https://huggingface.co/spaces/mteb/leaderboard)로 비교 후, **자기 데이터로 테스트**하고 결정
- **배포 패턴 (모델을 어디서 돌리나)**
  - ① **클라우드 API** — 키만 있으면 바로 사용 가능 (유료) : OpenAI·Cohere·Google·Voyage·Upstage
  - ② **로컬** — 무료 모델을 내 컴퓨터에 받아서 직접 실행 : HuggingFace·Ollama·BGE 등
  - ③·④ 도메인 특화 fine-tune / 대규모 self-host (고급, 학습 단계에선 생략 가능)
- **고려 요소**: 비용, 지연(클라우드는 +50~200ms), 차원(384~3072+), 컨텍스트 길이(512~8192), **다국어 지원**

## 3. 제공자 선택지

| 제공자 | 클래스 | 패키지 | 특징 |
|--------|--------|--------|------|
| OpenAI | `OpenAIEmbeddings` | `langchain-openai` | 클라우드, 표준 비교 기준 |
| Upstage | `UpstageEmbeddings` | `langchain-upstage` | 클라우드 API, 한글 문서에 활용 |
| HuggingFace | `HuggingFaceEmbeddings` | `langchain-huggingface` | 로컬·무료, 파이썬이 모델 받아 실행 |
| Ollama | `OllamaEmbeddings` | `langchain-ollama` | 로컬·무료, Ollama 런타임이 모델 실행 |

- 공식 목록(40+ 제공자): https://docs.langchain.com/oss/python/integrations/embeddings
- OpenAI 상세 → `openai/`

## 4. OpenAI 사용 예 (공식)

```python
from langchain_openai import OpenAIEmbeddings

embedding = OpenAIEmbeddings(
    model="text-embedding-3-large",   # dimensions=1024 로 차원 축소 가능
)

single_vector = embedding.embed_query("거주자의 정의는?")     # 쿼리 1개
two_vectors = embedding.embed_documents([t1, t2])             # 문서 여러 개

print(len(single_vector))   # 3072 (text-embedding-3-large 기본 차원)
```

- 설치: `pip install -qU langchain-openai`, 환경변수 `OPENAI_API_KEY` 필요

## 5. 폴더 구조

```
embedding/
├── embedding.md            ← (이 문서) 전체 개요·인덱스
└── openai/                OpenAIEmbeddings
    └── 01-docx-embedding.ipynb
```

## 6. 참고 문서

- LangChain 임베딩 통합: https://docs.langchain.com/oss/python/integrations/embeddings
- OpenAI 임베딩: https://docs.langchain.com/oss/python/integrations/embeddings/openai
