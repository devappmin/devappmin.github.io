---
title: "[JS] 바닐라 JS에서 정말 간단한 MVVM + Repository 패턴 사용하기"
date: 2022-11-25 00:09 +0900
categories: [Develop, Javascript]
tags: [Javascript]
---

# 🛫 시작하기에 앞서

![Vanilla JS Thumbnail](/uploads/vanilla-js-thumbnail.png)

간단한 바닐라 자바스크립트 프로젝트를 진행하면서 바닐라 자바스크립트에서의 **MVVM + Repository** 패턴을 학습한 내용을 기록하려고 합니다.

## 🤔 왜 하필 MVVM?

처음에는 안드로이드를 개발하면 많이 사용하는 **AAC-MVVM**(AAC-ViewModel을 활용한 MVVM) 패턴에 익숙해서 바닐라 자바스크립트 역시 MVVM으로 개발을 시작하였습니다. 하지만, AAC-MVVM과 Pure MVVM에는 차이가 있었고 Pure MVVM을 개발해본 적이 없어 AAC-ViewModel과 일반 ViewModel의 차이를 또한 공부해보고 싶어 본 프로젝트를 진행하였습니다!

## 🤨 AAC ViewModel vs MVVM ViewModel

일단 AAC의 ViewModel과 MVVM의 ViewModel은 이름만 같을 뿐 완전히 다른 ViewModel입니다. 하지만, AAC ViewModel을 활용해서 MVVM을 구현할 수도 있기 때문에 AAC ViewModel을 쓰면 MVVM이 아니라고 할 수도 없습니다!

### AAC ViewModel

> ViewModel의 대안은 UI에 표시되는 데이터를 보유하는 일반 클래스입니다. 이는 활동이나 탐색 대상 간에 이동할 때 문제가 될 수 있습니다. 이렇게 하면 [인스턴스 상태 저장 메커니즘](https://developer.android.com/topic/libraries/architecture/saving-states#onsaveinstancestate)을 사용하여 데이터를 저장하지 않을 경우 해당 데이터가 소멸됩니다. ViewModel은 데이터 지속성을 위한 편리한 API를 제공하여 이 문제를 해결합니다.
> 
> ViewModel 클래스의 주요 이점은 기본적으로 두 가지입니다.
> -   UI 상태를 유지할 수 있습니다.
> -   비즈니스 로직에 대한 액세스 권한을 제공합니다.
> 
>  [ViewModel 개요](https://developer.android.com/topic/libraries/architecture/viewmodel)

Google 공식 문서를 확인해보면 알 수 있듯이, MVVM 패턴에 대한 설명 없이 AAC(Android Architecture Component)의 ViewModel 만을 알려주는 것을 알 수 있습니다.

AAC ViewModel은 Android의 수명 주기를 고려할 때 UI와 관련된 데이터들을 저장하고 관리하는 것을 목적으로 설계되었습니다. 즉, 데이터를 관리하고 바인딩하고 정제하기 위해 사용하는 ViewModel이 아닌 화면이 회전했을 때와 같이 상태가 변화 했을 때도 데이터 라이프 사이클을 관리하고 유지하는 데에 사용됩니다.

![AAC ViewModel Lifecycle](/uploads/viewmodel-lifecycle.png)

### MVVM ViewModel

MVVM ViewModel은 View와 Model 사이에서 데이터를 관리, 정제 및 바인딩을 하기 위한 클래스로, 값이 변경이 됐을 때 그 상태 변화를 View에게 전달하는 작업을 합니다.

MVVM에서 View와 ViewModel의 관계는 연결 상태를 최소화하고 ViewModel은 데이터가 변한 것을 View에 전달하고 View에서는 화면 정보의 변화를 ViewModel에 전달해야 합니다.

# 🫠 MVVM + Repository 개발하기

## 🗂️ 폴더 구조

```
.
├── app.js
├── js
│   ├── abstract
│   │   ├── binder.js
│   │   ├── model.js
│   │   └── view.js
│   ├── binder
│   │   └── sub-binder.js
│   ├── config
│   │   └── *
│   ├── constant
│   │   └── *
│   ├── error
│   │   └── *
│   ├── model
│   │   └── sub-model.js
│   ├── repository
│   │   └── repository.js
│   ├── ui
│   │   └── *
│   ├── util
│   │   └── type-check.js
│   ├── view
│   │   └── sub-view.js
│   └── viewmodel
│       └── sub-view-model.js
├── scss
│   └── styles.scss
└── views
    └── index.html
```

- **abstract**: binder, model, view로 사용하기 위한 추상 클래스를 모아둔 폴더
- **binder**: **AddEventListener** 등 이벤트 바인딩을 위한 클래스를 모아둔 폴더
- **config**: Fetch 등을 통해 네트워크에 요청을 할 때 사용할 config 들을 모아둔 폴더
- **constant**: 상수들을 모아둔 폴더
- **error**: **Error**를 상속받은 클래스를 모아둔 폴더
- **model**: 모델 클래스를 모아둔 폴더
- **repository**: 데이터 통신을 통해 가져온 데이터들을 저장하는 클래스를 모아둔 폴더
- **ui**: UI 변경 (ProgressBar, Alert 등)을 사용하는 클래스를 모아둔 폴더
- **util**: 유틸리티 클래스를 모아둔 폴더
- **view**: 뷰 클래스를 모아둔 폴더
- **viewmodel**: 뷰모델 클래스를 모아둔 폴더

## 🐛 MVVM + Repository 로직

![MVVM Vanilla](/uploads/mvvm-vanilla.png)

대부분의 프로세스는 다음과 같습니다.
1. Binder를 통해서 이벤트가 들어온 것을 확인합니다.
2. 들어온 이벤트를 기반으로 적절한 ViewModel의 메소드를 실행시킵니다.
3. 네트워크 요청이 필요한 이벤트일 경우, ViewModel에서는 Repository에서 데이터를 가져오도록 합니다.
4. Repository에서는 Model에 맞게 데이터를 불러온 뒤에 값을 ViewModel에 전달합니다.
5. ViewModel에서는 Repository에서 받은 데이터를 적절히 가공하여 저장합니다.
6. View에서는 ViewModel의 가공된 데이터를 기준으로 화면에 출력합니다.

## 🛠️ 코드

간단한 자바스크립트 코드를 보면서 로직을 파악해보도록 하겠습니다.

### Binder

```js
// binder.js

export default class Binder {
	constructor() {
		if (this.constructor === Binder) {
			throw new Error('Abstract Error');
		}
	}

	bindEvents() {
		throw new Error('Abstract Error');
	}
}
```

Binder는 `BindEvents()` 추상 메서드를 가지고 있는 추상 클래스입니다. Binder를 상속받은 클래스를 사용해서 유저는 이벤트를 바인딩 할 수 있습니다.

```js
// sub-binder.js

class SubBinder extends Binder {
	bindEvents() {
		const tempEl = document.querySelector('.my-temp-button');
		const subViewModel = new SubViewModel();
		const subView = new SubView(subViewModel);
		
		tempEl.addEventListener('click', async () => {	
			await subViewModel.action();
			subView.render();
		});
	}
}

export default new SubBinder();
```

간단한 예를 살펴보면 이벤트가 일어났을 때 ViewModel을 통해 데이터를 가공하고 View를 통해 렌더링을 하게 하였습니다.

### View

```js
// view.js

export default class View {
	constructor() {
		if (this.constructor === View) {
			throw new Error('Abstract Error');
		}
	}

	render() {
		throw new Error('Abstract Error');
	}
}
```

View도 Binder와 동일하게 추상클래스로 만들어주었습니다. 자식 클래스들은 View를 상속받고 ViewModel의 데이터 가공이 끝났을 때 render()를 호출하여 화면을 업데이트 해줍니다.

```js
// sub-view.js

export default class SubView extends View {
	#subViewModel;
	
	constructor(subViewModel) {
		super();
		this.#subViewModel = subViewModel;
	}
	
	render() {
		const { name, age } = this.#subViewModel.getUser();
		
		const nameEl = document.querySelector('.my-name');
		const ageEl = document.querySelector('.my-age');
		
		nameEl.textContent = name;
		ageEl.textContent = ageEl;
	}
}
```

말 그대로 View에서는 ViewModel에서 가공된 데이터를 렌더링할 수 있습니다. `render()`는 Binder에서 모든 일이 끝난 뒤에 마지막에서 호출되기 때문에 업데이트된 데이터를 가지고 있어 새로운 데이터를 뿌릴 수 있습니다.

### ViewModel

```js
// sub-view-model.js

export default class SubViewModel {
	#subModel;
	#repository = new Repository();
	
	getUser() {
		return this.#subModel.getData();
	}
	
	async action() {
		this.#subModel = await this.#repository.fetchData();
	}
}
```

ViewModel은 그 범위가 달라 추상클래스 없이 개발을 진행하였습니다. 여기서는 알맞게 Model을 기준으로 데이터를 가공하거나 불러오는 작업을 합니다.

### Repository

```js
// repository.js

export default class Repository {
	async fetchData() {
		const res = await fetch('https://dummyjson.com/user/1');
		
		if (!res.ok) throw new Error('Fetch Error');
		
		return new SubModel(res.json());
	}
}
```

Repository에서는 데이터를 가져오는 작업을 합니다. 여기서 리턴하는 데이터는 특정 Model을 기반으로 합니다.

### Model

```js
// model.js

export default class Model {
	constructor() {
		if (this.constructor === Model) {
			throw new Error('Abstract Error');
		}
	}

	setData() {
		throw new Error('Abstract Error');
	}
	
	getData() {
		throw new Error('Abstract Error');
	}
}
```

Model은 위와 같은 추상클래스를 만들어서 진행하였습니다. 물론 굳이 `setData()`, `getData()`를 추상 메서드로 만들 필요도, 꼭 이런 방식으로 Model을 개발하지 않아도 되나 이번 프로젝트를 진행하면서는 위와 같이 모델을 제작하였습니다.

```js
// sub-model.js

export default class SubModel extends Model {
	#data;

	constructor(data}) {
		super();
		#this.data = data;
	}
	
	setData(data) {
		this.#data = {
			...this.#data
			...data,
		}
	}
	
	getData() {
		return this.#data;
	}
}
```

그냥 간단하게 데이터를 불러오고 저장할 수 있도록 하였습니다.

이와 같이 로직을 분리하여 간단한 바닐라 자바스크립트 MVVM 패턴을 사용할 수 있습니다..!

# 🙂 결론

간단하게 MVVM을 활용해서 개발을 해봤는데요! 사실 간단한 작업은 위와 같은 Role로도 작동이 되지만 더 큰 규모의 프로젝트가 되기 위해서는 Role Design을 더욱 상세하고 정밀하게 짜야할 것 같네요!

## 참조

- [객체지향 자바스크립트 2회차(MVVM 구현) , jyoon blog version2 (happyjy.netlify.app)](https://happyjy.netlify.app/%EA%B0%9D%EC%B2%B4%EC%A7%80%ED%96%A5-%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-2%ED%9A%8C%EC%B0%A8mvvm-%EA%B5%AC%ED%98%84)
- [MVVM System 만들기 , 개발자 황준일 (junilhwang.github.io)](https://junilhwang.github.io/TIL/CodeSpitz/Object-Oriented-Javascript/02-MVVM/)
- [ViewModel 개요 ,  Android 개발자 ,  Android Developers](https://developer.android.com/topic/libraries/architecture/viewmodel)
