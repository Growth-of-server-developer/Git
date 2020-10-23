## Branch
코드를 통째로 복사하고 나서 원래 코드와 상관없이 독립적으로 개발하는 것이 브랜치다.

### Git 브랜치 이해하기

Git이 데이터를 어떻게 저장하는지 알아야 한다.

- Git은 데이터를 변경사항으로 기록하지 않고 일련의 스냅샷으로 기록한다.
- 커밋하면 Git은 현재 Staging Area에 있는 데이터의 스냅샷에 대한 포인터, 저자나 커밋 메시지 같은 메타 데이터, 이전 커밋에 대한 포인터 등을 포함하는 커밋 Object를 저장한다.
- 이전 커밋 포인터가 있어서 비교가 가능

### git commit을 하면

메타데이터와 루트 트리를 가리키는 포인터가 담긴 커밋 개체 하나를 만든다. → 하나의 스냅샷

루트 트리를 가리키는 포인터를 따라가면 파일과 디렉터리 구조가 들어 있는 트리 개체가 있다.

트리 개체에 각 blob(파일)을 따라가보면 파일에 대한 정보가 담겨있다.

### Git 브랜치는 커밋 사이를 가볍게 이동할 수 있는 포인터 같은 것이다.

Git 브랜치는 어떤 한 커밋을 가리키는 40글자의 SHA-1 체크섬 파일에 불과 (줄 바꿈 문자 포함 41바이트 크기의 파일일 뿐) 

### Git은 HEAD라는 특수한 포인터가 있어 지금 작업 중인 브랜치가 무엇인지 파악할 수 있다.

- git log —oneline —decorate : 브랜치가 어떤 커밋을 가리키는지 확인할 수 있다.
- git log —oneline —decorate —graph —all : 트리로 확인 가능

git checkout을 하면 HEAD는 해당 브랜치를 가리킨다.

## Merge

c1 ← c2 ← c4

c2 ← A

c4 ← B

- fast-forward

A 브랜치에서 다른 B 브랜치를 Merge할 때 B 브랜치가 A 브랜치 이후의 커밋을 가리키고 있으면 그저 A브랜치가 B브랜치와 동일한 커밋을 가리키도록 이동시킬 뿐이다.

merge 하고 나면

c4 ← B ← A

- 3-way-merge (recursive)

```
c2 - c4 (master)
   \ 
     c3 - c5 (issue3)
```

현재 브랜치(master)가 가리키는 커밋이 merge할 브랜치(issue3)의 조상이 아니므로 공통 조상 하나를 사용하여 3-way-merge를 한다.

```
c2 - c4 ----- c6(master)
   \         / 
     c3 - c5 (issue3)
```

3-way-merge의 결과를 별도의 커밋(c6)으로 만들고 나서 해당 브랜치(master)가 그 커밋을 가리키도록 이동시킨다. 이런 커밋을 merge 커밋이라고 부른다.

### 충돌

Merge하는 두 브랜치에서 같은 파일의 한 부분을 동시에 수정하고 merge할 때 발생한다.

- git status로 어떤 파일을 merge할 수 없었는지 살펴볼 수 있다.

```
<<<<<< HEAD:README.md
merge 실행할 때 작업하던 브랜치(위에선 master)
======
merge할 브랜치 부분
>>>>>> issue3:README.md
```

다른 팀원이 master 브랜치를 업데이트 했다면?

팀원과의 히스토리는 서로 달라진다.

리모트 서버로부터 저장소 정보를 동기화해야 한다. → git fetch origin

위 명령어를 실행하면 "origin" 서버의 주소 정보를 찾아서 현재 로컬의 저장소가 갖고 있지 않은 새로운 정보가 있으면 모두 내려받고, 받은 데이터를 로컬 저장소에 업데이트하고 나서 origin/master 포인터의 위치를 최신 커밋으로 이동시킨다. (변경 사항이 없다고해도 remote tracking 브랜치가 해당 원격 서버의 브랜치가 가리키는 것을 가리키게 한다.) 

그저 수정 못하는 원격 브랜치 포인터가 생긱는 것임 → merge따로 해야 함

### 공유

- git push (remote) (branch)

만약 로컬 브랜치의 이름과 리모트 서버의 브랜치 이름이 다를 때는?

- git push (remote) (로컬브랜치):(리모트브랜치)

### Upstream 브랜치 = Tracking 브랜치

리모트 트래킹 브랜치를 로컬 브랜치로 checkout 하면 자동으로 tracking 브랜치가 만들어진다. 이 브랜치는 리모트 브랜치와 직접적인 연결고리가 있는 로컬 브랜치이다. 

따라서 git pull을 하면 리모트 저장소로부터 데이터를 내려받아 연결된 리모트 브랜치와 자동으로 merge한다.

특정 브랜치를 추적하게 하려면?

- git branch -u(—set-upstream-to) (remote서버)/(remote branch)

이런 정보 확인하기 (git fetch —all 으로 최신 데이터를 받아온 후에 추적 상황을 확인해야 한다.)

- git branch -vv
    - ahead 3, behind 1 표시의 의미

        해당 브랜치에 서버로 보내지 않은 커밋이 3개, 서버의 브랜치에서 아직 로컬 브랜치로 merge하지 않은 커밋이 1개 있다.

리모트 브랜치 삭제하기

- git push (remote) —delete (branch)

## Rebase

Git에서 한 브랜치를 다른 브랜치로 합치는 방법 중 하나다. (merege, rebase)

### Rebase 이해하기

한 브랜치에서 변경된 사항을 다른 브랜치에 적용할 수 있다.

```
c2 - c4 (master)
   \ 
     c3 - c5 (issue3)
```

```
git checkout issue3
git reabse master
```

```
c2 - c4(master) - c3 - c5(HEAD -> issue3)
```

두 브랜치가 나뉘기 전인 공통 커밋으로 이동하고 나서 그 커밋부터 지금 checkout한 브랜치가 가리키는 커밋까지 diff를 차례로 만들어 어딘가에 임시로 저장해 놓는다. 

rebase할 브랜치(issue3)가 합칠 브랜치(master)가 가리키는 커밋을 가리키게 하고 아까 저장해 놓았던 변경사항을 차례대로 적용한다. 결국 합칠 브랜치를 두고 그 위를 현재 작업 브랜치의 커밋들이 쌓여간다.  

### merge와 다른건 좀 더 깨끗한 히스토리를 만든다는 것! 최종 결과물은 같다.

히스토리가 선형이라 모든 작업이 차례대로 수행된 것처럼 보인다. 실제로 rebase를 진행할 때도 브랜치의 변경사항을 순서대로 다른 브랜치에 적용하면서 합친다.

필자의 경우 2개의 커밋을 하나의 커밋으로 합치는 작업을 진행할 때 rebase를 사용하다가 

### rebase 활용

- —onto 옵션
- git rebase [basebranch] [topicbranch]
    - git rebase develop issue3

### 주의

rebase는 기존 커밋을 그대로 사용하는 것이 아니라 내용은 같지만 다른 커밋을 새로 만든다. 

- git pull —rebase
    - git fetch 후 git rebase [remote/branch]한 것과 같다.
