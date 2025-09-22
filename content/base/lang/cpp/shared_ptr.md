## 数据结构

基于引用计数

```cpp
private:
	element_type* _Ptr[nullptr];
	_Ref_count_base* _Rep[nullptr];
```
在对某一指针操作时线程安全，但是一起操作时不安全了，存在Race Condition


## 线程安全

`shared_ptr`线程不安全，读安全，写不安全。