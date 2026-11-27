# 문서 유형별 검색 RAG

대학 안내, 웹페이지, 규정 PDF의 구조를 보존해 검색하고 답변 근거를 표시하는 RAG 코드입니다.

## 문제와 구현

목록형 질문을 단순 top-k 검색으로 처리하면 항목이 누락될 수 있습니다. 학과·문서 유형 메타데이터로 검색 범위를 제한하고, 목록형 질문은 조건에 맞는 항목을 모아 응답합니다. 규정 PDF는 조항 경계와 원문 페이지를 보존하며, 근거가 약할 때 응답을 보류합니다.

Chroma 검색과 로컬 Ollama 모델을 사용합니다. Neo4j 입시 그래프는 로컬 실행을 전제로 하며, 일반 문서 검색과 별도로 구성합니다. 실제 문서·벡터DB·대화 기록은 포함하지 않았습니다.

## 구성

- `apps/sch/`: 학과·입시·논문 검색, 문서 적재, 로컬 그래프 화면
- `src/document_rag/`: PDF·웹 문서 정제, 조항 검색, 근거 검증
- `apps/employment/`: 규정 PDF 검색 화면
- `crawl/`, `configs/`: Playwright 수집과 대상 설정
- `tests/`: 검색 범위, 중복, 목록 누락, 오류 처리 회귀 테스트

## 실행 준비

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt
pip install -e . --no-deps
cp .env.example .env
cp apps/sch/.env.example apps/sch/.env
python -m playwright install chromium
pytest -q
streamlit run apps/sch/app_streamlit.py
```

모델과 원문은 별도로 준비해야 합니다. 기본 로컬 모델은 `gemma3:4b`, 임베딩은 `jhgan/ko-sroberta-multitask`이며 환경변수로 변경할 수 있습니다. 원문 적재 스크립트는 `apps/sch/ingest/`, 범용 CLI는 `python -m document_rag.cli --help`에서 확인합니다. Neo4j 사용 시 로컬 주소와 계정 정보를 환경변수로 설정합니다.

## 검증 상태와 한계

기존 브랜치의 검색·목록 처리 회귀 테스트를 포함했습니다. 이번 배포 준비에서 전체 LLM·Neo4j·브라우저 흐름을 다시 실행하지 않았습니다. 코드를 받는 것만으로 이전 데이터의 검색 결과가 복원되지는 않습니다.

검색 성공률과 답변 정확성은 별도로 측정해야 합니다. 개선 전후 비교에는 같은 문서 인덱스·질문·정답 기준을 사용하고, 평가 질문 수와 실패 사례를 함께 공개해야 합니다. 사내 문서에서 파생된 질문·답변·스크린샷은 이 배포에 포함하지 않았습니다.
