---
tags:
  - rust
---

## 1. 线程基础

### 1.1 创建线程
```rust
use std::thread;

let handle = thread::spawn(|| {
    // 线程执行代码
    println!("New thread started!");
});
```
- 使用 `thread::spawn` 创建新线程，返回 `JoinHandle`
- 线程在闭包中执行代码
- 主线程结束时，未完成的子线程会被强制终止

### 1.2 等待线程
```rust
handle.join().expect("Failed to join thread");
```
- `JoinHandle.join()` 会阻塞当前线程直到目标线程完成
- 返回 `Result<(), Box<dyn Any + Send>>`，处理可能发生的 panic

## 2. 线程同步

### 2.1 共享内存模型

#### 原子类型 (Atomics)
```rust
use std::sync::atomic::{AtomicUsize, Ordering};

let counter = AtomicUsize::new(0);
counter.fetch_add(1, Ordering::SeqCst);
```
- 提供原子操作的基本类型（AtomicBool, AtomicIsize, AtomicUsize 等）
- 需要指定内存顺序（Ordering::Relaxed, Ordering::SeqCst 等）

#### 不可变共享 (`Arc<T>`)

```rust
use std::sync::Arc;

let data = Arc::new(vec![1, 2, 3]);
let data_clone = Arc::clone(&data);
```

- `Arc`（原子引用计数）实现线程间共享只读数据
- 克隆 Arc 只会增加引用计数

#### 可变共享 (`Arc<Mutex<T>>`)

```rust
use std::sync::{Arc, Mutex};

let counter = Arc::new(Mutex::new(0));
let mut num = counter.lock().unwrap();
*num += 1;
```
- `Mutex` 提供互斥访问，`RwLock` 支持读写分离
- 组合使用 `Arc` + `Mutex` 实现线程安全可变共享

### 2.2 消息传递模型 (CSP/Actor)

#### mpsc 多生产者单消费者
```rust
use std::sync::mpsc;

let (tx, rx) = mpsc::channel();
tx.send(42).unwrap();
let received = rx.recv().unwrap();
```
- 多生产者单消费者通道
- `send` 可能阻塞，`try_send` 立即返回
- `recv` 阻塞接收，`try_recv` 非阻塞

#### oneshot 单次通信
```rust
use std::sync::mpsc::oneshot;

let (tx, rx) = oneshot::channel();
tx.send(42).unwrap();
let result = rx.recv().unwrap();
```
- 专为单次消息传递设计的通道
- 常用于请求-响应模式

## 3. 并发数据处理

### 3.1 并行迭代器（rayon）
```rust
use rayon::prelude::*;

let sum = (0..1000).par_iter().sum();
```

### 3.2 数据分块处理
```rust
let chunks = data.chunks(data.len() / num_threads);
for chunk in chunks {
    // 分发到不同线程处理
}
```

## 4. 同步机制选择指南

| 场景                      | 推荐方案                 |
|--------------------------|-------------------------|
| 只读数据共享              | `Arc<T>`                  |
| 简单计数器/标志位         | Atomic 类型             |
| 复杂数据结构共享访问      | `Arc<Mutex<T>>` 或 `Arc<RwLock<T>>` |
| 任务间通信                | mpsc 或 oneshot        |
| 流水线式数据处理          | 通道组合                |

## 注意事项
1. 避免死锁：确保锁的获取顺序一致
2. 最小化锁粒度：减少持有锁的时间
3. 优先选择消息传递：更易维护和推理
4. 注意 `Send` 和 `Sync` trait 的自动推导
5. 使用 `cargo test -- --test-threads=1` 避免测试干扰