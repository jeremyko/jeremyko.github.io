---
layout: post
title: 'byte_rb : rust 에서 구현된 ring buffer'
date: 2023-02-25 11:17:57 +0900
# categories: python # 별도 폴더로 관리된다 !!!
tags: [rust]
---

rust 에서 고정길이 메모리 할당 후 memcpy 로 구현한 ring buffer 

[https://github.com/jeremyko/byte_rb](https://github.com/jeremyko/byte_rb)

[https://crates.io/crates/byte_rb](https://crates.io/crates/byte_rb)


### usage

```rust
use byte_rb::BrBuffer;

let mut cbuf = BrBuffer::new(6);

assert!(cbuf.append(6, b"123456").unwrap());
assert_eq!(cbuf.rpos(), 0);
assert_eq!(cbuf.wpos(), 6);
// "123456"
let result = cbuf.get(3).unwrap();
assert_eq!(result, b"123");
assert_eq!(cbuf.cumulated_len(), 3);
assert_eq!(cbuf.rpos(), 3);
assert_eq!(cbuf.wpos(), 6);
// "  456"

assert!(cbuf.append(3, b"789").unwrap());
assert_eq!(cbuf.cumulated_len(), 6);
assert_eq!(cbuf.rpos(), 3);
assert_eq!(cbuf.wpos(), 3);
// "789456"

let result = cbuf.get(1).unwrap();
assert_eq!(result, b"4");
assert_eq!(cbuf.rpos(), 4);
assert_eq!(cbuf.wpos(), 3);
assert_eq!(cbuf.cumulated_len(), 5);
// "789 56"

let result = cbuf.get(5).unwrap();
assert_eq!(result, b"56789");
assert_eq!(cbuf.rpos(), 3);
assert_eq!(cbuf.wpos(), 3);
assert_eq!(cbuf.cumulated_len(), 0);

assert_eq!(cbuf.capacity(), 6);
```
