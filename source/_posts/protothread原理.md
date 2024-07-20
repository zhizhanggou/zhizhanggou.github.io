---
title: protothread原理
date: 2024-05-28 14:56:40
categories: 嵌入式
tags: 操作系统
---
# 背景
在一些规模较小的项目中，系统的复杂度还没有到需要使用rtos的必要，并且rtos本身会占用大量的代码空间。此时可以使用protothread来对工程进行类操作系统task的包装，或者实现类似调度的功能。这样日后想上操作系统时可以方便的进行适配。
# protothread是什么
[维基百科](https://zh.wikipedia.org/wiki/Protothreads)

Protothreads是一种低开销的并发编程机制。Protothreads充当无栈的轻量级线程或协程，它使用了极小的每protothread内存：一个短整数保存执行位置，一个字节作为让步标志。

Protothreads可用于实现叫做协作式多任务的非抢占形式的并发计算，故而在一个线程yield（让步）给另一个线程的时候不会招致上下文切换。为了在一个protothread内达成yield，在线程函数内利用了达夫设备并在其switch语句内使用一个函数外部的变量。这允许在另一次函数调用时跳转（恢复）到上次的yield的地方。为了阻塞线程，这些yield要通过等待条件来守卫，使得后续的对同样这个函数的调用仍然yield，直到这个条件表达式是为真值为止。

# 原理
## 达夫设备
达夫设备利用了switch case语句的"穿透"特性，在case语句后如果没有break，则会一直执行到switch语句的结尾。以下方代码为例。
```
#include "stdio.h"
#include "stdlib.h"
void duffs_device_demo (int count) {
    int n = (count+7)/8;
    int val = count;
    switch(count % 8) {
        case 0: do { printf("%d ", val--);
        case 7:      printf("%d ", val--);
        case 6:      printf("%d ", val--);
        case 5:      printf("%d ", val--);
        case 4:      printf("%d ", val--);
        case 3:      printf("%d ", val--);
        case 2:      printf("%d ", val--);
        case 1:      printf("%d ", val--);
        } while (--n > 0);
    }
}

int main() {
    duffs_device_demo(10);
}
```
输入count的值为10，n为2，在执行到switch语句时条件值为10%8=2，会跳转到"case 2:"语句处。然后会因为“穿透”的特性一直执行到"while(--n > 0)"这个语句处，此时--n为1，会继续跳转到do语句处再开始执行，然后再次“穿透”到while判断，n=0，退出循环。

最后执行的结果就是打印为：10 9 8 7 6 5 4 3 2 1

## protothread的实现
protothread的代码都是用宏来定义。

一个典型的用法如下：
```
/*define and init pt struct*/
struct pt m_pt_demo;
PT_INIT(pt_demo);

/*thread code*/
int demo_thread()
{
    PT_BEGIN(&m_pt_demo);

    while (1) {
        /*user code*/

        PT_YIELD(&m_pt_demo);

        /*user code*/
    }

    PT_END(&m_pt_demo);
}

```

将宏进行展开
```
/*PT_INIT*/
(m_pt_demo)->lc = 0; 

int demo_thread()
{ 
    /*PT_BEGIN*/
    char PT_YIELD_FLAG = 1; 
    switch((m_pt_demo)->lc) {
        case 0:

        while(1) {
            /*user code*/

            /*PT_YIELD*/
            do {						
                PT_YIELD_FLAG = 0;	
                /*LC_SET*/
                (m_pt_demo)->lc = __LINE__; case __LINE__:						
                if(PT_YIELD_FLAG == 0) {			
                    return PT_YIELDED;			
                }
            } while(0)
        }

    /*PT_END*/
    };
    PT_YIELD_FLAG = 0; 
    (m_pt_demo)->lc = 0;  
    return PT_ENDED; 
}
```
在第一轮进入demo_thread时，(m_pt_demo)->lc = 0,因此执行user code，然后进入yield,把PT_YIELD_FLAG置0，并把当前运行到的行号赋给(m_pt_demo)->lc，用于记录目前执行到的位置。然后退出函数，让出cpu。

当下一轮再次进入时，PT_YIELD_FLAG先置为1，然后到switch时直接跳转到了case \_\_LINE\_\_:处，由于PT_YIELD_FLAG为一，所以又运行user code，然后重复第一轮的操作。这就实现了一种看起来类似于“调度”的效果。

此外还有PT_WAIT_UNTIL,可以在条件不满足时让出cpu，并在条件满足后从上次执行到的位置来运行，
```
#define PT_WAIT_UNTIL(pt, condition)	        \
  do {						\
    LC_SET((pt)->lc);				\
    if(!(condition)) {				\
      return PT_WAITING;			\
    }						\
  } while(0)
```

还支持“子线程”、信号量之类的功能。

# 缺点
不能保存现场，其“线程”并没有自己的栈，其局部变量的值无法保存到下一轮运行，所以只能用全局变量来保存需要使用的值。

不存在真正的“调度”过程，代码本质上还是按照顺序来执行。（也不能说时缺点，只能说特性）

# 总结
protothread仅仅用了4个.文件就实现了操作系统中的一些关键功能。非常适合一些轻量级的项目来使用，可以避免移植操作系统的麻烦以及操作系统的大量资源消耗。

但由于其无栈的特性，在使用时要小心现场的保存问题。