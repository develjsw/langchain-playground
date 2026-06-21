# Document Chunking 정리

`chunking/` 폴더에서 테스트한 청킹(chunking) 내용을 정리한 문서
> 로딩(파싱)·설치 등 전제 내용은 `../loading/loader.md` 참고

## 1. 전제 (구성)

- `langchain-unstructured`(어댑터) + `unstructured`(엔진) 조합으로 처리 (상세 → `loader.md`)
- 청킹은 별도 splitter 없이 **`UnstructuredLoader`의 옵션**으로 수행
- 확장자별 부분 설치 필요 ex) `pip install -qU "unstructured[docx]"`

## 2. 청킹 전략

| 전략 | 동작 | 적합한 문서 |
|------|------|-------------|
| `basic` | 구조 무시, 크기(`max_characters`) 채워서 자르기 | 구조 없는 줄글 (에세이·로그·자막) |
| `by_title` | 제목(섹션) 만나면 새 청크 시작 | 구조 있는 문서 (법령·계약서·논문·매뉴얼) |

### 전략 분류 + 도구 매핑

알려진 청킹 전략은 크게 5가지, unstructured는 그중 2가지(`basic`·`by_title`)만 제공하고 나머지는 LangChain 공식 문서의 splitter를 참고

**분류별 설명**

- **고정 길이** : 글자 수나 토큰 수 기준(예: 500자)으로 무조건 자르는 방식
- **재귀** : 단락(`\n\n`) → 줄바꿈(`\n`) → 공백 순서로 큰 단위부터 시도하며 크기가 맞을 때까지 작게 잘라가는 방식
- **문서 구조** : 마크다운 헤더(`#`, `##`)·HTML 태그·목차·표(Table) 등 문서 구조를 기준으로 나눔
- **의미 기반** : 임베딩 모델로 문맥을 분석해 주제·의미가 바뀌는 지점(Breakpoint)을 찾아 분할
- **에이전틱/LLM** : LLM이 문서 전체를 읽고 핵심 명제(Proposition)나 의도에 맞춰 직접 분할 지점을 결정

**도구 매핑**

| 분류 | unstructured 청킹 전략 옵션 | 대표 도구 (그 외) |
|------|------|------|
| 고정 길이 | △ `basic` (오픈소스, 요소를 크기로 채움·순수 고정길이 X) | LangChain `CharacterTextSplitter`<br>https://docs.langchain.com/oss/python/integrations/splitters/character_text_splitter |
| 재귀 | 없음 | LangChain `RecursiveCharacterTextSplitter`<br>https://docs.langchain.com/oss/python/integrations/splitters/recursive_text_splitter |
| 문서 구조 | `by_title`(섹션, 오픈소스) · `by_page`(페이지, 상용) | LangChain Markdown·HTML·JSON splitter<br>https://docs.langchain.com/oss/python/integrations/splitters |
| 의미 기반 | `by_similarity`(임베딩 유사도, 상용) | LangChain `SemanticChunker` (`langchain_experimental`)<br>- |
| 에이전틱/LLM | 없음 | **공식 구현 없음** → 직접 LLM 호출로 구현<br>- |

- → unstructured는 **문서 구조 기반에 강함**(`by_title`·`by_page`), 의미 기반은 상용 전용, 재귀·에이전틱은 미제공
- 출처(unstructured): [오픈소스 chunking](https://docs.unstructured.io/open-source/core-functionality/chunking)
- ※ `by_page`·`by_similarity`는 unstructured **상용 Platform/API 전용**(무료 15,000페이지 + 초과분 과금), 오픈소스 라이브러리엔 없음
- ※ `SemanticChunker`는 `langchain_experimental.text_splitter`에 실재(deprecated 아님, GitHub 이슈 #35553에서 확인)하나, 새 레퍼런스 사이트 개편으로 **개별 클래스의 안정적 공식 URL이 현재 없음**(딥링크가 검색 랜딩으로 리다이렉트됨)

## 3. 청킹 옵션

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
    '../files/pdf/medical-law-2026-04.pdf',
    chunking_strategy='by_title',
    max_characters=1000,
    new_after_n_chars=800,
    combine_text_under_n_chars=200,
    overlap=150,
    include_orig_elements=False,
    languages=['kor'],
)
```
## 4. 참고 문서

- `unstructured` 청킹 옵션: https://docs.unstructured.io/open-source/core-functionality/chunking

## 5. 참고 내용

- 청크 크기 가이드 (RAG best practices): https://unstructured.io/blog/chunking-for-rag-best-practices
  - **약 250 토큰(약 1,000자) 정도의 청크 크기가 실험을 시작하기에 적절한 지점**이라고 표현됨
