# Document Loader (Parsing) 정리

`loading/` 폴더에서 테스트한 문서 로딩(파싱) 내용을 정리한 문서
> 청킹(chunking) 내용은 `../chunking/chunking.md` 참고

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
- 청킹은 `unstructured`가 옵션으로 지원하므로, 별도 splitter 없이 로더 옵션으로 처리 (→ `chunking.md`)

## 3. 이미지 (OCR)

- 이미지는 텍스트가 없으므로 **OCR이 필요**함
- 방법: ① OCR 엔진 **로컬 설치 + unstructured** 사용, ② **국내/해외 OCR 서비스** 사용
- **품질 고려 시**:
  - **표가 포함**되어 있거나 레이아웃이 복잡하면 → unstructured 기본 OCR보다 **별도 서비스**가 더 정확함
  - **한글이 포함**되어 있으면 → **국내용 서비스**가 더 안정적

## 4. 참고 문서

- LangChain `UnstructuredLoader` 사용법: https://docs.langchain.com/oss/python/integrations/document_loaders/unstructured_file
- LangChain Document Loader 공식문서: https://docs.langchain.com/oss/python/integrations/document_loaders
