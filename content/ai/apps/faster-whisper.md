 
faster-whisper是一个比[whisper](whisper.md)转录更快的技术实现，转录速度是whisper的4倍，并且占用的显存更少，占用显存是whisper的1/2。而我们这次要讲的是faster-whisper-webui是内置了VAD的支持，可以很精准的定位到每一句话的开始和结束，对于转录长音视频很有意义，可以防止转录长音视频出现幻听的情况。