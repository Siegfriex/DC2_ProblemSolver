# 1001_GoalProgramming — Goal Programming (연습문제 9-4 현대가구)

> 의사결정2 · 홍익대 2026-2 · 류지환
> 브랜치 주제: **모든 목표를 동시에 만족할 수 없을 때, 어떤 목표를 얼마나 희생할 것인가 — Preemptive Goal Programming**
> 직전 브랜치: [`0926_FreeVariable`](../../tree/0926_FreeVariable) (STEP3 · Free Variable · Rank · Dimension)

## 이 브랜치에 있는 것

| 파일 | 내용 |
|---|---|
| `W03_Exercise_9_4_Goal_Programming_DeepDive.ipynb` | **이번 주제** — 연습문제 9-4 를 Goal Programming 으로 푼다 |
| `STEP3_류지환의생산공장.ipynb` | 직전 주제 (Free Variable · Rank · Dimension). 브랜치 규칙상 함께 들고 간다 |
| `STEP2_류지환의생산공장.ipynb` | 그 전 주제 (Standard Form · Basis · BFS) |
| `STEP1_류지환의생산공장.ipynb` | 그 전 주제 (SCIP Feasibility) |
| `requirements.txt` | 노트북들을 돌리기 위한 최소 의존성 (이번에 새로 추가된 패키지 없음) |
| `README.md` | 이 문서 |

## 이번 노트북이 다루는 것

> ⚠ 이 노트북은 생산공장 모델이 아니라 **교재 연습문제 9-4 (현대가구)** 다.
> Goal Programming 기법을 작은 문제에서 먼저 익히는 단계이고, 생산공장 STEP1 의 INFEASIBLE 에 적용하는 것은 아직 하지 않았다.

| 항목 | 책상 1 | 책상 2 |
|---|---:|---:|
| 작업시간 | 2시간 | 1시간 |
| 단위이익 | 2만원 | 3만원 |
| 최소 수요 | 280개 | 400개 |

목표 (우선순위 순): P1 정규 작업시간 800시간 초과 금지 → P2 책상 1·2 수요 충족 → P3 총이익 1500만원.

흐름: 문제 구조 → 결정변수 → Goal 행렬 `G` (ℝ² → ℝ⁴) → 수요만 맞춰 보기 → 전부 Hard Constraint 로 강제 (INFEASIBLE) → 2D 그림 →
Deviation Variable `Gx + d⁻ − d⁺ = t` → 편차 계산 함수 → **Priority 1 → 2 → 3** → 결과 그림.

핵심은 **다음 Priority 로 넘어갈 때 `x` 가 아니라 앞 단계의 최적 성과(`d₁⁺ ≤ 0` 등)를 고정한다**는 것이다.

확인한 수치:

| 절 | 내용 | 결과 |
|---|---|---|
| 4 | 수요만 정확히 맞춘 계획 `(280, 400)` | 작업시간 960 → 초과 160시간 · 이익 1760 |
| 5 | 네 목표를 전부 Hard Constraint 로 (SCIP) | `INFEASIBLE` — `2·280 + 400 = 960 > 800` |
| 10 | P1 `min d₁⁺` | **0** → `2x₁ + x₂ ≤ 800` 이 남는다 |
| 11 | P2 `min d₂⁻ + d₃⁻` (단 `d₁⁺ ≤ 0`) | **80** at `(200, 400)` — 같은 시간이면 책상 2 가 부족분을 2배 빨리 줄인다. 비교: `(280, 240)` 은 160 |
| 12 | P3 `min d₄⁻` (단 P1 · P2 성과 유지) | **0** — 해가 `(200, 400)` 하나뿐이고 이익 1600 ≥ 1500 |

**최종 해: 책상 1 = 200, 책상 2 = 400 · 작업시간 800 · 초과작업 0 · 책상 1 수요 80개 미달 · 이익 1600만원.**
희생된 목표는 책상 1 수요 하나뿐이다. 최종 해는 solver 의 편차 값이 아니라 `evaluate_plan(200, 400)` 으로 다시 계산해 대조했다
(목적함수에 없는 편차는 solver 가 임의 값을 낼 수 있기 때문).

### 원본에서 고친 것

- `build_goal_model()` 의 Goal 1 식 부호: `+ d_plus[1] − d_minus[1]` → `+ d_minus[1] − d_plus[1]`.
  마크다운 식 `2x₁ + x₂ + d₁⁻ − d₁⁺ = 800` 과 Goal 2~4 의 형태에 맞춘 것이다. 고치기 전에는 `d_plus[1]` 이 "남는 시간" 을 뜻해서,
  P1 이 초과작업이 아니라 남는 시간을 줄였고, P2 · P3 는 `x ≈ 1e20` 의 무의미한 해를 냈다
- 함수 끝에 남아 있던 `sp;` 를 지우고 `return solver, x1, x2, d_plus, d_minus` 를 넣었다
- 빈 마지막 셀 자리에 10 ~ 12절 (P1 · P2 · P3 셀 · 결과 표 · 그림) 과 13절 Part (2) 자리를 추가했다

## 난이도 단계 — 어디까지 왔나

| Level | 질문 | 어디서 |
|---|---|---|
| 1 — Feasibility | 모든 목표를 동시에 만족할 수 있는가? | ✅ STEP1 (`0919_SCIP`) — INFEASIBLE |
| — 기계장치 | Solver 는 그 답을 **어떻게** 내는가? | ✅ STEP2 (`0921_StandardForm`) — Standard Form · Basis · BFS = 꼭짓점 |
| — 개념 정리 | "자유변수" · "차원" 은 각각 무엇을 말하는가? | ✅ STEP3 (`0926_FreeVariable`) |
| 2 — Goal Programming | 전부 만족할 수 없다면 무엇부터 지킬 것인가? | 🔶 **이 브랜치** — 기법을 연습문제 9-4 로 학습 (Preemptive 완료 · Weighted 미작성). 생산공장 모델 적용은 미착수 |
| 3 — Resource Sensitivity | 인간시간 또는 Token 을 줄이면 결과가 어떻게 변하는가? | ⬜ 미착수 |

## 실행 방법 (다른 로컬에서 새로 시작할 때)

```bash
git clone -b 1001_GoalProgramming https://github.com/Siegfriex/DC2_ProblemSolver.git
cd DC2_ProblemSolver

python3 -m venv .venv                 # Python 3.12 권장 (3.10 이상)
source .venv/bin/activate             # Windows: .venv\Scripts\activate
pip install -r requirements.txt

jupyter lab W03_Exercise_9_4_Goal_Programming_DeepDive.ipynb
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

STEP2 · STEP3 · W03 의 그림 라벨은 전부 영문이므로 별도 폰트 설정 없이 어느 OS 에서나 깨지지 않는다.
(`axes.unicode_minus = False` 만 설정 — 마이너스 기호가 □ 로 나오는 것을 막는다.)

## 검증 환경

아래 조합에서 `jupyter nbconvert --execute` 로 W03 노트북을 **전 셀 위에서 아래로 재실행 · 오류 0 · 그림 2장 정상 생성**
을 확인했다 (2026-10-03). 노트북 안의 `assert` 가 `(x₁, x₂) = (200, 400)`, P1 = 0, P2 = 80, P3 = 0 을 검사한다.
STEP1 ~ 3 은 이전 브랜치에서 같은 환경으로 검증했고 이 브랜치에서 내용이 바뀌지 않았다.

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

- **pandas 3.x** 는 copy-on-write 와 string dtype 이 기본이다. 이 노트북들은 `DataFrame` 생성과 표시만 하므로 2.2 이상이면 무방하다.
- **SCIP** 는 ortools 휠에 포함되어 있어 별도 설치가 필요 없다.
- W03 은 STEP 노트북의 변수를 쓰지 않고 단독으로 돈다.
- Priority 단계마다 `build_goal_model()` 로 모형을 새로 만들고, 앞 단계 성과는 `≤ 최적값 + 1e-6` 제약으로 넘긴다.

### 아직 하지 않은 것 · 다듬지 않은 것

- **13절 Part (2) Weighted Goal Programming** — 원문 (2)의 벌금비용을 확인한 뒤 작성 (자리와 식만 있음)
- 학습 흐름의 Part C (대수 ↔ Matrix ↔ Geometry ↔ Python) · Part E (Reconstruction) 는 아직 별도 절이 없다
- 첫 셀의 `HUMAN_CAP` · `TOKEN_CAP` · `PROFIT_TARGET` 은 STEP 노트북에서 복사된 값으로 이 문제에서는 쓰지 않는다. `combinations` · `sympy` 는 import 만
- 7절 주석 "x 의 shape 는 2,1 → 결과 4,1" — 실제 NumPy shape 는 `(2,)` → `(4,)`
- 오탈자 — "나쁜 편자"(2곳) · "책상 1 생산량 S" · "GoalPrograming"
- STEP3 · STEP2: 이전 브랜치 README 의 '아직 다듬지 않은 것' 목록 그대로

## 브랜치 규칙

이 저장소는 **주제 하나 = 브랜치 하나 = 노트북 하나** 로 운영한다.

| 브랜치 | 주제 | 노트북 |
|---|---|---|
| `main` | (LICENSE 만) | — |
| `0919_SCIP` | SCIP 정수계획 Feasibility | `STEP1_류지환의생산공장.ipynb` |
| `0921_StandardForm` | Standard Form · Slack/Surplus/Artificial · Basis · BFS | `STEP2_류지환의생산공장.ipynb` (+ STEP1) |
| `0926_FreeVariable` | Free Variable · Rank · Dimension · Unrestricted · Nonbasic | `STEP3_류지환의생산공장.ipynb` (+ STEP1 · STEP2) |
| `1001_GoalProgramming` | Preemptive Goal Programming (연습문제 9-4) | `W03_Exercise_9_4_Goal_Programming_DeepDive.ipynb` (+ STEP1 ~ 3) |

새 주제를 시작할 때:

```bash
# ⚠ main 이 아니라 '직전 주제 브랜치' 에서 딴다
git checkout -b <날짜>_<주제> 1001_GoalProgramming
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
