# 청킹(Chunking) 개요

문서를 청크로 나누는 방법 정리, LangChain 공식 문서 기준으로 선택지가 여러 갈래로 나뉨
> 로딩(파싱)·설치 전제는 `../loading/loader.md` 참고
> 도구별 상세는 각 하위 폴더(`unstructured/`, `docling/`, `recursive-character-text-splitter/`) 참고

## 1. 두 갈래 (공식 문서 기준)

LangChain은 청킹을 크게 두 카테고리로 제공

| 갈래 | 무엇 | 공식 문서 |
|------|------|-----------|
| **로더 내장 청킹** | 로더가 파싱하면서 옵션으로 청킹/분할까지 수행 | https://docs.langchain.com/oss/python/integrations/document_loaders |
| **독립 splitter** | 이미 로드된 텍스트를 별도로 잘라줌 | https://docs.langchain.com/oss/python/integrations/splitters/recursive_text_splitter |

→ document_loaders에 나오는 docling·unstructured·upstage 등은 **로더 자체에 청킹 옵션 포함**, recursive 계열은 **독립 splitter**

## 2. 로더 내장 청킹 (docling · unstructured · upstage 등)

| 로더 | 내장 옵션 | 동작 | 공식 문서 |
|------|-----------|------|-----------|
| **unstructured** | `chunking_strategy` (`basic`·`by_title`) | 요소를 크기로 묶거나 섹션 경계로 분할 | https://docs.unstructured.io/open-source/core-functionality/chunking |
| **docling** | `export_type` (`DOC_CHUNKS`·`MARKDOWN`) + `chunker`(예: `HybridChunker`) | 구조·토큰 기반 청킹 또는 통째 마크다운 | https://docs.langchain.com/oss/python/integrations/document_loaders/docling |
| **upstage** | `split` (`none`·`page`·`element`) | 분할 안 함 / 페이지 / 요소 단위 (※ 크기 기반 청킹이 아니라 **구조 단위 분할**) | https://docs.langchain.com/oss/python/integrations/document_loaders/upstage |

- 상세: unstructured → `unstructured/chunking.md` · docling → `docling/chunking.md` · upstage → `upstage/chunking.md`

## 3. 독립 splitter — `RecursiveCharacterTextSplitter`

- **분류상 "재귀"**: 단락(`\n\n`) → 줄바꿈(`\n`) → 공백 순서로 큰 단위부터 시도하며 크기가 맞을 때까지 작게 잘라가는 방식
- 패키지: `langchain-text-splitters` (현역, sunset 아님)
- 사용 패턴: `loader.load()` → `splitter.split_documents(docs)` (`load_and_split`은 deprecated이므로 안 씀)

### 예전엔 docx2txt와 자주 썼지만 지금은 아님

- 과거: `langchain-community`의 `Docx2txtLoader` + `RecursiveCharacterTextSplitter` 조합이 흔했음
- 그러나 **`langchain-community`가 2026-06-19 아카이브(sunset)** → 신규 의존 비권장
  - 출처: [Sunsetting langchain-community (issue #674)](https://github.com/langchain-ai/langchain-community/issues/674)
- ∴ recursive는 이제 **다른 현역 로더와 함께** 써야 함

### Loader별 recursive 조합법

- **docling (권장)** : `export_type=ExportType.MARKDOWN`으로 전체를 1 Document로 받은 뒤 splitter 적용
  ```python
  from langchain_docling import DoclingLoader
  from langchain_docling.loader import ExportType
  from langchain_text_splitters import RecursiveCharacterTextSplitter

  loader = DoclingLoader(
      file_path='../files/doc/tax.docx',
      export_type=ExportType.MARKDOWN,   # 통째로 받기 (기본 DOC_CHUNKS는 자동 청킹됨)
  )
  docs = loader.load()
  chunks = RecursiveCharacterTextSplitter(
      chunk_size=1000, chunk_overlap=100,
  ).split_documents(docs)
  ```
- **unstructured (우회, 비권장)** : `mode` 미지원이라 `chunking_strategy="basic"` + `max_characters`=거대값 + `include_orig_elements=False`로 한 덩이로 받아야 함 → 부자연스러움
  - 출처: [LangChain Unstructured 통합](https://docs.langchain.com/oss/python/integrations/document_loaders/unstructured_file)
- **docx2txt 직접 호출** : langchain 래퍼 없이 `docx2txt.process()` → `Document` 직접 생성 → splitter (community 의존 0)
  ```python
  import docx2txt
  from langchain_core.documents import Document
  from langchain_text_splitters import RecursiveCharacterTextSplitter

  text = docx2txt.process('../files/doc/tax.docx')
  docs = [Document(page_content=text, metadata={'source': 'tax.docx'})]
  chunks = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=100).split_documents(docs)
  ```

## 4. 청킹 전략 5분류 (일반)

알려진 청킹 전략은 크게 5가지, 도구마다 제공 범위가 다름

- **고정 길이** : 글자 수나 토큰 수 기준(예: 500자)으로 무조건 자르는 방식
- **재귀** : 단락 → 줄바꿈 → 공백 순서로 큰 단위부터 시도하며 크기가 맞을 때까지 작게 잘라가는 방식
- **문서 구조** : 마크다운 헤더(`#`, `##`)·HTML 태그·목차·표(Table) 등 문서 구조를 기준으로 나눔
- **의미 기반** : 임베딩 모델로 문맥을 분석해 주제·의미가 바뀌는 지점(Breakpoint)을 찾아 분할
- **에이전틱/LLM** : LLM이 문서 전체를 읽고 핵심 명제(Proposition)나 의도에 맞춰 직접 분할 지점을 결정

| 분류 | unstructured 청킹 전략 옵션 | 대표 도구 (그 외) |
|------|------|------|
| 고정 길이 | △ `basic` (오픈소스, 요소를 크기로 채움·순수 고정길이 X) | LangChain `CharacterTextSplitter`<br>https://docs.langchain.com/oss/python/integrations/splitters/character_text_splitter |
| 재귀 | 없음 | LangChain `RecursiveCharacterTextSplitter`<br>https://docs.langchain.com/oss/python/integrations/splitters/recursive_text_splitter |
| 문서 구조 | `by_title`(섹션, 오픈소스) · `by_page`(페이지, 상용) | LangChain Markdown·HTML·JSON splitter<br>https://docs.langchain.com/oss/python/integrations/splitters |
| 의미 기반 | `by_similarity`(임베딩 유사도, 상용) | LangChain `SemanticChunker` (`langchain_experimental`)<br>- |
| 에이전틱/LLM | 없음 | **공식 구현 없음** → 직접 LLM 호출로 구현<br>- |

- ※ `by_page`·`by_similarity`는 unstructured **상용 Platform/API 전용**(무료 15,000페이지 + 초과분 과금), 오픈소스 라이브러리엔 없음
- ※ `SemanticChunker`는 `langchain_experimental.text_splitter`에 실재(deprecated 아님, GitHub 이슈 #35553에서 확인)하나, 새 레퍼런스 사이트 개편으로 **개별 클래스의 안정적 공식 URL이 현재 없음**(딥링크가 검색 랜딩으로 리다이렉트됨)

## 5. 폴더 구조

```
chunking/
├── chunking.md                          ← (이 문서) 전체 개요·인덱스
├── unstructured/                        UnstructuredLoader 청킹 (basic/by_title)
│   ├── chunking.md                      unstructured 전용 상세
│   ├── 01-docx-chunking.ipynb
│   ├── 02-csv-chunking.ipynb
│   └── 03.pdf-chunking.ipynb
├── docling/                             DoclingLoader 청킹 (export_type)
│   ├── chunking.md                      docling 전용 상세
│   └── 01-docx-chunking.ipynb
├── upstage/                             UpstageDocumentParseLoader 청킹 (split)
│   └── chunking.md                      upstage 전용 상세
└── recursive-character-text-splitter/   RecursiveCharacterTextSplitter 예제
    └── 01-docx-chunking.ipynb
```
