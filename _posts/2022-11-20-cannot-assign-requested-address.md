---
title: "[Python, Docker] smtplib 이용 시 Cannot Assign Requested Address 문제 해결하기"
date: 2022-11-21 17:33 +0900
categories: [Develop, Python]
tags: [Python, Docker]
---

# 🛫 시작하기에 앞서

```
OSError: [Errno 99] Cannot assign requested address
```

저는 도커에서 특정 요청이 들어오면 파이썬 라이브러리인 `smtplib`을 활용해서 지메일을 통해 메일을 보내는 기능을 구현하는 도중 해당 문제를 마주하였습니다.

사실 개발환경에서는 잘 동작했으나 서버에서 컨테이너로 동작시키면 해당 문제가 발생해서 처음에는 도커 자체만의 문제인 줄 알았습니다. 하지만, 도커 뿐만이 아니라 다른 문제도 해결해야 해당 문제를 해결할 수 있습니다..!

## 🤔 그래서 해당 에러가 뭔데?

> `OSError: [Errno 99] Cannot assign requested address` this means the provide `ip/port` is already taken. Change your `ip/port` and try again.

해당 `ip:port`가 이미 사용되고 있거나 다른 이유로 해당 `ip:port`를 사용할 수 없어서 생기는 에러입니다.

# 🫠 해결 과정

## 🐳 Docker-Compose에서 포트 열어주기

당연하겠지만 Docker-Compose에서 포트를 열어줘야 합니다. `SMTP`의 경우 **25번**포트를 사용하고 **Gmail**의 경우에는 **587번**포트를 사용하여 메일을 주고 받고 있습니다.

저의 경우에는 **Gmail**을 통해서 메일을 보내고 있으므로 프로젝트의 587번 포트와 도커 외부의 587번 포트를 연결시켜주도록 하겠습니다.

```yml
version: '3.7'
services:
    my-service:
        # ports
        ports:
            - 587:587
```

## 🐛 서버 방화벽에서 해당 포트 예외처리하기

AWS, 개인 서버, 다른 서버 업체 등을 통해서 파이썬 프로그램을 실행했을 때 메일을 보내는 포트가 방화벽에 의해서 막혀있을 경우에도 해당 문제가 발생할 수 있습니다.

저 또한, **iwinv**서버를 임대하면서 587번 포트를 열어주지 않았다는 것을 알게 된 후에 빠르게 Out-Bound 규칙에 587포트를 추가하였습니다.

![iwinv-setting](/uploads/iwinv-outbound.png)

---

도커에서 포트를 연결하고 서버 방화벽에서 Out Bound 규칙에 587포트를 허가하게 만들면 해당 문제를 해결할 수 있습니다..! 

# 🙂 결론

오늘은 `OSError: [Errno 99] Cannot assign requested address` 문제를 해결하는 방법을 알아보았습니다. 단순히 도커 문제라고 생각하고 계속 고민했다가 서버측 문제라는 것을 깨닫게 될 때까지 많은 고민을 했던 것 같습니다. 딱 저에게 맞는 문제를 겪은 분이 인터넷을 찾아봐도 잘 없어서 혹시라도 이 글을 보는 분들은 쉽게 해결할 수 있었으면 좋겠습니다!

## 참조

[python - How to fix error when sending mail with django in docker container? (Cannot assign requested address) - Stack Overflow](https://stackoverflow.com/questions/74002773/how-to-fix-error-when-sending-mail-with-django-in-docker-container-cannot-assi)
