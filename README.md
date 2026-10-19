# Stepwise Word Removal & Preprocessing Strategies in MIMIC-IV Clinical Notes (KSC 2025)

한국정보과학회 **KSC2025 학부생 논문 경진대회 장려상** · 포스터 발표 (여수, 2025.12)
저자: 이진아 (제1저자, 숭실대 AI소프트웨어학부) · 윤진혁 교수 (교신)

## Problem
퇴원 요약지(Discharge Summary)에는 예측 시점 이후의 정보(사망 시각, 퇴원 처분 등)가 섞여 있어, 전체 텍스트를 그대로 임베딩하면 모델이 `expired`, `deceased` 같은 결과론적 단어와 타겟 사이의 **허위 상관(spurious correlation)** 을 학습한다. 이 연구는 텍스트에서 단어·섹션을 단계적으로 제거하면서 성능과 Top 피처가 어떻게 바뀌는지 비교해, 데이터 유출을 막는 전처리 가이드라인을 제시한다.

## Data
- MIMIC-IV v2.2 `mimic-iv-note` Discharge Summary **329,940건** + `mimiciv/hosp/admissions.csv`
- PhysioNet credentialed access 필요 — **이 레포에는 데이터가 포함되어 있지 않다** (DUA). 노트북의 경로를 본인 환경에 맞게 바꿔 실행.

## Method
| 실험 | 전처리 | 설명 |
|---|---|---|
| A (Baseline) | 전체 텍스트 | 기본 정제만 수행 |
| B (Section Removal) | 미래 정보 섹션 제거 | `Hospital Course`, 퇴원 처분 등 결과 섹션 전체 제거 |
| C (White-list + Filtering) | 입원 초기 섹션만 추출 | `Chief Complaint`, `HPI`, `Past Medical History`, `Physical Exam` 정규식 추출 + "Patient died" 류 결과 문장 규칙 필터링 |

- 임베딩: **Bio_ClinicalBERT** (`emilyalsentzer/Bio_ClinicalBERT`), 512 토큰 초과 시 510-토큰 chunk로 나눠 `[CLS]` 벡터 평균(mean pooling, chunk 수 상한 4)
- 분류기: XGBoost (`scale_pos_weight`로 불균형 보정) / Logistic Regression, Top-K feature importance 비교
- 정형 데이터(Late Fusion) 결합 실험 포함

## Results (논문·포스터 기준)
| 단계 | AUROC | Top 피처 |
|---|---|---|
| A 전체 텍스트 | 0.9995 | expired, deceased, died, time of death |
| B 섹션 제거 | 0.9727 | unresponsive, failure, **comfort**(완화의료 — 암시적 유출), pleural |
| C White-list 추출 + 문장 필터링 | 0.9552 | **hypotension, intubated**, admission, history |

B에서도 `comfort care` 같은 단어가 상위에 남아 **암시적 유출(implicit leakage)** 이 있음을 확인했고, C에서야 임상적으로 타당한 피처가 상위에 올라왔다. 성능이 떨어지는 것이 아니라 "모델이 무엇을 보고 판단하는지"가 바뀐 것이 핵심 결과.

### 추가 실험: TF-IDF + Logistic Regression / Random Forest (`notebooks/03`)

같은 사망 타겟으로 임베딩 대신 TF-IDF(1–2gram, 5,000 피처)를 써서 A/B 전처리를 비교한 실험도 함께 수행했다.
(퇴원 요약지가 있는 입원 331,732건, train 265,385 / test 66,347)

| 단계 | LR AUROC | RF AUROC |
|---|---|---|
| A 전체 텍스트 | 0.9995 | 0.9994 |
| B 결과 관련 섹션(Hospital Course, Discharge *, Follow-up) 이후 제거 | 0.9727 | 0.9429 |

표현 방식(TF-IDF vs Bio_ClinicalBERT)과 무관하게 전체 텍스트에서는 AUROC가 사실상 1에 가깝고, 결과 섹션을 지우면 떨어진다 —
전처리 단계가 유출을 얼마나 막는지가 성능 수치를 좌우한다는 같은 결론.

## Repository layout
```
notebooks/
  01_label_tfidf_rf_baseline.ipynb      # EDA, 30일 재입원 라벨 생성, TF-IDF(50) + RandomForest 베이스라인, Hospital Course 섹션 추출
  02_whitelist_clinicalbert_xgb.ipynb   # White-list 섹션 추출 → Bio_ClinicalBERT chunk 임베딩 → LR / XGBoost, 정형 데이터 Late Fusion
  03_mortality_tfidf_lr_expA_expB.ipynb # 사망 타겟 TF-IDF + LR / RF, Exp A(전체) vs Exp B(결과 섹션 제거) — 연구실 GPU 서버 실행본
docs/
  KSC2025_poster.png                    # 포스터
  KSC2025_paper_draft.pdf               # 논문 (초안 버전; 최종 제출본은 '학부생 논문_숭실대_이진아.pdf')
```
노트북은 **출력 셀을 모두 비운 상태**로 올렸다 (MIMIC 텍스트 샘플이 출력에 포함되기 때문). 절대 경로는 `data/physionet.org/files/...` 상대 경로로 바꿔 두었다.

## How to run
```bash
pip install pandas numpy scikit-learn xgboost torch transformers tqdm matplotlib seaborn
# PhysioNet에서 mimic-iv-note 2.2 / mimiciv 3.1(hosp/admissions.csv) 를 받아 data/physionet.org/files/ 아래에 둔다
# Bio_ClinicalBERT 를 ./bio_clinicalbert 에 내려받거나 model_path 를 HF id 로 바꾼다
jupyter lab notebooks/
```

## My role
연구 설계·전처리 전략(A/B/C) 설계·실험·포스터/논문 작성 전체 (단독 학부생 저자, 지도: 윤진혁 교수).

## Note
- 같은 MIMIC-IV를 쓴 **심부전 30일 재입원 예측 팀프로젝트**는 별도 레포([mimic-iv-hf-readmission](https://github.com/Binary-Ah/mimic-iv-hf-readmission))이며 이 연구와 다른 작업이다.
