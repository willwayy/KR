# CSS Keylogger

7월 5, 2018 willwayy 이정모



항상 그렇듯이 설명하기 앞서 실제로 해보자.

일단 github에서 css-keylogger 를 다운받자.



#### 설치

```bash
$ git clone https://github.com/maxchehab/CSS-Keylogging
$ ls
README.md               css-keylogger-extension  package.json            yarn.lock
build.go                node_modules             server.js
```

>  여기서 크롬 플러그인 폴더가 생성되므로 크롬 플러그인을 설정해주러 가자.



#### 크롬 플러그인 설치

<img src="https://user-images.githubusercontent.com/24206298/42323848-7cfb5da0-809c-11e8-9fcf-4e5c82d7bac5.png" style="zoom:50%" />

>  [chrome://extensions/](chrome://extensions/)  에 접속을 하던지 아니면 확장 프로그램 메뉴에 들어가서 개발자 모드를 on 해준다.



<img src="https://user-images.githubusercontent.com/24206298/42323851-7d5d440c-809c-11e8-99f7-c40c3e0a09ff.png" style="zoom:90%" />

> 다운받은 폴더에 크롬 확장 프로그램 폴더가 있으니 선택해서 추가해준다.



<img src="https://user-images.githubusercontent.com/24206298/42323850-7d2a9a16-809c-11e8-9d6a-fb17b212e6e1.png" style="zoom:0%" />

> 잘 진행이 되었다면 이런식으로 일반적인 크롬 플러그인이 만들어졌을 것이다.



#### 서버 설치 (MacOS)

```bash
$ brew install yarn
$ yarn start
```

<img src="https://user-images.githubusercontent.com/24206298/42323852-7d8c3e60-809c-11e8-8aaa-e28d7dd03e2f.png" style="zoom:40%" />

> 이렇게 정상적으로 표시가 되면 이쪽으로 키로깅이 되는것이다.



#### 실습방법

1. [instagram](https://instagram.com) 과 같은 사이트를 접속한다.
2. 웹 페이지 오른쪽 상단의 확장 프로그램 중 우리가 설치한 키로거 프로그램을 실행시킨다. (`C` 표시로 된 프로그램)
3. 아무런 암호를 입력해보자.
4. 현재 켜진 서버를 봐보자!



<img src="https://user-images.githubusercontent.com/24206298/42323765-390a7b62-809c-11e8-9712-63d4104e791d.gif" style="zoom:60%" />

> OMG



* 추가 작업중 ...





