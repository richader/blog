---
title: 对比linux和c++的原子操作
date: 2019-12-10 13:09:44
categories: 
    - 编程
tags: 
    - linux 
    - c++
---


先说结论，在相同的机器上多次执行，感觉linux的原子操作比c++的原子操作快10%左右

<!-- more -->
### c++原子操作
使用std::atomic,然后变量就可以正常使用。代码：
``` c++  
#include <iostream>
#include <atomic>
#include <thread>
#include <sys/time.h>

using namespace std;
atomic<int> value(0);

void thread_function(){
    for (size_t i = 0; i < 100000000; i++)
    {
        value++;
    }
}

int main(){
    struct timeval tv1,tv2;
    gettimeofday(&tv1,NULL);
    thread thread1(thread_function);
    thread thread2(thread_function);
    thread thread3(thread_function);
    thread thread4(thread_function);
    thread1.join();
    thread2.join();
    thread3.join();
    thread4.join();
    gettimeofday(&tv2,NULL);
    cout<<"result:"<<value<<"  time:"<<(tv2.tv_sec-tv1.tv_sec)*1000000+(tv2.tv_usec-tv1.tv_usec)<<endl;
}
```   

编译：g++ -std=c++11 -pthread atomic.cpp -o au.out  
执行：./au.out

### Linux原子操作
使用__sync_add_and_fetch系列操作函数。代码：  
``` c++  
#include <iostream>
#include <thread>
#include <sys/time.h>

using namespace std;
volatile uint32_t value = 0;

void thread_function(){
    for (size_t i = 0; i < 100000000; i++)
    {
        __sync_add_and_fetch(&value,1);
    }
}

int main(){
    struct timeval tv1,tv2;
    gettimeofday(&tv1,NULL);
    thread thread1(thread_function);
    thread thread2(thread_function);
    thread thread3(thread_function);
    thread thread4(thread_function);
    thread1.join();
    thread2.join();
    thread3.join();
    thread4.join();
    gettimeofday(&tv2,NULL);
    cout<<"result:"<<value<<"  time:"<<(tv2.tv_sec-tv1.tv_sec)*1000000+(tv2.tv_usec-tv1.tv_usec)<<endl;

}
```  
编译：g++ -std=c++11 -pthread volatile.cpp -o vo.out  
执行：./vo.out
