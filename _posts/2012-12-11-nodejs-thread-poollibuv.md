---
layout: post
title: node.js 의 thread pool(libuv) 부분
date: '2012-12-11T12:21:00.000+09:00'
tags:
    - node.js
modified_time: '2012-12-11T12:29:04.363+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-3741120756531587307
blogger_orig_url: https://jeremyko.blogspot.com/2012/12/nodejs-thread-poollibuv.html
---

node.js 에서의 입출력 동작은 멀티쓰레드(thread pool)로 구현된다.
소스 코드에서 이 부분을 정리해본다.

다음과 같이 파일을 비동기로 읽어오는 스크립트가 있을때, 함수 호출을 따라가 보면 다음과 같다.

```js
var fs = require('fs');
fs.readFile('./bigFile.txt', 'utf8', function (err, data) {
    console.log(data);
});
console.log('exiting...');
```

사용자의 스크립트에서 호출한 `fs.readFile` 함수는 `/lib/fs.js` 에 존재하며
내부적으로 다음 함수 호출을 한다.

    binding.read(fd, buffer, offset, length, position, wrapper);

`node_file.cc` 를 보면 javascript 에서 호출한 read 함수는 다음과 같이 정의 되었기 때문에

    NODE_SET_METHOD(target, "read", Read);

결국 C++ 소스의 Read 함수를 호출하게 된다.

Read함수 정의는 다음과 같다.

<span style="color:{{site.span_emphasis_color}}">
Read (src/node_file.cc)
</span>

```cpp
static Handle<Value> Read(const Arguments& args) {
  HandleScope scope;
  ...
  int fd = args[0]->Int32Value();

  Local<Value> cb;
  ...
  pos = GET_OFFSET(args[4]);

  buf = buffer_data + off;

  cb = args[5];

  if (cb->IsFunction()) {
    ASYNC_CALL(read, cb, fd, buf, len, pos);
  } else {
    SYNC_CALL(read, 0, fd, buf, len, pos)
    Local<Integer> bytesRead = Integer::New(SYNC_RESULT);
    return scope.Close(bytesRead);
  }
}
```

비동기 호출을 하는경우, 콜백이 존재하고 이때 `ASYNC_CALL` 매크로로 처리된다.

<span style="color:{{site.span_emphasis_color}}">
ASYNC_CALL (src/node_file.cc)
</span>

```cpp
#define ASYNC_CALL(func, callback, ...)                           \
  FSReqWrap* req_wrap = new FSReqWrap(#func);                     \
  int r = uv_fs_##func(uv_default_loop(), &req_wrap->req_,        \
      __VA_ARGS__, After);                                        \
  req_wrap->object_->Set(oncomplete_sym, callback);               \
  req_wrap->Dispatched();                                         \
  if (r < 0) {                                                    \
    uv_fs_t* req = &req_wrap->req_;                               \
    req->result = r;                                              \
    req->path = NULL;                                             \
    req->errorno = uv_last_error(uv_default_loop()).code;         \
    After(req);                                                   \
  }                                                               \
  return scope.Close(req_wrap->object_);
```

이 매크로 내부에서 read 요청은 libuv 함수 `uv_fs_read` 호출로 변경된다.

<span style="color:{{site.span_emphasis_color}}">
uv_fs_read (deps/uv/src/unix/fs.c)
</span>

```cpp
int uv_fs_read(uv_loop_t* loop, uv_fs_t* req,
               uv_file file,
               void* buf,
               size_t len,
               int64_t off,
               uv_fs_cb cb) {
  INIT(READ);
  req->file = file;
  req->buf = buf;
  req->len = len;
  req->off = off;
  POST;
}
```

먼저 INIT처리, 이후 POST 처리를 하고 있다.

<span style="color:{{site.span_emphasis_color}}">
INIT (deps/uv/src/unix/fs.c)
</span>

```cpp
#define INIT(type)                                                            \
  do {                                                                        \
    uv__req_init((loop), (req), UV_FS);                                       \
    (req)->fs_type = UV_FS_ ## type;                                          \
    (req)->errorno = 0;                                                       \
    (req)->result = 0;                                                        \
    (req)->ptr = NULL;                                                        \
    (req)->loop = loop;                                                       \
    (req)->path = NULL;                                                       \
    (req)->new_path = NULL;                                                   \
    (req)->cb = (cb);                                                         \
  }                                                                           \
  while (0)
```

<span style="color:{{site.span_emphasis_color}}">
POST (deps/uv/src/unix/fs.c)
</span>

```cpp
#define POST                                                                  \
  do {                                                                        \
    if ((cb) != NULL) {                                                       \
      uv__work_submit((loop), &(req)->work_req, uv__fs_work, uv__fs_done);    \
      return 0;                                                               \
    }                                                                         \
    else {                                                                    \
      uv__fs_work(&(req)->work_req);                                          \
      uv__fs_done(&(req)->work_req);                                          \
      return (req)->result;                                                   \
    }                                                                         \
  }                                                                           \
  while (0)
```

여기서 thread pool 로 처리가 되기 위한 `uv__work_submit` 함수 호출을 볼수 있다.

<span style="color:{{site.span_emphasis_color}}">
uv__work_submit (deps/uv/src/unix/threadpool.c)
</span>

이 함수는 초기화 작업이 한번만 수행되도록 `pthread_once` 를 호출하였다. 이 초기화 작업에 쓰레드 풀을 생성하는것이 포함된다.

```cpp
void uv__work_submit(uv_loop_t* loop,
                     struct uv__work* w,
                     void (*work)(struct uv__work* w),
                     void (*done)(struct uv__work* w)) {
  pthread_once(&once, init_once);
  w->loop = loop;
  w->work = work;
  w->done = done;
  post(&w->wq);
}
```

<span style="color:{{site.span_emphasis_color}}">
init_once (deps/uv/src/unix/threadpool.c)
</span>

다음처럼 threads 배열의 크기만큼 쓰레드 풀을 생성하고 있다.
linux/mac 의 경우 쓰레드풀을 위해 고정된 쓰레드수가 4로 지정되어 있다.
windows 경우는 OS 차원의 쓰레드풀을 활용하므로(`QueueUserWorkItem` 호출)
이런 고정 갯수는 존재하지 않는다.

```cpp
static pthread_t threads[4];
...
static void init_once(void) {
  unsigned int i;

  ngx_queue_init(&wq);

  for (i = 0; i < ARRAY_SIZE(threads); i++)
    if (pthread_create(threads + i, NULL, worker, NULL))
      abort();

  initialized = 1;
}
```

<span style="color:{{site.span_emphasis_color}}">
post (deps/uv/src/unix/threadpool.c)
</span>

여기서는 큐에 작업을 넣고, 조건변수에 신호를 보낸다.

```cpp
static void post(ngx_queue_t* q) {
  pthread_mutex_lock(&mutex);
  ngx_queue_insert_tail(&wq, q);
  pthread_cond_signal(&cond);
  pthread_mutex_unlock(&mutex);
}
```

한편, 이 thread pool에서 작업을 처리하는 쓰레드 함수는 다음과 같다.
큐에 작업이 삽입될때 까지 조건변수를 기다리고 신호를 받으면 작업큐에서 가져와서 처리한다.

```cpp
static void* worker(void* arg) {
  struct uv__work* w;
  ngx_queue_t* q;

  (void) arg;

  for (;;) {
    if (pthread_mutex_lock(&mutex))
      abort();

    while (ngx_queue_empty(&wq))
      if (pthread_cond_wait(&cond, &mutex))
        abort();

    q = ngx_queue_head(&wq);

    if (q == &exit_message)
      pthread_cond_signal(&cond);
    else
      ngx_queue_remove(q);

    if (pthread_mutex_unlock(&mutex))
      abort();

    if (q == &exit_message)
      break;

    w = ngx_queue_data(q, struct uv__work, wq);
    w->work(w);

    uv_mutex_lock(&w->loop->wq_mutex);
    ngx_queue_insert_tail(&w->loop->wq, &w->wq);
    uv_mutex_unlock(&w->loop->wq_mutex);
    uv_async_send(&w->loop->wq_async);
  }

  return NULL;
}
```

<span style="color:{{site.span_emphasis_color}}">
ngx_queue_insert_tail (deps/uv/include/uv-private/ngx-queue.h)
</span>

큐에 작업을 넣는 부분은 다음과 같이 구현된다. 리스트를 이용해서 큐를 구현하였다.

```cpp
#define ngx_queue_insert_tail(h, x)                                           \
  do {                                                                        \
    (x)->prev = (h)->prev;                                                    \
    (x)->prev->next = x;                                                      \
    (x)->next = h;                                                            \
    (h)->prev = x;                                                            \
  }                                                                           \
  while (0)
```

소스를 따라가면 누구나 알수 있는 내용이지만, 한번 간단히 요약해 보았다.
