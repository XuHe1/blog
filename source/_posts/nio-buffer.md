---
title: NIO之buffer
date: 2018-10-19 15:01:37
tags: 
	- IO
	- JVM
---
### Buffer分类

ByteBuffer官方文档：
 >  
 <p>A byte buffer is either direct or non-direct. Given a direct byte buffer, the Java virtual machine will make a best effort to perform native I/O operations directly upon it.  That is, it will attempt to avoid copying the buffer's content to (or from) an intermediate buffer before (or after) each invocation of one of the underlying operating system's native I/O operations.</p>

根据文档，可将buffer分为两种：    

* Direct: java heap内存
* Non-Direct:堆外内存
<!-- more -->
#### Direct Buffer
* 堆外内存（native 堆）

    任何涉及到文件的读取和写入都会涉及到堆内内存和对外内存的数据拷贝。
    1. 文件读取
    
        内核读取数据到堆外内存，堆外内存将数据拷贝到堆内内存
        
    2. 文件写入
        
        程序将读取堆内内存数据，写入堆外内存，内核再将数据写入文件
       
* 堆外内存回收
    - new DirectByteBuffer对象时在java heap上分配的内存 
    
	    ```
	        public static ByteBuffer allocateDirect(int capacity) {
	            return new DirectByteBuffer(capacity);
	        }
	        
	    ```
	    <p></p>
    - 查看DirectByteBuffer源码
    
	    ```
	        DirectByteBuffer(int cap) {                   // package-private
	    
	            super(-1, 0, cap, cap);
	            boolean pa = VM.isDirectMemoryPageAligned();
	            int ps = Bits.pageSize();
	            long size = Math.max(1L, (long)cap + (pa ? ps : 0));
	            Bits.reserveMemory(size, cap);
	    
	            long base = 0;
	            try {
	                base = unsafe.allocateMemory(size);
	            } catch (OutOfMemoryError x) {
	                Bits.unreserveMemory(size, cap);
	                throw x;
	            }
	            unsafe.setMemory(base, size, (byte) 0);
	            if (pa && (base % ps != 0)) {
	                // Round up to page boundary
	                address = base + ps - (base & (ps - 1));
	            } else {
	                address = base;
	            }
	            cleaner = Cleaner.create(this, new Deallocator(base, size, cap));
	            att = null;
	
	    }
	    
	    ```
	    <p></p>
    - 重点在倒数第二行代码 `Cleaner`
    
	    ```
	    public class Cleaner extends PhantomReference<Object> {
	    
	    ```
	    <p></p>
	    
     结论：
	  
	  > <p>可以看出，Cleaner 继承了PhantomReference（虚引用），Cleaner通过this关键字 虚引用了DirectByteBuffer对象，DirectByteBuffer对象被回收时会通知Cleaner,Cleaner收到通知调用clean()方法回收内存（通过JNI Unsafe)</p>

#### 另外
* native 本地化是什么概念？

    JNI （Java本地接口）是一种编程框架，使得Java虚拟机中的Java程序可以调用本地应用/或库，也可以被其他程序调用。 ==本地程序==一般是用其它语言（C、C++或汇编语言等）编写的，并且被编译为==基于本机硬件和操作系统的程序==。[1]
    * 本地方法又称为原生方法，可以直接与操作系统或硬件通信，无需系统调用
    * java里涉及到IO、Thread、内存分配、回收等与操作系统相关的功能的方法都是由==native==修饰
    * ==Native代码（如汇编语言）分配的内存和资源，需要其自身负责进行显式的释放。==
    
* native堆，属于jvm进程的私有空间,内核可以直接访问

* jni调用是系统调用，是从用户态切换到内核态

#### Reference
* https://www.jianshu.com/p/007052ee3773
* http://www.importnew.com/14486.html

