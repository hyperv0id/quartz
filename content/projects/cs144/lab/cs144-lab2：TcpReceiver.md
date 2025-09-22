## description
[lab2.pdf (cs144.github.io)](https://cs144.github.io/assignments/lab2.pdf)

## solution

### WrappingInt32
这里需要在 isn、abs_seq_no，seq_no之间相互转换

转换图如下：
![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2022/11/15/20221115210405-a4b03e.png)

```cpp
//! Transform an "absolute" 64-bit sequence number (zero-indexed) into a WrappingInt32
//! \param n The input absolute 64-bit sequence number
//! \param isn The initial sequence number
WrappingInt32 wrap(uint64_t n, WrappingInt32 isn) {
    // 将 64位 Absolute Sequence Numbers 转换成 32位
    // 其中 n 为 64比特 序列号（32bit只占4GB，会被很快用完）
    // isn 为初始的序列号（随机，加强安全性）
    return WrappingInt32(static_cast<uint32_t>(n) + isn.raw_value());
}

//! Transform a WrappingInt32 into an "absolute" 64-bit sequence number (zero-indexed)
//! \param n The relative sequence number
//! \param isn The initial sequence number
//! \param checkpoint A recent absolute 64-bit sequence number
//! \returns the 64-bit sequence number that wraps to `n` and is closest to `checkpoint`
//!
//! \note Each of the two streams of the TCP connection has its own ISN. One stream
//! runs from the local TCPSender to the remote TCPReceiver and has one ISN,
//! and the other stream runs from the remote TCPSender to the local TCPReceiver and
//! has a different ISN.
uint64_t unwrap(WrappingInt32 n, WrappingInt32 isn, uint64_t checkpoint) {
    // 将Wrappinglnt32 类型的 数值n 转换为 absolute seqNO
    // 参数:n -相对序号
    // isn -初始随机序列号
    // 检查点——最近的64位绝对序列号
    // 返回:包装到“n”并最接近“检查点”的64位序列号
    // 注意，tcp的两个流都有自己独立的 ISN
    uint64_t INT32 = 1l<<32;
    uint32_t offset = n - isn; // 当前序列号到初始序列号之差
    // abs_seq_no % INT_32 == offset
    if(checkpoint > offset){
        uint64_t real_checkpoint = (checkpoint - offset) + (1l<<31);
        uint64_t wrap_no = real_checkpoint / INT32;
        return wrap_no * INT32 + offset;
    }
    return offset;
}
```

### tcp_receiver header

贴一个状态转移图：
![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2022/11/15/20221115210254-e6ba52.png)

```cpp
//! Receives and reassembles segments into a ByteStream, and computes
//! the acknowledgment number and window size to advertise back to the
//! remote TCPSender.
class TCPReceiver {
    WrappingInt32 _isn;
    bool _is_syn_got;

    //! Our data structure for re-assembling bytes.
    StreamReassembler _reassembler;

    //! The maximum number of bytes we'll store.
    size_t _capacity;
    ...
```

- 接收端只有在`syn`收到之后才会接收，否则直接丢弃
- `isn`表示初始设置的编号（需要转化为绝对编号）

### tcp_receiver source



```cpp
/**
 * \brief Tcp Receiver 有三种状态
 * 1. LISTEN：等待SYN包，不能传数据，如果不是SYN直接丢掉
 * 2. SYN_RECV：SYN包到达，可以传输数据
 * 3. FIN_RECV：FIN包到达，结束输入，输出神域的
*/

using namespace std;

void TCPReceiver::segment_received(const TCPSegment &seg) {
    TCPHeader header = seg.header();
    if(!_is_syn_got){
        if(!header.syn){
            // 丢弃
            return;
        }
        _is_syn_got = true;
        _isn = header.seqno;
    }
    uint64_t abs_ackno = _reassembler.stream_out().bytes_written() + 1;
    uint64_t cur_ackno = unwrap(header.seqno, _isn, abs_ackno);
    uint64_t stream_idx = cur_ackno - 1 + header.syn;

    _reassembler.push_substring(seg.payload().copy(), stream_idx, header.fin);

}

optional<WrappingInt32> TCPReceiver::ackno() const { 
    if(!_is_syn_got){return nullopt;}
    // 添加 SYN 标志
    uint64_t abs_ack_no = _reassembler.stream_out().bytes_written() + 1;
    // 添加 FIN 标志
    if(_reassembler.stream_out().input_ended()){
        abs_ack_no++;
    }
    return WrappingInt32(_isn) + abs_ack_no;
 }

size_t TCPReceiver::window_size() const {
    return _capacity - _reassembler.stream_out().buffer_size();
}
```

- 接收端收到消息`seg`
  - 判断`syn`是否收到过，不是`syn`直接丢掉
  - 接受`seg`
- 告知发送端`ackno`
- 窗口大小

## 测试

```
make
make check_lab2
```

![image-20221115205447006](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2022/11/15/image-20221115205447006-4f5789.png)
