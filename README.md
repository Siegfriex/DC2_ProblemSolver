# 0919_SCIP — 생산공장 모델 STEP1 (Feasibility)

> 의사결정2 · 홍익대 2026-2 · 류지환
> 브랜치 주제: **OR-Tools / SCIP 로 정수계획 Feasibility 문제 풀기**

## 이 브랜치에 있는 것

| 파일 | 내용 |
|---|---|
| `STEP1_류지환의생산공장.ipynb` | 본 학습 노트북 — 아래 참조 |
| `requirements.txt` | 이 노트북만 돌리기 위한 최소 의존성 |
| `README.md` | 이 문서 |

## 노트북이 다루는 것

강의의 Operations Research 예제를 **내 실제 사업(MVP 수주)** 과 **AI Harness 운영 문제** 로
다시 모델링한다. 같은 개념을 두 관점에서 두 번 푼다.

- **A. Human / Principal Model** — 인간 류지환이 어떤 상품(P0~P3)을 몇 건 받을 것인가
- **B. Harness / Orchestrator Model** — 수주 후 Top Orchestrator 가 어떤 Harness 구조를 택할 것인가

난이도 단계:

| Level | 질문 |
|---|---|
| 1 — Feasibility | 모든 목표를 **동시에** 만족할 수 있는가? ← **이 노트북(STEP1)** |
| 2 — Goal Programming | 전부 만족할 수 없다면 무엇부터 지킬 것인가? |
| 3 — Resource Sensitivity | 인간시간 또는 Token 을 줄이면 결과가 어떻게 변하는가? |

STEP1 의 결론: 네 목표(이익 ≥ 210만원 / Human Time ≤ 60h / Token ≤ 65U / P3 ≥ 1건)를
동시에 만족하는 해는 **INFEASIBLE**. 진단 모델로 다시 풀면 "나머지 목표를 모두 지킬 때
이익은 최대 160만원" 이라는 구체적 수치가 나온다 — 이게 Level 2 Goal Programming 이 필요한 이유다.

핵심 기술 요소:

- `pywraplp.Solver.CreateSolver("SCIP")` — 정수 결정변수를 다룰 수 있는 MIP solver
- `IntVar(0, infinity, name)` — 프로젝트 수는 음수·소수가 불가능하므로 정수변수
- `Minimize(0)` — 목적함수를 상수 0 으로 두어 **최적화가 아닌 제약 만족(constraint satisfaction)** 만 검사
- `STATUS_LABEL` 로 solver 상태 코드를 사람이 읽는 문자열로 변환

## 실행 방법 (다른 로컬에서 새로 시작할 때)

```bash
git clone -b 0919_SCIP https://github.com/Siegfriex/DC2_ProblemSolver.git
cd DC2_ProblemSolver

python3 -m venv .venv                 # Python 3.12 권장 (3.10 이상)
source .venv/bin/activate             # Windows: .venv\Scripts\activate
pip install -r requirements.txt

jupyter lab STEP1_류지환의생산공장.ipynb
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

## 검증 환경

아래 조합에서 `jupyter nbconvert --execute` 로 **전 셀 재실행 · 오류 0** 을 확인했다 (2026-09-19).
확인된 주요 결과: `Solver_Status = INFEASIBLE` · 진단 모델 최대이익 `160.0만원` (목표 210 대비 50 부족).

| 항목 | 버전 |
|---|---|
| OS | WSL2 · Ubuntu 24.04 |
| Python | 3.12.3 |
| ortools | 9.10.4067 |
| pandas | 3.0.3 |
| IPython | 9.14.1 |
| ipykernel | 7.3.0 |
| jupyterlab | 4.6.0 |

주의할 점:

- **pandas 3.x** 는 copy-on-write 와 string dtype 이 기본이다. 2.x 관용구(체이닝 대입 등)는 동작이 다르다.
  이 노트북은 `DataFrame` 생성과 표시만 하므로 2.2 이상이면 무방하다.
- **SCIP** 는 ortools 휠에 포함되어 있어 별도 설치가 필요 없다.
- 동일 목적함수값을 갖는 **다른 최적해** 가 나올 수 있다 (degenerate solution). ortools 버전이 다르면
  `x` 조합이 달라 보여도 목적함수값이 같으면 정상이다. 0 이 `-0.0` 으로 출력되는 것도 SCIP 의 정상 동작.

## 브랜치 규칙

이 저장소는 **주제 하나 = 브랜치 하나 = 노트북 하나** 로 운영한다.

| 브랜치 | 주제 | 노트북 |
|---|---|---|
| `main` | (LICENSE 만) | — |
| `0919_SCIP` | SCIP 정수계획 Feasibility | `STEP1_류지환의생산공장.ipynb` |

새 주제를 시작할 때:

```bash
# ⚠ main 이 아니라 '직전 주제 브랜치' 에서 딴다
git checkout -b <날짜>_<주제> 0919_SCIP
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
