# Document Loader (Parsing) 개요

문서를 읽어 텍스트/구조로 파싱하는 방법 정리, LangChain은 여러 로더를 선택지로 제공
> 청킹(chunking) 내용은 `../chunking/chunking.md` 참고
> 도구별 상세는 각 하위 폴더(`unstructured/`, `docling/`) 참고

## 1. 로더 선택지

LangChain document_loaders에는 다양한 로더가 동급으로 나열됨 (특정 권장 없음)

| 로더 | 패키지 | 특징 | 참고 문서 |
|------|--------|------|-----------|
| **unstructured** | `langchain-unstructured` + `unstructured` | 요소(element) 단위 파싱, 청킹 옵션 내장 | https://docs.langchain.com/oss/python/integrations/document_loaders/unstructured_file |
| **docling** | `langchain-docling` | PDF·DOCX·PPTX·HTML 등을 레이아웃·표 포함 구조로 파싱 | https://docs.langchain.com/oss/python/integrations/document_loaders/docling |
| **upstage** | `langchain-upstage` | `UpstageDocumentParseLoader`, 레이아웃 분석 기반 파싱 | https://docs.langchain.com/oss/python/integrations/document_loaders/upstage |

- 전체 목록: https://docs.langchain.com/oss/python/integrations/document_loaders
- unstructured 상세 → `unstructured/loader.md`
- docling 상세 → `docling/loader.md`

## 2. 공통 동작 (대부분의 로더)

- `loader.load()` 는 **`List[Document]`** 반환
- 로더에 따라 기본 출력 단위가 다름 (unstructured=요소 단위, docling=`export_type`에 따름, upstage=`split`에 따름)
- `load_and_split()` 은 **Deprecated** → 표준은 `load()` 후 splitter의 `split_documents()` 사용
- 로더 상당수가 **파싱과 동시에 청킹/분할 옵션**을 제공 (→ `../chunking/chunking.md`)

## 3. 이미지 (OCR) — 공통 고려사항

- 이미지는 텍스트가 없으므로 **OCR이 필요**함
- 방법: ① OCR 엔진 **로컬 설치 + 로더** 사용, ② **국내/해외 OCR 서비스** 사용
- **품질 고려 시**:
  - **표가 포함**되어 있거나 레이아웃이 복잡하면 → 기본 OCR보다 **별도 서비스**가 더 정확함
  - **한글이 포함**되어 있으면 → **국내용 서비스**가 더 안정적
- 로더별 OCR 세부는 각 하위 문서 참고 (예: `unstructured/loader.md`)

## 4. 폴더 구조

```
loading/
├── loader.md                ← (이 문서) 전체 개요·인덱스
├── unstructured/            UnstructuredLoader 파싱
│   ├── loader.md            unstructured 전용 상세
│   ├── 01-docx.ipynb
│   ├── 02-csv.ipynb
│   ├── 03-png.ipynb
│   └── 04.pdf.ipynb
├── docling/                 DoclingLoader 파싱
│   ├── loader.md            docling 전용 상세
│   ├── 01-docx.ipynb
│   ├── 03-png.ipynb         (이미지 OCR — docling 적합 용도)
│   └── 04.pdf.ipynb
└── upstage/                 UpstageDocumentParseLoader 파싱 (클라우드 API)
    ├── loader.md            upstage 전용 상세
    ├── 03-png.ipynb
    └── 04-pdf.ipynb
```

> ※ CSV 등 이미 구조화된 데이터엔 docling·upstage 부적합 → pandas 등 경량 도구 권장

## 5. 참고 문서

- LangChain Document Loader 문서: https://docs.langchain.com/oss/python/integrations/document_loaders
