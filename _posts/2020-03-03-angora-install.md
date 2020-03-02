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

