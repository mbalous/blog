# Portable stack traces in C++
#### Posted May 23rd, 7:00 PM

### Background
I am currently deep into the process of porting a 5-kloc [system](https://github.com/andreas-gone-wild/snackis) from Golang to C++; since it's a reasonably complex system with a custom database-engine, encryption, networking, ui etc.; any assistance with tracking down errors is more than welcome. Like Golang, C++ doesn't provide a standard way of attaching stack-traces to errors. While there are various more or less involved libraries floating around; I prefer a simpler, more portable approach.

### Traces
Any C compiler worth it's name provides support for getting the current filename and line via the ```__FILE__``` and ```__LINE__``` macros. The code below implements a trace struct and provides a macro to simplify user code, RAII is used to keep a thread-local stack updated as traces are allocated/freed.

```
#include <sstream>
#include <string>
#include <vector>

#define _CONCAT(x, y)				\
  x ## y					\

#define CONCAT(x, y)				\
  _CONCAT(x, y)					\

#define UNIQUE(prefix)				\
  CONCAT(prefix, __COUNTER__)			\

#define TRACE(msg)				\
  Trace UNIQUE(trace)(msg, __FILE__, __LINE__)	\

struct Trace {
  const std::string msg;
  const char *file;
  const int line;

  Trace(const std::string &msg, const char *file, int line);
  ~Trace();
};

thread_local std::vector<const Trace *> stack;
  
Trace::Trace(const std::string &msg, const char *file, int line):
  msg(msg), file(file), line(line) {
  stack.push_back(this);
}

Trace::~Trace() {
  stack.pop_back();
}
  
std::string stack_trace() {
  std::stringstream out;

  for (const Trace *t: stack) {
    out << t->msg << " in file " << t->file << ", line " << t->line << ":\n";
  }
    
  return out.str();
}
```

### Errors
A general purpose tracing-facility is usable enough in itself, but the focus of this post is adding stack-traces to errors. The ```ERROR``` macro traces the current stack-frame and and throws the specified error type, an ```xError``` naming convention is adopted for error-types to simplify user code.

```
#include <stdexcept>
#include <string>

#define ERROR(type, msg) do {					\
    TRACE("Error thrown");					\
    throw CONCAT(type, Error)(msg);				\
  } while (false)						\

struct Error: public std::runtime_error {
  Error(const std::string &msg);
};

Error::Error(const std::string &msg): std::runtime_error(stack_trace() + msg) { }
```

### Example
Given facilities described above, and a custom error type like this:

```
  struct ImapError: public Error {
    ImapError(const std::string &msg);
  };

  ImapError::ImapError(const std::string &msg):
    Error(std::string("ImapError: ") + msg) { }
```

The following two lines of code:

```
TRACE("Running email_tests");
...
ERROR(Imap, std::string("Failed fetching: ") + curl_easy_strerror(res));
```

Throws an exception with a message like this:

```
Running email_tests in file /home/a/Projects/snackis/src/main.cpp, line 66:
Error thrown in file /home/a/Projects/snackis/src/snackis/net/imap.cpp, line 112:
ImapError: Failed fetching: Couldn't resolve host name
```

Until next time; be well,<br/>
A
