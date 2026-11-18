# 2026-11-18 Git Day 2 | 브랜치, 병합, 충돌 해결

오늘은 VS Code 터미널에서 기능 브랜치를 만들고, 변경을 커밋한 뒤 `main`에 병합했다. 같은 `git merge`를 실행했는데도 어떤 때는 `main`이 기존 커밋으로 이동했고, 어떤 때는 새 병합 커밋이 생겼다. 마지막에는 `hello.py`에서 충돌이 발생해 해결 결과를 커밋했다. GitHub 저장소를 만들고 로컬 작업을 push하는 과정도 실습했다.

[실습 저장소 · moonjy613/OzWeb-git-basics](https://github.com/moonjy613/OzWeb-git-basics.git)

## 1. 오전 트러블슈팅 | VS Code 소스 제어에 10K 표시

프로젝트보다 상위에 있는 홈 폴더에 `/Users/admin/.git`이 잘못 잡혀 있었다. Git이 프로젝트에서 상위 폴더의 저장소를 찾아 사용하면, 소스 제어의 범위가 홈 폴더 아래까지 넓어질 수 있다. 프로젝트 파일 몇 개만 보려고 열었는데 VS Code 소스 제어에 10K가 표시된 증상을 이 저장소 범위로 설명할 수 있다.

같은 증상이 보이면 VS Code 터미널에서 현재 위치와 Git이 인식한 저장소를 먼저 비교한다.

```bash
pwd
git rev-parse --show-toplevel
git rev-parse --absolute-git-dir
git status --short
```

`pwd`는 터미널의 현재 위치를 보여준다. `--show-toplevel`은 Git이 인식한 작업 트리의 최상위 폴더를, `--absolute-git-dir`은 Git 메타데이터의 위치를 보여준다. 프로젝트 안에서 실행했는데 최상위 경로가 `/Users/admin`으로 나오면 홈 저장소를 보고 있다는 뜻이다. 이때는 의도한 프로젝트 폴더에서 올바른 저장소를 열고 두 경로를 다시 확인한다. 홈의 `.git`을 정리해야 한다면 먼저 보관할 이력이 있는지 확인한다. 위 명령은 재진단용 예시다.

## 2. 파일을 수정한 뒤 Git에 기록하기

`feat/ui`에서 실행한 첫 `git status`에는 `.DS_Store`가 추적되지 않은 파일로 나왔다. 프롬프트에는 `main`이라고 표시됐지만, 명령 출력의 `On branch feat/ui`를 기준으로 현재 브랜치를 읽었다. `rm .DS_Store`를 실행하고 다시 `git status`를 보니 `nothing to commit, working tree clean`이 나왔다. 이후 `ui.py`를 만들자 다시 추적되지 않은 파일이 표시됐다.

```text
작업 디렉터리 ── git add ──▶ 스테이징 영역 ── git commit ──▶ 로컬 저장소
                                                                    │
                                                                 git push
                                                                    ▼
                                                               GitHub 원격 저장소
```

- **작업 디렉터리**에서 파일을 생성하거나 수정한다. `git status`로 추적 여부와 스테이징 상태를 살핀다.
- **스테이징 영역(인덱스)**에는 다음 커밋에 넣을 파일 상태가 들어간다. `git add ui.py`는 명령을 실행한 시점의 파일 상태를 올린다. 이후 다시 편집한 내용까지 커밋하려면 다시 스테이징한다.
- **로컬 저장소**에는 `git commit`이 스테이징된 상태를 스냅샷으로 기록한다. 브랜치에서 커밋하면 해당 브랜치가 새 커밋을 가리킨다.
- **원격 저장소**는 `git push`로 필요한 커밋을 받고, 전송 대상 브랜치를 갱신한다.

## 3. 기능 브랜치 병합 | Fast-forward와 3-way

| 순서 | 터미널에서 한 작업 | 결과 |
| --- | --- | --- |
| 1 | `feat/ui`에서 `git add ui.py` → `git commit -m "Create ui.py"` | `eb8e387` 생성 |
| 2 | `git switch main` → `git merge feat/ui` | `7f4f48f..eb8e387` Fast-forward |
| 3 | `main`에서 `git branch feat/ai` | `feat/ai`가 `eb8e387`에서 시작 |
| 4 | `git switch feat/ui` → `git add ui.py` → `git commit -m "Update ui.py"` | `6dcad8e` 생성 |
| 5 | `git switch feat/ai` → `git add ai.py` → `git commit -m "Create ai.py"` | `8017ed0` 생성 |
| 6 | `git switch main` → `git merge feat/ai` | `eb8e387..8017ed0` Fast-forward |
| 7 | `git branch -D feat/ai` | 로컬 `feat/ai` 삭제. 삭제 직전 가리키던 커밋은 `8017ed0` |
| 8 | `main`에서 `git merge feat/ui` | `Merge made by the 'ort' strategy.` 출력. 3-way 병합 커밋 생성 |
| 9 | `git branch -D feat/ui` | 로컬 `feat/ui` 삭제. 삭제 직전 가리키던 커밋은 `6dcad8e` |

### Fast-forward | main이 이미 대상 커밋의 조상일 때

첫 병합에서 `main`이 가리키던 `7f4f48f`는 `eb8e387`의 조상이었다. 그래서 Git은 `main`을 기존 커밋 `eb8e387`로 이동했다. `feat/ai`를 병합할 때도 당시 `main`의 `eb8e387`에서 `8017ed0`까지 같은 방식으로 이동했다. 두 경우 모두 새 병합 커밋은 생기지 않았다.

### 3-way merge | 공통 조상에서 양쪽 이력이 갈라졌을 때

두 번째 `git merge feat/ui` 직전에는 `main`이 `8017ed0`을, `feat/ui`가 `6dcad8e`를 가리켰다. 둘 다 `eb8e387`에서 출발해 별도로 커밋됐으므로, Git은 공통 조상 `eb8e387`과 양쪽 끝 커밋의 세 스냅샷을 비교했다. 이 실습에서는 `ort` 전략이 두 변경을 자동으로 합쳤고, 두 끝 커밋을 부모로 갖는 병합 커밋을 만들었다. 병합 명령 출력에는 해시가 표시되지 않았다. 이어진 브랜치 생성과 `left` 병합의 시작점 `c557247`을 통해 이 병합 직후의 `main` 위치를 읽을 수 있다.

```text
            ┌── 8017ed0 ──┐
eb8e387 ────┤             ├── 병합 커밋 (main)
            └── 6dcad8e ──┘
```

### 브랜치 포인터와 커밋 DAG

커밋은 부모 커밋을 가리킨다. 이 부모 관계가 순환 없이 연결된 방향 그래프인 **DAG(Directed Acyclic Graph)**를 이룬다. 위 그림은 흐름을 읽기 쉽게 왼쪽에서 오른쪽으로 그렸으며, 실제 부모 참조는 새 커밋에서 이전 커밋으로 향한다.

브랜치는 그래프 안의 특정 커밋을 가리키는 이름이다. `git branch feat/ai`는 현재 커밋에 이름을 하나 추가하고, `git switch feat/ai`는 그 브랜치에서 작업하도록 전환한다. 브랜치에서 커밋하거나 fast-forward하면 해당 브랜치 포인터가 이동한다. `git branch -D`는 병합 여부 검사 없이 로컬 브랜치 이름을 삭제한다. 실습에서 삭제한 두 브랜치의 커밋은 병합 후에도 `main`에서 부모 관계를 따라갈 수 있다. [Git 공식 문서 · git merge](https://git-scm.com/docs/git-merge)

## 4. left와 right | hello.py 충돌 해결

1. `main`에서 `git branch left`, `git branch right`를 차례로 실행했다. 두 브랜치는 같은 커밋 `c557247`에서 시작했다.
2. `git switch left` 후 변경된 `hello.py`를 `git add hello.py`로 올리고 `git commit -m "Hello, left"`로 `d28d5d1`을 만들었다.
3. `git switch right` 후 변경된 `hello.py`를 `git add hello.py`로 올리고 `git commit -m "Hello, right"`로 `baf2117`을 만들었다.
4. `git switch main` 후 `git merge left`를 실행하자 `Updating c557247..d28d5d1`과 `Fast-forward`가 출력됐다.
5. 이어서 `git merge right`를 실행하자 `CONFLICT (content): Merge conflict in hello.py`가 출력됐다. 이제 `main`은 `d28d5d1`에 있고, `right`는 공통 조상 `c557247`에서 따로 뻗은 `baf2117`에 있었다. Git이 세 스냅샷을 비교하는 과정에서 `hello.py`의 변경을 자동으로 합치지 못해 병합을 멈췄다.
6. `git status`의 `Unmerged paths` 아래에 `both modified:   hello.py`가 나왔다. 공통 조상과 비교했을 때 양쪽에서 수정된 파일이 아직 병합되지 않은 상태였다.
7. 충돌 해결본은 에디터에서 정리한다. 이어서 기록에 나온 `git add hello.py`가 현재 파일을 스테이징하고 Git에 해당 경로의 충돌 처리가 끝났다고 표시했다. `git commit -m "Fix conflict"`로 `main`에 병합 커밋 `8363501`을 만들었다. 이 커밋의 두 부모는 병합 당시의 `main`인 `d28d5d1`과 `right`인 `baf2117`이다.

```text
           ┌── d28d5d1 (left) ───┐
c557247 ───┤                     ├── 8363501 (main)
           └── baf2117 (right) ──┘
```

충돌은 3-way 병합 과정에서 자동으로 합칠 수 없는 변경을 만났을 때 발생한다. 앞선 `feat/ui` 병합에서는 Git이 병합 커밋까지 만들었다. `right` 병합에서는 Git이 충돌에서 멈췄고, 파일을 스테이징한 뒤 직접 커밋해 병합을 마쳤다. Python 소스와 에디터에서 정리한 결과는 터미널 기록에 포함되어 있지 않다. [Git 공식 문서 · 충돌 해결](https://git-scm.com/docs/git-merge#_how_to_resolve_conflicts)

## 5. GitHub 저장소 생성과 push

오늘 GitHub에서 실습 저장소를 만들고 push까지 했다. 위 터미널 기록의 마지막 커밋은 `8363501 Fix conflict`이며, 제공된 터미널 기록에는 `git push` 명령과 출력이 없다.

GitHub에서 저장소를 만든 뒤 로컬 Git에서 그 주소를 원격 저장소로 연결하고 브랜치를 push한다. 아래는 기존 로컬 저장소에 `origin`이 아직 등록되지 않은 경우, `main`을 처음 올리는 복습용 예시다.

```bash
git remote -v
git remote add origin https://github.com/moonjy613/OzWeb-git-basics.git
git push -u origin main
```

`git remote -v`로 등록된 원격 주소를 확인한다. 올바른 `origin`이 이미 있다면 바로 push 단계로 넘어간다. `-u`는 이후 `git push`나 `git pull`이 기본으로 참조할 업스트림 브랜치를 설정한다. [GitHub Docs · 로컬 프로젝트를 GitHub에 추가하기](https://docs.github.com/en/repositories/creating-and-managing-repositories/adding-locally-hosted-code-to-github)
