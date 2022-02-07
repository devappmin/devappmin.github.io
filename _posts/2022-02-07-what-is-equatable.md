---
title: "[Flutter] Equatable, 이렇게 좋은데도 안 써?"
date: 2022-02-07 21:12 +0900
categories: [Mobile, Flutter]
tags: [Flutter, Equatable]
---

## Before Start.

### Equatable 패키지를 어쩌다 알게 되었나

`BLoC` 아키텍처 패턴을 공부하기 위해서 다른 개발자 분들의 코드를 살펴보던 중, 많은 분들이 **state, event** 클래스를 생성할 때 `Equatable` 클래스를 상속받는 것을 보고 처음에는 아는 것 없이 무지성으로 저도 `Equatable`을 상속받아서 생성 했었습니다.

하지만 아무것도 모르면서 쓰는 것 보다는 알고 사용하는 것이 훨씬 좋겠죠? 결국 궁금증에 못 이겨 `Equatable`에 대해서 알아본 결과 생각한 것 보다 좋은 패키지라는 것을 알 수 있었습니다.

### 그래서 Equatable이 뭔데?

결론부터 알려드리자면 두 개의 인스턴스가 같은 인스턴스인지를 쉽게 판단해주는 패키지입니다.

## Getting Started.

### Equatable을 사용하지 않을 시

#### 클래스 선언

```dart
class Shoes {
  final int productId;
  final String name;
  final String brand;
  final int size;

  Shoes({
    required this.productId,
    required this.name,
    required this.brand,
    required this.size,
  });
}
```

위와 같은 `Shoes` 클래스를 생성하고 아래 두 개의 인스턴스를 생성했습니다.

#### 인스턴스 생성

```dart
final myJordan1 = Shoes(productId: 10000, name: "Jordan 1", brand: "Nike", size: 280);
final yourJordan1 = Shoes(productId: 10000, name: "Jordan 1", brand: "Nike", size: 280);
```

위와 같이 `myJordan1`과 `yourJordan1`을 생성했습니다.

#### 내꺼 == 니꺼??

위 두 인스턴스는 모두 같은 값을 가지고 있으니까 `myJordan1 == yourJordan1`을 하면 `true`가 나올까요? 아닙니다. 두 값은 다른 메모리에 저장되어 있는 독립된 객체들이기 때문에 두 값을 비교하면 `false`가 나오게 됩니다.

#### 두 값이 같은지 확인하는 방법

그러면 두 인스턴스가 같은지 확인을 하려면 어떻게 해야할까요? 다른 언어들처럼 `operator`를 `override`하여 두 인스턴스를 비교할 수 있습니다.

```dart
class Shoes {
  final int productId;
  final String name;
  final String brand;
  final int size;

  Shoes({
    required this.productId,
    required this.name,
    required this.brand,
    required this.size,
  });

  @override
  bool operator ==(Object other) =>
      other is Shoes &&
      other.productId == productId &&
      other.name == name &&
      other.brand == brand &&
      other.size == size;

  @override
  int get hashCode => productId;
}

```

`operator ==`에서 `productId`와 `name`, `brand`, `size`가 같을 경우 `true`를 리턴하게 했습니다. 이를 통해서 `myJordan1 == yourJordan1`이 `true`를 리턴할 수 있겠네요!

#### hashCode 함수는 뭐야?

근데 `operator` 이외에도 `override`된 것이 하나 더 보이네요. 아마 직접 `operator`를 넣으면 파란색 밑줄이 생기면서 아래와 같이 `hashCode`도 정의하라고 합니다.

```
Override `hashCode` if overriding `==`.
Implement `hashCode`.
```

`hashCode`는 `Map`이나 `Set`과 같이 중복되는 값이 들어가지 못할 때 그 중복되는 것을 의미하는 **Key**값을 정의할 수 있습니다.

이렇게 하면 어렵지 않게 두 인스턴스를 비교할 수 있습니다! 그러면 도대체 `Equatable`은 왜 쓰는 걸까요?? 이보다 더 쉽게 비교할 수 있기 때문입니다! 😄

### Equatable을 사용하여 해결

`Equatable`을 사용하기 위해서는 `Equatable`을 우선 추가 해야겠죠? 터미널에 아래 커멘드를 입력하여 설치를 할 수 있습니다!

```bash
$ flutter pub add equatable
```

#### 클래스 생성

위에 있던 `Shoes` 클래스에서 `operator`와 `hashCode`를 삭제한 후 `equatable`을 상속 받겠습니다.

또한 상속을 받으면 함수 하나를 `override`하라고 하면서 빨간 줄이 뜨기 때문에 해당 함수도 overriding 해줄게요!

```dart
class Shoes extends Equatable {
  final int productId;
  final String name;
  final String brand;
  final int size;

  Shoes({
    required this.productId,
    required this.name,
    required this.brand,
    required this.size,
  });

  @override
  // TODO: implement props
  List<Object?> get props => throw UnimplementedError();
}
```

#### 비교할 값들을 props에 추가

저는 `productId`, `name`, `brand`, `size`가 전부 같아야지만 동일한 인스턴스라고 판단할거기 때문에 이 값들을 props에 추가했습니다! :)

```dart
class Shoes extends Equatable {
  final int productId;
  final String name;
  final String brand;
  final int size;

  Shoes({
    required this.productId,
    required this.name,
    required this.brand,
    required this.size,
  });

  @override
  List<Object?> get props => [productId, name, brand, size];
}
```

이러면 끝입니다! `myJordan1 == yourJordan1`을 확인해보면 `true`인 것을 확인할 수 있었습니다.

## Conclusion

### References

[equatable | Dart Package (pub.dev)](https://pub.dev/packages/equatable)

[felangel/equatable: A Dart package that helps to implement value based equality without needing to explicitly override == and hashCode. (github.com)](https://github.com/felangel/equatable)
