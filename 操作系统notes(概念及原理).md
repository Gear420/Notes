# 操作系统notes(概念及原理)

## 计算机的启动，中断，异常和系统调用：

### 启动：

bios启动。

bootloader：将操作系统装载到内存（512字节——>第一个扇区）  

### 中断：外设（计时器or网络的中断）：

中断的保护与恢复机制；

1.硬件：设置中断标记（cpu初始化）

2.软件：操作系统—》保存当前处理状态

中断服务程序处理

清除中断标记

恢复之前保存的处理状态

### 异常：不良的应用程序。

保存现场

异常处理（kill program —》 retry）

恢复现场

### 系统调用：来源于应用程序—>服务请求

printf() ->write() (系统调用)

![屏幕快照 2018-09-02 下午10.19.49](/Users/zhangxingyu/Desktop/屏幕快照 2018-09-02 下午10.19.49.png)

用户态（权限低）和内核态（）——》系统调用进行状态转换:![屏幕快照 2018-09-02 下午10.24.41](/Users/zhangxingyu/Desktop/屏幕快照 2018-09-02 下午10.24.41.png)

## 进程的概念：

一个程序应对多个进程

并行与并发的概念



进程的创建与等待：

自身阻塞自己 使得自己成为等待状态（running ready block）

 进程状态的变化模型：（时间片用完又运行变为ready）



线程与进程    

线程共享地址空间。可以并发执行

 进程是资源分配单位，线程是cpu调度单位

进程的创建比线程慢（线程只独享寄存器和栈）



###  管程机制：

  



## 操作系统的内存管理：

### 内存与虚拟内存

页表：保证进程的独立性



###  非连续内存 与 连续内存：  

初次分配：第一个满足大小的放置。

最优适配：使用最小的块。

最坏适配：空闲最大的块


内碎片：
	分配单元中的碎片
外碎片
	分配单元间的碎片

压缩与交换式碎片处理

​	

#### 非连续内存地址：

一个程序的物理地址空间是非连续的。

分段机制与分页机制：

​	段号和偏移

​	程序的分段地址空间。

​	分段寻址机制。

​	（有无段寄存器）

分段：肢解。



分页：

​	分页与分段的区别：——> 页的大小是固定不变的 为2的幂

​	加速地址转换 页帧->物理层面的地址寻址。

​	base+页号-》帧号-》物理地址



页表-》数组



虚拟内存管理技术：



页面置换算法：



页面置换轨迹：

​	



## 进程调度算法：

​	

## 信号量和管程：

![屏幕快照 2018-09-02 下午11.38.40](/Users/zhangxingyu/Desktop/屏幕快照 2018-09-02 下午11.38.40.png)  

锁机制

同步操作-》信号量为0-》》实现同步操作![屏幕快照 2018-09-02 下午11.43.57](/Users/zhangxingyu/Desktop/屏幕快照 2018-09-02 下午11.43.57.png)



```c++
//生产者消费者代码

#include <stdio.h>
#include <pthread.h>
#include <semaphore.h>
#include <string.h>
#include <time.h>
 
#define TRUE  1
#define FALSE 0
#define SIZE 11
 
typedef int QueueData;
typedef struct _queue   //队列结构体
{
	int data[SIZE];
	int front;     // 指向队头的下标
	int rear;      // 指向队尾的下标
}Queue;
 
struct data             //信号量结构体
{
	sem_t empty;
	sem_t full;
	Queue q;
};
 
pthread_mutex_t mutex;
 
int num = 0;
 
struct data sem;
 
int InitQueue (Queue *q)   // 队列初始化
{
	if (q == NULL)
	{
		return FALSE;
	}
	
	q->front = 0;
	q->rear  = 0;
	
	return TRUE;
}
 
int QueueEmpty (Queue *q)      //判断空对情况
{
	if (q == NULL)
	{
		return FALSE;
	}
	
	return q->front == q->rear;
}
 
int QueueFull (Queue *q)     //判断队满的情况
{
	if (q == NULL)
	{
		return FALSE;
	}
	
	
	return q->front == (q->rear+1)%SIZE;
} 
 
int DeQueue (Queue *q, int *x)   //出队函数
{
	if (q == NULL)
	{
		return FALSE;
	}
	
	if (QueueEmpty(q))
	{
		return FALSE;
	}
	q->front = (q->front + 1) % SIZE;
	*x = q->data[q->front];
	
	return TRUE;
}
 
int EnQueue (Queue *q, int x)   //进队函数
{
	if (q == NULL)
	{
		return FALSE;
	}
	
	if (QueueFull(q))
	{
		return FALSE;
	}
	
	q->rear = (q->rear+1) % SIZE;
	q->data[q->rear] = x;
	
	return TRUE;
}
 
void *Producer()
{
	while(1)
	{
		int time = rand() % 10 + 1;          //随机使程序睡眠0点几秒
		usleep(time * 100000);                 
		                                      
		sem_wait(&sem.empty);                 //信号量的P操作
		pthread_mutex_lock(&mutex);           //互斥锁上锁
		                                      
		num++;                                
		EnQueue (&(sem.q), num);              //消息进队
		printf("生产了一条消息：%d\n", num);  
		                                      
		pthread_mutex_unlock(&mutex);         //互斥锁解锁
		sem_post(&sem.full);                  //信号量的V操作
	}
}
 
void *Consumer()
{
	while(1)
	{
		int time = rand() % 10 + 1;   //随机使程序睡眠0点几秒
		usleep(time * 100000);
		
		sem_wait(&sem.full);           //信号量的P操作
		pthread_mutex_lock(&mutex);    //互斥锁上锁
		
		num--;
		DeQueue (&sem.q, &num);       //消息出队
		printf("消费了一条消息\n");
		
		pthread_mutex_unlock(&mutex);  //互斥锁解锁
		sem_post(&sem.empty);         //信号量的V操作
	}
}
 
int main()
{
	srand((unsigned int)time(NULL));
	
	sem_init(&sem.empty, 0, 10);    //信号量初始化（做多容纳10条消息，容纳了10条生产者将不会生产消息）
	sem_init(&sem.full, 0, 0);
	
	pthread_mutex_init(&mutex, NULL);  //互斥锁初始化
	
	InitQueue(&(sem.q));   //队列初始化
	
	pthread_t producid;
	pthread_t consumid;
	
	pthread_create(&producid, NULL, Producer, NULL);   //创建生产者线程
	pthread_create(&consumid, NULL, Consumer, NULL);   //创建消费者线程
	
	pthread_join(consumid, NULL);    //线程等待，如果没有这一步，主程序会直接结束，导致线程也直接退出。
	
	sem_destroy(&sem.empty);         //信号量的销毁
	sem_destroy(&sem.full);    
	
	pthread_mutex_destroy(&mutex);   //互斥锁的销毁
	
	return 0;
}
```

### 管程的思想

![屏幕快照 2018-09-03 上午12.19.59](/Users/zhangxingyu/Desktop/屏幕快照 2018-09-03 上午12.19.59.png)



## 死锁问题：

  构造资源分配图 —> 用来判断死锁

算法角度分析 —>  

死锁的特征：互斥。  持有并等待。无抢占。循环等待。

死锁 —> 鸵鸟政策

### 银行家算法：

死锁避免算法：n  = 进程数量 m = 资源类型数量

Max【i,j】 = k 表示pi需求Rj的k个实例

ava和allo向量 

need矩阵 

## 资源管理问题：

## extra：微内核的设计思想



