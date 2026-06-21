# Document Loader (Parsing + Chunking) 정리

`loading/` 폴더에서 테스트한 문서 로딩/청킹 내용을 정리한 문서

## 1. 구성 (어댑터 + 엔진)

| 패키지 | 역할 |
|--------|------|
| `langchain-unstructured` | **어댑터(인터페이스)** — LangChain의 `UnstructuredLoader` 제공 |
| `unstructured` | **실제 파싱 엔진** — 문서를 읽고 element로 분해 |

- `unstructured` 전체를 받으면 의존성이 무거우므로, **필요한 확장자만** 설치
- `"unstructured[확장자]"` 형태로 부분 설치

```bash
# 예시: CSV만 필요할 때
pip install -qU langchain langchain-unstructured "unstructured[csv]"
# pdf → "unstructured[pdf]", docx → "unstructured[docx]" ...
```

## 2. 기본 동작

- `UnstructuredLoader(...).load()` 는 **항상 `List[Document]`** 반환 (파일 종류 무관)
- 기본은 문서를 **의미 요소(element) 단위로 분해**하므로, 청킹 없이도 `len(docs)`가 매우 클 수 있음 ex) PDF 2304개
- `load_and_split()` 은 **Deprecated** → 사용하지 말 것
- 청킹은 `unstructured`가 옵션으로 지원하므로, 별도 splitter 없이 로더 옵션으로 처리

## 3. 청킹 전략

| 전략 | 동작 | 적합한 문서 |
|------|------|-------------|
| `basic` | 구조 무시, 크기(`max_characters`) 채워서 자르기 | 구조 없는 줄글 (에세이·로그·자막) |
| `by_title` | 제목(섹션) 만나면 새 청크 시작 | 구조 있는 문서 (법령·계약서·논문·매뉴얼) |

## 4. 청킹 옵션

| 옵션 | 의미 | 비고 |
|------|------|------|
| `max_characters` | 청크 최대 글자 수 (**하드 리밋**, 절대 못 넘음) | 우선순위 가장 높음 |
| `new_after_n_chars` | 이 글자 수 넘으면 다음 경계에서 미리 끊음 (**소프트 리밋**) | 보통 `max`의 70~80% |
| `combine_text_under_n_chars` | 이보다 짧은 청크는 다음 것과 합침 | 잘게 쪼개짐·노이즈 방지 |
| `overlap` | 겹칠 글자 수 | **기본은 큰 element가 강제 분할될 때만** 적용 |
| `overlap_all` | `True`면 **모든 인접 청크 경계**에 overlap 적용 | 중복↑·표 등 부작용 주의 |
| `include_orig_elements` | 합쳐지기 전 원본 요소를 메타데이터로 보존 | `False`면 경량화 |
| `languages` | 파싱 언어 지정 (예: `['kor']`) | 한글 문서 정확도↑ |

> **우선순위**: `max_characters`(하드) > `by_title`(의미 경계) > `new_after_n_chars`(소프트) > 먼저 걸리는 조건에서 끊되, 하드 리밋은 절대 못 넘음

### 추천 예시 (구조 있는 한글 PDF, RAG 용도)

```python
loader = UnstructuredLoader(
    'files/medical-law-2026-04.pdf',
    chunking_strategy='by_title',
    max_characters=1000,
    new_after_n_chars=800,
    combine_text_under_n_chars=200,
    overlap=150,
    include_orig_elements=False,
    languages=['kor'],
)
```

## 5. 이미지 (OCR)

- 이미지는 텍스트가 없으므로 **OCR이 필요**함
- 방법: ① OCR 엔진 **로컬 설치 + unstructured** 사용, ② **국내/해외 OCR 서비스** 사용
- **품질 고려 시**:
  - **표가 포함**되어 있거나 레이아웃이 복잡하면 → unstructured 기본 OCR보다 **별도 서비스**가 더 정확함
  - **한글이 포함**되어 있으면 → **국내용 서비스**가 더 안정적

## 6. 참고 문서

- LangChain `UnstructuredLoader` 사용법: https://docs.langchain.com/oss/python/integrations/document_loaders/unstructured_file
- `unstructured` 청킹 옵션: https://docs.unstructured.io/open-source/core-functionality/chunking
- LangChain Document Loader 공식문서: https://docs.langchain.com/oss/python/integrations/document_loaders

## 7. 참고 내용

- 청크 크기 가이드 (RAG best practices): https://unstructured.io/blog/chunking-for-rag-best-practices
  - **약 250 토큰(약 1,000자) 정도의 청크 크기가 실험을 시작하기에 적절한 지점**이라고 표현됨