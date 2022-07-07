## Git의 데이터베이스

### Git Commit과 Reset의 이해

커밋을 만들 때는 첫번째로 커밋 메시지, 커밋한 사람의 이메일, 날짜, 시간. 이런 것들을 합쳐서 메타 데이터라고 한다.   

또 필요한 건 프로젝트 스냅샷 hash   

그리고 부모 커밋의 hash값   

어떤 가상의 프로젝트가 있다고 가정.   

```
a.txt의 내용 aa
b.txt의 내용 bb
c.txt의 내용 aa
d.txt의 내용 d
e.txt의 내용 e
```

a.txt: hash 0a123   
b.txt: hash 17cd6   
c.txt: hash는 a.txt와 내용이 같기 때문에 같다.   


파일은 5개인데 저장되는 내용은 4개 => git은 중복되는 내용을 효율적으로 저장하게 된다.    
파일의 내용을 저장하는것을 blob이라고 한다.

이제부터는 파일의 이름과 경로를 저장하는데 이것을 트리라고 한다.   

src 폴더 아래 파일들

```
{ name: a.txt, cont: 0a123 }
{ name: b.txt, cont: 17cd6 }
{ name: c.txt, cont: 0a123 }
```

이것을 문자열이라고 생각하고 해싱했더니 d6789라고 하고 저장을 한다.   

res 폴더 아래 파일들   

```
{ name: d.txt, cont: 76c12 }
{ name: e.txt, cont: bb234 }
```

이것을 문자열이라고 생각하고 해싱했더니 112f3라고 하고 저장을 한다.   

root 폴더아래에는 src폴더와 res폴더가 존재.

```
{ name: src, cont: d6789 }
{ name: res, cont: 112f3 }
```

root 폴더는 c627b라고 하고 저장을 한다.    

```
{ message: 최초 커밋, root: c627b,parent: null }
```

이렇게 데이터를 얻었고 다시 문자열로 표현한 다음에 해시한 값이 있다. 그게 bb178이라고 가정하면 이것이 commit의 hash값이다.   

이제 프로젝트를 복원해보는 작업을 해본다.   
깃을 사용하면서 중요한 파일을 날렸다거나 삭제한 경우 그 복원과정이 어떻게 발생하는지 살펴본다.   

우리는 해시값 (bb178)하나가 남아만 있다.   

```bash
git reset --hard bb178
```

위 명령어를 입력시 hash값으로 찾는다. bb178 => root의 hash값 c627b를 찾는다.   
이 root 디렉터리는 src, res의 hash값을 가지고 있다. src는 a, b, c.txt에 대한 정보를 가지고 있다.   
res디렉터리는 d, e.txt 파일에 대한 정보를 가지고 있다.   

근데, 파일의 대한 이름과 정보만 복원했다. 내용을 복원되지 않았다.   
a.txt의 hash값으로 내용을 찾는다. 마찬가지로 b, c, d, e.txt의 hash값으로 내용을 찾는다.   

한가지 신경써야할 건 기존의 DB는 insert, select, update, delete가 다 되지만, git의 db는 insert, select만 가능하다. 그래서 git의 db로 들어간 이상 삭제되거나 변경되지 않기때문에 안전하게 보관된다. (일부로 git의 db를 날리지 않는 이상)   
그렇기 때문에 commit hash만 알고 있으면 언제든지 복원할 수 있다.   

```
> git cat-file -p 9af46
tree 9ec3138d35c9ef44dcb5de383efa95740ad1d122 => git commit으로 스냅샷을 찍었던 프로젝트 전체의 hash값
parent 0cb437be3dc888c45564a586720b9c5f9ccbf27c
author saseungmin <dbd02169@naver.com> 1657027373 +0900 => metadata
committer saseungmin <dbd02169@naver.com> 1657027373 +0900 => metadata

git commit lecture 1 test
```

tree를 다시 검색

```
> git cat-file -p a1a972b636028c21434d4e451d960930fb77889a
100644 blob f28d007a5d4769c93baa485a12b345d578200409    README.md
040000 tree 6797b0bfef3c78dd6191b13c647f1370a2f1fa03    res
040000 tree 323696e217f6b124878abb3a1636ba18b145cb5b    src
```

res의 정보

```
git cat-file -p 6797b0bfef3c78dd6191b13c647f1370a2f1fa03
100644 blob 4bcfe98e640c8284511312660fb8709b0afa888e    d.txt
100644 blob d905d9da82c97264ab6f4920e20242e088850ce9    e.txt
```

d.txt를 검색

```
> git cat-file -p 4bcfe98e640c8284511312660fb8709b0afa888e
d
```

a.txt 와 c.txt의 내용이 같기 떄문에 hash값이 같다

```
100644 blob e61ef7b965e17c62ca23b6ff5f0aaf09586e10e9    a.txt
100644 blob e61ef7b965e17c62ca23b6ff5f0aaf09586e10e9    c.txt
```

만약에 res와 src 폴더를 삭제한 경우

```bash
> rm -rf ./res ./src
```

git log로 commit 내용을 확인할 수 있다 commit의 hash값을 사용해 복원할 수 있다.

```bash
> git reset --hard 9af462f133b2b2dc8351348ae35f0e717e64ee14
```

### Git Objects
Git은 단순한 Key-value 데이터 저장소. Git에 저장되는 데이터에는 3가지 객체가 있다.

blob   
blob은 파일의 내용을 담고 있는 객체. 파일의 이름은 담고 있지 않다.

tree   
tree는 디렉터리를 저장하는 객체. tree에서 파일의 이름을 저장한다.

commit   
commit은 프로젝트의 루트 트리의 해시값, 부모 commit의 해시값 그리고 메타데이터를 저장한다.

#### git diff   

```
$ git diff <commit> <other commit>
```

diff명령어는 커밋과 커밋 사이 혹은 현재 작업 중인 디렉터리에서 변경사항을 출력해 주는 명령어이다.
깃의 db에서 두 커밋을 조회하면 root 먼저 비교한다. 같으면 끝나게되고 두개가 같지 않으면, 자식을 조회한다. 그 자식을 조회할 때 그 자식의 hash가 같으면 조회를 하지 않고, 다를 경우에만 다시 그 자식을 비교한다.   

만약에 d.txt의 내용이 d에서 ddd로 변경되면 아래와 같이 나타난다. 

```
d.txt
- d
+ ddd
```

이러한 비교로 인해 빠른 퍼포먼스를 낼 수 있다. 

git은 전체 프로젝트 hash값 하나만으로 특정 순간을 기록만해두면 언제든지 안심하고 쉽게 돌아갈 수 있다.

#### git add

```
$ git add <pathspec>
```

add명령어는 커밋을 위해 작업 중인 트리에서 현재 내용들을 index에 추가한다.

#### git restore

```
$ git restore <pathspec>
```

restore명령어는 작업중인 트리에서 파일을 복원하는 명령어이다.

#### git show

```
$ git show <object>
```

show명령어는 blobs, trees 그리고 커밋의 내용을 출력해 주는 명령어이다.

#### git log

```
$ git log
```

log명령어는 커밋 로그를 출력해 주는 명령어이다.

#### git ls-files

```
$ git ls-files
```

ls-files명령어는 작업중인 트리에서 파일 목록을 출력해 주는 명령어이다.
