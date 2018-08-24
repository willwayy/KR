# libFuzzer Tutorial

7월 11, 2018 willwayy 이정모



## 소개

- 이 튜토리얼에서는 coverage-guided in-process fuzzing engine 인  [libFuzzer](http://libfuzzer.info/) 를 사용하는 법을 배울 것이다.
- 또한 C / C++ 용 동적 메모리 오류를 탐지하는 [AddressSanitizer](http://clang.llvm.org/docs/AddressSanitizer.html) 의 기본적인 내용을 배울 것이다.
  - 학습하기 전 요구사항 : C / C++ 및 Unix 쉘 경험.

<br>

## 환경 구성

첫째, 환경을 준비해야합니다. [GCE ](https://cloud.google.com/compute/)에서 VM을 사용하는 것을 권장한다. 좀 더 간단한 해결책은 **Docker**를 사용하는 것이다 (클라우드 저장소와 관련된 작업을 수행하지 못할 수 있음). 또한 자신의 리눅스 머신을 사용할 수 있지만.. YMMV.

<br>

### VM on GCE

- [GCE ](https://cloud.google.com/compute/)에 로그인을 하거나, 계정이 없다면 새로 만드시오.

- 새로운 VM을 생성하고 ssh로 접속한다.

  - Ubuntu 16.04 권장, 다른 VM은 작동하지 않을 수 있음.
  - 가능한 한 많은 CPU를 선택하십시오.
  - "Access scopes" = "Allow full access to all Cloud APIs" 선택하시오 (엑세스 범위를 전제 엑세스 허용으로).

- 설치

  ```bash
  $ git clone https://github.com/google/fuzzer-test-suite.git FTS
  $ ./FTS/tutorial/install-deps.sh  # Get deps
  $ ./FTS/tutorial/install-clang.sh # Get fresh clang binaries
  ```

  > 마지막꺼는 설치하는데 시간이 좀 걸린다. (번역자는 20분정도 걸림)



<br>

### Docker

- [Install Docker](https://docs.docker.com/engine/installation/)
- 설치 후 실행 `docker run --cap-add SYS_PTRACE -ti libfuzzertutorial/prebuilt`

<br>

## Verify the setup

실행:

```bash
$ clang++ -g -fsanitize=address,fuzzer FTS/tutorial/fuzz_me.cc
./a.out 2>&1 | grep ERROR
```

제대로 설치가 되었다면 다음과 같은 문구를 볼 수 있을 것이다.

```bash
==31851==ERROR: AddressSanitizer: heap-buffer-overflow on address...
```



## 'Hello world' fuzzer

정의: 퍼징 타겟은 아래와 같은 시그니처가 있고 그 인자값으로 흥미로운것을 수행하는 함수이다.

```c
extern "C" int LLVMFuzzerTestOneInput(const uint8_t *Data, size_t Size) {
  DoSomethingWithData(Data, Size);
  return 0;
}
```

자 그럼 fuzz 타겟인  [./fuzz_me.cc](https://github.com/google/fuzzer-test-suite/blob/master/tutorial/fuzz_me.cc) 이것을 예로 한번 보자. 버그가 보이는가?

이 타겟에 대한 fuzzer 바이너리를 빌드하려면 다음 추가 옵션을 사용하여 최근 Clang 컴파일러를 사용하여 컴파일해야한다.

예:

```bash
$ clang++ -g -fsanitize=address,fuzzer FTS/tutorial/fuzz_me.cc
```

자 그럼 실행시켜보자:

```
./a.out
```

제대로 수행이 되었다면 이런것들이 보일 것이다.

```bash
INFO: Seed: 3918206239
INFO: Loaded 1 modules (14 guards): [0x73be00, 0x73be38),
INFO: Loaded 1 PC tables (7 PCs): 7 [0x52f8c8,0x52f938),
INFO: -max_len is not provided; libFuzzer will not generate inputs larger than 4096 bytes
INFO: A corpus is not provided, starting from an empty corpus
#0      READ units: 1
#1      INITED cov: 3 ft: 3 corp: 1/1b exec/s: 0 rss: 26Mb
#8      NEW    cov: 4 ft: 4 corp: 2/29b exec/s: 0 rss: 26Mb L: 28 MS: 2 InsertByte-InsertRepeatedBytes-
#3405   NEW    cov: 5 ft: 5 corp: 3/82b exec/s: 0 rss: 27Mb L: 53 MS: 4 InsertByte-EraseBytes-...
#8664   NEW    cov: 6 ft: 6 corp: 4/141b exec/s: 0 rss: 27Mb L: 59 MS: 3 CrossOver-EraseBytes-...
#272167 NEW    cov: 7 ft: 7 corp: 5/201b exec/s: 0 rss: 51Mb L: 60 MS: 1 InsertByte-
=================================================================
==2335==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x602000155c13 at pc 0x0000004ee637...
READ of size 1 at 0x602000155c13 thread T0
    #0 0x4ee636 in FuzzMe(unsigned char const*, unsigned long) FTS/tutorial/fuzz_me.cc:10:7
    #1 0x4ee6aa in LLVMFuzzerTestOneInput FTS/tutorial/fuzz_me.cc:14:3
...
artifact_prefix='./'; Test unit written to ./crash-0eb8e4ed029b774d80f2b66408203801cb982a60
...
```

위와 같이 비슷하게 떴다면 다행이다. 독자는 fuuzer 를 만들고 버그를 발견했다! 그럼 이제 결과를 살펴보겠다.

```bash
INFO: Seed: 3918206239
```

퍼저는 위와 같이 랜덤한 시드값을 가지고 시작된다. 동일한 결과를 얻으려면 `-seed=3918206239` 로 다시 실행하라.

```bash
INFO: -max_len is not provided; libFuzzer will not generate inputs larger than 4096 bytes
INFO: A corpus is not provided, starting from an empty corpus	
```

기본적으로 libFuzzer는 모든 입력이 4096 byte 이하라고 가정한다. 이를 변경하려면 -max_len = N을 사용하거나 비어 있지 않은 [seed corpus](https://github.com/google/fuzzer-test-suite/blob/master/tutorial/libFuzzerTutorial.md#seed-corpus) 를 사용하여 실행하라.

```bash
#0      READ units: 1
#1      INITED cov: 3 ft: 3 corp: 1/1b exec/s: 0 rss: 26Mb
#8      NEW    cov: 4 ft: 4 corp: 2/29b exec/s: 0 rss: 26Mb L: 28 MS: 2 InsertByte-InsertRepeatedBytes-
#3405   NEW    cov: 5 ft: 5 corp: 3/82b exec/s: 0 rss: 27Mb L: 53 MS: 4 InsertByte-EraseBytes-...
#8664   NEW    cov: 6 ft: 6 corp: 4/141b exec/s: 0 rss: 27Mb L: 59 MS: 3 CrossOver-EraseBytes-...
#272167 NEW    cov: 7 ft: 7 corp: 5/201b exec/s: 0 rss: 51Mb L: 60 MS: 1 InsertByte-
```

libFuzzer는 적어도 272167 개의 입력 (# 272167)을 시도했으며 함께 7 개의 수신 지점 (cov : 7)을 커버하는 총 201 byte (입력 : 5 / 201b)의 5 개의 입력을 발견했습니다.

```bash
==2335==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x602000155c13 at pc 0x0000004ee637...
READ of size 1 at 0x602000155c13 thread T0
    #0 0x4ee636 in FuzzMe(unsigned char const*, unsigned long) FTS/tutorial/fuzz_me.cc:10:7
    #1 0x4ee6aa in LLVMFuzzerTestOneInput FTS/tutorial/fuzz_me.cc:14:3
```

입력 중 하나에서 AddressSanitizer가 `heap-buffer-overflow` 버그를 감지하고 실행을 중단했다.

```bash
artifact_prefix='./'; Test unit written to ./crash-0eb8e4ed029b774d80f2b66408203801cb982a60
```

프로세스를 종료하기 전 libFuzzer가 crash 를 일으킨 바이트를 가진 디스크에 파일을 생성했습니다. 이 파일을 봐봐라. 뭐가 보이나? 왜 crash를 일으켰습니나?

fuzzing w / o를 사용하여 crash 를 다시 재현하려면,

```bash
$ ./a.out crash-0eb8e4ed029b774d80f2b66408203801cb982a60
```



## HeartBleed

