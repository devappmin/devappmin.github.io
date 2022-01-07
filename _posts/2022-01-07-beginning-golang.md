---
title: golang을 처음 접할 때 알아야할 알짜배기
date: 2022-01-07 17:48 +0900
categories: [Language, Go]
tags: [Go]
---

## Before start.

[노마드코더](https://nomadcoders.co)에서 `쉽고 빠른 Go 시작하기`를 학습하며 새로 배운 사실들을 공부하면서 적은 글 입니다.

### References

[Golang Study](https://github.com/devappmin/golang_study)에 공부를 하며 작성한 예제 프로그램과 각 챕터마다 `README`를 적어놓았습니다.

## Getting Started.

### formatting package

```go
import "fmt"
```

### Print something

```go
import "fmt"
fmt.Println("Hello world!")
```

### Private / Public

`Public` - 변수 혹은 함수 명을 대문자로 시작할 경우<br>
`Private` - 변수 혹은 함수 명을 소문자로 시작할 경우

### Constant / Variable

`const` 상수<br>
`var`변수

```go
// 둘은 동일
// :=은 새로 만들 때만 사용
var name string = "hello"
name := "hello" // 함수 내부에서만 사용 가능
```

## Function

### General

파라미터 및 리턴 값은 타입을 지정해야 함.<br>

```go
func multiply(a int, b int) int {
    return a * b
}
```

or

```go
func multiply(a, b int) int {
    return a * b
}
```

**리턴 값은 두 개 이상일 수도 있다!!**

```go
func lenAndUpper(name string) (int, string) {
	return len(name), strings.ToUpper(name)
}

func main() {
    // _ means ignore
    totalLength, upperName := lenAndUpper("Hong Gil Dong")
    second, _ := lenAndUpper("Hong Gil Dong")
}
```

**`...`을 통해서 여러 개의 값을 파라미터로 가져올 수 있다.**

```go
func repeatMe(words ...string) {
	fmt.Println(words)
}

func main() {
	repeatMe("Kim", "Lee", "Han", "Hong", "Kyu")
}
```

똑같이 `...`을 통해서 여러 개의 값을 파라미터로 넣을 수 있다.

```go
hi := []string{"hello", "hi"}
hello := []string{}
hello = append(hello, hi...) // ...을 이용하여 여러개를 넣을 수 있음.
```

### Naked return

아래와 같이 리턴 값을 미리 정해둘 수 있다.

```go
func lenAndUpper(name string) (length int, uppercase string) {
	length = len(name)
	uppercase = strings.ToUpper(name)

	return
}

func main() {
	// _ means ignore
	totalLength, upperName := lenAndUpper("Hong Gil Dong")
	fmt.Println(totalLength, upperName)
}
```

### defer

Really cool feature<br>
defer로 입력된 부분은 함수가 끝난 다음에 호출이 됨.

```go
func lenAndUpper(name string) (length int, uppercase string) {
	defer fmt.Println("I'm done")

	length = len(name)
	uppercase = strings.ToUpper(name)

	return
}
```

## Loop

Go에는 for 이외에 loop를 도는 방법이 없음.

### For each 느낌

```go
// Index only
for number := range numbers {
    fmt.Println(number)
}
```

```go
// Index and number
for index, number := range numbers {
    fmt.Println(index, number)
}
```

### 전통적인 for 느낌

```go
for i := 0; i < len(numbers); i++ {
    fmt.Println(numbers[i])
}
```

## if else

기본 모습

```go
if age < 18 {
    return false
} else {
    return true
}
```

go에서는 if문 내에서도 변수를 선언할 수 있다.

```go
// Variable expression
koreanAge := age + 2
if koreanAge < 18 {
    return false
}
// Is equal as below
if koreanAge := age + 2; koreanAge < 18 {
    return false
}
```

## Switch

```go
switch age {
case 16:
    return false
case 18:
    return true
}
```

go에서는 switch안에 expression이 들어가도 됨.

```go
switch {
case age < 16:
    return false
case age == 18:
    return true
case age > 50:
    return false
}
```

또한 `if else`와 동일하게 `Variable expression`이 가능하다.

```go
switch koreanAge:= age + 2; koreanAge {
    // Put case here..
}
```

## Pointer

기본적으로 C와 동일

```go
func pointerExample() {
    a := 5
    b := &a
    fmt.Println(&a) // Address of a
    fmt.Println(b)  // Same as &a
    fmt.Println(*b) // point value of b
}
```

## Array

### 정적 할당 배열

```go
func arrayExample() {
    // [length]type{init values}
    names := [5]string{"hello", "world", ":)"}
    names[3] = "sup"
    names[4] = "have a nice day!"
}
```

### Slices

동적으로 크기를 변경하는 배열을 생성할 수도 있다.

```go
func slicesExample() {
    // 정적 배열을 생성할 때에서 배열의 크기 부분만 없애면 됨.
    names := []string{"hello", "world", ":)"}

    // 값 추가는 append 함수를 사용해서 추가하면 됨.
    names = append(names, "sup")
}
```

## Map

```go
variable := map[string]string{"Hello": "World", "Nice": "day!"}
// map[key]value{init values}
```

다른 언어들과 같이 `map`을 `for loop`에 돌릴 수 있음.

```go
for key, value := range variable {
    fmt.Println(key, "is", value)
}
```

```go
value, ok := variable["Hello"]
```

위와 같이 값을 잘 받아왔는지 ok를 통해서 받아올 수 있음.

## struct

`C`나 `C++`처럼 `struct`를 사용하여 구조체를 생성할 수 있음.

```json
{
  "name": "kim",
  "age": 18
}
```

위와 같이 value 부분에 dynamically하게 object가 들어가게 하는 방법은 `struct`를 사용하는 것.

구조체에는 메소드를 생성할 수 있음.

### 구조체 선언

만드는 방식은 다른 언어들과 비슷하나 `,`를 추가해서는 안됨.

```go
type person struct {
	name string
	age int
	favFood []string
}
```

### 구조체 생성

생성하는 것도 다른 언어랑 비슷하나 map과 같이 key값을 명시하여 출력하는 것을 선호.

```go
favFood := []string{"sushi", "kimbap", "ramen", "rice", "eggplant"}
// 명확하지 않음.
kim := person{"Hong Gil Dong", 26, favFood}

// key 값을 명시해줌.
kim := person{name:"Hong Gil Dong", age:26, favFood: favFood}

// 전체 출력
fmt.Println(kim)

// 일부 출력
fmt.Println(kim.name)
```

## After basic course.

`go`에는 클래스가 없다.

## Comments

### 구조체 export 시

상단에 Comments를 생성해아하며 첫 문자는 구조체 명을 적어야 함.

```go
// Account Struct
type Account struct {

}
```

## go.mod

go 디렉토리 내부에서 코딩을 하는 것이 아니라 로컬 디렉토리에서 모듈을 새로 만드니까 `import`를 할 수 없는 오류가 생기면 `go.mod`를 통해서 해결해야 함.

1. 프로젝트 Root Directory에서 `go.mod` 생성

   ```terminal
   $ go mod init main
   ```

   go mod를 초기화 하여 현재 디렉토리를 main으로 한다.

2. 생성하고자 하는 Module Directory 내에서 `go.mod` 생성

   ```terminal
   $ cd banking
   $ go mod init banking
   ```

   go mod를 생성한 뒤에 다시 Root Directory로 이동한다.

3. 프로젝트 Root Directory에서 Module Directory 변경

   ```mod
   replace main.com/banking => ./banking
   ```

   `main.com/banking`을 import 할 경우 현재 directory 내에 있는 `banking` directory를 자동으로 import 하게 함.

4. Module Linking
   ```terminal
   $ go mod tidy
   ```
   해당 명령어를 입력하면 자동으로 연결이 된다.

## Constructor

`go`의 구조체는 constructor가 없으므로 함수로 직접 빼서 만들어야 한다.

```go
// NewAccount creates a new account.
func NewAccount(owner string) *account {
	account := account{owner: owner, balance: 0}
	return &account
}
```

## Method

`go`에서 메소드를 생성하기 위해서는 `func`와 메소드 이름 사이에 구조체를 적어주면 된다.

```go
func (a Account) Deposit(amount int) {
    // (a Account) means receiver.
    // 보통은 receiver의 명을 struct 명의 첫 글자를 소문자로 한다.
}
...
account // Account type object.
account.Deposit() // 이렇게 method가 된다.
```

값을 변경하기 위해서는 receiver가 pointer야 한다.

```go
func (a *Account) Deposit(amount int) {
    // a를 포인터로 받아와야한다.
    // 이유는 C나 C++와 동일.
}
```

### String()

`String()` 메소드는 해당 구조체를 print할 때 출력하는 값을 의미한다. `python`의 `**str**``와 동일.

```go
func (a Account) String() string {
    return fmt.Sprint(a.Owner(), "'s account.\nHas: ", a.Balance())
}
```

## Error handling

```go
// Withdraw x amount from your account.
func (a *Account) Withdraw(amount int) error {
	if a.balance < amount {
		return errors.New("Can't withdraw from account")
	}
	a.balance -= amount

	// nil means no error
	return nil
}
```

`errors.Error()` 혹은 `errors.New()`로 에러를 생성하면 된다. 에러가 없으면 `nil`을 리턴해주는 것으로 처리.

```go
err := account.Withdraw(20)
if err != nil {
    log.Fatalln(err)
}
```

`log.Fatalln`은 프로그램을 출력 후 종료시켜준다.

```go
var errNoMoney = errors.New("Can't withdraw from account")
```

에러 변수를 만들 때는 변수 이름을 `err`로 시작해야한다.

## type

struct에서 만들었을 때도 봤겠지만 `type`은 뒤에 있는 것의 alias를 입력하게 해주는 것이다.

`C`나 `C++`에서의 `typedef`와 비슷.

```go
type Dictionary map[string]string
```

## http

### GET

`get` 메소드를 실행하기 위해서는 `http`를 이용해야 한다.

```go
resp, err := http.Get(url)
```

이렇게 얻은 `response`의 값에서 `status code`는 아래 방식으로 얻을 수 있다.

```go
resp.StatusCode
```

## make

`map`과 같은 것은 초기화를 해야하고 초기화를 하지 않았을 시에 오류가 발생한다.

```go
// OK
results := map[string]string{}
var results = make(map[string]string)

// Panic
results := map[string]string
var results map[string]string
```

위에서 `Panic`부분은 값을 추가하려고 하면 정상적으로 진행되지 않는다.

## Concurrency

### Goroutines

단순히 함수를 호출할 때 앞에다가 `go`를 붙여주면 됨.

```go
func main() {
    go hello("kim")
    hello("lee")
}

func hello(name string) {
    fmt.Println("Hello," name)
}
```

하지만 `main function`이 종료가 되면 병렬처리가 되고 있던 작업들도 전부 종료됨.

### Channels

`pipe`를 통해서 값을 전달하는 것을 `channel`이라고 한다.

```go
c := make(chan bool) // chan type
go myFunc(..., c)
```

위와 같은 방식으로 channel을 만들고 `goroutine` 함수에 인자값으로 넣어주면 된다.

그 후에는 함수 안에서 인자값을 처리해주면 된다.

```go
func myFunc(..., c chan bool) {
    // Your codes here..
    c <- true
}
```

```go
// chan 대신 chan<-으로 적어서 send only로 만들 수 있음.
func myFunc(..., c chan<- bool) {
    // Your codes here..

    fmt.Prinlnt(<-c) // error!
}
```

값을 넣어줄 때에는 `<-` 화살표를 통해서 넣어주면 된다.

```go
c := make(chan bool)
...
result := <-c
// 바로 출력할 경우 fmt.Println(<-c)
```

동일하게 `channel`의 값을 변수로 넣어줄 때도 `<-`를 활용하면 된다.<br>
이 떄는 호출을 한 함수(예제의 경우 `main`)에서 값을 가져올 때까지 기다린다.<br>
`C`의 `pthread_join`과 `Dart`의 `await`와 비슷하게 다음 코드로 넘어가지 않는 것 같음.

```go
c := make(chan bool)

people := [2]string{"kim", "lee"}

for _, person := range people {
    go isAwesome(person, c)
}

fmt.Println(<-c)
fmt.Println(<-c)
```

위와 같이 `channel`을 두 번 호출했으면 값을 두 번 받을 수 있음. 만약에 **두 번 호출**하고 **세 번 이상** 값을 받으려고 하면 `deadlock`에 걸림.

## Go query

`jQuery`와 비슷한데 `Go`버전의 라이브러리임.

```terminal
$ go get github.com/PuerkitoBio/goquery
```

### 페이지에서 읽기

```go
res, err := http.Get(/* Your URL Here.. */)

doc, err := goquery.NewDocumentFromReader(res.Body)

doc.Find("Class name of website")
```

해당 클래스의 값을 갖는 오브젝트를 생성하는 방법이다.

```go
doc.Find("Class name of website").Each(func(i int, s *goquery.Selection) {
    // Each values..
    // i => index
    // s => each values
})
```

얻은 오브젝트에서 값을 하나씩 돌아가면서 loop를 도는 방법이다.

`Find`가 아니라 `Attribute`를 통해서도 값을 얻을 수 있다.

```go
val, exists = doc.Attr("Attributes")
```

## String

### String Conversion

`strconv`를 사용하여 값을 String으로 변경할 수 있음.

```go
strconv.Itoa(5000)
```

### strings.TrimSpace

`TrimSpace`를 이용해서 string 앞뒤에 있는 빈 공간을 제거할 수 있음.

```go
strings.TrimSpace("Hello\n\nWor    ld")
```

### strings.Fields

텍스트를 텍스트의 배열로 나눠줌.

```go
// Return value is string[]
strings.Fields(strings.TrimSpace("Hello\n\nWor     ld"))
```

### strings.Join

텍스트 배열을 하나로 합치면서 `sep`을 넣어줌.

```go
strings.Join(strings.Fields(strings.TrimSpace("Hello\n\nWor     ld")), " ")
```

### TrimSpace + Fields + Join

```go
strings.Join(strings.Fields(strings.TrimSpace("   Hello\n\nWor     ld   ")), " ")
```

를 실행할 경우 아래와 같이 변경 됨.

1. TrimSpace
   ```go
   "Hello\n\nWor     ld"
   ```
2. Fields
   ```go
   "Hello", "Wor", "ld"
   ```
3. Join
   ```go
   "Hello Wor ld"
   ```

## CSV

### create file

```go
file, err := os.Create()
```

### create New CSV / Flush it

```go
w := csv.NewWriter(file)
defer w.Flush()
```

### CSV에 쓰기

```go
header := []string{"id", "title", "location", "company", "salary", "summary", "upload"}
wErr := w.Write(header)
```

## ECHO를 통한 웹 통신

### install echo

```terminal
$ go get github.com/labstack/echo
```

### 프로젝트에 echo 추가

`main.go`

```go
import "github.com/labstack.echo"
```

```terminal
$ go mod tidy
```

### echo 오브젝트 생성 및 GET 처리

```go
func handleHome(c echo.Context) error {
    return .String(http.StatusOK, /* Put value here*/ )
}
...
e := echo.New()
e.GET("/", handleHome)
```

### echo 서버 시작

```go
e.Logger.Fatal(e.Start(":PORT_NO"))
```
