# Docling 로딩(파싱) 정리

`DoclingLoader`로 문서를 파싱하는 내용 정리
> 전체 로더 선택지 개요는 상위 `../loader.md` 참고
> 청킹은 `../../chunking/chunking.md` 참고

## 1. 적합 / 부적합 (용도)

| 구분 | 형식 |
|----|------|
| 적합 | PDF · 이미지(PNG 등) · DOCX · PPTX |
| 부적합 | CSV · Excel 등 이미 구조화된 데이터 |

- **적합한 이유**: 레이아웃·표·이미지가 섞여 있어, 구조를 분석해 파싱하는 docling의 강점이 살아남
- **부적합한 이유**: 이미 표로 정리된 데이터는 docling이 오히려 무겁게 처리해 비효율 → pandas 등 경량 도구가 더 빠르고 적합
- 실측: 4.1MB·8296행짜리 CSV를 docling으로 돌렸더니 약 700만 토큰으로 부풀어 **27분 넘게 안 끝나고 메모리도 폭증** (직접 확인)

## 2. 형식별 처리 경로

docling은 **형식마다 다른 backend·pipeline**을 사용함 (같은 docling이라도 내부 동작이 다름)

- **DOCX·PPTX·HTML·MD·CSV** → 파서 라이브러리로 **텍스트를 바로 추출** (모델 없이 빠름)
- **PDF·이미지** → 레이아웃·표·OCR **모델 파이프라인**을 태움 → 모델 다운로드 필요, **메모리도 무거움**
  - 관측: DOCX는 모델 없이 바로 처리되고, PDF·이미지는 모델을 받음 / 대용량 PDF에서 메모리 부족(`std::bad_alloc`) 발생
- 출처: [Docling architecture](https://docling-project.github.io/docling/concepts/architecture/) (형식별 backend·pipeline) · [pipeline options](https://docling-project.github.io/docling/reference/pipeline_options/) (PDF의 `do_ocr`·레이아웃 옵션)

## 3. OCR (PDF·이미지에서만)

- **OCR을 타는 형식**: PNG 등 이미지 · PDF / **안 타는 형식**: DOCX·CSV 등 (CSV는 OCR 불필요, 텍스트를 바로 읽음) — 근거는 §2 형식별 처리 경로
- 사용 가능한 엔진: EasyOCR · Tesseract · RapidOCR · Mac OCR
- ⚠️ **지금 설치된 환경에서는 RapidOCR이 쓰임** (실행 로그의 `ch_PP-OCRv4`로 확인) — RapidOCR은 Baidu PaddleOCR(중국) 기반이라 **한글에는 부적절**
- 한글 문서라면 언어를 바꿔야 함 → `pipeline_options.ocr_options.lang = ["ko"]` 로 지정, 또는 텍스트 PDF면 `do_ocr = False`로 OCR 자체를 끄기
- 출처: [Docling FAQ](https://docling-project.github.io/docling/faq/) (엔진 목록·`ocr_options.lang` 설정) · [pipeline options](https://docling-project.github.io/docling/reference/pipeline_options/) (`do_ocr` 등 OCR 옵션)

## 4. Windows 주의 (HF 모델 다운로드 권한)

- 첫 실행 때 HF에서 모델을 받는데, Windows에서는 심볼릭 링크 권한 부족으로 `WinError 1314`가 날 수 있음
- 해결 방법 (셋 중 하나)
  - ① 개발자 모드 켜기 (권장)
  - ② 관리자 권한으로 실행
  - ③ `os.environ["HF_HUB_DISABLE_SYMLINKS"] = "1"` (import 전에, 커널 재시작 후)
- 출처: [HF Hub 환경변수](https://huggingface.co/docs/huggingface_hub/package_reference/environment_variables)

## 5. 출력 모드 (`export_type`)

- `export_type`(`DOC_CHUNKS`/`MARKDOWN`)은 **출력 단위(청킹) 관련** → `../../chunking/docling/chunking.md` 참고

## 6. 참고 문서

- LangChain Docling 통합: https://docs.langchain.com/oss/python/integrations/document_loaders/docling
- Docling 지원 포맷: https://docling-project.github.io/docling/usage/supported_formats/
