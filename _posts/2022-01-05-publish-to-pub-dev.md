---
title: "[Flutter] 패키지 생성 후 pub.dev에 publish하기"
date: 2022-01-05 21:00 +0900
categories: [Develop, Flutter]
tags: [Flutter]
---

## Before Start.

### Dart Packages vs Flutter Packages.

![Dart vs Plugin](/assets/uploads/2022-01-05/1.png)
_Figure 1_

1. **Dart Packages**

   Dart로만 짜여진 일반적인 패키지를 의미합니다. 각 Platform마다 따로 처리를 할 필요가 없으면 다트 패키지로 만드는 것 같습니다.

2. **Plugin Packages**

   다트 패키지와 다르게 각 Platfrom마다 다르게 처리를 해 줄 필요가 있을 경우에는 플러그인 패키지로 생성하는 것 같습니다.

추가적으로 **플러그인 패키지**로 패키지를 생성할 경우 example 폴더가 기본적으로 생성되나 **다트 패키지**로 패키지를 생성할 경우에는 example 폴더가 기본적으로 생기지 않습니다.

이번 포스트에서는 Dart 코드를 통해서 퍼블리싱을 진행하겠습니다.

### References

[Flutter Heatmap Calendar](https://github.com/devappmin/flutter_heatmap_calendar/)<br>
위 링크를 참조하면서 진행하시면 더욱 쉽게 진행하실 수 있습니다!

### 시작 전 주의사항

패키지명이 이미 pub.dev에 올라와 있을 경우 올릴 수 없고 파일 명과 패키지 명을 하나 하나 전부 수정한 후 다시 검사를 해야하니 사전에 pub.dev에서 패키지명을 먼저 **꼭!!** 검색해본 후에 프로젝트를 생성해주시길 바랍니다.

## Getting Started.

### 패키지 만들기

다트 패키지를 생성하기 위해서는 `--template=package` flag를 추가하여 `flutter create`를 호출해야 합니다.

```terminal
$ flutter create --template=package YOUR_PACKAGE_NAME
```

만약에 다트 패키지가 아니라 플러그인 패키지를 생성하기 위해서는 `--org`, `--platforms`, `-a` 혹은 `-i` 옵션 또한 추가해야 합니다.

```terminal
$ flutter create --org com.example --template=plugin --platforms=android,ios -a kotlin YOUR_PACKAGE_NAME
```

`--org` 옵션을 사용하여 organization을 명시할 수 있습니다.<br>
`--platforms` 옵션을 사용하여 적용할 플랫폼을 명시할 수 있으며, 적용 가능한 플랫폼은 `android, ios, web, linux, macos, windows`가 있습니다.<br>
`-a` 옵션은 적용할 플랫폼 중 안드로이드가 있을 경우 사용할 언어를 의미하며 `kotlin, java`중에서 하나를 선택할 수 있습니다.<br>
`-i` 옵션은 적용할 플랫폼 중 iOS가 있을 경우 사용할 언어를 의미하며 `objc, swift`중 하나를 선택할 수 있습니다.

이번 포스트에서는 다트 패키지를 통해서 패키지를 개발하였습니다.

```terminal
$ flutter create --template=package flutter_heatmap_calendar
```

![Created Cli](/assets/uploads/2022-01-05/2.png)
_Figure 2_

성공적으로 생성이 완료된 경우 **Figure 2**와 같이 출력됩니다.

### example 폴더 생성하기

다트 패키지로 프로젝트를 생성했기 때문에 폴더 내에 예제를 보여줄 `example`폴더가 없으므로 아래 명령어를 통해 example 폴더 또한 생성합니다.

```terminal
$ cd flutter_heatmap_calendar
$ flutter create example
```

여기까지 준비하셨다면 본격적으로 패키지 개발을 시작할 수 있겠네요!

## Set Project.

### Directory 구조

![Directory Structure](/assets/uploads/2022-01-05/3.png)
_Figure 3_

위의 순서를 따라오셨으면 **Figure 3**와 같은 디렉토리 구조를 가지게 됩니다.

### example에 패키지 추가

![Add Package to Example](/assets/uploads/2022-01-05/4.png)
_Figure 4_

`exmaple/pubspec.yaml`에 자신이 생성한 패키지를 입력해줍니다.

### 패키지 개발하기

`/lib` 폴더에 들어가보면 프로젝트명과 동일한 dart 파일이 하나가 있습니다.<br>
깃허브를 둘러보면서 다른 개발자 분들이 어떻게 개발을 하는지 확인을 해보니 해당 파일은 `export`용으로만 사용하고 실질적인 개발은 `/lib/src` 디렉토리를 생성하여 내부에서 개발을 진행합니다.

![Develop Package](/assets/uploads/2022-01-05/5.png)
_Figure 5_

저 또한 동일하게 **Figure 5**같이 `src` 폴더를 추가로 생성해서 내부에서 개발을 했으며 프로젝트명과 동일한 dart 파일은 `export`용으로 사용하였습니다.

![export](/assets/uploads/2022-01-05/6.png)
_Figure 6_

이런 식으로 말이죠!

### pubspec.yaml 변경

개발을 완료 했으면 이제 `pubspec.yaml`을 수정해야 합니다.

```yaml
name: flutter_heatmap_calendar
description: A new Flutter package project.
version: 0.0.1
homepage:
```

기존에는 `1 ~ 4`번 라인에 위와 같이 적혀있을 것 입니다. 여기서 version은 `1.0.0`으로, homepage는 자신의 깃허브 프로젝트 url을 입력해주세요.

```yaml
name: flutter_heatmap_calendar
description: Flutter heatmap calendar inspired by github contribution chart which includes traditional mode / calendar mode.
version: 1.0.0
homepage: https://github.com/devappmin/flutter_heatmap_calendar
```

저는 다음과 같이 적었습니다!

### README, CHANGELOG 변경

README와 CHANGELOG 또한 자신의 프로젝트에 맞게 변경을 해주시면 됩니다.

README는 깃허브 뿐만이 아니라 pub.dev에서도 가장 첫 화면에 등장합니다.

![readme](/assets/uploads/2022-01-05/9.png)
_Figure 7_

CHANGELOG는 pub.dev에서 changelog 탭을 통해서 확인할 수 있는 변경사항을 넣어두는 부분입니다.

![changelog](/assets/uploads/2022-01-05/10.png)
_Figure 8_

### 라이센스 추가

1. 자신의 프로젝트 깃허브에 들어간 뒤 `LICENSE`를 클릭합니다.

   ![pencil](/assets/uploads/2022-01-05/7.png)
   _Figure 9_

2. 연필 모양 아이콘을 클릭합니다.

   ![Choose a license template](/assets/uploads/2022-01-05/8.png)
   _Figure 10_

3. **Choose a license template**를 클릭하여 자신이 원하는 라이센스를 클릭해줍니다.

## Publish it.

### --dry-run을 통한 오류 발견

pub.dev에 퍼블리싱을 하기 전에 아래 명령어를 통해서 현재 개발하고 있는 패키지에 오류가 있는지 확인할 수 있습니다.

```terminal
$ flutter pub publish --dry-run
```

만약에 `Package has 0 warnings. The server may enforce additional checks.`라고 뜨는 경우에는 다음 step으로 넘어가도 됩니다.

### pub.dev에 퍼블리싱 하기.

`--dry-run`을 무사히 통과하셨다면 해당 옵션을 빼고 퍼블리싱을 하시면 됩니다.

```terminal
$ flutter pub publish
```

중간에 최초 1회 pub.dev 계정으로 로그인하라고 뜨는데 그 때 로그인을 하시면 pub.dev에 등록이 됩니다.

### pub.dev에서 확인하기

퍼블리싱이 완료된 후에는 1시간 이내로 `pub points`와 `사용 가능한 플랫폼`, `Null safety` 여부를 자동으로 올려줍니다!

[Flutter Heatmap Calendar](https://pub.dev/packages/flutter_heatmap_calendar)

저 또한 제대로 잘 올라간 것을 확인할 수 있었습니다!
