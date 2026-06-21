# Upstage 로딩(파싱) 정리

`UpstageDocumentParseLoader`로 문서를 파싱하는 내용 정리
> 전체 로더 선택지 개요는 상위 `../loader.md` 참고
> 청킹은 `../../chunking/chunking.md` 참고

## 1. 구성 (클라우드 API)

| 항목 | 내용 |
|------|------|
| 패키지 | `langchain-upstage` 의 `UpstageDocumentParseLoader` |
| 동작 방식 | **Upstage 클라우드 API 호출** (로컬 모델 X) → 인터넷 필요 |
| 인증 | API 키 필요 (`UPSTAGE_API_KEY` 환경변수 또는 생성자 인자) |

- unstructured·docling이 **로컬 처리**인 것과 달리, Upstage는 **문서를 서버로 전송**해 파싱 → 보안·네트워크 고려 필요

## 2. 지원 형식 / 제약

| 구분 | 내용 |
|-----|------|
| 지원 형식 | JPEG · PNG · BMP · PDF · TIFF · HEIC · DOCX · PPTX · XLSX |
| 미지원 | **CSV** 등 → `415 Unsupported Media Type` (CSV는 pandas 등으로 처리) |
| 페이지 한도 | 동기 요청 **최대 100페이지** (초과 시 `413`) |
| 파일 크기 | 최대 50MB |

- 실측: `tax.docx`(144페이지)는 한도 초과로 `413` / `hospital.csv`는 미지원으로 `415`
- 출처: [Upstage Document Parse 공식 문서](https://console.upstage.ai/docs/capabilities/document-digitization/document-parsing) · API 에러 메시지

## 3. 출력 단위 (`split`)

- `split` 옵션(`none`/`page`/`element`)은 **출력 단위(청킹) 관련** → `../../chunking/upstage/chunking.md` 참고

## 4. 참고 문서

- LangChain Upstage 통합: https://docs.langchain.com/oss/python/integrations/document_loaders/upstage
- Upstage Document Parse: https://console.upstage.ai/docs/capabilities/document-digitization/document-parsing
