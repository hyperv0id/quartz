学习资料：[Go 语言上手 - 基础语法 .pptx](https://bytedance.feishu.cn/file/boxcnDQ57K0wtcZtA3Y26ORKwec) 

## 01 - 简介

### 1.1 - golang特点

1. 高性能、高并发：高性能标准库
2. 语法简单、学习曲线平缓：基于C语言
3. 丰富的标准库：
4. 完善的工具链：test、fmt、性能优化
5. 静态链接：编译成可执行文件
6. 快速编译：静态语言中几乎最快
7. 跨平台：根据平台不同会编译成不同可执行文件
8. 垃圾回收：三色消除法

## 2 - 基础语法

略

## 3 - 实战

## 3.1 - 猜谜游戏

注意字符串处理时可能以 `\r\n` 结尾，会使得字符串转换时失败

```Go
package main

import (
   "bufio"
   "fmt"
   "math/rand"
   "os"
   "strconv"
   "strings"
   "time"
)

func main() {
   maxNum := 100
   rand.Seed(time.Now().Unix())
   secretNum := rand.Intn(maxNum)
   //fmt.Println(secretNum)
   fmt.Print("Please guess a number: ")
   reader := bufio.NewReader(os.Stdin)
   for {
      input, err := reader.ReadString('\n')
      if err != nil {
         fmt.Println("read failed")
         continue
      }
      input = strings.TrimSuffix(input, "\r\n")
      guess, err := strconv.Atoi(input)
      if err != nil {
         fmt.Println("Invalid input")
         continue
      }
      fmt.Println("Your guess is:", guess)
      if guess > secretNum {
         fmt.Println("Larger, guess a smaller one")
      } else if guess < secretNum {
         fmt.Println("Smaller, guess a larger one")
      } else {
         fmt.Println("You win!!!")
         break
      }

   }
}
```

输出：

```Plain
Please guess a number: 50
Your guess is: 50
Smaller, guess a larger one
75
Your guess is: 75
Smaller, guess a larger one
80
Your guess is: 80
Smaller, guess a larger one
82
Your guess is: 82
Larger, guess a smaller one
81
Your guess is: 81
You win!!!
```

## 3.2 - 在线词典

```Go

```

## 3.3 - socks5协议