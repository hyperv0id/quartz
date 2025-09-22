---
tags: [" #algorithm"]
aliases: [雪花算法]
---

## 简介

雪花算法是一种生成分布式ID的算法，能够在多台机器上生成，并保证**全局唯一**，**递增**，**高并发**，**高可用**

SnowFlake是Twitter公司采用的一种算法，目的是在分布式系统中产生全局唯一且趋势递增的ID。

![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2022/12/04/20221204153627-b614cc.png)

### 组成部分（64bit）

1. **第一位** 占用1bit，其值始终是0，没有实际作用。 
2. **时间戳** 占用41bit，精确到毫秒，总共可以容纳约69年的时间。 
3. **工作机器id** 占用10bit，其中高位5bit是数据中心ID，低位5bit是工作节点ID，做多可以容纳1024个节点。 
4. **序列号** 占用12bit，每个节点每毫秒0开始不断累加，最多可以累加到4095，一共可以产生4096个ID。

SnowFlake算法在同一毫秒内最多可以生成多少个全局唯一ID呢：： 同一毫秒的ID数量 = 1024 X 4096 = **4194304**

注意：JS 的整数最大只支持 53 位，不能给前端传整形，会溢出

## 实现与变体

### golang
```go
package snow

import (
    "errors"
    "sync"
    "time"
)

// 0(1位，且始终为0)|时间戳(41位)|工作机器id(10位)|序列号(12位)
type SNOW struct {
    machineID     int64
    snow          int64
    lastTimeStamp int64
    lock          sync.Mutex
}

func (snow *SNOW) initTimeStamp() {
    snow.lastTimeStamp = time.Now().UnixNano() / 1e6
}

func newSnow(machineID int64) (*SNOW, error) {
    snow := &SNOW{
        machineID:     0,
        snow:          0,
        lastTimeStamp: time.Now().UnixNano() / 1e6,
        lock:          sync.Mutex{},
    }
    err := snow.setMachineID(machineID)
    if err != nil {
        return &SNOW{}, err
    }
    return snow, err
}

func (snow *SNOW) setMachineID(id int64) error {
    if id > 1024 {
        return errors.New("Machine id must lower than 1024!")
    }
    snow.machineID = id << 12 // 左移12位变成机器号
    return nil
}

func (snow *SNOW) getID() int64 {
    // snow.lock.Lock()
    // defer snow.lock.Unlock()
    return snow.snowID()
}

func (snow *SNOW) snowID() int64 {
    curTimeStamp := time.Now().UnixNano() / 1e6
    if curTimeStamp == snow.lastTimeStamp { // 请求id的时间发生冲突
        // fmt.Println("current time is ", curTimeStamp, "last time is ", snow.lastTimeStamp, " cur snow is ", snow.snow)
        snow.snow++ // 防止冲突直接加一
        if snow.snow > 4095 {
            time.Sleep(time.Nanosecond) // 无法加一的时候直接睡眠并置0，一定不会冲突，相当于同时修改snow以及时间戳
            curTimeStamp = time.Now().UnixNano() / 1e6
            snow.lastTimeStamp = curTimeStamp
            snow.snow = 0
        }
        timeStampInSnmow := curTimeStamp & 0x1FFFFFFFFFF // 保留低41位的值
        timeStampInSnmow <<= 22                          // 作为时间戳的41位时间值
        return timeStampInSnmow | snow.machineID | snow.snow
    } else {
        snow.snow = 0
        snow.lastTimeStamp = curTimeStamp
        timeStampInSnmow := curTimeStamp & 0x1FFFFFFFFFF // 保留低41位的值
        timeStampInSnmow <<= 22                          // 作为时间戳的41位时间值
        return timeStampInSnmow | snow.machineID | snow.snow
    }
}
```

