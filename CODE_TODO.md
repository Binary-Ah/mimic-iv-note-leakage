# CODE_TODO — mimic-iv-note-leakage

윈도우 PC에서 찾은 코드는 **파이프라인 구성 요소**(White-list 정규식 추출, ClinicalBERT chunk 임베딩, XGBoost/LR, 30일 재입원 라벨링)이고, 타겟이 30일 재입원인 버전이다.
논문 최종 표(A/B/C, AUROC 0.9995 / 0.9727 / 0.9553, 사망 타겟)를 만든 실행 스크립트는 여기 없다.

- [ ] 연구실 GPU 서버 `/data/jinagg/medi/` 에서 `medi.py`(노트북 주석에 언급) 및 Exp A/B/C 실행 스크립트를 가져와 `src/` 로 추가
  - 02 노트북 CONFIG 주석: `"model_path": "./bio_clinicalbert"  # 모델 폴더는 medi.py와 같은 위치`
- [ ] "Patient died" 류 결과 문장 규칙 필터링(Exp C 두 번째 단계) 코드 확인 — 현재 노트북에는 섹션 White-list만 있음
- [ ] Top-K feature importance 그래프(포스터 Exp B/C 막대그래프) 생성 코드 추가
- [ ] 맥북 `/Users/jina/Documents/무제 폴더/학부생 논문_숭실대_이진아.pdf` (저자 순서 이진아, 윤진혁* 최종본)로 `docs/KSC2025_paper_draft.pdf` 교체
- [ ] KSC2025 장려상 **상장 파일** — 윈도우/맥 어디에도 없음. 학회 메일 또는 지도교수님께 요청
- [ ] 커밋 전 `jupyter nbconvert --clear-output --inplace notebooks/*.ipynb` 로 출력 비어 있는지 재확인 (MIMIC 텍스트 출력 금지)
