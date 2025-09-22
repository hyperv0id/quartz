

## 简介

[lab1.pdf (cs144.github.io)](https://cs144.github.io/assignments/lab1.pdf)

 

### 环境配置

1. 确保lab0的代码已经提交

2. 运行git fetch来检索最新版本的实验室作业

3. 使用下面的合并分支命令来获得lab1的代码，图中红框部分就是lab1的任务

```bash
git merge origin/lab1-startercode  
```

| ![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2022/11/15/clip_image002-f0422e.gif) | ![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2022/11/15/clip_image004-f663ed.gif) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |

### **我是谁？我在哪？我要干什么？**

下面是一个cs144的任务模型，咱们要从零开始实现一个TCP，本次lab需要实现ByteStream部分

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2022/11/15/clip_image006-b06828.gif)

 

 

## Solution

 

### Bitmap版本

#### 头文件

```C++
#ifndef SPONGE_LIBSPONGE_STREAM_REASSEMBLER_HH
#define SPONGE_LIBSPONGE_STREAM_REASSEMBLER_HH

#include "byte_stream.hh"

#include <cstdint>
#include <string>
#include <deque>

//! \brief A class that assembles a series of excerpts from a byte stream (possibly out of order,
//! possibly overlapping) into an in-order byte stream.
class StreamReassembler {
  private:
    // Your code here -- add private members as necessary.

    size_t _unass_idx = 0; //!< 第一次未处理的数据段位置
    size_t _unass_size = 0; //!< 未处理数据段长度

    bool _eof = false; //!> 结束标志，可能还有未处理的数据段需要读取

    std::deque<char> _buf={}; //!< 未处理的数据段
    std::deque<bool> _bitmap={}; //!< 和 _buf 一起表示这个位置上有没有数据

    ByteStream _output;  //!< The reassembled in-order byte stream
    size_t _capacity;    //!< The maximum number of bytes

    //! \brief 向 _output 中写入数据
    void write_to_buf();

  public:
    //! \brief Construct a `StreamReassembler` that will store up to `capacity` bytes.
    //! \note This capacity limits both the bytes that have been reassembled,
    //! and those that have not yet been reassembled.
    StreamReassembler(const size_t capacity);

    //! \brief Receive a substring and write any newly contiguous bytes into the stream.
    //!
    //! The StreamReassembler will stay within the memory limits of the `capacity`.
    //! Bytes that would exceed the capacity are silently discarded.
    //!
    //! \param data the substring
    //! \param index indicates the index (place in sequence) of the first byte in `data`
    //! \param eof the last byte of `data` will be the last byte in the entire stream
    void push_substring(const std::string &data, const uint64_t index, const bool eof);

    //! \name Access the reassembled byte stream
    //!@{
    const ByteStream &stream_out() const { return _output; }
    ByteStream &stream_out() { return _output; }
    //!@}

    //! The number of bytes in the substrings stored but not yet reassembled
    //!
    //! \note If the byte at a particular index has been pushed more than once, it
    //! should only be counted once for the purpose of this function.
    size_t unassembled_bytes() const;

    //! \brief Is the internal state empty (other than the output stream)?
    //! \returns `true` if no substrings are waiting to be assembled
    bool empty() const;
};

#endif  // SPONGE_LIBSPONGE_STREAM_REASSEMBLER_HH
```

#### 源文件

```C++
#include "stream_reassembler.hh"

// Dummy implementation of a stream reassembler.

// For Lab 1, please replace with a real implementation that passes the
// automated checks run by `make check_lab1`.

// You will need to add private members to the class declaration in `stream_reassembler.hh`

template <typename... Targs>
void DUMMY_CODE(Targs &&... /* unused */) {}

using namespace std;

StreamReassembler::StreamReassembler(const size_t capacity) 
    : _unass_idx(0)
    , _unass_size(0)
    , _eof(false)
    , _buf(capacity, 0)
    , _bitmap(capacity, false)
    , _output(capacity)
    , _capacity(capacity){}

void StreamReassembler::write_to_buf(){
    string seg;
    while (_bitmap.front())
    {
        // 添加数据
        seg.push_back(_buf.front());
        _buf.pop_front();
        _bitmap.pop_front();
        // 维持capacity恒定
        _buf.push_back(0);
        _bitmap.push_back(false);
    }
    if(!seg.empty()){
        _output.write(seg);
        // 更新 unassembled 相关参数，可以改成 length
        _unass_idx += seg.size();
        _unass_size -= seg.size();
    }
}

//! \details This function accepts a substring (aka a segment) of bytes,
//! possibly out-of-order, from the logical stream, and assembles any newly
//! contiguous substrings and writes them into the output stream in order.
void StreamReassembler::push_substring(const string &data, const size_t index, const bool eof) {
    if(eof){_eof = true;}
    // 太远了装不下，直接丢掉
    if(index >= _unass_idx + _capacity){return;}
    size_t len = data.size();
    if(index >= _unass_idx){
        int offset = index - _unass_idx;
        size_t times = min(len, _capacity - _output.buffer_size() - offset);
        if(times < len){_eof = false;}
        for (size_t i = 0; i < times; i++)
        {
            if(_bitmap[i+offset]){continue;}
            _bitmap[i+offset] = true;
            _buf[i+offset] = data[i];
            _unass_size++;
        }
        
    }else if(index + len > _unass_idx){
        int offset = _unass_idx - index;
        size_t times = min(len - offset, _capacity - _output.buffer_size());
        if(times < len - offset){_eof = false;}
        for (size_t i = 0; i < times; i++)
        {
            if(_bitmap[i]){continue;}
            _bitmap[i] = true;
            _buf[i] = data[i+ offset];
            _unass_size++;
        }
        
    }
    
    write_to_buf();

    if(_eof && _unass_size ==0){
        // 停止向 ByteStream 中输入
        _output.end_input();
    }
}

size_t StreamReassembler::unassembled_bytes() const { return this->_unass_size; }

bool StreamReassembler::empty() const { return _unass_size==0; }
```

 

#### **Check**

```bash
cd build
make
make check_lab1  
```

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2022/11/15/clip_image008-6de9d7.gif)

### Set版本

TODO



 