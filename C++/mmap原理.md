![](C:\Users\ljc\Documents\GitHub\--\C++\图片\mmap原理.webp)

mmap会返回一个指针ptr，它指向进程逻辑地址空间中的一个地址，这样以后，进程无需再调用read和write对文件进行读写，而只需要通过ptr就能够操作文件。但是ptr所指向的是一个逻辑地址，要操作其中的数据，必须通过MMU将逻辑地址转换成物理地址。这个过程与内存映射无关。

建立内存映射并没有实际拷贝数据，这时，MMU在地址映射表中无法找到与ptr相对应的物理地址，也就是MMU失败，将产生一个缺页中断，缺页中断的中断响应函数会在swap中寻找相对应的页面，如果找不到（从未读入内存），则会通过mmap()建立的映射关系，从硬盘上将文件读取到物理内存中，这个也与内存映射无关。

如果拷贝数据的时候，发现内存不够用，则会通过swap机制，将暂时不用的物理页面交换到硬盘上，这个过程也与内存映射无关。

## 效率

从代码层面上看，从硬盘上将文件读入内存，都要经过文件系统进行数据拷贝，并且数据拷贝操作是由文件系统和硬件驱动实现的，理论上来说，拷贝数据的效率是一样的。但是通过内存映射的方法访问硬盘上的文件，效率要比read和write要搞，为什么呢？

原因是read是系统调用，其中进行了数据拷贝，他首先将文件内容从硬盘拷贝到内核空间的一个缓冲区，然后再将这些数据拷贝到用户空间，这个过程中实际产生了两次数据拷贝；而mmap也是系统调用，但是他没有进行数据拷贝，而是在发生缺页中断的时候发生了一次数据拷贝，由于mmap将文件直接映射到用户空间，所以中断处理函数根据这个映射关系，直接将文件从硬盘拷贝到用户空间，只进行了一次数据拷贝。因此效率要高。

## 使用

在龙芯SPMV优化中，出现了mmap比read更慢的情况

场景，读取矩阵文件，每行三个浮点数

关闭开启文件预加载如下：

![](C:\Users\ljc\Documents\GitHub\--\C++\图片\loongArch下mmap和read对比.PNG)

关闭文件预加载如下

![](C:\Users\ljc\Documents\GitHub\--\C++\图片\关闭文件预加载.PNG)
