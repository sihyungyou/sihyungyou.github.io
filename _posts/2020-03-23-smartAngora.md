---
layout: post
title: "Angora의 Smart Parallelization 위한 idea들"
tags: [일상]
comments: true
---

> 캡스톤 아자아자  

일단 지금까지의 진행상황은 이렇다. Angora는 처음 오픈소스를 다운로드 받아 사용할 때 부터 이미 multi-core fuzzing은 가능했다. 하지만 이것을 distributed environment로 확장하고, multi-core, multi-process 레벨까지 구현했다. fuzzing을 실제로 진행하는 fuzz_loop 함수에서는 executor를 생성하고 이를 SearchHandler에 담아서 이것저것(?)을 하는데 이 지점에서 slave CPU들이 각각 갖고있는 local stat, branch 정보를 master의 global stat, branch로 넘겨오는 것 까지는 됐다. 또한 slave들이 new internal state를 찾을 때마다 master와 자신을 제외한 다른 slave들과 depot을 공유하기도 한다.  

하지만 이는 단순한 브랜치 데이터, 혹은 스탯의 숫자들을 가져와서 add 해주는 것에 불과하다. 진짜 분산환경에서의 fuzzing이 performance가 더 좋으려면 input 배분에 있어서 기똥찬 알고리즘을 가져와서 최대한 mutually exclusive 하게 일을 하도록 해준다거나 branch 선택 과정에 개입해서 좀 더 path를 빨리 발견하는 등의 추가적인 아이디어가 필요한 상황이다. 이를 생각날 때 마다 정리해놓으려한다.  

먼저 논문에서 가장 눈여겨 볼 만한 Algorithm 1을 실제 Angora code와 비교해가며 큰 흐름을 다시 정리해야한다. 그래야 언제 어디에 개입해서 스마트하게 발전시킬지 구상할 수 있으니까.  

## Algorithm 1  
~~~rust
pub fn sync_depot(executor: &mut Executor, running: Arc<AtomicBool>, dir: &Path) {
    executor.local_stats.clear();
    // line 4 : seeds(queue directory)에 있는 모든 input들을에 대해 하나씩 가지고온다.  
    let seed_dir = dir.read_dir().expect("read_dir call failed");
    for entry in seed_dir {
        if let Ok(entry) = entry {
            if !running.load(Ordering::SeqCst) { break; }
            // line 5 : 그 input의 경로가 path다. 그리고 그 path는 file이다.  
            let path = &entry.path();
            if path.is_file() {
                ...
                let buf = read_from_file(path);
                // run_sync -> run_inner -> do_if_has_new -> has_new
                executor.run_sync(&buf);
            }
        }
    }
}

// line 5 : run taint program with input
fn run_inner(&mut self, buf: &Vec<u8>) -> StatusType {
    self.write_test(buf);

    self.branches.clear_trace();

    unsafe { asm!("" ::: "memory" : "volatile") };
    let ret_status = if let Some(ref mut fs) = self.forksrv {
        fs.run()
    } else {
        self.run_target(&self.cmd.main, self.cmd.mem_limit, self.cmd.time_limit)
    };
    unsafe { asm!("" ::: "memory" : "volatile") };

    ret_status
}

// line 7 : branches[b] <- input
fn do_if_has_new(&mut self, buf: &Vec<u8>, status: StatusType, _explored: bool, cmpid: u32) {
    // check if there's new branch found
    let (has_new_path, has_new_edge, edge_num) = self.branches.has_new(status);

    if has_new_path {
        self.has_new_path = true;
        self.local_stats.find_new(&status);
        // if new branch exists, save into depot. (in fuzz_main, it will check if depot is empty of not to decide execute fuzz_loop of not)
        let id = self.depot.save(status, &buf, cmpid);

        if status == StatusType::Normal {
            ...
        }
    }
}
~~~
~~~rust
// line 11 : select b from branches
// get_entry를 통해 Option<(CondStmt, QPriority)>를 반환하고 priority, cond를 통해 still un explored인지 확인
let entry = match depot.get_entry() {
    Some(e) => e,
    None => break,
};

let mut cond = entry.0;
let priority = entry.1;

if priority.is_done() {
    break;
}

if cond.is_done() {
    depot.update_entry(cond);
    continue;
}

let belong_input = cond.base.belong as usize;
// 여기서 buf가 input
let buf = depot.get_input_buf(belong_input);

let handler = SearchHandler::new(running.clone(), &mut executor, &mut cond, buf);
// line 13, 14 : mutate branches [b] to get a new input input', run none taint program with input'
// OneByteFuzz::run -> OneByteFuzz::execute -> SearchHandler::execute_input -> Executor::run -> run_inner 
OneByteFuzz::new(handler).run();
// line 15 : if input' explored new branches
// Executor::do_if_has_new -> GlobalBranches::has_new 과정을 거치며 update entry, fuzz loop 진행
~~~