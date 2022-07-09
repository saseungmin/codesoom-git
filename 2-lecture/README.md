## Git References

hash값을 쉽게 기억하기 위해서 branch와 tag를 사용한다.   

브랜치를 사용하면 특정 시점에 옮겨다닐 수 있다.   

tag는 일단 만들계되면 옮겨다니지 않는다.   

branch는 기본적으로는 tag와 비슷해보이지만 다른동작이 하나 있다.  main브랜치가 c123이라고 할 때, 새로운 커밋을 생성하면 main은 새로운 커밋을 바라본다.   

branch는 새로만드는 커밋으로 옮겨다니고 tag는 일부로 삭제하지 않는이상 옮겨다니지 않는다. 이 둘이 어떤 방식으로 작동하는지 알아본다.   

`.git`이라는 디렉터리 밑에 `refs`(references)라는 디렉터리가 있다. (`.git/refs`) 

`.git/refs`에는 `.git/refs/heads`와 `.git/refs/tags`가 있다. tag는 `.git/refs/tags`안에 있다. 만약 `.git/refs/tags`밑에 있는 파일을 삭제하면 tag가 삭제된다.   

만약 수동으로 내용을 변경하면

```bash
> cat .git/ref/tag/test
```

`cat`명령어를 읽어보면 40자의 hash값이 있다. test파일의 내용을 지우고 새로운 내용으로 변경하면 그 새로운 파일을 바라보게 된다.   
실제로는 `git tag <tag name>`을 사용한다. tag를 삭제할 땐 `git tag -d <tag name>`을 사용한다.   

브랜치는 `.git/refs/heads` 디렉터리를 확인해본다. main branch는 `.git/refs/heads/main`이다. 직접 이 파일을 수정하면 문제가 발생한다.   

```bash
// main 브랜치 내용을 확인
> cat .git/refs/head/main
```

역시나 40자 hash값이 있다. 만약에 파일을 수정을 하고 `add`후 `commit`을 하면 자동으로 다른 hash값으로 변경되어있을 것이다. 이것이 새로운 커밋의 내용을 바라보는 hash값이다.   
여러개의 브랜치중 지금 바라보고 있는 브랜치는 어떻게 알 수 있을까?   
예를 들어, develop, hotfix, feature 브랜치 등.. 내가 어떤 기준으로 브랜치를 작업하고 있는지 어떻게 알까?? 그 방법은 `HEAD`를 본다.    

`HEAD`가 저장되어있는 위치는 `.git/HEAD`위치에 존재한다.

```bash
> cat .git/HEAD
ref: refs/heads/main
```

브랜치를 변경할 때

```bash
> git checkout develop
// 또는
> git switch develop
```

만약 `feature/login`와 같은 슬래시가 존재하는 브랜치를 만들면 다음과 같이 된다.   
이렇게 만들 경우 `feature`는 파일이 아니라 디렉터리가 된다.

```
.git/refs/heads/feature/login
.git/refs/heads/feature/password
```

`git reflog`는 HEAD가 변경되었던 이력을 조회할 수 있는 명령어이다.
