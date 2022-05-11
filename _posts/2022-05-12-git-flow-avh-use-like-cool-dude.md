---
title: "[Git] git-flow-avh를 멋쟁이처 쓰는 법"
date: 2022-05-12 00:08 +0900
categories: [Tool, Git]
tags: [Git, Git-Flow]
---

# 시작하기에 앞서

<img title="" src="/uploads/0c81fdc9e1b3739a4dbd4e611b76c6f2dbd17473.png" alt="loading-ag-1603" width="275" data-align="center">

작년에 학교에서 협업을 하면서 `git-flow-avh`를 통해 Git Flow 브랜치 전략을 활용하여 개발을 진행했습니다.

사실 `git-flow-avh`를 사용하는 처음 사용했을 당시에는 완전 신세계였고 깔끔하게 관리할 수 있다는 것에 감탄을 했었으나 개발을 더 진행하면서 몇 가지 문제를 마주하였습니다.

그 중 가장 불편했던 점은 **Pull Request -> Merge -> Pull**와 같이 **PR**을 하는 방식으로 Workflow를 활용하고 싶었으나, 인터넷을 찾아보면 feature branch를 예로 들면 `git flow feature finish` 명령어를 통해서 feature branch를 merge하는 것만 나와있었기 때문에 이러한 형식으로만 사용해야 한다고 생각했었습니다.

하지만, `git flow feature finish` 명령어를 사용하지 않고 **Pull Request**를 사용하는 Workflow를 최근에 발견하고 사용해보니 너무 마음에 들었고 뭔가 섹시하게 Git을 관리할 수 있었습니다.

## Git-flow-avh

### Git-flow-avh는 무엇일까요?

혹시라도 `git-flow-avh`를 모르시는 분이 있을 수 있으니 간략하게 소개하고자 합니다. 이미 알고 계시는 분이라면 아래 **Git-flow-avh 활용**으로 바로 넘어가시면 됩니다.

`git-flow`는 Vincent Driessen의 브랜칭 모델을 위한 고수준 저장소 작업을 제공하는 git의 확장입니다. 여기서 `git-flow`에 추가적인 명령을 속도 향상하기 위해서 개발한 것이 `git-flow-avh`으로 avh는 **A Virtual Home**을 의미합니다.

> git-flow AVH Edition is a collection of Git extensions to provide high-level repository operations for Vincent Driessen's [branching model](http://nvie.com/posts/a-successful-git-branching-model/).
>
> The AVH Edition adds more functionality to the existing git-flow and several of the internal commands have been rewritten to speed up the software.

<img title="" src="/uploads/f1015928c499dd75f0b777e69d76b53306aad9dc.png" alt="loading-ag-1594" width="550" data-align="center">

### Git-flow-avh 설치하기

`git-flow-avh`를 설치하는 방법은 아래와 같습니다.

- Mac OS

```bash
$ brew install git-flow-avh
```

- Windows

```bash
$ wget -q -O - --no-check-certificate https://raw.github.com/petervanderdoes/gitflow-avh/develop/contrib/gitflow-installer.sh install stable | bash
```

- Linux

```bash
$ apt-get install git-flow
```

### Git-flow-avh 둘러보기

`git-flow-avh`를 사용하려는 git 저장소 내에서 아래 명령어를 입력하여 `git-flow`를 시작할 수 있습니다.

```bash
$ git flow init
```

또한, `git-flow`는 다음과 같은 명령어들의 조합으로 브랜치를**start, finish, publish, pull**할 수 있습니다.

<img title="" src="/uploads/6b112175d9db0e1595d09a20f5f7ec78537e26f3.png" alt="loading-ag-1595" width="540" data-align="center">

더욱 자세한 내용은 [git-flow cheatsheet](https://danielkummer.github.io/git-flow-cheatsheet/index.ko_KR.html)를 통해서 살펴볼 수 있습니다.

# Git-flow-avh 활용

## Git-flow-avh를 통해 협업하기

앞서 말했듯이, 제가 `git-flow-avh`를 사용했을 때는 **start**를 통해서 브랜치를 생성한 후, **finish**를 통해서 해당 브랜치를 merge 시켰습니다.

하지만 이런 방식이 아니라 **Pull Request**를 하고 싶었는데 이 때는 `finish`를 하지 않고 다른 방식으로 접근을 해야 합니다. 그 방식은 다음과 같습니다.

1. 개발을 진행한 브랜치를 `publish` 명령어를 통해서 origin에 push합니다.

2. PR를 생성하고 모든 확인이 끝난 경우 Merge를 합니다.

3. `pull` 명령어를 통해서 origin에서 pull합니다.

위에 올라온 방식처럼 `finish`를 사용하지 않고 다른 방식으로 관리를 해도 정상적으로 되는 것을 확인할 수 있습니다.

# 결론

`git-flow-avh`를 사용하면서도 PR를 할 수 있다는 사실을 배우면서 더욱 `git-flow-avh`에 관심을 가지게 되지 않았나 싶습니다.

## References

[git-flow 시작하기](https://til.younho9.dev/log/2021/2021/07/06/hello-git-flow/)

[git-flow cheatsheet](https://danielkummer.github.io/git-flow-cheatsheet/index.ko_KR.html)

[A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model/)

[petervanderdoes/gitflow-avh: AVH Edition of the git extensions to provide high-level repository operations for Vincent Driessen's branching model (github.com)](https://github.com/petervanderdoes/gitflow-avh)
