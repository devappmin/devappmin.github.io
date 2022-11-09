---
title: "[Linux] 리눅스에서 스왑메모리를 생성해주기"
date: 2022-10-21 1:00 +0900
categories: [Develop, Linux]
tags: [Linux]
---

# 🛫 시작하기에 앞서

AWS EC2 프리티어 기간이 끝난지만 최근에 프로젝트를 진행하면서 서버를 이용할 일이 생겨서 저렴한 국내 서버 호스팅 업체에서 1vCore/1GB 리눅스 서버를 활용하여 프로젝트를 배포하였습니다.

만족하면서 사용하고 있었으나 도커와 젠킨스, 장고 서버까지 틀어져 있다보니 메모리가 아슬아슬한 상황이 되었고 2GB로 서버를 증축하기에는 비용이 2배가 들어 약간 꺼려지는 상황이었습니다. 1GB 중에는 60MB 정도는 항상 free 상태였고 가끔 튀었을 때 젠킨스에서 메모리 부족 문제가 발생하는 부분만 해결하고 싶었기 때문에 `Swap 메모리`를 활용하여 이를 해결하기로 했습니다! 

## SWAP 메모리란?

`Swap 메모리`란 실제 메모리가 가득 찼지만 더 많은 메모리 용량이 필요할 때 디스크 공간을 치환하여 부족한 메모리를 대체하는 것을 의미합니다. 실제로 존재하는 메모리가 아니라 디스크 공간을 메모리처럼 활용하는 것이기 때문에 가상 메모리이기도 합니다.

아무래도 RAM보다 HDD나 SSD의 속도가 훨씬 느리기도 하고 HDD나 SSD에 접근하는 시간도 문제이기 때문에 일반적인 메모리보다는 속도가 매우 느려 `Swap 메모리`를 사용하게 된다면 전반적인 속도가 느려질 수 밖에 없습니다.

하지만, 메모리가 부족해서 ssh 연결도 안되고 툭 꺼져버리는 상황을 방지하기 위해서 미리 임시방편으로 Swap 메모리를 만들어 놓는게 좋을 것 같습니다! 🥲

# 🫠 리눅스에서 Swap 메모리 확인

`Swap 메모리`를 확인하기 위해서는 **swapon -s**나 **free -h**, 또는 **top** 명령어를 활용할 수 있습니다.

![loading-ag-482](/uploads/swaponfreetop.png)

![loading-ag-488](/uploads/swaponfreetop2.png)

저는 미리 4GB만큼 스왑메모리를 생성했기 때문에 정상적으로 뜨는 것을 확인할 수 있습니다.

# 🐛 리눅스에서 Swap 메모리 만들기

1. 처음으로는 Swap 메모리를 Swap 파일인 **swapfile**로 잡습니다.
   
   ```bash
   $ sudo dd if=/dev/zero of=/swapfile bs=128M count=32
   ```
   
   **bs**는 포맷의 단의로 128M는 128MB를 의미합니다. M을 생략하여 바이트 단위로 생성도 가능하므로 `bs=1024`와 같이 1KB로 집을 수도 있습니다.
   
   **count**는 블록의 개수를 의미합니다. 즉, 128M의 블록을 32개 생성한 것이므로 4GB 만큼의 Swap 파일을 포맷했다고 할 수 있겠네요.

2. 다음은 **Swap 파일**에 퍼미션을 부여합니다.
   
   ```bash
   $ sudo chmod 600 /swapfile
   ```
   
   Swap 파일에 rw- 퍼미션을 부여하여 읽기와 쓰기가 가능하도록 하였습니다.

3. **Swap 영역**을 할당해줍니다.
   
   ```bash
   $ sudo mkswap /swapfile
   ```

4. **Swap 영역**에 **Swap 파일**을 추가하여 스왑 파일을 즉시 사용할 수 있도록 합니다.
   
   ```bash
   $ sudo swapon /swapfile
   ```

5. 여기까지 제대로 진행됐는지 확인하기 위해서 **swapon** 명령어를 통해 확인합니다.
   
   ```bash
   $ sudo swapon -s
   ```
   
   제대로 진행이 됐을 경우 `/swapfile`이 표시되는 것을 확인할 수 있습니다.

6. **/etc/fstab**에 명령어 추가합시다.
   
   vim이나 편하신 에디터를 통해서 **/etc/fstab**를 수정할 겁니다. 저는 vim을 통해서 수정할 것이기 때문에 다음과 같이 파일을 열었습니다.
   
   ```bash
   $ sudo vim /etc/fstab
   ```
   
   이후 파일 맨 아래에 다음 명령어를 추가하였습니다.
   
   ```textile
   (... 상위 텍스트 ...)
   /swapfile swap swap defaults 0 0
   ```
   
   해당 명령어를 통해서 시스템이 재시작되더라도 자동으로 활성화하게 할 수 있습니다.

7. 마지막으로 Free 명령어를 통해서 Swap 메모리가 잘 생성이 됐는지 확인하겠습니다.
   
   ```bash
   $ free
   ```

# 🐋 Swap 메모리 비활성화/삭제

## Swap 메모리 비활성화

스왑메모리를 더이상 사용하고 싶지 않을 경우에는 다음 명령어를 통해서 Swap 메모리를 비활성화 할 수 있습니다.

```bash
# 단일 Swap 메모리 비활성
$ sudo swapon /swapfile

# 모든 Swap 메모리 비활성화
$ swapon -a
```

## Swap 메모리 삭제

폴더를 삭제하듯이 `/swapfile`을 삭제하면 됩니다.

```bash
$ sudo rm -r /swapfile
```

# 🙂 결론

오늘은 Swap 메모리를 만드는 방법에 대해서 알아보았습니다.

앞으로는 메모리 용량이 작은 서버를 이용할 때 메모리가 부족해서 사용하기 불편한 것을 예방하기 위해 미리 Swap 메모리를 미리 만들어야겠네요!

## 참조

- [리눅스 : Swap 메모리란?](https://jw910911.tistory.com/122#:~:text=%EC%8A%A4%EC%99%91%20%EB%A9%94%EB%AA%A8%EB%A6%AC%EB%9E%80%2C%20%EC%8B%A4%EC%A0%9C%20%EB%A9%94%EB%AA%A8%EB%A6%AC,%EB%A9%94%EB%AA%A8%EB%A6%AC%EB%9D%BC%EA%B3%A0%20%ED%95%A0%20%EC%88%98%20%EC%9E%88%EC%8A%B5%EB%8B%88%EB%8B%A4.)
- [리눅스 공유 메모리 - Voyager of Linux](https://linux.systemv.pe.kr/%EB%A6%AC%EB%88%85%EC%8A%A4-%EA%B3%B5%EC%9C%A0-%EB%A9%94%EB%AA%A8%EB%A6%AC/)
- [젠킨스 JVM 메모리 설정](https://devroach.tistory.com/32)
