# AFL

#### American Fuzzy Lop

7월 9, 2018 willwayy 이정모



- AFL 은 테스트 케이스의 코드 적용 범위 (Code Coverage) 를 효율적으로 늘리기 위해 유전자 알고리즘 (Genetic Algorithm) 을 사용하는 fuzzer 이다. 



#### Install

```bash
$ wget http://lcamtuf.coredump.cx/afl/releases/afl-latest.tgz
$ tar -xvf afl-latest.tgz
$ cd afl-2.49b/
$ make
$ sudo make install
```



#### Command

| Command     |   Description    |   Basic methods of use    |
| ----------- | :--------------: | :-----------------------: |
| afl-analyze | 파일 포맷 분석기 | afl-analyze -i target_app |
| afl-fuzz    |  AFL 코드 퍼지   | afl-fuzz -i -o target_app |
| afl-gotcpu  | CPU선점비율출력  |        afl-gotcpu         |



#### AFL Fuzzing Method

```mermaid
graph LR
A(AFL 설치) -->B(Fuzzing 대상 소스코드 컴파일) 
    B --> C(AFL Fuzzing Start)
    C --> D(Segmentation fault inputcase 조사)
```

> testcase가 반드시 있어야 하는데, 이것은 타겟 바이너리에 임의의 입력값을 넣은뒤에 새로운 path가 생기면 거기 에 대한 mutation 우선순위를 높여서 fuzzing의 효율을 높인다. 

<img src="https://user-images.githubusercontent.com/40850499/42444692-aeb02e5e-83ab-11e8-88e3-8425ed3c5f51.png" style="zoom:0%" />

> afl-fuzzer 디렉토리의 testcases 라는 디렉토리에 가보면 여러가지 파일에 대한 input case 가 있다. 



#### test-instr.c

```c
 #include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
int main(int argc, char** argv) {
char buf[8];
  if (read(0, buf, 8) < 1) {
    printf("Hum?\n");
    exit(1);
}
  if (buf[0] == '0')
    printf("Looks like a zero to me!\n");
else
    printf("A non-zero value? How quaint!\n");
  exit(0);
}
```



<img src="https://user-images.githubusercontent.com/40850499/42444679-ac627f30-83ab-11e8-9822-00699d91e5a9.png" style="zoom:0%" />

> afl-gcc 를 이용한 컴파일이다.  (컴파일시에 afl 메시지도 같이 나와야 정상적으로 컴파일 된 것이다) 



실행하기 앞서...



#### input 데이터가 Standard input 인 경우 

```bash
$ ./afl-fuzz -i testcase_dir -o findings_dir /path/to/program [...params...]
```

> -i "Testcase 디렉토리" -o "결과가 만들어진 디렉토리 이름" "Fuzzing할 바이너리 이름" "파라미터이름..." 



#### input 데이터가 File 인 경우 

```bash
 -i "Testcase 디렉토리" -o "결과가 만들어진 디렉토리 이름" "Fuzzing할 바이너리 이름" @@ 
```

> -i "Testcase 디렉토리" -o "결과가 만들어진 디렉토리 이름" "Fuzzing할 바이너리 이름" @@ 



위의 test-instr 파일은 argv는 받지 않고 input을 키보드 등으로 입력받으니 첫번째 케이스로 하면 된다. 

> Test case 디렉토리는 afl 디렉토리의 ./testcases/other/text로 지정할것이고, 결과 디렉토리는 afl-out 디렉토 리를 만들어서 이걸로 지정해줄것이다. 



<img src="https://user-images.githubusercontent.com/40850499/42444680-ac9397fa-83ab-11e8-887d-b5e2eb1430a2.png" style="zoom:50%" />

음? 🤔  위와 같은 에러를 뿜어내면서 실행이 되지 않는다.  

> 그래서 추가적인 정보를 찾아보다 보니 echo core >/proc/sys/kernel/core_pattern 를 해주라고 한다. 



<img src="https://user-images.githubusercontent.com/40850499/42444681-acc4f3cc-83ab-11e8-9682-35188fe82b9b.png" style="zoom:50%" />

🤒 약 80도 까지 올라가지만 정상이다. 자 이제 퍼징을 끝내고 결과 파일을 분석해보자. 

> 유니크 크래시가 나지 않는 것은 정상이다. 



<img src="https://user-images.githubusercontent.com/40850499/42444682-ad19a7e6-83ab-11e8-8c92-832a581344dc.png" style="zoom:50%" />

위에서 fuzzing한 결과는 afl-out 디렉토리로 지정했었다. 디렉토리로 가보면, crashes, hangs, queue 디렉토리가 만들어져있는곳에 있다. 

> crashes ➔  실제 sig fault (프로그램오류)등이 나왔을때의 unique input 파일
>
> queue   ➔   디렉토리는 각 path 별 입력값들  
>
> hangs    ➔   지정된 시간을 지났을때까지 멈췄었던 input 데이터 case를 볼수있다. 



<img src="https://user-images.githubusercontent.com/40850499/42444683-ad4c2464-83ab-11e8-9b32-a70cff655e10.png" style="zoom:0%" />

원래대로라면 유심히 봐야할 부분은 crashes 라는 디렉토리 이지만, 

크래쉬가 나지 않는 샘플이므로 그냥 2개 path에 대한 input 값만 볼 수 있다. 

결론적으로 test-instr 이라는 바이너리를 퍼징하고 크래시가 나면 그 크래시 파일을 이용하여 crash를 reproduce 할 수 있다. 



------



이론적인 내용과 대충 어떻게 돌아가는지 알았으니, 

부가적인 옵션들과 실제로 크래시가 나는 예제를 통해 좀 더 알아보자. 



#### test.c

```c
 #include <stdio.h>
#include <string.h>
int main() {
    char login[16];
    char password[16];
    printf("Login : ");
    scanf("%s", login);
    printf("Password : ");
    scanf("%s", password);
    if(strcmp(login, "root") == 0)
    {
        if(strcmp(password, "toor") == 0)
        {
            printf("Success.\n");
			return 0; 
		}
	}
    printf("Fail.\n");
    return 1;
}
```



#### afl-analyze

- 해당 바이너리는 Test Case 의 파일 포맷을 분석한다.
  -  데이터 스트림으로 부터 순차적으로 전달되는 데이터들을 가져오며, 매 입력마다 바이너리의 동작을 관찰한다. 

<img src="https://user-images.githubusercontent.com/40850499/42444685-ad87221c-83ab-11e8-858c-23c51a9de9c5.png" style="zoom:50%" />

#### afl-gotcpu

- 해당 바이너리는 afl-fuzz 에서 사용중인 cpu 선점 비율을 출력한다. <img src="https://user-images.githubusercontent.com/40850499/42444686-adef45c2-83ab-11e8-8aa8-0862cb71df46.png" style="zoom:60%" />

## Crash Test

<img src="https://user-images.githubusercontent.com/40850499/42444687-ae1f7b52-83ab-11e8-9360-226e3031fa17.png" style="zoom:0%" />

./testcase 안에는 다음과 같이 Test case 를 생성해놓았다 

- ID가 틀린경우 
- Password가 틀린경우
- ID, Password 모두 틀린경우
- ID, Password 정확한 경우 



<img src="https://user-images.githubusercontent.com/40850499/42444689-ae4ee432-83ab-11e8-8703-887cb185c304.png" style="zoom:50%" />

> 유니크crash가생성된걸볼수있다. 



#### Check for the crash

- 다음과 같이 발견된 uniq crashes 는 result 디렉토리에 저장되어 있다. 
  - 해당파일을이용해crash를재현할수있다.



<img src="https://user-images.githubusercontent.com/40850499/42444691-ae7f009a-83ab-11e8-980c-1e4acf113606.png" style="zoom:50%" />

7월 9, 2018 willwayy 이정모