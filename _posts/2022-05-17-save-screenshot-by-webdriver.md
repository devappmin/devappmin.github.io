---
title: "[Python] Selenium으로 원하는 웹사이트 스크린샷 찍기"
date: 2022-05-12 17:01 +0900
categories: [Develop, Python]
tags: [Python, Webdriver, Selenium]
---

# 시작하기에 앞서

<img title="" src="/uploads/f4ea9afd4663e2b42da8035f141d70c1b2b833c9.png" alt="selenium_logo_large (1).png" width="441" data-align="center">

최근에 프로젝트를 진행하면서 웹사이트의 특정 노트, 혹은 웹사이트 전체를 스크린샷을 찍어 저장해야 하는 상황이 생겼습니다. 파이썬으로 웹사이트를 캡쳐하는 방법은 다양하나, 저는 이번에 `Selenium`을 활용하여 스크린샷을 찍는 방법을 택하였습니다.

## Selenium 설치 및 Chromedriver 다운로드

이 부분은 `Selenium`을 써본 적이 없는 분들을 위해서 적으려고 합니다. `Selenium`을 사용할 수 있고 `Chromedriver`를 이미 다운로드하신 분은 **스크린샷 찍기**로 넘어가주시면 됩니다.

1. **pip**를 통해서 Selenium을 설치합니다.
    ```bash
    $ pip3 install selenium
    ```

2. 자신의 버전에 맞는 `Chromedriver`를 [링크](https://chromedriver.chromium.org/downloads)에서 다운 받습니다.

3. 다운 받은 `Chromedriver`를 자신의 프로젝트로 옮깁니다.

# 스크린샷 찍기

본 포스트에서는 **기본 스크린샷 찍기, 전체 화면 스크린샷 찍기, 특정 요소(노드) 스크린샷 찍기**로 나누어 스크린샷을 찍는 방법을 알아보겠습니다.

## 1. 기본 설정

일단 스크린샷을 찍기 전에 `Selenium`을 통해서 웹 드라이버를 설정해야 합니다. 저는 아래와 같이 설정했으며, 원하시는 입맛대로 수정하시면 됩니다.

```python
from selenium import webdriver

DRIVER = '자신의 chromedriver 위치' # 'chromedriver', './driver/chromedriver' 등

options = webdriver.ChromeOptions()
options.headless = True # 크롬 창을 띄우지 말고 숨기기

driver = webdriver.Chrome(DRIVER, options=options)
driver.get('원하는 웹사이트 URI')
```

저는 이번에 네이버를 스크린샷 찍어보겠습니다.

## 2. 스크린샷

### 2-1. 기본 스크린샷 찍기

그냥 기본적으로 스크린샷을 찍는 방법은 매우 간단합니다. `Selenium`에서 제공하는 `save_screenshot()` 메서드를 통해서 쉽게 스크린샷을 찍을 수 있습니다.

```python
driver.save_screenshot('filename 혹은 path') # 'screenshot.png', './temp/screenshot.png' 등
```

위 메서드를 통해서 스크린샷을 찍으면 다음과 같은 결과물을 얻습니다.

![Hello.png](/uploads/31be86e70a5e6ce14811e7e8af764aa909624fc9.png)

말 그대로 `Selenium`에서 표시되고 있는 화면을 스크린샷을 찍었기에 해당 웹사이트를 보여주는 섬네일 사진으로 활용하기에 적합해 보입니다.

### 2-2. 전체 화면 스크린샷 찍기

하지만 위와 같은 상황이 아니라 해당 페이지의 전체를 스크린샷으로 찍고 싶을 때도 있을 것입니다. 이러한 상황에서는 다음 코드를 통해서 스크린샷을 찍을 수 있습니다.

```python
from selenium.webdriver.common.by import By

original_size = driver.get_window_size()
required_width = driver.execute_script('return document.body.parentNode.scrollWidth')
required_height = driver.execute_script('return document.body.parentNode.scrollHeight')
driver.set_window_size(required_width, required_height)
# driver.save_screenshot('filename 혹은 path')  # 스크롤바를 포함하고 찍고 싶을 경우
driver.find_element(by=By.TAG_NAME, value='body').screenshot('filename 혹은 path')  # 스크롤바를 제외하고 찍고 싶을 경우
driver.set_window_size(original_size['width'], original_size['height'])
```

스크롤바를 넣고 싶은 경우에는 `driver.save_screenshot()`를 사용하시고, 스크롤바를 제외하고 싶을 경우에는 `driver.find_element(by=By.TAG_NAME, value='body').screenshot()`를 사용해서 스크린샷을 찍으면 됩니다.

추가적으로, deprecated 됐지만, `driver.find_element()` 대신 `driver.find_element_by_tag_name()`을 사용하셔도 동일하게 `screenshot()` 메서드를 호출할 수 있습니다.

해당 코드의 로직은 다음과 같습니다.

1. 현재 `Selenium`에서 보여주는 화면의 크기를 구합니다.

2. 웹사이트의 **body** 크기를 구하여 `Selenium`의 화면 크기를 **body** 크기로 변경합니다.

3. **body** 태그를 통해서 element를 구한 뒤에 스크린샷을 찍습니다.

4. 다시 화면 크기를 이전으로 변경합니다.

위 코드를 통해서 스크린샷을 찍으면 다음과 같은 결과물을 얻습니다.

<img title="" src="/uploads/5caae086abf75f2c6d7ef56373abd70e0e9d9472.png" alt="Hello2.png" width="313" data-align="center">

다음과 같이 네이버 웹사이트 전체를 저장한 것을 확인할 수 있습니다.

### 2-3. 특정 요소(노드) 스크린샷 찍기

마지막으로, 특정 요소(Element)를 저장하고 싶을 경우도 있을 텐데요, 이 때도 **전체화면 스크린샷 찍기**와 거의 유사한 경우로 스크린샷을 찍을 수 있습니다.

저는 네이버에서 검색창 부분만 스크린샷을 해보겠습니다. 또한, 검색창 부분의 클래스가 `search_area`인 것을 생각하여 클래스로 접근하여 스크린샷을 하겠습니다.

```python
from selenium.webdriver.common.by import By

original_size = driver.get_window_size()
required_width = driver.execute_script('return document.body.parentNode.scrollWidth')
required_height = driver.execute_script('return document.body.parentNode.scrollHeight')
driver.set_window_size(required_width, required_height)
driver.find_element(by=By.CLASS_NAME, value='search_area').screenshot('filename 혹은 path')
driver.set_window_size(original_size['width'], original_size['height'])
```

위 코드를 통해서 확인할 수 있듯이, 전체화면을 스크린샷 찍을 때랑 선택한 `element`만 다르게 해서 특정 부위만 스크린 샷을 찍을 수 있었습니다.

위 코드를 통해서 스크린샷을 찍으면 다음과 같은 결과물을 얻습니다.

<img src="/uploads/b6455aa0c0146962b95f08270e11b751bac60835.png" title="" alt="Hello3.png" data-align="center">

딱 저희가 원하는 검색창만 가져온 것을 확인할 수 있습니다.

## 3. Driver 종료하기

스크린샷을 전부 찍은 후에는 `Selenium`을 다음 코드를 통해서 종료하면 됩니다.

```python
driver.quit()
```

# 스크린샷을 찍는 클래스

위 내용을 종합하여 저는 아래와 같이 클래스를 만들어서 사용하고 있으니 참고하셔도 좋을 것 같습니다!

```python
# screenshot.py

from selenium import webdriver
from selenium.webdriver.common.by import By

class Screenshot:
    def __init__(self, driver_path='chromedriver'):
        DRIVER = driver_path

        options = webdriver.ChromeOptions()
        options.headless = True

        self.driver = webdriver.Chrome(DRIVER, options=options)

    def __del__(self):
        self.driver.quit()

    def save_screenshot(self, uri: str, file_name: str, path: str = './temp', element_class: str = None) -> None:
        self.driver.get(uri)
        required_width = self.driver.execute_script('return document.body.parentNode.scrollWidth')
        required_height = self.driver.execute_script('return document.body.parentNode.scrollHeight')
        self.driver.set_window_size(required_width, required_height)
        if element_class is None:
            self.driver.fine_element(by=By.TAG_NAME, value='body').screenshot('{}/{}'.format(path, file_name))
        else:
            self.driver.fine_element(by=By.CLASS_NAME, value=element_class).screenshot('{}/{}'.format(path, file_name))
```

해당 클래스는 다음과 같이 활용할 수 있습니다.

```python
from YOUR_PATH import Screenshot

c = Screenshot()
c.save_screenshot("https://www.naver.com", "searchbar.png", path='.', element_class='search_area')
c.save_screenshot("https://www.naver.com", "fullscreen.png")
```

# 결론

## References

[Take screenshot of full page with Selenium Python with chromedriver - Stack Overflow](https://stackoverflow.com/a/52572919/)