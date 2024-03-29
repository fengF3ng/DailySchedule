# DailySchedule

## 2022-7-1~2022-7-7

第一周的学习任务集中在Rust部分，通过[《Rust程序设计语言》](https://rustwiki.org/zh-CN/book/title-page.html)和[《Rust语言圣经》](https://course.rs/about-book.html)奠定了语法基础。

通过[Rustling](https://github.com/rust-lang/rustlings)和[Rust By Practice](https://zh.practice.rs/why-exercise.html)两本习题册巩固了所学知识，可以发现在学习过程中有较多薄弱项，主要集中在高级错误处理、生命周期和迭代闭包。

简要浏览了实验0的内容

### 计划

在第一周的基础上加强Rust语法的练习，适当对错题进行巩固，并使用leetcode练习基础。

进行实验0和实验1，及时复习基础理论知识，打好基础后再进行下一步。

## 2022-7-8~2022-7-14

完成了实验0~1，其中问题主要出现在实验1上面。

花了一定时间理解实验1的运作机制，并做了简单的总结。首先在启动阶段设置时钟中断，并设置trap.S为异常处理向量入口。之后每次接收到时钟中断后，在trap.S代码的作用下会保存上下文内容并进入trap_handler处理进程切换。trap_handler中会调用switch.S维护进程上下文的切换并改变运行指针等，至此进程切换结束

### 计划

下周完成实验2和实验3，争取对RISC相关的寄存器特性有更进一步的认识。

## 2022-7-15~2022-7-21

实验2的过程中遇到了三个主要问题：

1. 错误的理解了find_pte方法，认为其在页表项未建立映射时会返回None，事实上就算是没有建立映射的页表项也可能返回pte，只要它的前两级页表均合法。这就导致了实验过程中会将一些没用的页表项当成已经映射的，为了方便起见，直接修改了find_pte的逻辑，使其保证返回pte时映射一定存在。当然，这种做法不可取，会引发其他未知的错误，好在测试成功了。
2. 计算时间的时候出现了两个问题，一是照搬了实验1的处理办法，在sys_get_time函数中没有正确处理虚拟地址和物理地址之间的关系；二是使用了get_time函数而非get_time_us导致精度不匹配，与测试样例存在误差
3. 最后的问题就是处理一片连续地址的sys_munmap，由于每个进程维护自己的MapArea容器，一片连续的地址可能存在多个MapArea中，所以一开始通过遍历MapArea并计算和目标vpn_range的交集来确定需要释放哪些MapArea的一部分。但计算交集时没有考虑到vpn_range的上限是达不到的，导致多释放了一点内存

实验3过程中遇到了

1. 采用stride算法时，BIG_STRIDE设置不当，导致主进程usertest的优先级溢出后占用了进程大多时间，出现了超时错误。调整BIT_STRIDE的值，不光考虑到溢出情况，还需要考虑公式要求的1.5倍范围。
2. 在mmap的过程中，对memory_set.translate的返回值使用了unwrap方法，并采用is_valid()判断是否为合法的PTE。奇怪的是针对返回None的情况，编译器没有对unwrap方法报错，只是在运行过程中出现了死等待的现象。通过match语句分别处理Some和None情况后程序运转正常，不知道是否为这个原因。

### 计划

下周是最后一周，需要完成实验4和实验5，如果时间充足的话可以从整体上对所有代码进行一次梳理

## 2022-7-22~2022-7-28

实验4遇到的问题

1. 在fstat函数中current_user_token会要求task的使用权，如果先在fstat中申请了current_task的inner_exclusive_access则导致同一时间内task被两次借用。
2. 搜索目录下所有文件的时候错误的调用了当前fd的inode的read_disk_inode导致没有正确搜索所有文件，应该对ROOT_INODE使用read_disk_inode得到所有文件id。