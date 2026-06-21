# Upstage 청킹 정리

`UpstageDocumentParseLoader`의 출력 단위(`split`) 옵션 정리
> 전체 청킹 선택지 개요는 상위 `../chunking.md` 참고
> 로딩(파싱)·제약 등 전제 내용은 `../../loading/upstage/loader.md` 참고

## `split` 옵션 (출력 단위)

| 값 | 동작 |
|------|------|
| `none` (기본) | 문서 전체를 1개 Document로 |
| `page` | 페이지 단위로 분할 |
| `element` | 요소(문단·표·그림 등) 단위로 분할 |

- ※ 크기 기반 청킹이 아니라 **구조 단위 분할**임
- 크기 기반으로 자르려면 `split` 결과를 `RecursiveCharacterTextSplitter` 등 외부 splitter와 조합 (→ `../chunking.md`)
- 출처: [LangChain Upstage 통합](https://docs.langchain.com/oss/python/integrations/document_loaders/upstage)
