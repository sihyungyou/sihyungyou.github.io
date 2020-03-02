---
layout: post
title: "Angora Fuzzer 설치"
tags: [실프2]
comments: true
---

> 앙고라 설치하기  

## git clone  
~~~bash
$ git clone https://github.com/AngoraFuzzer/Angora.git
~~~

## install rustup for Rust  
~~~bash
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
~~~

## 환경설정  
shell configuration file `~/.bashrc`, `~/.zshrc`에 다음과 같이 설정해준다.  
~~~
export PATH=/path-to-clang/bin:$PATH
export LD_LIBRARY_PATH=/path-to-clang/lib:$LD_LIBRARY_PATH
~~~
참고로 path-to-clang은 LLVM 설치를 하면 그 안에 있다! 그리고 이 설정을 적용하기 위해서 `./source ~/.bashrc`를 실행해주자.  

## Fuzzer Compilation  
앙고라에 들어가서 다음 커맨드를 실행해주자  
~~~bash 
$ ./build/build.sh
~~~

## Test  
정상적으로 설치되었는지 확인하기 위해 앙고라에서 제공하는 테스트 프로그램을 돌려보자!  
~~~bash
$ cd /path-to-angora/tests
$ ./test.sh mini
~~~

![Center example image](https://user-images.githubusercontent.com/35067611/75654519-424d2c80-5ca3-11ea-92a9-7f316c5d304c.png "Center"){: .center-image}  
위와 같은 결과가 나오면 설치에 성공한 것이고 fuzzing을 할 준비가 된 것이다!  

