# lab1

## 实现一个trace
在 ch3 中，我们的系统已经能够支持多个任务分时轮流运行，我们希望引入一个新的系统调用 ``sys_trace``（ID 为 410）用来追踪当前任务系统调用的历史信息，并做对应的修改。定义如下。
```rust
fn sys_trace(_trace_request: usize, _id: usize, _data: usize) -> isize
```
调用规范：
这个系统调用有三种功能，根据 ``trace_request`` 的值不同，执行不同的操作：
如果 ``trace_request`` 为 ``0``，则 id 应被视作 ``*const u8`` ，表示读取当前任务 ``id`` 地址处一个字节的无符号整数值。此时应忽略 ``data`` 参数。返回值为 ``id`` 地址处的值。
如果 ``trace_request`` 为 ``1``，则 ``id`` 应被视作 ``*mut u8`` ，表示写入 ``data`` （作为 ``u8``，即只考虑最低位的一个字节）到该用户程序 ``id`` 地址处。返回值应为 ``0``。
如果 ``trace_request`` 为 ``2``，表示查询当前任务调用编号为 ``id`` 的系统调用的次数，返回值为这个调用次数。本次调用也计入统计 。
否则，忽略其他参数，返回值为 ``-1``。

## 实现过程
sys_trace函数的0状态和1状态都比较容易实现，不赘述。这里只介绍2状态如何做。
如果你在每一个TCB结构体里放一个 [usize;1000]的数组，就会发生一些奇怪的事情。。。数据似乎爆栈影响到了其他段的数据，导致``sys_get_time``函数一直返回0.
所以我的做法是，在TCB结构体里放一个 syscall_count: [usize;10]的数组，用于记录每个系统调用的调用次数；TaskControlBlockInner结构体中放一个 syscall_count_idx: [usize;10]的数组，用于记录 从系统调用id到 syscall_count数组索引的映射关系。
在所有系统调用（包括sys_trace）发生时，TASKMANAGER找到当前进程，并根据系统调用id从syscall_count_idx数组中找到 syscall_count数组的索引，然后对syscall_count中相应的元素进行自增；在sys_trace调用发生时，仍然需要先找到索引，然后返回syscall_count数组中对应元素即可。

当然也可以使用类似哈希一样的存储（比如BTreeMap）来完成节省内存的存储。
