---
layout: post
title: "Functions of Angora Fuzzer"
tags: [Fuzzing, Angora, 캡스톤]
comments: true
---

> 앙고라 함수 정리  

[Angora](https://github.com/AngoraFuzzer/Angora)는 모듈화가 잘 되어있어서 코드 분석할 때 여기저기 왔다갔다를 많이 해야한다. 그러다보니 한 함수의 역할을 공부하는 데 또 다른 함수들이 많이 포함되어있어 자꾸 까먹게된다. 그래서 fuzz_main.rs 에서 시작해 Angora의 흐름을 타고 가면서 만나는 함수들의 역할들을 차례대로 간단하게 정리해놓았다.  

### sync_depot (executor: &mut Executor, running: Arc<AtomicBool>, dir: &Path)  
하는 일은 크게 네 가지. executor.local_stats clear, read seed direcotry, validity check for every entry of seeds, 그리고 읽은 seed를 가지고 run_sync. fuzz_main에서 생성되는 stats이 executor 생성 시점에 인자로 전달되는데 이는 global_stats이라는 이름으로 넘겨진다. 즉, local_stats는 executor마다 갖고있는 fuzzing 진행정보로써 global_stats와는 다르다.  

### run_sync (&mut self, buf: &Vec<u8>)  
run_init, run_inner, do_if_has_new  세 함수로 이루어졌다. run_init()은 has_path_new 변수를 false로 설정하고 local_stats의 실행횟수(num_exec)를 카운트한다.  

### run_inner (&mut self, buf: &Vec<u8>) -> StatusType  
run_inner는 먼저 읽어들인 seed를 받아서 write_test, branches clear trace(이 때 global branches는 아닌 것 같다), 그리고 compiler_fence로 스레드들간의 충돌을 방지한 후에 forksrv가 존재하면 fs.run(), 그렇지 않다면 run_target한다. 두 경우에 대해 결과가 ret_status에 담기고 return, do_if_has_new_path의 status 인자로 전달됨.  

### write_test (&mut self, buf: &Vec<u8>), rewind(&mut self)  
write_buf, cmd가 stdin이라면 rewind. write_buf는 파일에 buf를 쓰고 길이를 확인하고 flush하는 말그대로 write_test의 과정. rewind는 write_buf했으니 커서를 0으로 돌려놓는 함수.  

### compiler_fence (order: Ordering)  
이 함수는 Angora defined가 아닌 Rust 함수다. 간단하게 fence를 쳐놓고 그 구간에서는 re-ordering을 하지 못하도록 하는 것이다. 이는 싱글스레드에서는 문제가 되지 않으나 여러 스레드들이 동시에 메모리를 modify하려 할 때 필요한 함수다. order에는 SeqCst, Release 등 여러 ordering semantics가 온다.    

### run_target (&self, target: &(String, Vec<String>), mem_limit: u64, time_limit: u64,) -> StatusType  
전달받은 input으로 프로그램을 돌리고 crash, normal, timeout 판단하여 return  

### forksrv.run (&mut self) -> StatusType  
흠.. 여긴 잘 모르겠다  

### do_if_has_new (&mut self, buf: &Vec<u8>, status: StatusType, _explored: bool, cmpid: u32)  
branches.has_new를 통해 has_new_path, has_new_edge, edge_num의 값을 설정. 만약 new path가 있다면 local_stats에서 find_new, depot에서 save. (추후 보충)  

### has_new (&mut self, status: StatusType) -> (bool, bool, usize) 
num_new_edge가 0보다 크면 has_new_edge도 true로 설정. to_write이 empty거나 status type이 virgin / timeout / crash 중 어느것도 아니라면 has_new_path와 함께 false로 return. (추후 보충)  

### depot.save(&self, status: StatusType, buf: &Vec<u8>, cmpid: u32) -> usize  
status에 따라 normal / timeout / crash로 save_input 한다. save_input은 각 status 타입의 경우(num_inputs, num_hangs, num_crashes)가 몇 번 있었는지 기록하는 변수를 증가시킨다. 그리고 normal이면 input_dir, timeout은 hands_dir, crash는 crashes_dir에 저장한다.  

### init_cpus_run_fuzzing_threads  
free cpu 개수 파악, 만약 num_jobs가 그것보다 많다면 어떤 스레드도 cpu core와 bind되지 않음(아마 OS에서 임의로 스레드들을 적절히 스케줄링할 듯) free cpu core가 충분하다면 앙고라 실행 시 num_jobs 옵션만큼 스레드를 spawn 해서 core 하나씩과 bind시킨다. 그리고 각각의 코어에서 실제 fuzzing 시작.  

### fuzz_loop  
executor 생성. 이는 fuzz_main에서의 executor와 별개로 child process를 spawn하여 target program을 돌린다. 그리고 running.load일 동안 다음을 수행한다. 

### main_thread_sync_and_log  
  