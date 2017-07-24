# DIY C++ Channels
#### Posted July 18th, 11:00 AM

### Intro
From my perspective, Channel semantics is one of the things that Go got mostly right. Luckily; comparable functionality is only a dynamic array, a mutex and a pair of condition-variables away in any language. This post describes a take on that idea in 60 lines of portable C++, taken from the database I wrote for [Snackis](https://github.com/andreas-gone-wild/snackis).

### Implementation

```
#include <condition_variable>
#include <mutex>
#include <optional>
#include <vector>

template <typename T>
struct Chan {
  const size_t max;
  std::vector<T> buf;
  size_t pos;
  std::mutex mutex;
  std::condition_variable get_ok, put_ok;
  bool closed;

  Chan(size_t max);
};

using ChanLock = std::unique_lock<std::mutex>;
  
template <typename T>
Chan<T>::Chan(size_t max):
  max(max), pos(0), closed(false)
{ }

template <typename T>
void close(Chan<T> &c) {    
  ChanLock lock(c.mutex);
  assert(!c.closed);
  c.closed = true;
  c.get_ok.notify_all();
  c.put_ok.notify_all();
}

template <typename T>
bool put(Chan<T> &c, const T &it, bool wait=true) {
  ChanLock lock(c.mutex);
  if (c.closed) { return false; }
  
  if (wait && c.buf.size() == c.max) {
    c.put_ok.wait(lock, [&c](){ return c.closed || c.buf.size() < c.max; });
  }

  if (c.buf.size() == c.max) { return false; }
  c.buf.push_back(it);
  c.get_ok.notify_one();
  return true;
}

template <typename T>
std::optional<T> get(Chan<T> &c, bool wait=true) {
  ChanLock lock(c.mutex);
    
  if (wait && c.pos == c.buf.size()) {
    c.get_ok.wait(lock, [&c](){ return c.closed || c.pos < c.buf.size(); });
  }
    
  if (c.pos == c.buf.size()) { return nullopt; }
  
  auto out(c.buf[c.pos]);
  c.pos++;

  if (c.pos == c.buf.size()) {
    c.buf.clear();
    c.pos = 0;
  }
  
  c.put_ok.notify_one();
  return out;
}
```

### Usage

```
const int MAX(100);
Chan<int> c(MAX);

assert(!get(c, false));

for (int i = 0; i < MAX; i++) { assert(put(c, i)); }

assert(!put(c, 42, false));

for (int i = 0; i < MAX; i++) { assert(*get(c) == i); }

assert(!get(c, false));
assert(put(c, 42));

close(c);
```

### Performance
The implementation above is fast enough for many needs but I was still curious how it stacked up against Go, so I wrote a basic benchmark loop to get an idea. The short story is that Go 1.8 is about twice as fast.

```
#include <vector>
#include <thread>
#include "snackis/core/chan.hpp"

using namespace snackis;

void run_pub(Chan<int> *ch, int reps) {
  for (int i(0); i < reps; i++) {
    put(*ch, i);
  }
}

void run_con(Chan<int> *ch, int reps) {
  for (int i(0); i < reps; i++) {
    get(*ch);
  }
}

void run(int workers, int reps, int buf) {
  std::vector<std::thread> wg;
  Chan<int> ch(buf);

  for (int i(0); i < workers; i++) {
    wg.emplace_back(run_pub, &ch, reps);
    wg.emplace_back(run_con, &ch, reps);
  }

  for (auto &t: wg) { t.join(); }
}

const int
  MAX_WORKERS(  5),
  MAX_REPS(1000000),
  MAX_BUF(    1000);

int main() {
  for (int workers(1); workers < MAX_WORKERS; workers++) {
    for(int reps(10); reps < MAX_REPS; reps *= 10) {
      for(int buf(1); buf < MAX_BUF; buf *= 10) {
	run(workers, reps, buf);
      }
    }
  }

  return 0;
}
```

```
package main

import (
	"runtime"
	"sync"
)

func run_pub(wg *sync.WaitGroup, ch chan int, reps int) {
	runtime.LockOSThread()
	
	for i := 0; i < reps; i++ {
		ch<- i
	}
	
	wg.Done()
}

func run_con(wg *sync.WaitGroup, ch chan int, reps int) {
	runtime.LockOSThread()

	for i := 0; i < reps; i++ {
		<-ch
	}
	
	wg.Done()
}

func run(workers, reps, buf int) {
	var wg sync.WaitGroup
	ch := make(chan int, buf)
	
	for i := 0; i < workers; i++ {
		wg.Add(2)
		go run_pub(&wg, ch, reps)
		go run_con(&wg, ch, reps)
	}

	wg.Wait()
}

const (
	MAX_WORKERS =    5
	MAX_REPS = 1000000
	MAX_BUF =     1000
)

func main() {
	for workers := 1; workers < MAX_WORKERS; workers++ {
		for reps := 10; reps < MAX_REPS; reps *= 10 {
			for buf := 1; buf < MAX_BUF; buf *= 10 {
				run(workers, reps, buf)
			}
		}
	}
}
```


Until next time; be well,<br/>
A