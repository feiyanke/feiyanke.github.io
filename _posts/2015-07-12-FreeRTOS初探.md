---
date: 2015-07-12
layout: 'post'
tags:
    - 'c'
status: 'public'
---

# FreeRTOS初探

用了半天时间对`FreeRTOS`有了一个初步的认识，大概总结一下，其中混杂了系统实现和实际应用方面的问题。

现只是以应用为目的，实现方面待以后进一步研究。

1. `FreeRTOS`提供的功能包括：任务管理、时间管理、信号量、消息队列、内存管理。与平台有关的文件包含在`portable`文件夹中，主要是`port.c`, `portmacro.h`两个文件。平台无关的文件主要是：`list.c`(基本链表结构), `queue.c`(包括消息队列，信号量的实现), `croutine.c`,`tasks.c`(任务管理，时间管理)。

2. 命名协定：
  `FreeRTOS`内核与范例程序源代码使用下面的协定： 
  变量 
  `char`类型的变量以 `c` 为前缀 
  `short`类型的变量以 `s` 为前缀 
  `long`类型的变量以 `l` 为前缀 
  `float`类型的变量以 `f` 为前缀 
  `double`类型的变量以 `d` 为前缀 
  枚举变量以 `e` 为前缀 
  其他类型（如结构体）以 `x` 为前缀 
  指针有一个额外的前缀 `p` , 例如`short`类型的指针前缀为 `ps` 
  无符号类型的变量有一个额外的前缀 `u` , 例如`unsigned short`类型的变量前缀为 `us` 
  函数 
  文件内部函数以`prv`为前缀 
  API函数以其返回值类型为前缀，按照前面对变量的定义 
  函数的名字以其所在的文件名开头。如`vTaskDelete`函数在`Task.c`文件中定义 
  数据类型
  数据类型并不直接在内核内部引用。相反，每个平台都有其自身的定义方式。例如，`char`类型定义为`portCHAR`，`short`类型定义为`portSHORT`等。范例程序源代码使用的就是这种符号，但这并不是必须的，你可以在你的程序中使用任何你喜欢的符号。 
  此外，有两种额外的类型要为每种平台定义。分别是： 
  `portTickType`
  可配置为16位的无符号类型或32位的无符号类型。参考API文档中的 定制部分获取详细信息。
  `portBASE_TYPE`
  为特定体系定义的最有效率的数据类型。 
  如果`portBASE_TYPE`定义为`char`则必须要特别小心的保证用来作为函数返回值的`signed char`可以为负数，用于指示错误。



3. `FreeRTOS`内核支持优先级调度算法，每个任务可根据重要程度的不同被赋予一定的优先级，CPU总是让处于就绪态的、 优先级最高的任务先运行。`FreeRT0S`内核同时支持轮换调度算法，系统允许不同的任务使用相同的优先级，在没有更高优先级任务就绪的情况下，同一优先级的任务共享CPU的使用时间。


4. `freertos`既可以配置为可抢占内核也可以配置为不可抢占内核。当`FreeRTOS`被设置为可剥夺型内核时，处于就绪态的高优先级任务能剥夺低优先级任务的CPU使用权，这样可保证系统满足实时性的要求；当`FreeRTOS`被设置为不可剥夺型内核时，处于就绪态的高优先级任务只有等当前运行任务主动释放CPU的使用权后才能获得运行，这 样可提高CPU的运行效率。


5. 任务管理:
  系统为每个任务分配一个TCB结构
```
typedef struct tskTaskControlBlock

{

 volatile portSTACK_TYPE     *pxTopOfStack;//指向堆栈顶

 xListItem    xGenericListItem;   //通过它将任务连入就绪链表或者延时链表或者挂起链表中， xListItem包含其TCB指针

 xListItem    xEventListItem;//通过它把任务连入事件等待链表

 unsigned portBASE_TYPE    uxPriority;//优先级

 portSTACK_TYPE      *pxStack;              //指向堆栈起始位置

 signed portCHAR     pcTaskName[ configMAX_TASK_NAME_LEN ];

//省略一些次要结构

} tskTCB;
```


系统的全局变量：
```
static xList pxReadyTasksLists[ configMAX_PRIORITIES ]; 就绪队列

static xList xDelayedTaskList1;

static xList xDelayedTaskList2; 两个延时任务队列

static xList * volatile pxDelayedTaskList;

static xList * volatile pxOverflowDelayedTaskList; 两个延时队列的指针，应该是可互换的。

static xList xPendingReadyList; 

static volatile xList xTasksWaitingTermination;   等待结束队列

static volatile unsigned portBASE_TYPE uxTasksDeleted = ( unsigned portBASE_TYPE ) 0;  结束队列中的个数？？？？？

static xList xSuspendedTaskList;   挂起队列

static volatile unsigned portBASE_TYPE uxCurrentNumberOfTasks；记录了当前系统任务的数目

static volatile portTickType xTickCount;是自启动以来系统运行的ticks数

static unsigned portBASE_TYPE uxTopUsedPriority；记录当前系统中被使用的最高优先级，

static volatile unsigned portBASE_TYPE uxTopReadyPriority；记录当前系统中处于就绪状态的最高优先级。

static volatile signed portBASE_TYPE xSchedulerRunning  ;表示当前调度器是否在运行，也即内核是否启动了

任务建立和删除，挂起和唤醒
```




6. 时间管理:
  操作系统总是需要个时钟节拍的，这个需要硬件支持。`freertos`同样需要一个time tick产生器，通常是用处理器的硬件定时器来实现这个功能。（时间片轮转调度中和延时时间控制？？）
  它周期性的产生定时中断，所谓的时钟节拍管理的核心就是这个定时中断的服务程序。`freertos`的时钟节拍isr中除去保存现场，灰度现场这些事情外，核心的工作就是调用`vTaskIncrementTick()`函数。`vTaskIncrementTick()`函数主要做两件事情：维护系统时间（以tick为单位，多少个节拍）；处理那些延时的任务，如果延时到期，则唤醒任务。
  任务可用的延时函数：`vTaskDelay();vTaskDelayUntil();`
  特别之处在于`vTaskDelayUntil()`是一个周期性任务可以利用它可以保证一个固定的（确定的）常数执行频率，而`vTaskDelay()`无法保证。


7. 任务间的通信（详见“FreeRTOS任务间通讯”）

    1) 当然可以用全局变量的形式通信，但是不安全。

    2) 队列(`xQueueHandle`)是`FreeRTOS`中通信所需的主要数据结构。

    3) 信号量(`xSemaphoreHandle`),有二进制信号量，计数信号量和互斥信号量，其都是以队列为基础结构建立。二进制信号量可以用于中断和任务间的同步。也就是说希望任务随外部中断而执行。即外设给出“数据已就绪”信号，系统中断，任务收到此中断信号接收数据。互斥一般用于都共享资源或数据结构的保护。因为任务调度不能保证数据不被破坏。当一个任务需要访问资源，它必须先获得 ('take') 令牌；当访问结束后，它必须释放令牌 - 允许其他任务能够访问这个资源。(对此还有待进一步实验研究)。

8. 系统配置

freeRTOS 配置在：`FREERTOS_CONFIG.H` 里面，条目如下： 
```
/* 是否配置成抢先先多任务内核，是1的时候，优先级高的任务优先执行。 为0任务就没有优先级之说，用时间片轮流执行 */

#define configUSE_PREEMPTION                1 

 

/* IDLE任务的HOOK函数，用于OS功能扩展，需要你自己编相应函数， 名字是void vApplicationIdleHook( void ) */

#define configUSE_IDLE_HOOK                   0    

 

/* SYSTEM TICK的HOOK函数，用于OS功能扩展，需要你自己编相应函数， 名字是 void vApplicationTickHook(void ) */

 #define configUSE_TICK_HOOK                 0 

 

/* 系统CPU频率，单位是Hz */

#define configCPU_CLOCK_HZ                  58982400 

 

/* 系统SYSTEM TICK每秒钟的发生次数， 数值越大系统反应越快，但是CPU用在任务切换的开销就越多 */

#define configTICK_RATE_HZ                      250 

/* 系统任务优先级数。5 说明任务有5级优先度。这个数目越大耗费RAM越多 */

#define configMAX_PRIORITIES                      5  

 

/* 系统最小堆栈尺寸，注意128不是128字节，而是128个入栈。比如ARM32位，128个入栈就是512字节 */ 

#define configMINIMAL_STACK_SIZE            128 

 

/* 系统可用内存。一般设成除了操作系统和你的程序所用RAM外的最大RAM。 比如20KRAM你用了2K，系统用了3K，剩下15就是最大HEAP 尺寸。你可以先设小然后看编译结果往大里加*/

#define configTOTAL_HEAP_SIZE                       10240

 

/* 任务的PC名字最大长度，因为函数名编译完了就不见了，所以追踪时不知道哪个名字。16表示16个char */

#define configMAX_TASK_NAME_LEN             16

 

/* 是否设定成追踪，由PC端TraceCon.exe记录，也可以转到系统显示屏上 */

#define configUSE_TRACE_FACILITY                  0

 

/* 就是SYSTEM TICK的长度，16是16位，如果是16位以下CPU， 一般选1；如果是32位系统，一般选0 */

#define configUSE_16_BIT_TICKS                          0

 

/* 简单理解以下就是和IDLE TASK同样优先级的任务执行情况。建议设成1，对系统影响不大 */

#define configIDLE_SHOULD_YIELD                     1

 

/* 是否用MUTEXES。 MUTEXES是任务间通讯的一种方式，特别是用于任务共享资源的应用，比如打印机，任务A用的时候就排斥别的任务应用，用完了别的任务才可以应用 */

#define configUSE_MUTEXES                                     0  

 

/*  确定是否用递归式的MUTEXES */

#define configUSE_RECURSIVE_MUTEXES              0

 

/* 是否用计数式的SEMAPHORES，SEMAPHORES也是任务间通讯的一种方式 */
#define configUSE_COUNTING_SEMAPHORES       0

 

/* 是否应用可切换式的API。freeRTOS 同一功能API有多个，有全功能但是需求资源和时间较多的，此项使能后就可以用较简单的API， 节省资源和时间，但是应用限制较多 */

#define configUSE_ALTERNATIVE_API                       0 

 

 

 /* 此项用于DEBUG，来看是否有栈溢出，需要你自己编相应检查函数void vApplicationStackOverflowHook(xTaskHandle *pxTask, signed portCHAR *pcTaskName )  */
#define configCHECK_FOR_STACK_OVERFLOW      0

 

/* 用于DEBUG，登记SEMAPHORESQ和QUEUE的最大个数，需要在任务用应用函数vQueueAddToRegistry()和vQueueUnregisterQueue()  */

#define configQUEUE_REGISTRY_SIZE                        10 

 

 /* 设定可以改变任务优先度 */

 #define INCLUDE_vTaskPrioritySet                               1

 

 /* 设定可以查询任务优先度 */
#define INCLUDE_uxTaskPriorityGet                              1

 

 /* 设定可以删除任务 */

 #define INCLUDE_vTaskDelete                                       1    

 

/* 据说是可以回收删除任务后的资源（RAM等）*/

 #define INCLUDE_vTaskCleanUpResources                 0 

 

/* 设置可以把任务挂起 */

#define INCLUDE_vTaskSuspend                                     1 

/* 设置可以从中断恢复（比如系统睡眠，由中断唤醒 */

#define INCLUDE_vResumeFromISR                                1 

 

 /* 设置任务延迟的绝对时间，比如现在4：30，延迟到5：00。时间都是绝对时间 */

#define INCLUDE_vTaskDelayUntil                                  1 

 

 /* 设置任务延时，比如延迟30分钟，相对的时间，现在什么时间，不需要知道 */
#define INCLUDE_vTaskDelay                                           1  

 

/* 设置 取得当前任务分配器的状态 */

#define INCLUDE_xTaskGetSchedulerState                      1

 

/* 设置当前任务是由哪个任务开启的 */

#define INCLUDE_xTaskGetCurrentTaskHandle              1 

 

/* 是否使能这一函数，还数的目的是返回任务执行后任务堆栈的最小未用数量，同样是为防止堆栈溢出 */

#define INCLUDE_uxTaskGetStackHighWaterMark       0 

 

/* 是用用协程。协程公用堆栈，节省RAM，但是没有任务优先级高，也无法和任务通讯 */

#define configUSE_CO_ROUTINES                                   0   

 

 /* 所有协程的最大优先级数，协程优先级永远低于任务。就是系统先执行任务，所有任务执行完了才执行协程。*/

#define configMAX_CO_ROUTINE_PRIORITIES          1 

 

/*  系统内核的中断优先级，中断优先级越低，越不会影响其他中断。一般设成最低 */

#define configKERNEL_INTERRUPT_PRIORITY            [dependent of processor] 

 

/* 系统SVC中断优先级，这两项都在在M3和PIC32上应用 */

#define configMAX_SYSCALL_INTERRUPT_PRIORITY  [dependent on processor and application] 

 

#endif /* FREERTOS_CONFIG_H */

```

一般来说，如果用不上的功能都要设成0，可以减少代码和资源。