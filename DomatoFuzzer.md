# Domato 🍅

##### 7월 2, 2018 willwayy 이정모



- Domato 는 DOM fuzzer 이다.
  - 2개의 python 스크립트 파일에 의해 동작한다.
    - `generator.py` 파일은 main 스크립트이다.
    - `grammer.py` 파일은 라이브러리 및 DOM 퍼징을 위한 추가 도움말 코드가 포함되어 있다.
    - *.txt 파일에는 HTML, CSS 및 JavaScript 문법이 저장되어 있다.



## 설치

```
$ git clone https://github.com/google/domato.git ~/domato
$ cd domato
```



#### 다음과 같이 한개의 샘플 파일을 생성 할 수 있다.

```markdown
** $ python generator.py <output file> **
$ python generator.py sample.html
```



#### 다음과 같은 방법으로 여러개의 샘플 파일을 생성 할 수 있다.

```markdown
** $ python generator.py --output_dir <output directory> --no_of_files <number of output files> **
$ mkdir sample
$ python generator.py --output_dir sample --no_of_files 100
```

> 생성 된 샘플은 지정된 디렉토리에 저장되며 fuzz- <number> .html로 이름이 지정된다.



#### fuzz-1.html

<img src="https://user-images.githubusercontent.com/24206298/42162537-ff4bff5e-7e39-11e8-8c49-615c62b91f5d.png" style="zoom:100%" />



#### fuzzer-3.html

<img src="https://user-images.githubusercontent.com/24206298/42162542-00d0bfae-7e3a-11e8-8568-b62413248861.png" style="zoom:50%" />