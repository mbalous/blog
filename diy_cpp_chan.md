# DIY C++ Channels
#### Posted July 18th, 11:00 AM

### Intro
From my perspective, Channel semantics is one of the things that Go got mostly right. Luckily; comparable functionality is only a dynamic array, a mutex and a couple of atomic variables away in any language. This post describes a take on that idea in 70 lines of portable C++, from the database I wrote for [Snackis](https://github.com/andreas-gone-wild/snackis).

### Implementation
This implementation uses atomic variables in combination with yielding. The same effect may be achieved using condition variables, one for putting and one for getting; only around 5 times slower.

```
#include <atomic>
#include <mutex>
#include <optional>
#include <vector>

template <typename T>
struct Chan {
  const size_t max;
  std::vector<T> buf;
  size_t pos;
  std::atomic<size_t> size;
  std::mutex mutex;
  std::atomic<bool> closed;

  Chan(size_t max);
};

using ChanLock = std::unique_lock<std::mutex>;
  
template <typename T>
Chan<T>::Chan(size_t max):
  max(max), pos(0), size(0), closed(false)
{ }

template <typename T>
void close(Chan<T> &c) {    
  ChanLock lock(c.mutex);
  assert(!c.closed.load());
  c.closed.store(true);
}

template <typename T>
bool put(Chan<T> &c, const T &it, bool wait=true) {
  while (true) {
    if (c.size.load() == c.max) {
      if (!wait || c.closed.load()) { break; }
      std::this_thread::yield();
      continue;
    }
    
    ChanLock lock(c.mutex);
    if (c.buf.size() == c.max) { continue; }      
    c.buf.push_back(it);
    c.size++;
    return true;
  }

  return false;
}

template <typename T>
opt<T> get(Chan<T> &c, bool wait=true) {
  while (true) {
    if (c.size.load() == 0) {
      if (!wait || c.closed.load()) { break; }
      std::this_thread::yield();
      continue;
    }

    ChanLock lock(c.mutex);
    if (c.pos == c.buf.size()) { continue; }
    auto out(c.buf[c.pos]);
    c.pos++;
      
    if (c.pos == c.buf.size()) {
      c.buf.clear();
      c.pos = 0;
    }
      
    c.size--;
    return out;
  }

  return nullopt;
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
The implementation above is fast enough for many needs but I was still curious how it stacked up against Go, so I wrote a basic benchmark loop to get an idea. The short story is that it's more than twice as fast as the built-in channels in Go 1.8.

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