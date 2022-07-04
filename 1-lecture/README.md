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
