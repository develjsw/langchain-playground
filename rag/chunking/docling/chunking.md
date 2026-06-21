# Docling 청킹 정리

`DoclingLoader`의 출력 모드(`export_type`) 정리
> 전체 청킹 선택지 개요는 상위 `../chunking.md` 참고
> 로딩(파싱) 전제는 `../../loading/docling/loader.md` 참고

## `export_type` (출력 모드)

| 모드 | 동작 |
|------|------|
| `DOC_CHUNKS` (기본) | 내장 청커로 청크를 나눠서 반환 |
| `MARKDOWN` | 문서 전체를 1개 Document(마크다운)로 반환 → 외부 splitter와 조합 |

- 크기 기반으로 자르려면 `MARKDOWN`으로 받은 뒤 `RecursiveCharacterTextSplitter` 등과 조합 (→ `../chunking.md`)
- 출처: [LangChain Docling 통합](https://docs.langchain.com/oss/python/integrations/document_loaders/docling)
