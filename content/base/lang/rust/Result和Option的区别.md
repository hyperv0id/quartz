---
tags:
  - rust
---

### Rust 中 `Result` 与 `Option` 的区别

---

#### **1. 核心用途**
| **类型**       | **用途**                                                     |
| -------------- | ------------------------------------------------------------ |
| `Result<T, E>` | 表示操作可能**成功**（`Ok(T)`）或**失败**（`Err(E)`），需处理错误原因。例如文件**操作**、网络请求等可能失败且需要错误信息的场景。 |
| `Option<T>`    | 表示值可能**存在**（`Some(T)`）或**不存在**（`None`）。适用于无需错误信息的场景，如**查找**元素、可选配置等。 |

**示例**

```rust
// Result：文件读取（可能成功或失败）
let file = File::open("hello.txt"); // Result<File, std::io::Error>

// Option：从集合中查找元素
let vec = vec![1, 2, 3];
let value = vec.get(5); // Option<&i32>（此处为 None）
```

---

#### **2. 错误信息**
| **类型**       | **错误处理**                                                 |
| -------------- | ------------------------------------------------------------ |
| `Result<T, E>` | 失败时通过 `Err(E)` 携带具体错误信息（如 `std::io::Error`），便于调试和处理。 |
| `Option<T>`    | 仅表示值缺失（`None`），**不提供错误细节**。                 |

**示例**
```rust
// Result 的错误信息传递
fn read_file() -> Result<String, std::io::Error> {
    let content = std::fs::read_to_string("missing.txt")?; // 错误时返回 Err
    Ok(content)
}

// Option 的 None 无额外信息
fn find_even(numbers: &[i32]) -> Option<&i32> {
    numbers.iter().find(|&&x| x % 2 == 0)
}
```

---

#### **3. 泛型参数**
| **类型**       | **泛型参数**                                 |
| -------------- | -------------------------------------------- |
| `Result<T, E>` | 两个参数：`T`（成功类型），`E`（错误类型）。 |
| `Option<T>`    | 一个参数：`T`（值的类型）。                  |

**自定义 Result 别名示例**
```rust
type MyResult = Result<u32, String>; // 成功返回 u32，失败返回 String
fn div(x: u32, y: u32) -> MyResult {
    if y == 0 { Err("Division by zero".to_string()) }
    else { Ok(x / y) }
}
```

---

#### **4. 处理方式**
两者均需通过 `match` 或 `?` 操作符处理所有可能情况，确保代码健壮性。

**`match` 处理示例**
```rust
// 处理 Result
match File::open("file.txt") {
    Ok(file) => println!("File opened: {:?}", file),
    Err(e) => println!("Error: {:?}", e),
}

// 处理 Option
match vec.get(2) {
    Some(val) => println!("Value: {}", val),
    None => println!("Index out of bounds"),
}
```

**`?` 操作符的差异**
- `Result` 的 `?` 遇到 `Err(e)` 会提前返回错误。
- `Option` 的 `?` 遇到 `None` 会提前返回 `None`。

---

#### **5. 转换方法**

`Option<T>`相当于`Result<T, ()>`

![image-20250322113003733](https://assets.hypervoid.top/img/2025/03/22/image-20250322113003733-8645.png)

| **操作**                | **示例**                                                     |
| ----------------------- | ------------------------------------------------------------ |
| **`Option` → `Result`** | 使用 `ok_or` 或 `ok_or_else` 添加错误信息：<br>`let res: Result<i32, &str> = Some(5).ok_or("Value missing");` |
| **`Result` → `Option`** | 使用 `ok()` 丢弃错误，或 `err()` 获取错误：<br>`let opt = res.ok();` |

---

#### **6. 适用场景总结**
| **场景**                         | **推荐类型**   |
| -------------------------------- | -------------- |
| 需要处理错误原因（如I/O、网络）  | `Result<T, E>` |
| 仅需表示值是否存在（如查找元素） | `Option<T>`    |
| 需传播错误（链式调用）           | `Result` + `?` |
| 可选参数或配置项                 | `Option<T>`    |

---

#### **关键对比表**
| **特性**     | `Result<T, E>`         | `Option<T>`              |
| ------------ | ---------------------- | ------------------------ |
| **含义**     | 成功或失败（含错误）   | 值存在或不存在           |
| **变体**     | `Ok(T)`, `Err(E)`      | `Some(T)`, `None`        |
| **错误信息** | 通过 `E` 类型传递      | 无                       |
| **泛型参数** | 2个（`T` 和 `E`）      | 1个（`T`）               |
| **适用操作** | 高风险操作（可能失败） | 低风险操作（简单值缺失） |

---

![image-20250322112433398](https://assets.hypervoid.top/img/2025/03/22/image-20250322112433398-7883.png)


# 参考资料

- [https://rustwiki.org/zh-CN/rust-by-example/std/result.html](https://rustwiki.org/zh-CN/rust-by-example/std/result.html)
- [https://rustwiki.org/zh-CN/rust-by-example/std/option.html](https://rustwiki.org/zh-CN/rust-by-example/std/option.html)