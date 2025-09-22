# CPU分支预测(Branch Prediction)详解

## 基本概念

分支预测是现代CPU中的一种重要优化机制，用于预测条件分支指令（如if-else语句）的执行路径。CPU会根据历史执行记录来预测下一次分支的走向，从而提前执行预测路径上的指令。

### 工作原理
- **流水线执行**：CPU采用指令流水线技术提高效率
- **预测执行**：在条件判断结果出来之前就开始执行预测路径
- **投机执行**：预测正确则继续，错误则回滚并执行正确路径
- **分支目标缓冲器(BTB)**：存储分支历史信息

### 预测策略
1. **静态预测**
   - 向前分支默认不跳转
   - 向后分支默认跳转（适用于循环）

2. **动态预测**
   - 一位预测器：记录上次是否跳转
   - 两位预测器：需要连续两次预测错误才改变预测
   - 相关预测：考虑多个分支之间的关系

## 测试分支预测

既然CPU在IF语句执行前就将对应指令放入流水线执行了，即使可能因为预测失败而被冲洗掉，仍然可以从**缓存**中查看CPU的痕迹。

### 预测代码

```c
// 使用GCC编译: gcc -O0 predict.c

#include <stdint.h>
#include <stdio.h>
#include <string.h>
#include <x86intrin.h>

#define PAGE_NUM 256
#define CACHE_HIT_THRESHOLD 100

// 获取访问时间
static inline int get_access_time(uint8_t *page) {
  unsigned long long tick1, tick2;
  unsigned int aux;

  tick1 = __rdtscp(&aux);
  tick2 = __rdtscp(&aux);

  return (int)(tick2 - tick1);
}

static uint8_t mem_pages[PAGE_NUM][1024 * 4] __attribute__((aligned(4096)));

// 检测缓存中的页面
static int detect_cached_page() {
  for (int i = 0; i < PAGE_NUM; i++) {
    // 混合访问顺序以防止步长预测
    uint8_t mix_i = ((i * 167) + 13) & 255;
    if (get_access_time(mem_pages[mix_i]) < CACHE_HIT_THRESHOLD) {
      return mix_i;
    }
  }

  return -1;
}

int data = 100;

int main() {
  // 初始化内存页
  memset(mem_pages, 1, sizeof(mem_pages));

  // 测试循环
  for (int i = 0; i < 10; i++) {
    // 刷新缓存
    _mm_clflush(&data);
    for (int j = 0; j < PAGE_NUM; j++) {
      _mm_clflush(mem_pages[j]);
    }

    // 分支预测测试
    if (data == 0) {
      data = mem_pages[33][0];
    }

    // 检测缓存页面
    int cached_page = detect_cached_page();

    printf("cached page: %d, data: %d\n", cached_page, data);
  }

  return 0;
}

```

- cpp的likely/unlikely就是用来优化分支预测的
## 动态预测

- [ ] tage算法

## Reference

1. [阿布編程 | GitHub](https://github.com/idea4good/AbuCoding?tab=readme-ov-file)