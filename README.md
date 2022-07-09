# thymeleaf
> Java Template Engine Library <br>
> [tymeleaf docs](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#introducing-thymeleaf)
***
<details>
<summary>(#01) Escape vs UnEscaped</summary>
<div markdown="1">

## (#01) Escape vs UnEscaped
HTML 문서는 ``<``, ``>`` 같은 특수문자를 기반으로 정의된다.
따라서 뷰 템플릿으로 HTML 화면을 생성할 때는 출력하는 데이터에 이러한 특수 문자가 있는 것을 주의해서 사용해야 한다.

* 변경 전 : Hello Spring!

```java
model.addAttribute("data", "Hello Spring!");
```

* 변경 후 : Hello ``<b>``Spring!``</b>``

```java
model.addAttribute("data", "<b>Hello Spring!</b>");
```

Escape 문법 때문에 ``<b>`` 태그가 적용되지 않는 모습이다. 따라서 타임리프에서 ``<b>`` 태그를
적용시키려면 아래의 2가지 기능을 사용해야 한다.

* ``th:text`` -> ``th:utext``
* ``[[${data}]]`` -> ``[(${data)]``

### 적용
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<ul>
    <li>th:text = <span th:text = "${data}"></span></li>
    <li>th:text = <span th:utext = "${data}"></span></li>
</ul>

<h1><span th:inline="none">[[...]] vs [(...)]</span></h1>
<ul>
    <li><span th:inline = "none"></span>[[${data}]]</li>
    <li><span th:inline = "none"></span>[(${data})]</li>
</ul>
</body>
</html>
```
### 결과
![](img/UnEscaped.png)
***

</div>
</details>

<details>
<summary>(#02) Variable - SpringEL</summary>
<div markdown="1">

## (#02) Variable - SpringEL
타임리프에서 변수를 사용할 때는 ``변수 표현식``을 사용한다.
> 변수 표현식 : ``${...}``

그리고 이 변수 표현식에는 ``스프링 EL``이라는 스프링이 제공하는 표현식을 사용할 수 있다.

### Controller에 User 객체 생성
```java
@GetMapping("/variable")
    public String variable(Model model){
        User userA = new User("userA", 10);
        User userB = new User("userB", 20);

        List<User> arr = new ArrayList<>();
        arr.add(userA);
        arr.add(userB);

        Map<String, User> map = new HashMap<>();
        map.put("userA", userA);
        map.put("userB", userB);

        model.addAttribute("user", userA);
        model.addAttribute("users", arr);
        model.addAttribute("userMap", map);

        return "basic/variable";
    }

    @Data
    static class User {
        private String username;
        private int age;

        public User(String username, int age) {
            this.username = username;
            this.age = age;
        }
    }
```

### variable.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Variable</title>
</head>
<body>
<h1>SpringEL 표현식</h1>

<h2>Object</h2>
<ul>
    <li>${user.username} : <span th:text = "${user.username}"></span></li>
    <li>${user['username'} : <span th:text = "${user['username']}"></span></li>
    <li>${user.getUsername()} : <span th:text = "${user.getUsername()}"></span></li>
</ul>

<h2>List</h2>
<ul>
    <li>${users[0].username} : <span th:text="${users[0].username}"></span></li>
    <li>${users[0]['username']} : <span th:text="${users[0]['username']}"></span></li>
    <li>${users[0].getUsername()} : <span th:text="${users[0].getUsername()}"></span></li>
<br>
    <li>${users[1].username} : <span th:text="${users[1].username}"></span></li>
    <li>${users[1]['username']} : <span th:text="${users[1]['username']}"></span></li>
    <li>${users[1].getUsername()} : <span th:text="${users[1].getUsername()}"></span></li>
</ul>

<h2>Map</h2>
<ul>
    <li>${userMap['userA'].username} : <span th:text = "${userMap['userA'].username}"></span></li>
    <li>${userMap['userA']['username']} : <span th:text = "${userMap['userA']['username']}"></span></li>
    <li>${userMap['userA'].getUsername()} : <span th:text = "${userMap['userA'].getUsername()}"></span></li>

    <br>

    <li>${userMap['userB'].username} : <span th:text = "${userMap['userB'].username}"></span></li>
    <li>${userMap['userB']['username']} : <span th:text = "${userMap['userB']['username']}"></span></li>
    <li>${userMap['userB'].getUsername()} : <span th:text = "${userMap['userB'].getUsername()}"></span></li>

</ul>

<h2> 지역 변수 - (th:with)</h2>
<div th:with="first=${users[0]}">
    <p>첫 번째 사람의 이름 : <span th:text = "${first.username}"></span></p>
    <p>첫 번째 사람의 나이 : <span th:text = "${first.age}"></span>세</p>
</div>

</body>
</html>
```
* `list`는 ``index``( [0], [1] .. )에 접근하여 변수를 사용할 수 있다.
* `map`은 `key`와 `value`로 이루어져 있기 때문에 `key`로 접근해야 한다.
* `username`, `['username']`, `getUsername()`은 모두 같다.
* `지역 변수`는 **선언한 태그 내에서만** 사용 가능하다.

### 결과
![](img/Variable.png)

</div>
</details>

<details>
<summary>(#03) LocalDateTime</summary>
<div markdown="1">

## (#03) LocalDateTime
### BasicController에 date 메서드 추가
```java
    @GetMapping("/date")
    public String date(Model model){
        model.addAttribute("localDateTime", LocalDateTime.now());
        return "basic/date";
    }
```

### date.html
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
  <meta charset="UTF-8">
  <title>Title</title>
</head>
<body>
<h1>LocalDateTime</h1>
<ul>
  <li>default = <span th:text="${localDateTime}"></span></li>
  <li><b>포맷팅</b> : yyyy-MM-dd HH:mm:ss = <span th:text="${#temporals.format(localDateTime, 'yyyy-MM-dd HH:mm:ss')}"></span></li>

</ul>
<h1>LocalDateTime - Utils</h1>

<ul>
  <li>${#temporals.day(localDateTime)} = <span th:text="${#temporals.day(localDateTime)}"></span></li>
  <li>${#temporals.month(localDateTime)} = <span th:text="${#temporals.month(localDateTime)}"></span></li>
  <li>${#temporals.monthName(localDateTime)} = <span th:text="${#temporals.monthName(localDateTime)}"></span></li>
  <li>${#temporals.monthNameShort(localDateTime)} = <span th:text="${#temporals.monthNameShort(localDateTime)}"></span></li>
  <li>${#temporals.year(localDateTime)} = <span th:text="${#temporals.year(localDateTime)}"></span></li>
  <li>${#temporals.dayOfWeek(localDateTime)} = <span th:text="${#temporals.dayOfWeek(localDateTime)}"></span></li>
  <li>${#temporals.dayOfWeekName(localDateTime)} = <span th:text="${#temporals.dayOfWeekName(localDateTime)}"></span></li>
  <li>${#temporals.dayOfWeekNameShort(localDateTime)} = <span th:text="${#temporals.dayOfWeekNameShort(localDateTime)}"></span></li>
  <li>${#temporals.hour(localDateTime)} = <span th:text="${#temporals.hour(localDateTime)}"></span></li>
  <li>${#temporals.minute(localDateTime)} = <span th:text="${#temporals.minute(localDateTime)}"></span></li>
  <li>${#temporals.second(localDateTime)} = <span th:text="${#temporals.second(localDateTime)}"></span></li>
  <li>${#temporals.nanosecond(localDateTime)} = <span th:text="${#temporals.nanosecond(localDateTime)}"></span></li>
</ul>
</body>
</html>
```
날짜를 `formating` 할 수 있는 `#temporals.format()`을 자주 사용함

### 결과
![](img/LocalDateTime.png)
</div>
</details>

<details>
<summary>(#04) URL 링크</summary>
<div markdown="1">

타임리프에서 URL을 생성할 때는 `@{...}` 문법을 사용하면 된다.

## BasicController에 link 메서드 추가
```java
    @GetMapping("/link")
    public String link(Model model){
        model.addAttribute("param1", "data1");
        model.addAttribute("param2", "data2");
        return "basic/link";
    }
```

## link.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>URL Link</title>
</head>
<body>
<ul>
<li><a th:href="@{/hello}">basic url</a></li>
<li><a th:href="@{/hello(param=${param1}, param2=${param2})}">query param</a></li>
<li><a th:href="@{/hello/{param1}/{param2}(param1=${param1}, param2=${param2})}">path variable</a></li>
<li><a th:href="@{/hello/{param1}(param1=${param1}, param2=${param2})}"> query param + path variable </a></li>
</ul>
</body>
</html>
```

***

#### 단순한 URL
> `@{/hello}` : `/hello`
***
#### 쿼리 파라미터
> `@{/hello(param1=${param1}, param2=${param2})}` : `/hello?param1=data1&param2=data2`

`()` 에 있는 부분은 쿼리 파라미터로 처리된다.
***
#### 경로 변수
> `@{/hello/{param1}/{param2}(param1=${param1}, param2=${param2})}` : `/hello/data1/data2`
***
* 상대경로, 절대경로, 프로토콜 기준을 표현할 수 도 있다.

* `/hello` : 절대경로
* `hello` : 상대경로
</div>
</details>

<details>
<summary>(#04) Literal</summary>
<div markdown="1">

`리터럴`은 소스 코드상에 `고정된 값`을 말하는 용어이다.
예를 들어서 다음 코드에서 "Hello" 는 문자 리터럴, 10 , 20 는 숫자 리터럴이다.
```java
String a = "Hello";
int a = 10 * 20;
```

타임리프에서 문자 리터럴은 항상 `' (작은 따옴표)`로 감싸야 한다.
```html
<span th:text="'hello'">
```

### BasicController에 literal 메서드 추가
```java
@GetMapping("/literal")
    public String literal(Model model){
        model.addAttribute("data", "Spring!");
        return "basic/literal";
    }
```

### literal.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Literal</title>
</head>
<body>
<ul>
    <li>'hello' + ' world!' = <span th:text = "'hello' + ' world!'"></span></li>
    <li>'hello world!' = <span th:text="'hello world!'"></span></li>
    <li>'hello ' + ${data} = <span th:text="'hello ' + ${data}"></span></li>
    <li>리터럴 대체 |hello ${data}| = <span th:text="|hello ${data}|"></span></li>
</ul>
</body>
</html>
```

### 결과
![](img/Literal.png)
</div>
</details>

<details>
<summary>(#05) Operation - 연산</summary>
<div markdown="1">

타임리프 연산은 자바와 크게 다르지 않다. 
HTML 안에서 사용하기 때문에 HTML 엔티티를 사용하는 부분만 주의하자.

## BasicController 추가
```java
    @GetMapping("/operation")
    public String operation(Model model){
        model.addAttribute("nullData", null);
        model.addAttribute("data", "타임리프 제대로 배우기");
        return "basic/operation";
    }
```

## operation.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Operation</title>
</head>
<body>

<ul>
  <li>산술 연산
    <ul>
      <li>10 + 2 = <span th:text="10 + 2"></span></li>
      <li>10 % 2 == 0 = <span th:text="10 % 2 == 0"></span></li>
    </ul>
  </li>
    <li> 비교 연산
        <ul>
            <li>1 > 10 = <span th:text="1 &gt; 10"></span></li>
            <li>1 gt 10 = <span th:text="1 gt 10"></span></li>
            <li>1 >= 10 = <span th:text="1 >= 10"></span></li>
            <li>1 ge 10 = <span th:text="1 ge 10"></span></li>
            <li>1 == 10 = <span th:text="1 == 10"></span></li>
            <li>1 != 10 = <span th:text="1 != 10"></span></li>
        </ul>
    </li>
    <li>조건식
        <ul>
            <li>(10 % 2 == 0) ? '짝수' : '홀수' = <span th:text="(10 % 2 == 0) ? '짝수' : '홀수'"></span></li>
        </ul>
    </li>
    <li>Elvis 연산자
        <ul>
            <li>${data} ? : '데이터가 없습니다.' = <span th:text="${data} ?: '데이터가 없습니다.'"></span></li>
            <li>${nullData} ? : '데이터가 없습니다.' = <span th:text="${nullData} ?: '데이터가 없습니다.'"></span></li>
        </ul>
    </li>
    <li>No Operation
        <ul>
            <li>${data} ?: _ = <span th:text="${data} ?: _">데이터가 없습니다.</span></li>
            <li>${nullData} ?: _ = <span th:text="${nullData} ?: _">데이터가 없습니다.</span></li>
        </ul>
    </li>
</ul>
</body>
</html>
```

## 결과
![](img/operation.png)
</div>
</details>