# 0921_StandardForm — 생산공장 모델 STEP2 (Standard Form · Basis)

> 의사결정2 · 홍익대 2026-2 · 류지환
> 브랜치 주제: **부등식을 등식으로 — Slack / Surplus / Artificial Variable 과 Standard Form `Az=b`**
> 직전 브랜치: [`0919_SCIP`](../../tree/0919_SCIP) (STEP1 · Feasibility)

## 이 브랜치에 있는 것

| 파일 | 내용 |
|---|---|
| `STEP2_류지환의생산공장.ipynb` | **이번 주제** — Standard Form 과 Basis |
| `STEP1_류지환의생산공장.ipynb` | 직전 주제 (SCIP Feasibility). 브랜치 규칙상 함께 들고 간다 |
| `requirements.txt` | 두 노트북을 돌리기 위한 최소 의존성 |
| `README.md` | 이 문서 |

## STEP2 가 다루는 것

STEP1 은 Solver 에게 "가능한가?" 를 물었고 `INFEASIBLE` 이라는 답을 받았다.
STEP2 는 **Solver 안쪽**으로 들어간다 — Simplex 가 왜 부등식을 등식으로 바꾸는지,
그 과정에서 생기는 변수들이 각각 무슨 뜻인지를 손으로 계산하고 그림으로 확인한다.

노트북이 스스로 던진 질문 6개:

| # | 질문 | 상태 |
|---|---|---|
| 1 | 왜 부등식을 등식으로 바꾸는가? | ✅ 2-1 |
| 2 | Slack / Surplus 는 무엇인가? | ✅ 2-1 · 2-2 |
| 3 | Artificial Variable 은 왜 필요한가? | ✅ 2-3 |
| 4 | Basis 란 무엇인가? | ✅ 2-4 · 2-5 |
| 5 | Basic Solution 은 어떻게 계산하는가? | ✅ 2-5 · 2-6 |
| 6 | Basic Feasible Solution 과 그래프의 꼭짓점은 왜 같은가? | ✅ 2-6 · 2-7 |

마지막에 **STEP 2 Reconstruction** 마크다운으로 Slack → Surplus → Artificial → Basis → Basic Solution → BFS → Geometry 를 한 번에 정리했다.

확인한 수치:

| 절 | 내용 | 결과 |
|---|---|---|
| 2-1 Slack | 샘플 계획 (P0, P3) = (1, 2) 의 Human Time | 사용 62h → `s_H = 18`, `6x+28y+s_H = 80` 성립 |
| 2-2 Surplus | 같은 계획의 Profit | 실제 220 → `s_P = 10`, `20x+100y-s_P = 210` 성립 |
| 2-3 Artificial | Phase I 직관 — `max(0, 210 - profit)` 등고선 | Profit 목표선 바깥쪽에서만 `a_P > 0` |
| 2-4 Standard Form | 계수행렬 | `A.shape = (3, 6)` · `rank(A) = 3` |
| 2-5 Basis | 초기 Basis `B₀ = [A_sH, A_sT, A_aP] = I` | `x_B = b = (80, 100, 210)` · `A z₀ = b` 성립 |
| 2-6 전수조사 | `C(6,3) = 20` 개 column 조합 → rank 검사 → `B x_B = b` → `x_B ≥ 0` | rank 3 인 Basis **16개** (4개는 특이) · Phase-I BFS **6개** · 원 문제 BFS (`a_P = 0`) **5개** |
| 2-6 꼭짓점 | 원 문제 BFS 5개를 (x, y) 평면에 | V1 (6.67, 1.43) · V2 (10.5, 0) · V3 (13.33, 0) · V4 (0, 2.1) · V5 (0, 2.38) = feasible 다각형의 꼭짓점 5개 |
| 2-7 Linear Combination | Basis `[A_x, A_y, A_sP]` | `x_B = (6.67, 1.43, 66.19)` → 세 column 기여의 합이 `b = (80, 100, 210)` 과 일치 · 3D 화살표 이어붙이기 그림 |

Phase-I BFS 6개 중 원 문제 BFS 가 아닌 하나는 초기 Basis `(s_H, s_T, a_P)` 다 — 여기서는 `a_P = 210`.
나머지 5개는 전부 `a_P = 0` 이므로 Phase I 의 `min a_P` 가 0 까지 내려갈 수 있고, 즉 이 교보재 인스턴스는 feasible 하다.

핵심은 **`Az = z₁A₁ + ... + z₆A₆`** — A 의 column 들을 변수 값만큼 섞어 `b` 를 만드는 문제로
읽는 시점부터 선형대수와 OR 이 같은 물건이 된다는 것.

### ⚠ STEP1 과 모델이 달라진 점 — 읽고 넘어갈 것

STEP2 는 **그림을 그리기 위해** 모델을 두 군데 바꿨다. 같은 사업 이야기지만 같은 문제 인스턴스가 아니다.

| 항목 | STEP1 | STEP2 | 이유 |
|---|---|---|---|
| 결정변수 | P0~P3 4개 · **정수** | P0, P3 2개 (`x`, `y`) · **연속** | 2차원이어야 feasible 영역을 평면에 그릴 수 있다. Simplex 의 기하를 보는 것이 목적이므로 정수 제약도 잠시 푼다 |
| Human Capacity | ≤ **60** h | ≤ **80** h | ↓ |
| AI Token Capacity | ≤ **65** U | ≤ **100** U | ↓ |
| Profit 목표 | ≥ 210 만원 | ≥ 210 만원 (그대로) | |
| 공헌이익 · 소요량 | (20, 6, 6) / (100, 28, 42) | 동일 | |

**상한을 올린 이유:** 원래 상한(60 / 65)을 그대로 두고 P0·P3 만 남기면,
이 2변수 모델의 최대 이익은 꼭짓점 (x, y) = (8.33, 0.357) 에서 **202.38 만원** 이다.
목표 210 에 못 미치므로 세 제약을 모두 만족하는 영역이 **빈 집합**이 되어 그릴 그림이 없다.
(정수로 풀면 (10, 0) 에서 200 만원 — 역시 미달.)
상한을 80 / 100 으로 열면 최대 이익이 **276.19 만원** 이 되어 목표선 안쪽에 면적이 생긴다.

즉 **STEP1 의 `INFEASIBLE` 과 STEP2 의 feasible 영역은 모순이 아니다.**
STEP2 는 "이 사업이 가능한가" 를 다시 판정하는 것이 아니라,
Simplex 의 기계장치를 눈으로 보기 위해 일부러 여유를 준 **교보재 인스턴스**다.
원래 문제의 판정은 STEP1 의 `INFEASIBLE` 과 "나머지를 다 지키면 이익 상한 160 만원" 이 그대로 유효하다.

## 난이도 단계 — 어디까지 왔나

| Level | 질문 | 어디서 |
|---|---|---|
| 1 — Feasibility | 모든 목표를 동시에 만족할 수 있는가? | ✅ STEP1 (`0919_SCIP`) — INFEASIBLE |
| — 기계장치 | Solver 는 그 답을 **어떻게** 내는가? | ✅ **STEP2 (이 브랜치)** — Standard Form · Basis · BFS = 꼭짓점 |
| 2 — Goal Programming | 전부 만족할 수 없다면 무엇부터 지킬 것인가? | ⬜ 미착수 |
| 3 — Resource Sensitivity | 인간시간 또는 Token 을 줄이면 결과가 어떻게 변하는가? | ⬜ 미착수 |

STEP2 는 Level 1 → 2 사이에 끼워 넣은 단계다. Level 2 (Goal Programming) 는 아직 숙제로 남아 있다.

## 실행 방법 (다른 로컬에서 새로 시작할 때)

```bash
git clone -b 0921_StandardForm https://github.com/Siegfriex/DC2_ProblemSolver.git
cd DC2_ProblemSolver

python3 -m venv .venv                 # Python 3.12 권장 (3.10 이상)
source .venv/bin/activate             # Windows: .venv\Scripts\activate
pip install -r requirements.txt

jupyter lab STEP2_류지환의생산공장.ipynb
```

`uv` 를 쓴다면:

```bash
uv venv --python 3.12
uv pip install -r requirements.txt
```

### 커널 관련 메모

노트북의 `kernelspec` 은 이식성을 위해 일부러 표준 이름인 **`python3`** 로 두었다.
방금 만든 venv 를 주피터에 커널로 등록하려면:

```bash
python -m ipykernel install --user --name dc2-scip --display-name "Python (DC2 SCIP)"
```

VS Code 에서 열 경우엔 등록 없이 우측 상단에서 `.venv` 인터프리터를 고르면 된다.

> 원 저자의 로컬 환경(`hongik_univ_26_2` 모노레포)에서는 `hongik_26_2` 커널을 쓴다.
> 그 커널 이름은 그 머신에만 존재하므로 이 저장소에는 반영하지 않았다.

### 그래프의 한글

STEP2 의 그림 라벨은 전부 영문이므로 별도 폰트 설정 없이 어느 OS 에서나 깨지지 않는다.
(`axes.unicode_minus = False` 만 설정 — 마이너스 기호가 □ 로 나오는 것을 막는다.)

## 검증 환경

아래 조합에서 `jupyter nbconvert --execute` 로 **전 셀 위에서 아래로 재실행 · 오류 0 · 그림 6장 정상 생성**
을 확인했다 (2026-09-26, 2-7 까지 포함).
확인된 주요 결과: `s_H = 18` · `s_P = 10` · `A.shape = (3, 6)` · `rank(A) = 3` · rank-3 Basis 16개 · Phase-I BFS 6개 · 원 문제 BFS 5개.

| 항목 | 버전 |
|---|---|
| OS | WSL2 · Ubuntu 24.04 |
| Python | 3.12.3 |
| ortools | 9.10.4067 |
| numpy | 2.4.6 |
| pandas | 3.0.3 |
| matplotlib | 3.11.0 |
| sympy | 1.14.0 |
| IPython | 9.14.1 |
| ipykernel | 7.3.0 |
| jupyterlab | 4.6.0 |

주의할 점:

- **pandas 3.x** 는 copy-on-write 와 string dtype 이 기본이다. 2.x 관용구(체이닝 대입 등)는 동작이 다르다.
  이 노트북은 `DataFrame` 생성과 표시만 하므로 2.2 이상이면 무방하다.
- **SCIP** 는 ortools 휠에 포함되어 있어 별도 설치가 필요 없다.
- 노트북에 저장된 `execution_count` 는 저자가 학습하며 셀을 오가던 순서 그대로다.
  숫자가 연속이 아닌 것은 정상이며, 위에서 아래로 한 번에 돌려도 같은 결과가 나오는 것은 위에서 확인했다.

### 아직 다듬지 않은 것 (다음 커밋 예정)

이 커밋도 **공부한 지점까지를 그대로** 남기는 것이 목적이라 노트북 본문은 손대지 않았다. 계산 결과에는 영향이 없는 것들이다.

- `print(md(...))` 여러 곳에서 출력 아래에 `None` 이 한 줄씩 찍힌다 — `md()` 가 이미 `display` 를 하므로 `print` 는 불필요
- 2-4 뒤 마크다운의 `Z ... A ... = B ...` 수식 블록이 깨져 있다
- 2-6 전수조사 셀 마지막 `md(...)` 문자열의 `\boxed`·`\rightarrow` 가 일반 문자열이라 `SyntaxWarning: invalid escape sequence` 가 뜨고 수식이 렌더되지 않는다 (`r"..."` 로 바꾸면 해결)
- 2-7 의 3D 시각화 셀 아래쪽에 바로 앞 셀(Basis 선택 · contribution 표) 코드가 한 번 더 붙어 있다 — 같은 표가 두 번 출력된다
- 오탈자: `oringinal_bfs_df` · `Humena Equation` · `FEASIBL`
- `sympy` 는 여전히 import 만 되어 있다

## 브랜치 규칙

이 저장소는 **주제 하나 = 브랜치 하나 = 노트북 하나** 로 운영한다.

| 브랜치 | 주제 | 노트북 |
|---|---|---|
| `main` | (LICENSE 만) | — |
| `0919_SCIP` | SCIP 정수계획 Feasibility | `STEP1_류지환의생산공장.ipynb` |
| `0921_StandardForm` | Standard Form · Slack/Surplus/Artificial · Basis | `STEP2_류지환의생산공장.ipynb` (+ STEP1) |

새 주제를 시작할 때:

```bash
# ⚠ main 이 아니라 '직전 주제 브랜치' 에서 딴다
git checkout -b <날짜>_<주제> 0921_StandardForm
```

1. 위처럼 **직전 주제 브랜치에서** 새 브랜치를 만든다
2. `.gitignore` 의 화이트리스트에 새 노트북 파일명 한 줄 추가
3. 그 주제의 `requirements.txt` 와 이 `README.md` 를 주제에 맞게 갱신
4. 커밋 · 푸시

> **왜 `main` 에서 따면 안 되는가**
> `main` 에는 `LICENSE` 밖에 없다. `git checkout main` 을 하는 순간 git 은
> 작업 디렉터리를 `main` 의 내용으로 맞추므로 — 이 저장소는 로컬 `DC2/` 폴더 그 자체다 —
> **작업 중이던 `.ipynb` 와 `.gitignore` 가 디스크에서 사라진다.**
> 커밋돼 있으니 되돌릴 수는 있지만, 그 사이 `.gitignore` 가 없는 상태가 되어
> 다른 주차 과제 노트북들이 전부 커밋 후보로 노출된다. 직전 브랜치에서 따면 둘 다 안 생긴다.
> (그 대가로 새 브랜치는 이전 주제 노트북들을 함께 들고 간다. 학습 기록이 누적되는 쪽이
>  "다른 로컬에서 클론해서 바로 본다" 는 목적에 오히려 맞다.)

## 라이선스

`LICENSE` 참조.
