---
title: "[Flutter] BLoC 패턴을 케이크처럼 쉽게 이해할 수 있는 글"
date: 2022-02-19 20:50 +0900
categories: [Mobile, Flutter]
tags: [Flutter, BLoC]
---

## Before Start.

<img title="" src="/assets/uploads/2022-02-19/1.png" alt="BLoC Logo" width="191">

이 포스트에서는 BLoC 패턴을 이해하기 어려운 개발자 분들에게 매우 쉽게 BLoC을 풀어서 설명하려고 합니다.

### 그래서 BLoC 패턴이 도대체 뭐야?

`BLoC`은 `Business Logic Component`로 구글 개발자가 개발한 상태관리 아키텍처 패턴입니다.

이 부분은 다른 블로그에서 쉽게 찾아볼 수 있으므로 길게 설명하지 않겠습니다.

### BLoC 패턴을 써야하는 이유

솔직히 크기가 작은 앱을 개발하다보면 아키텍처 패턴과 상태관리가 별로 중요한 것처럼 느껴지지 않습니다.

하지만 개발하는 앱의 사이즈가 커질수록 아키텍처 패턴과 상태관리는 매우 중요한 부분 중 한 곳을 차지하며 보다 멋있는 개발자가 되기 위해서는 꼭 필요한 요소 중 하나라고 생각합니다.

또한 비즈니스 로직과 UI를 분리했기 때문에 코드가 상당히 깔끔해지고 보기 좋아집니다.

### 이번 포스트에서 만들어 볼 프로젝트

BLoC 패턴을 통해서 `pub.dev`에 올라온 패키지들을 크롤링하여 리스트로 출력하는 프로젝트를 만들어보겠습니다.

### References

아래 링크에 완성된 코드를 올려놨으니 참고하시면서 보면 이해하기 훨씬 쉬울겁니다.

[devappmin/BLoC_Practice](https://github.com/devappmin/BLoC_Practice)

## Getting Started.

### BLoC 패턴의 모습

![BLoC Pattern](/assets/uploads/2022-02-19/2.png)

`BLoC`패턴은 위 그림 한 장으로 설명이 가능합니다.

쉽게 이해가 안 갈 수도 있으니 이번에 저희가 만들 `pub.dev` 크롤링 앱 개발을 예로 들어보겠습니다.

1. **events**

   - 화면이 띄어졌을 때 `pub.dev`에 있는 패키지를 불러오기.
   - FAB을 눌러서 현재 패키지 리스트에다가 값 추가로 불러오기.

2. **states**

   - 패키지가 없고 빈 상태일 때 (Empty)
   - 크롤링을 통해서 값을 가져오는 중이라 로딩 중일 때 (Loading)
   - 값을 성공적으로 불러왔을 때 (Loaded)
   - 값을 불러오던 도중 에러가 발생했을 때 (Error)

3. **request / response**

   - 크롤링을 하기 위해서 웹에 요청을 하고 해당 값을 불러왔을 때

### 사용할 패키지

이번 프로젝트를 진행하기 위해서 사용한 패키지는 다음과 같습니다.

- http
- html
- flutter_bloc
- equatable
- build_runner
- freezed
- json_serializable

### 프로젝트 구조

```
lib
│   main.dart
│
├───bloc
│       package_bloc.dart
│       package_event.dart
│       package_state.dart
│
├───domain
│   ├───colors
│   │       colors.dart
│   │
│   └───constant
│           pub_dev.dart
│           score_value.dart
│
├───model
│       package.dart
│       package.freezed.dart
│       package.g.dart
│
└───repository
        package_repository.dart
```

이제 차근차근 개발을 진행해보겠습니다.

## Code it.

### Domain

#### colors/colors.dart

```dart
import 'package:flutter/widgets.dart';

class ViewColors {
  static const Color primaryColor = Color(0xFF0175c2);
  static const Color primaryDarkColor = Color(0xFF00539b);

  static const Color primaryTextColor = Color(0xFF4A4A4A);
  static const Color secondaryTextColor = Color(0xFF6d7278);
}
```

`colors/colors.dart`에는 제가 사용하고 싶은 색깔들을 저장했습니다.

#### constant/pub_dev.dart

```dart
class PubDev {
  static const String pubDevUrl = 'https://pub.dev/packages';
  static const String packageQuery = '?page=';

  static Uri packageUrl(int page) =>
      Uri.parse(pubDevUrl + packageQuery + page.toString());
}
```

`pub.dev`에서 크롤링하기 위해서 링크를 저장했습니다. `page` 쿼리에 따라서 페이지가 이동하기 때문에 페이지 번호를 입력 받으면 Uri를 리턴하는 함수도 하나 생성했습니다.

#### constant/score_value.dart

```dart
class ScoreValue {
  static const String likes = "LIKES";
  static const String pubPoints = "PUB POINTS";
  static const String popularity = "POPULARITY";
}
```

따봉 / pub 점수 / 인기도 스트링을 저장해놓은 클래스입니다.

### Model

`Model`은 저희가 `pub.dev`에서 가져올 패키지에서 필요한 값을 저장할 `package.dart`를 생성했습니다.

#### package.dart

```dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'package.freezed.dart';
part 'package.g.dart';

@freezed
class Package with _$Package {
  factory Package({
    required String name,
    required String description,
    required String version,
    required bool nullSafety,
    required int likes,
    required int pubPoints,
    required int popularity,
  }) = _Package;

  factory Package.fromJson(Map<String, dynamic> json) =>
      _$PackageFromJson(json);
}
```

패키지에서 가져올 값은 아래와 같습니다.

- **name**: 패키지의 이름
- **description**: 패키지의 상세 내용
- **version**: 패키지 버전
- **nullSafety**: Null Safety 여부
- **likes**: 따봉 개수
- **pubPoints**: Pub 점수
- **popularity**: 인기도

또한 `package.dart`는 `freezed`를 통해서 Model을 생성할 것이기 때문에 `with _$Package`를 넣어주고 `JSON`을 통해서 값을 불러올 것이기 때문에 `fromJson`을 생성합니다.

그 후에는 아래 커맨드를 통해서 `package.freezed.dart`와 `package.g.dart`를 생성합니다.

```bash
$ flutter pub run build_runner build
```

이를 통해서 Model 생성을 완료하였습니다.

### Repository

`Repository`는 Web에서 데이터를 가져오고 이를 처리하는 것을 담당합니다.

이번 프로젝트에서는 크롤링을 통해서 값을 가져오고 리턴하는 부분이 필요하겠네요!

#### package_repository.dart

```dart
import 'package:bloc_practice/domain/constant/pub_dev.dart';
import 'package:html/dom.dart';
import 'package:html/parser.dart' as parser;
import 'package:http/http.dart' as http;

class PackageRepository {
  Future<List<Map<String, dynamic>>> getPackages(int page) async {
    // Get the packages list with the given page number on pub.dev
    final response = await http.get(
      PubDev.packageUrl(page),
      headers: {
        'Accept': 'application/json',
      },
    );

    // Check if the response is successful
    if (response.statusCode == 200) {
      Document document = parser.parse(response.body);
      List<Element> packageElements =
          document.querySelectorAll('.packages-item');

      List<Map<String, dynamic>> packages = [];

      for (var pkgs in packageElements) {
        Element? name = pkgs.querySelector('.packages-title');
        Element? description = pkgs.querySelector('.packages-description');
        Element? version = pkgs.querySelector('.packages-metadata-block');
        Element? nullSafety = pkgs.querySelector('.package-badge');
        Element? likes = pkgs
            .querySelector('.packages-score-like .packages-score-value-number');
        Element? pubPoints = pkgs.querySelector(
            '.packages-score-health .packages-score-value-number');
        Element? popularity = pkgs.querySelector(
            '.packages-score-popularity .packages-score-value-number');

        packages.add({
          'name': name?.text,
          'description': description?.text,
          'version': version?.text,
          'nullSafety': nullSafety == null ? false : true,
          'likes': int.parse(likes?.text.toString() ?? '0'),
          'pubPoints': int.parse(pubPoints?.text.toString() ?? '0'),
          'popularity': int.parse(popularity?.text.toString() ?? '0'),
        });
      }

      return packages;
    } else {
      // If the response is not successful, throw an error
      throw Exception('Failed to load packages');
    }
  }
}
```

`pub.dev`에서 크롤링을 하고 가져온 패키지들을 리턴하는 함수입니다. 오늘은 크롤링이 아니라 `BLoC`을 배우는 것이 목적이므로 간단하게 설명하겠습니다.

`http`를 통해서 `pub.dev`에 요청을 보냅니다. 그 후, `parser.parse`를 통해서 파싱을 한 다음에 패키지 리스트에 값을 저장하고 이를 리턴합니다.

### BLoC

`BLoC` 아키텍처는 기본적으로 **xxxx_bloc.dart, xxxx_state.dart, xxxx_event.dart**(xxxx는 BLoC 이름) 파일을 생성해야 합니다.

처음에는 `state`부터 살펴보겠습니다.

#### package_state.dart

```dart
part of 'package_bloc.dart';

@immutable
abstract class PackageState extends Equatable {}

class Empty extends PackageState {
  @override
  List<Object?> get props => [];
}

class Loading extends PackageState {
  @override
  List<Object?> get props => [];
}

class Loaded extends PackageState {
  final List<Package> packages;

  Loaded({required this.packages});

  @override
  List<Object?> get props => [packages];
}

class Error extends PackageState {
  final String message;

  Error({required this.message});

  @override
  List<Object?> get props => [message];
}
```

`package_event.dart`는 패키지를 불러오는 상태를 관리하는 클래스들을 저장합니다.

일단 모든 클래스의 부모가 되는 `PackageState` abstract 클래스를 생성하고 `Equatable`을 상속받습니다.

`Equatable`가 뭔지 모르시면 [[Flutter] Equatable, 이렇게 좋은데도 안 써?](https://petabyte.studio/posts/what-is-equatable/)를 참조해주세요.

저희는 위에 언급했듯이 4개의 state를 생성할 것입니다.

- 패키지가 없고 빈 상태일 때 (Empty)
- 크롤링을 통해서 값을 가져오는 중이라 로딩 중일 때 (Loading)
- 값을 성공적으로 불러왔을 때 (Loaded)
- 값을 불러오던 도중 에러가 발생했을 때 (Error)

`Loaded` 상태에서는 불러온 패키지의 리스트를 가져올 수 있게 하였고, `Error`가 발생했으면 에러 메시지를 가지고 올 수 있게 하였습니다.

#### package_event.dart

```dart
part of 'package_bloc.dart';

@immutable
abstract class PackageEvent extends Equatable {}

class GetPackageEvent extends PackageEvent {
  final int pages;

  GetPackageEvent({
    required this.pages,
  });

  @override
  List<Object?> get props => [pages];
}

class AppendPackageEvent extends PackageEvent {
  final int pages;

  AppendPackageEvent({
    required this.pages,
  });

  @override
  List<Object?> get props => [pages];
}
```

`package_event.dart`역시 `Equatable`을 상속받은 PackageEvent를 통해서 두 개의 이벤트를 생성해줍니다.

이 또한 위에서 언급했듯이 아래 두 기능을 구현하기 위해서 사용하려고 합니다.

- 화면이 띄어졌을 때 `pub.dev`에 있는 패키지를 불러오기. (GetPackageEvent)
- FAB을 눌러서 현재 패키지 리스트에다가 값 추가(Append)로 불러오기. (AppendPackageEvent)

#### package_bloc.dart

```dart
import 'package:bloc/bloc.dart';
import 'package:bloc_practice/model/package.dart';
import 'package:bloc_practice/repository/package_repository.dart';
import 'package:equatable/equatable.dart';
import 'package:freezed_annotation/freezed_annotation.dart';

part 'package_event.dart';
part 'package_state.dart';

class PackageBloc extends Bloc<PackageEvent, PackageState> {
  final PackageRepository packageRepository;

  PackageBloc({required this.packageRepository}) : super(Empty()) {
    on<GetPackageEvent>(_onGetPackageEvent);
    on<AppendPackageEvent>(_onAppendPackageEvent);
  }

  void _onGetPackageEvent(
      GetPackageEvent event, Emitter<PackageState> emit) async {
    try {
      emit(Loading());

      final resp = await packageRepository.getPackages(event.pages);

      final packages = resp
          .map<Package>(
            (e) => Package.fromJson(e),
          )
          .toList();

      emit(Loaded(packages: packages));
    } catch (e) {
      emit(Error(message: e.toString()));
    }
  }

  void _onAppendPackageEvent(
      AppendPackageEvent event, Emitter<PackageState> emit) async {
    try {
      if (state is Loaded) {
        final parsedState = (state as Loaded);

        final resp = await packageRepository.getPackages(event.pages);
        final packages = resp
            .map<Package>(
              (e) => Package.fromJson(e),
            )
            .toList();

        final prevPackages = [
          ...parsedState.packages,
        ];

        final newPackages = [
          ...prevPackages,
          ...packages,
        ];

        emit(Loaded(packages: newPackages));
      }
    } catch (e) {
      emit(Error(message: e.toString()));
    }
  }
}
```

이제 대망의 `BLoC` 클래스입니다. 클래스를 하나씩 뜯어보면서 살펴볼까요?

```dart
class PackageBloc extends Bloc<PackageEvent, PackageState> {
```

클래스를 생성할 때 `Bloc<Event 클래스, State 클래스>`를 상속 받아줍니다.

```dart
final PackageRepository packageRepository;
```

저희가 아까 만들었던 `pub.dev`를 크롤링하는 클래스의 인스턴스를 생성해줍니다.

```dart
  PackageBloc({required this.packageRepository}) : super(Empty()) {
    on<GetPackageEvent>(_onGetPackageEvent);
    on<AppendPackageEvent>(_onAppendPackageEvent);
  }
```

Constructor를 생성해줍니다. 이 떄, `super()`에 초기 값을 넣어줘야 하는데 저희는 아무 것도 없는 상태를 넣을 것 이므로 `Empty`를 넣어주겠습니다. 또한, packageRepository를 props로 받고 싶으므로 이를 넣어줍니다.

저희는 아직 이벤트를 받으면 처리하는 부분을 만들지 않았잖아요?

패키지를 받는 이벤트를 받았으면 이를 처리해주고, 현재 패키지에 새로 받은 패키지를 이어주는 이벤트를 받았으면 이를 처리해주는 로직이 필요한데 아직 저희가 그거를 구현하지 않았거든요.

이 부분은 `on<GetPackageEvent>();`에서 해결이 가능합니다. `on`의 기본적인 문법은 아래와 같습니다.

```dart
on<EventName>((event, emit) {
   // Logic here..
});
```

이 말은 즉슨 익명 함수를 밖으로 꺼내서도 사용할 수 있다는 의미겠죠? 저희는 더 깔끔하게 코드를 보기 위해서 로직 부분을 담당하는 함수를 두 개 생성해주고 함수 이름을 대신 적어주도록 합니다.

```dart
  void _onGetPackageEvent(
      GetPackageEvent event, Emitter<PackageState> emit) async {
    try {
      emit(Loading());

      final resp = await packageRepository.getPackages(event.pages);

      final packages = resp
          .map<Package>(
            (e) => Package.fromJson(e),
          )
          .toList();

      emit(Loaded(packages: packages));
    } catch (e) {
      emit(Error(message: e.toString()));
    }
  }
```

저희가 꺼낸 로직은 다음과 같습니다.

함수를 살펴보면 `GetPackageEvent`의 event와 `Emitter<PackageState>`의 emit을 파라미터로 가지는 것을 확인할 수 있습니다.

`event`는 말 그대로 이 함수를 호출한 이벤트를 의미하고 `emit`은 하위 위젯에서 상위 위젯으로 이벤트를 전달하기 위한 것입니다.

간단하게 표현하면 현재 해당 BLoC을 사용중인 위젯에서 받는 `state`를 변경하는 부분이라고 하면 되겠네요.

일단 무지성 try-catch로 에러를 잡아주도록 하겠습니다. 그리고 에러가 발생할 경우 `emit`을 통해서 `Error` state로 변경하고 message prop에 에러 값을 넣어줍니다.

그게 아닐 경우, `emit`을 통해서 `Loading` state로 변경해줍니다.

로딩 상태에 들어간 이후에는 저희가 아까 만들었던 `packageRepository` 오브젝트에서 패키지 리스트를 불러옵니다. 이 때, `event.pages` (GetPackageEvent의 멤버)를 통해서 페이지 값을 가져와서 넣어줍니다.

`resp`에서 받은 `JSON`값을 packages 변수에 넣어주고 `Loaded` state로 변경을 해줍니다. 이 때, `package` prop에 지금 정보를 갖고 있는 `package` 변수를 넣어줍니다.

```dart
  void _onAppendPackageEvent(
      AppendPackageEvent event, Emitter<PackageState> emit) async {
    try {
      if (state is Loaded) {
        final parsedState = (state as Loaded);

        final resp = await packageRepository.getPackages(event.pages);
        final packages = resp
            .map<Package>(
              (e) => Package.fromJson(e),
            )
            .toList();

        final prevPackages = [
          ...parsedState.packages,
        ];

        final newPackages = [
          ...prevPackages,
          ...packages,
        ];

        emit(Loaded(packages: newPackages));
      }
    }
```

`_onAppendPackageEvent`도 거의 동일합니다. 하지만, 이는 `state`가 `Loaded` state 일 때만 동작을 하고, `Loaded` state에서 가져온 값과 현재 값을 이어서 다시 넣어주는 방식으로 진행합니다.

### UI

#### main.dart

```dart
import 'package:bloc_practice/bloc/package_bloc.dart';
import 'package:bloc_practice/domain/constant/score_value.dart';
import 'package:bloc_practice/repository/package_repository.dart';
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

import 'domain/colors/colors.dart';
import 'model/package.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'BLoC Practice',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: BlocProvider(
        create: (context) =>
            PackageBloc(packageRepository: PackageRepository()),
        child: const MyHomePage(),
      ),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({Key? key}) : super(key: key);

  @override
  State<StatefulWidget> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  int pages = 1;

  @override
  void initState() {
    super.initState();

    BlocProvider.of<PackageBloc>(context).add(GetPackageEvent(pages: pages));
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('BLoC Practice'),
      ),
      body: BlocBuilder<PackageBloc, PackageState>(
        builder: (context, state) {
          if (state is Empty) {
            return Container();
          } else if (state is Loading) {
            return const Center(
              child: CircularProgressIndicator(),
            );
          } else if (state is Loaded) {
            final packages = state.packages;

            return ListView.separated(
              itemCount: packages.length,
              itemBuilder: (context, index) {
                final package = packages[index];
                return _ListTile(package: package);
              },
              separatorBuilder: (context, index) => const Divider(),
            );
          } else if (state is Error) {
            return Text(state.message);
          }

          return Container();
        },
      ),
      floatingActionButton: FloatingActionButton(
        child: const Icon(Icons.add),
        onPressed: () {
          context.read<PackageBloc>().add(
                AppendPackageEvent(pages: ++pages),
              );
        },
      ),
    );
  }
}

class _ListTile extends StatelessWidget {
  final Package package;

  const _ListTile({required this.package});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(8.0),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: <Widget>[
          Text(
            package.name,
            style: const TextStyle(
              fontSize: 18,
              color: ViewColors.primaryColor,
              fontWeight: FontWeight.bold,
            ),
          ),
          const SizedBox(height: 8.0),
          Text(
            package.description,
            style: const TextStyle(
              color: ViewColors.primaryTextColor,
            ),
          ),
          const SizedBox(height: 8.0),
          Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              Column(
                children: <Widget>[
                  Text(
                    package.likes.toString(),
                    style: const TextStyle(
                      color: ViewColors.primaryColor,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const Text(
                    ScoreValue.likes,
                    style: TextStyle(fontSize: 10),
                  ),
                ],
              ),
              const SizedBox(width: 16),
              Column(
                children: <Widget>[
                  Text(
                    package.pubPoints.toString(),
                    style: const TextStyle(
                      color: ViewColors.primaryColor,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const Text(
                    ScoreValue.pubPoints,
                    style: TextStyle(fontSize: 10),
                  ),
                ],
              ),
              const SizedBox(width: 16),
              Column(
                children: <Widget>[
                  Text(
                    package.popularity.toString() + '%',
                    style: const TextStyle(
                      color: ViewColors.primaryColor,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const Text(
                    ScoreValue.popularity,
                    style: TextStyle(fontSize: 10),
                  ),
                ],
              ),
            ],
          ),
          const SizedBox(height: 16.0),
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: <Widget>[
              Text(
                package.version,
                style: const TextStyle(
                  color: ViewColors.primaryColor,
                  fontSize: 10,
                ),
              ),
              package.nullSafety
                  ? Container(
                      child: const Text(
                        "Null safety",
                        style: TextStyle(
                          color: ViewColors.primaryColor,
                          fontSize: 10,
                        ),
                      ),
                      margin: const EdgeInsets.only(left: 8),
                      padding: const EdgeInsets.symmetric(
                          horizontal: 8, vertical: 6),
                      decoration: BoxDecoration(
                        borderRadius: const BorderRadius.all(
                          Radius.circular(20),
                        ),
                        border: Border.all(color: ViewColors.primaryColor),
                      ),
                    )
                  : Container(),
            ],
          ),
        ],
      ),
    );
  }
}
```

`_ListTile` 클래스는 리스트 타일을 의미하는 클래스이므로 대충 보고 넘겨도 됩니다.

저희가 중요하게 봐야할 알짜배기들만 모아서 아래에 설명하겠습니다.

```dart
class MyApp extends StatelessWidget {
  // More codes here..
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      // More codes here..
      home: BlocProvider(
        create: (context) =>
            PackageBloc(packageRepository: PackageRepository()),
        child: const MyHomePage(),
      ),
  // More codes here..
}
```

`BLoC`을 사용하기 위해서는 `BLoC`을 사용하고자 하는 위젯 상단에 `BlocProvider`를 넣어줘야 합니다.

여기서는 사용할 `bloc` 클래스를 `create`에 정의해줘야하는데 저희는 `PackageBloc`으로 하겠습니다.

이 후, 홈 위젯이 생성이 되면 `event`를 호출해줍니다.

```dart
  @override
  void initState() {
    super.initState();
    BlocProvider.of<PackageBloc>(context).add(GetPackageEvent(pages: pages));
  }
```

초기 page의 값을 1로 두고 해당 페이지 값을 prop 값으로 넘겨 `GetPackageEvent`를 호출해주었습니다.

이를 처리하기 위해서는 `BlocProvider.of<>().add(Event)`를 호출해야 합니다.

또한, FAB를 누를 때 마다 `AppendPackageEvent`를 호출해주기 위해서 이 값을 넣어준 것입니다.

```dart
      floatingActionButton: FloatingActionButton(
        child: const Icon(Icons.add),
        onPressed: () {
          context.read<PackageBloc>().add(
                AppendPackageEvent(pages: ++pages),
              );
        },
      ),
```

이제 BLoC State에 따라서 값을 출력해주기만 하면 됩니다.

```dart
      body: BlocBuilder<PackageBloc, PackageState>(
        builder: (context, state) {
          if (state is Empty) {
            return Container();
          } else if (state is Loading) {
            return const Center(
              child: CircularProgressIndicator(),
            );
          } else if (state is Loaded) {
            final packages = state.packages;

            return ListView.separated(
              itemCount: packages.length,
              itemBuilder: (context, index) {
                final package = packages[index];
                return _ListTile(package: package);
              },
              separatorBuilder: (context, index) => const Divider(),
            );
          } else if (state is Error) {
            return Text(state.message);
          }

          return Container();
        },
      ),
```

`BLoCBuilder`를 통해서 `state`에 따라 값을 출력하려고 합니다.

저희는 아래와 같이 생성할게요.

- state가 `Empty`일 경우, 빈 공간 생성

- state가 `Loading`일 경우, 로딩하는 동글동글한 프로그래스 인디케이터 생성

- state가 `Loaded`일 경우, `Loaded` state의 멤버인 `packages`값을 불러와서 그 값을 토대로 리스트를 생성

- state가 `Error`일 경우 텍스트에 오류 출력

이게 끝입니다. 생각보다 간단하고 어렵지 않죠?

## Conclusion

### 완성된 앱의 모습

![App](/assets/uploads/2022-02-19/3.png)

### 진짜 끝

지금까지 BLoC 패턴을 알아보았습니다. BLoC은 비즈니스 로직과 UI를 나누는 부분에 있어서 좋은 점이 있지만, 이러한 로직마다 최소한 3개의 클래스를 만들어야 하니까 클래스의 개수가 방대하게 늘어날 수 밖에 없는 단점이 있습니다.

고로 다음에는 BLoC의 보다 간단한 `Provider`를 준비해오겠습니다!
