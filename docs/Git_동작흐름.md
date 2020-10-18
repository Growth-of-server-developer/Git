## 학습 목표
- Git이 어떻게 변경된 파일을 알 수 있는지 설명할 수 있다.
- git status를 입력할 때 내부에서 어떤 동작이 일어나는지 설명할 수 있다.
- git reset의 각 단계를 설명할 수 있다.
- git reset과 git checkout의 차이점을 알 수 있다.
----
### HEAD

현재 브랜치를 가리키는 포인터, 브랜치는 브랜치에 담긴 커밋 중 가장 마지막 커밋.

- 마지막 커밋의 스냅샷이다.

    아래는 저수준 명령어로 HEAD 스냅샷의 디렉터리 리스팅과 각 파일의 SHA-1 체크섬을 보여주는 명령어다.

- git cat-file -p HEAD
    - **스냅샷에는 트리 개체와 부모 커밋 개체, author, committer, 커밋 메시지**가 담겨있다.
- git ls-tree -r HEAD

### Index

바로 다음에 커밋할 것들이다. (Staging Area: 즉 git commit할 때 처리할 것들이 있는 곳)

워킹 디렉토리에서 **마지막으로 checkout한 브랜치의 파일 목록과 파일 내용으로 채워진다.** 이후 파일을 변경하면 Staging Area에 있는 파일 목록들이 추가된다.

- git ls-files -s

### 워킹 디렉터리

위의 두 트리는 파일과 그 내용을 효율적인 형태로 .git 디렉터리에 저장한다.

워킹 디렉터리는 실제 파일로 존재한다. 바로 눈에 보이기에 편집하기 수월하다.

커밋하기 전에는 Index(Staging Area)에 올려놓고 얼마든지 변경할 수 있다.

- tree (mac os에서는 설치해야 함)
----

### 🌟 Git의 주목적은 프로젝트의 스냅샷을 지속적으로 저장하는 것

이 트리 3개를 사용해 더 나은 상태로 관리한다.

1. git init

Git 저장소가 생기고 HEAD는 아직 없는 브랜치를 가리킨다.

```
? (HEAD -> master)

HEAD  |  Index  | Working Directory
      |         |    file.txt(v1)
```

2. git add file.txt

워킹 디렉터리의 내용이 Index로 복사한다.

```
? (HEAD -> master)

HEAD  |  Index        | Working Directory
      |  file.txt(v1) |    file.txt(v1)
```

3. git commit

Index의 내용을 스냅샷으로 영구히 저장하고 그 **스냅샷을 가리키는 커밋 객체**를 만든다. 

그리고 master가 그 커밋 객체를 가리키도록 한다.

```
 eb423d13 (HEAD -> master)

  HEAD                  |  Index        | Working Directory
eb423d13(file.txt(v1))  |  file.txt(v1) |    file.txt(v1)
```

이때 git status를 실행하면 아무런 변경사항이 없다고 나온다. **(세 트리 모두가 같기 때문)**

4. 파일 내용을 바꾼다.

```
 eb423d13 (HEAD -> master)

  HEAD                  |  Index        | Working Directory
eb423d13(file.txt(v1))  |  file.txt(v1) |    file.txt(v2)
```

이때 git status를 실행하면 Changes not staged for commit 아래 변경된 파일 목록(빨강)을 볼 수 있다.

**Index와 working directory가 다른 내용을 담고 있기 때문이다.**

5. git add file.txt

변경사항을 Index에 올려준다.

```
 eb423d13 (HEAD -> master)

  HEAD                  |  Index        | Working Directory
eb423d13(file.txt(v1))  |  file.txt(v2) |    file.txt(v2)
```

이때 git status를 실행하면 Changes to be committed 아래에 Staged 상태의 파일 목록(녹색)을 볼 수 있다.

**Index에 있는 파일과 HEAD에 있는 파일들 중 다른 파일들이 여기에 표시된다.** 

즉 다음 커밋할 것과 지금 마지막 커밋이 다르다는 뜻이다.

6. git commit

```
 eb423d13(v1) <- 9ba1b23(v2) (HEAD -> master)

  HEAD                  |  Index        | Working Directory
9ba1b23(file.txt(v2))   |  file.txt(v2) |    file.txt(v2)
```

다시 git status 명령을 실행하면 아무것도 출력하지 않는다. (세 개의 트리 내용이 다시 같아졌기 때문)

- 브랜치를 checkout하거나 clone도 비슷하다.

브랜치를 checkout하면 HEAD가 새로운 브랜치를 가리키도록 바뀌고, 새로운 커밋의 스냅샷을 Index에 놓는다.

그리고 Index의 내용을 워킹 디렉터리로 복사한다.

### Reset하기

```
 eb423d13(v1) <- 9ba1b23(v2) <- b1a12bc(v3) (HEAD -> master)

  HEAD                  |  Index        | Working Directory
b1a12bc(file.txt(v3))   |  file.txt(v3) |    file.txt(v3)
```

git reset 9ba1b23

1. HEAD 이동
checkout 명령처럼 HEAD가 가리키는 브랜치를 바꾸진 않는다. HEAD는 계속 현재 브랜치를 가리키고 있고, 현재 브랜치가 가리키는 커밋을 바꾼다. (git reset의 —soft 옵션이 여기까지 진행)

```
 eb423d13(v1) <- 9ba1b23(v2)(HEAD -> master) <- b1a12bc(v3) 

  HEAD                  |  Index        | Working Directory
9ba1b23(file.txt(v2))   |  file.txt(v3) |    file.txt(v3)
```

가장 최근의 git commit 명령을 되돌린다. (브랜치가 가리키는 커밋만 이전으로 되돌린다.)

2. Index 업데이트 (—mixed 옵션이 여기까지 진행, 기본 옵션임)
       Index를 현재 HEAD가 가리키는 스냅샷으로 업데이트한다. (Staging Area를 비운다.) git add 도 되돌리는 것

```
 eb423d13(v1) <- 9ba1b23(v2)(HEAD -> master) <- b1a12bc(v3) 

  HEAD                  |  Index        | Working Directory
9ba1b23(file.txt(v2))   |  file.txt(v2) |    file.txt(v3)
```

3. 워킹 디렉터리 업데이트 (—hard 옵션)

```
 eb423d13(v1) <- 9ba1b23(v2)(HEAD -> master) <- b1a12bc(v3) 

  HEAD                  |  Index        | Working Directory
9ba1b23(file.txt(v2))   |  file.txt(v2) |    file.txt(v2)
```

워킹 디렉터리의 내용까지도 되돌린다. **reset 명령을 위험하게 만드는 유일한 옵션이다.** Git에는 데이터를 실제로 삭제하는 방법이 별로 없는데 그 중 하나이다. **즉 되돌리는 것이 불가능한 옵션이다.** 워킹 디렉터리의 파일까지 강제로 덮어쓴다. (커밋한 적이 없다면 Git이 덮어쓴 데이터는 복원할 수 없다.) 

즉 add까지만 진행하고 reset하면 add한 파일을 되돌릴 수 없음.

→ 파일의 v3버전을 아직 Git이 커밋으로 보관하고 있기 때문에 reflog를 이용해서 다시 복원할 수 있다. (reset 하기 전 커밋으로 reset하면 된다.)

### Checkout

git reset —hard [branch]과 비슷하게 [branch] 스냅샷을 기준으로 세 트리를 조작하지만 두 가지 사항이 다르다.

1. 워킹 디렉터리를 안전하게 다룬다.

    워킹 디렉터리에서 Merge 작업을 한번 시도해보고 변경하지 않은 파일만 업데이트한다.
    반면에 reset —hard 명령은 확인하지 않고 모든 것을 바꿔버린다.

2. HEAD 자체를 다른 브랜치로 옮긴다. (reset은 HEAD가 가리키는 브랜치를 움직임)

    develop(현재 워킹 디렉터리)와 master가 있을 때

    - git reset master

    → develop 브랜치는 master 브랜치가 가리키는 커밋과 같은 커밋을 가리킴

    ```
     a <- b(master) <- c <- d (HEAD -> develop)
    # git reset master
     a <- b(master, HEAD -> develop) <- c <- d
    # git checkout master
     a <- b(HEAD -> master) <- c <- d (develop)
    ```
