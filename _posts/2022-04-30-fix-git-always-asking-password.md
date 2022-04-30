---
title: "[Git] 항상 GitHub 비밀번호를 요구하는 오류 해결하기"
date: 2022-04-30 19:58 +0900
categories: [Tool, Git]
tags: [Git, Error]
---

## 시작하기에 앞서

<img src="https://www.freecodecamp.org/news/content/images/2019/10/clone.png" title="" alt="clone" data-align="center">

GitHub에서 HTTPS를 통해 클론을 해서 프로젝트를 수정한 후 Push를 하려고 하니까 매번 GitHub 계정으로 로그인을 하라는 오류가 떠서 생각 없이 다시 로그인을 하고는 했습니다. 

하지만 너무 불편해서 인터넷에 검색해보니 생각보다 흔하게 발생하는 오류로 간단하게 해결할 수 있었습니다.

## 오류 해결 방법

### Origin Remote의 URL을 HTTPS에서 SSH로 수정하기

```bash
$ git remote set-url origin git@github.com:username/repo.git
```

첫 번째 방법은 origin remote를 HTTPS에서 SSH로 변경하는 것입니다. 해당 오류는 HTTPS에서만 발생하므로 SSH로 연결할 경우 해당 문제를 해결할 수 있습니다.

### Git Store 활용

```bash
$ git config --global credential.helper store
$ git config --global credential.helper cache
$ git config --global credential.helper 'cache --timeout=600'
```

두 번째 순서는 Git store와 cache를 활용하는 방법입니다. 위 커맨드는 다음과 같은 의미를 가지고 있습니다.

- username과 password를 저장하기 위한 Git store를 생성한다.

- Session의 username과 password를 저장(cache)한다.

- 필요시, 위 설정에 timeout을 설정한다.

## 결론

보안과 관련된 이유로 사실 Git Store를 활용하여 오류를 해결하는 것보다 SSH를 통해서 GitHub와 통신하는 것이 바람직합니다.

저는 아직까지 SSH를 통해서 연결할 생각을 하진 않았었는데 회사에서 일하거나 제 컴퓨터가 아닌 컴퓨터에서 작업을 할 때를 고려하여 SSH로 통신하는 것으로 천천히 바꿔야겠네요.

> However, due to security reasons, it is advisable that you use SSH to interact with GitHub, especially if you work for a company or you’re using a computer that isn’t yours.
> 
> Using the SSH protocol, you can connect to GitHub without supplying your username or password every time.



### References

[How to fix Git always asking for user credentials (freecodecamp.org)](https://www.freecodecamp.org/news/how-to-fix-git-always-asking-for-user-credentials/)


