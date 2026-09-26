# 0926_FreeVariable — 생산공장 모델 STEP3 (Free Variable · Rank · Dimension)

> 의사결정2 · 홍익대 2026-2 · 류지환
> 브랜치 주제: **"자유변수" 라는 말 아래 섞여 있는 세 가지 개념을 분리한다 — RREF Free Variable · LP Unrestricted Variable · Simplex Nonbasic Variable**
> 직전 브랜치: [`0921_StandardForm`](../../tree/0921_StandardForm) (STEP2 · Standard Form · Basis)

## 이 브랜치에 있는 것

| 파일 | 내용 |
|---|---|
| `STEP3_류지환의생산공장.ipynb` | **이번 주제** — Free Variable · Rank · Dimension |
| `STEP2_류지환의생산공장.ipynb` | 직전 주제 (Standard Form · Basis · BFS = 꼭짓점). 브랜치 규칙상 함께 들고 간다 |
| `STEP1_류지환의생산공장.ipynb` | 그 전 주제 (SCIP Feasibility) |
| `requirements.txt` | 세 노트북을 돌리기 위한 최소 의존성 (STEP3 에서 새로 추가된 패키지 없음) |
| `README.md` | 이 문서 |

## STEP3 가 다루는 것

STEP2 에서 Basis 를 고르면 나머지 변수를 0 으로 둔다고 했다. 그런데 선형대수에서는 "Pivot 이 없는 변수는 **자유롭게** 움직인다" 고 배운다.
둘 다 "free" 처럼 들리지만 정반대의 이야기다. STEP3 는 이 혼동을 세 개념으로 쪼개고, 각각을 계산과 그림으로 확인한다.

| 개념 | 등장 장소 | 뜻 | 노트북 |
|---|---|---|---|
| RREF Free Variable | 선형대수 | Pivot 이 없어서 해집합의 자유도를 만드는 변수 | 3-1 · 3-2 |
| Unrestricted Variable | LP Modeling | 부호 제한이 없는 변수 — `u = u⁺ − u⁻` 로 분해 | 3-3 |
| Nonbasic Variable | Simplex | 현재 Basis 밖이라 Basic Solution 계산에서 **0 으로 고정**하는 변수 | 3-4 |

확인한 수치:

| 절 | 내용 | 결과 |
|---|---|---|
| 3-1 RREF | P0·P1·P2 3변수, 등식 2개 (`20x₀+30x₁+40x₂=120`, `6x₀+12x₁+14x₂=44`) 의 `[A\|b]` 를 sympy `rref()` | Pivot column `(0, 1)` → `x₂` 가 free. 해 `x = p + t·d`, `p = (2, 8/3, 0)`, `d = (−1, −2/3, 1)` |
| 3-1 검증 | `A @ p`, `A @ d` · `t = 0 ~ 2` 9점 | `A p = (120, 44) = b` · `A d = (0, 0)` · t 를 바꿔도 Profit 120 · Human 44 그대로 · 해집합 직선 3D 그림 |
| 3-2 Dimension | 차원 네 종류 + NumPy `ndim` | Ambient 3 · `rank(A) = 2` · Nullity 1 · 해집합 차원 1 · `ndim = 1` (`shape = (3,)`) — 수학적 차원과 배열 축 수는 다른 것 |
| 3-3 Unrestricted | `u = −3` 을 `u⁺ − u⁻` 로 | `(u⁺, u⁻) = (0,3), (1,4), …, (5,8)` 모두 `−3` — 분해가 유일하지 않다 · 그림 1장 |
| 3-4 Nonbasic | STEP2 의 `A` (3×6), `b = (80, 100, 210)` 에서 Basis `{x, s_H, s_T}` | `x = 10.5` · `s_H = 17` · `s_T = 37`, Nonbasic `y = s_P = a_P = 0` (STEP2 의 꼭짓점 V2 와 같은 점) |

마지막에 Decision · Slack · Surplus · Artificial · RREF free · Unrestricted · Basic · Nonbasic 8종 변수를 한 표로 정리했다.

핵심은 **`x = p + t·d`, `A d = 0`** — `d` 방향으로 아무리 움직여도 `A` 가 재는 양(Profit, Human)은 변하지 않는다.
"벡터가 많다고 공간이 넓어지는 게 아니라 **독립 방향**이 늘어야 한다" 는 rank / basis / null-space 이야기와 같은 물건이다.

### ⚠ 모델 인스턴스 — 절마다 다르다

| 절 | 쓰는 모델 | STEP1 · STEP2 와의 관계 |
|---|---|---|
| 3-1 · 3-2 | P0 · P1 · P2 **3변수**, 제약 2개를 **등식**으로 둔 교보재 | 새 인스턴스. 부등식·목표·상한 없이 "미지수 3 > 방정식 2 → 자유도 1" 만 보기 위한 설정. 원 문제의 판정과 무관 |
| 3-3 | 가상의 변수 `u` 하나 | 생산공장 모델과 무관한 개념 예시 |
| 3-4 | STEP2 교보재 (P0·P3 2변수 연속, H ≤ 80, T ≤ 100, Profit ≥ 210) | STEP2 와 **동일** — 상한을 STEP1 (60 / 65) 에서 올린 이유는 `0921_StandardForm` README 참조 |

원 문제의 판정은 여전히 STEP1 의 `INFEASIBLE` 과 "나머지를 다 지키면 이익 상한 160 만원" 이다.

## 난이도 단계 — 어디까지 왔나

| Level | 질문 | 어디서 |
|---|---|---|
| 1 — Feasibility | 모든 목표를 동시에 만족할 수 있는가? | ✅ STEP1 (`0919_SCIP`) — INFEASIBLE |
| — 기계장치 | Solver 는 그 답을 **어떻게** 내는가? | ✅ STEP2 (`0921_StandardForm`) — Standard Form · Basis · BFS = 꼭짓점 |
| — 개념 정리 | "자유변수" · "차원" 은 각각 무엇을 말하는가? | ✅ **STEP3 (이 브랜치)** — RREF free / Unrestricted / Nonbasic · rank · nullity |
| 2 — Goal Programming | 전부 만족할 수 없다면 무엇부터 지킬 것인가? | ⬜ 미착수 |
| 3 — Resource Sensitivity | 인간시간 또는 Token 을 줄이면 결과가 어떻게 변하는가? | ⬜ 미착수 |

STEP2 · STEP3 는 Level 1 → 2 사이에 끼워 넣은 단계다. Level 2 (Goal Programming) 는 아직 숙제로 남아 있다.

## 실행 방법 (다른 로컬에서 새로 시작할 때)

```bash
git clone -b 0926_FreeVariable https://github.com/Siegfriex/DC2_ProblemSolver.git
cd DC2_ProblemSolver

python3 -m venv .venv                 # Python 3.12 권장 (3.10 이상)
source .venv/bin/activate             # Windows: .venv\Scripts\activate
pip install -r requirements.txt

jupyter lab STEP3_류지환의생산공장.ipynb
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

STEP2 · STEP3 의 그림 라벨은 전부 영문이므로 별도 폰트 설정 없이 어느 OS 에서나 깨지지 않는다.
(`axes.unicode_minus = False` 만 설정 — 마이너스 기호가 □ 로 나오는 것을 막는다.)

## 검증 환경

아래 조합에서 `jupyter nbconvert --execute` 로 STEP3 를 **전 셀 위에서 아래로 재실행 · 오류 0 · 그림 2장 정상 생성**
을 확인했다 (2026-09-26).
확인된 주요 결과: Pivot columns `(0, 1)` · `A p = b` · `A d = 0` · `rank = 2` · nullity 1 · 3-4 Basis `{x, s_H, s_T}` → `(10.5, 17, 37)`.
STEP2 는 `0921_StandardForm` 에서 같은 환경으로 검증했고 이 브랜치에서 내용이 바뀌지 않았다.

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
  이 노트북들은 `DataFrame` 생성과 표시만 하므로 2.2 이상이면 무방하다.
- **SCIP** 는 ortools 휠에 포함되어 있어 별도 설치가 필요 없다. STEP3 는 ortools 를 import 만 하고 solver 는 부르지 않는다.
- STEP3 는 STEP2 의 변수를 가져다 쓰지 않는다 — 3-4 의 `A`, `b` 를 노트북 안에서 다시 정의하므로 단독으로 돈다.
- 노트북에 저장된 `execution_count` 는 저자가 학습하며 셀을 오가던 순서 그대로다.
  숫자가 연속이 아닌 것은 정상이며, 위에서 아래로 한 번에 돌려도 같은 결과가 나오는 것은 위에서 확인했다.

### 아직 다듬지 않은 것 (다음 커밋 예정)

이 커밋은 **오늘 공부한 지점까지를 그대로** 남기는 것이 목적이라 노트북 본문은 손대지 않았다. 계산 결과에는 영향이 없는 것들이다.

- STEP3: `print(md(...))` 세 곳에서 출력 아래에 `None` 이 찍힌다 (STEP2 와 같은 패턴)
- STEP3: 맨 끝에 빈 코드 셀 2개
- STEP3: 첫 셀의 `combinations` · `pywraplp` 는 import 만 되어 있다 (STEP2 에서 복사한 환경 셀)
- STEP3: 오탈자 — `ecelon` · `parametrix solutiond` · `non-nagetive` · `1-dimentional` · 열 이름 `x2_p2`
- STEP2: `0921_StandardForm` README 의 '아직 다듬지 않은 것' 목록 그대로

## 브랜치 규칙

이 저장소는 **주제 하나 = 브랜치 하나 = 노트북 하나** 로 운영한다.

| 브랜치 | 주제 | 노트북 |
|---|---|---|
| `main` | (LICENSE 만) | — |
| `0919_SCIP` | SCIP 정수계획 Feasibility | `STEP1_류지환의생산공장.ipynb` |
| `0921_StandardForm` | Standard Form · Slack/Surplus/Artificial · Basis · BFS | `STEP2_류지환의생산공장.ipynb` (+ STEP1) |
| `0926_FreeVariable` | Free Variable · Rank · Dimension · Unrestricted · Nonbasic | `STEP3_류지환의생산공장.ipynb` (+ STEP1 · STEP2) |

새 주제를 시작할 때:

```bash
# ⚠ main 이 아니라 '직전 주제 브랜치' 에서 딴다
git checkout -b <날짜>_<주제> 0926_FreeVariable
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
