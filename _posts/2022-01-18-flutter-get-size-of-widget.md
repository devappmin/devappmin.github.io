---
title: "[Flutter] 위젯의 크기 / 위치 값을 가져오는 법"
date: 2022-01-18 23:49 +0900
categories: [Mobile, Flutter]
tags: [Flutter]
---

## Before Start.

### 위젯의 크기 및 위치를 얻는 방법들

- `LayoutBuilder`
- `Global Key` 및 `RenderBox`를 활용
- [MeasuredSize](https://pub.dev/packages/measured_size)

이번 포스트에서는 `Global Key`를 통해서 구하는 방법을 알아보겠습니다.

## Getting Started.

### 기본 코드

```dart
class MyHomePage extends StatefulWidget {
  const MyHomePage({Key? key}) : super(key: key);

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('위젯 사이즈 구하기'),
      ),
      body: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        crossAxisAlignment: CrossAxisAlignment.center,
        children: <Widget>[
          Text("컨테이너 정보: "),
          Center(
            child: Container(
              color: Colors.blue,
              padding: const EdgeInsets.all(20),
              child: const Text("컨테이너입니다."),
            ),
          ),
        ],
      ),
    );
  }
}
```

![Figure 1](/assets/uploads/2022-01-18/1.png)

플러터 프로젝트를 새로 생성한 뒤에 크기 및 위치를 구하려고 하는 컨테이너와 사이즈 값을 출력할 텍스트를 배치 하였습니다.

여기서 이제 `Global Key`와 `RenderBox`를 활용하여 파란색 컨테이너의 크기와 위치를 구해보겠습니다.

### 크기 / 위치를 얻으려는 컨테이너에 Global Key 넣기

```dart
class _MyHomePageState extends State<MyHomePage> {
  final GlobalKey _containerkey = GlobalKey();
  // (... 생략 ...)
```

`Global Key`를 위와 같이 생성합니다. 키를 생성한 뒤에는 해당 키를 위젯에 부여해야겠죠?<br>
저는 파란색 컨테이너의 정보를 얻고 싶은 것이므로 해당 위젯에 값을 넣어주겠습니다.

```dart
class _MyHomePageState extends State<MyHomePage> {
  // (... 생략 ...)
  @override
  Widget build(BuildContext context) {
          // (... 생략 ...)
          child: Container(
            key: _containerKey,
            color: Colors.blue,
            padding: const EdgeInsets.all(20),
            child: const Text("컨테이너입니다."),
          ),
          // (... 생략 ...)
```

`key: _containerKey`로 키 값을 컨테이너에 넣었습니다. 이제 해당 키 값을 통해서 `RenderBox`를 생성하여 크기와 위치를 구해보겠습니다!

### 크기를 구하는 함수 만들기

```dart
class _MyHomePageState extends State<MyHomePage> {
  // (... 생략 ...)
  Size? _getSize() {
    if (_containerKey.currentContext != null) {
      final RenderBox renderBox =
          _containerKey.currentContext!.findRenderObject() as RenderBox;
      Size size = renderBox.size;
      return size;
    }
  }
  // (... 생략 ...)
```

`Global Key`를 통해서 생성한 파란색 컨테이너의 RenderBox를 구하고 위젯의 크기를 리턴합니다. 크기는 `RenderBox`의 `size`를 통해서 구할 수 있습니다.

### 위치를 구하는 함수 만들기

```dart
class _MyHomePageState extends State<MyHomePage> {
  // (... 생략 ...)
  Offset? _getOffset() {
    if (_containerKey.currentContext != null) {
      final RenderBox renderBox =
          _containerKey.currentContext!.findRenderObject() as RenderBox;
      Offset offset = renderBox.localToGlobal(Offset.zero);
      return offset;
    }
  }
  // (... 생략 ...)
```

위치를 구하는 방법은 크기를 구하는 방법이랑 거의 유사합니다. 위치는 `RenderBox`의 `localToGlobal()` 메소드를 통해서 구할 수 있습니다.

두 값을 한 번에 구하는 함수를 생성해도 되지만 이번 포스트에서는 함수를 따로 제작하여 진행하려고 합니다!

### 함수 호출

당연한 이야기지만 위젯의 크기나 위치의 값을 받아오기 위해서는 위젯이 먼저 화면에 존재해야 합니다. 따라서 위젯이 존재하지 않는데 크기나 위치를 구하면 해당 값은 `null`으로 반환됩니다.

![Figure 2](/assets/uploads/2022-01-18/2.png)

하지만 `build`는 적합하지 않고 `initState`나 `didChangeDependencies`는 화면에 위젯을 그리기 전이니 어디서 호출을 해야할까요..

정말 간단하게 `WidgetsBinding`의 `addPostFrameCallback()`을 통해서 해결 가능합니다. 해당 콜백 함수는 위젯이 바인딩 된 후에 호출하는 함수로 해당 함수 내에서 위젯의 크기 및 위치를 받는 함수를 호출하면 되겠네요!

```dart
class _MyHomePageState extends State<MyHomePage> {
  // (... 생략 ...)
  Size? size;
  Offset? offset;

  @override
  void initState() {
    super.initState();
    WidgetsBinding.instance!.addPostFrameCallback((_) {
      setState(() {
        size = _getSize();
        offset = _getOffset();
      });
    });
  }
  // (... 생략 ...)
```

크기 값을 저장할 `size` 변수와 위치 값을 저장할 `offset` 변수를 생성한 후 `initState()`에서 `WidgetsBinding.instance!.addPostFrameCallback((_) {...});`를 통해 위젯이 바인딩 된 후에 `size`와 `offset`을 받아오도록 하였습니다.

### 값을 잘 받아왔는지 확인

```dart
Text("컨테이너 정보\nwidth:${size?.width}\nheight:${size?.height}\ndx:${offset?.dx}\ndy:${offset?.dy}"),
```

파란색 컨테이너의 값을 잘 받아왔는지 확인하기 위해서 텍스트에다가 사이즈 값과 위치 값을 출력 시켜보겠습니다.

![Figure 3](/assets/uploads/2022-01-18/3.png)

값이 제대로 잘 출력이 되네요!

## Conclusion

### 전체 코드

코드 전체를 보여드리면서 마무리 하겠습니다.

```dart
class MyHomePage extends StatefulWidget {
  const MyHomePage({Key? key}) : super(key: key);

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  final GlobalKey _containerKey = GlobalKey();
  Size? size;
  Offset? offset;

  @override
  void initState() {
    super.initState();
    WidgetsBinding.instance!.addPostFrameCallback((_) {
      setState(() {
        size = _getSize();
        offset = _getOffset();
      });
    });
  }

  Size? _getSize() {
    if (_containerKey.currentContext != null) {
      final RenderBox renderBox =
          _containerKey.currentContext!.findRenderObject() as RenderBox;
      Size size = renderBox.size;
      return size;
    }
  }

  Offset? _getOffset() {
    if (_containerKey.currentContext != null) {
      final RenderBox renderBox =
          _containerKey.currentContext!.findRenderObject() as RenderBox;
      Offset offset = renderBox.localToGlobal(Offset.zero);
      return offset;
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('위젯 사이즈 구하기'),
      ),
      body: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        crossAxisAlignment: CrossAxisAlignment.center,
        children: <Widget>[
          Text(
              "컨테이너 정보\nwidth:${size?.width}\nheight:${size?.height}\ndx:${offset?.dx}\ndy:${offset?.dy}"),
          Center(
            child: Container(
              key: _containerKey,
              color: Colors.blue,
              padding: const EdgeInsets.all(20),
              child: const Text("컨테이너입니다."),
            ),
          ),
        ],
      ),
    );
  }
}

```

### References

[How to get height of a widget](https://stackoverflow.com/questions/49307677/how-to-get-height-of-a-widget)

해당 Stack Overflow 글을 참조하였습니다.
