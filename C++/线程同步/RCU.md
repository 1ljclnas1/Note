一种空间防同步的巧妙方法。首先它只用于指针访问的动态数据。其次它采用读者写者分离的方式，读者先复制数据指针，用这个复制的指针访问数据，这个数据是只读的，很多读者可以同时访问。写者不直接更改数据，而是先申请一块内存空间，把数据都复制过来，在这个复制的数据上修改数据，由于数据是私有的所以可以随意更改。然后将指针指向这个修改后的地址。


