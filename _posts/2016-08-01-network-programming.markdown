---
layout:     post
title:      "网络编程分享"
subtitle:   "网络编程分享——by 刘佳"
date:       2016-07-31 10:00:00
author:     "liujia"
header-img: "img/post-bg-03.jpg"
---

## PPT内容

<img src="/blog/img/network-programming/网络编程模型简介.001.jpeg" style="display: block; margin: 10px auto; width: 800px; height: auto;" />
<img src="/blog/img/network-programming/网络编程模型简介.002.jpeg" style="display: block; margin: 10px auto; width: 800px; height: auto;" />
<img src="/blog/img/network-programming/网络编程模型简介.003.jpeg" style="display: block; margin: 10px auto; width: 800px; height: auto;" />
<img src="/blog/img/network-programming/网络编程模型简介.004.jpeg" style="display: block; margin: 10px auto; width: 800px; height: auto;" />
<img src="/blog/img/network-programming/网络编程模型简介.005.jpeg" style="display: block; margin: 10px auto; width: 800px; height: auto;" />
<img src="/blog/img/network-programming/网络编程模型简介.006.jpeg" style="display: block; margin: 10px auto; width: 800px; height: auto;" />
<img src="/blog/img/network-programming/网络编程模型简介.007.jpeg" style="display: block; margin: 10px auto; width: 800px; height: auto;" />
<img src="/blog/img/network-programming/网络编程模型简介.008.jpeg" style="display: block; margin: 10px auto; width: 800px; height: auto;" />
<img src="/blog/img/network-programming/网络编程模型简介.009.jpeg" style="display: block; margin: 10px auto; width: 800px; height: auto;" />
<img src="/blog/img/network-programming/网络编程模型简介.010.jpeg" style="display: block; margin: 10px auto; width: 800px; height: auto;" />
<img src="/blog/img/network-programming/网络编程模型简介.011.jpeg" style="display: block; margin: 10px auto; width: 800px; height: auto;" />
<img src="/blog/img/network-programming/网络编程模型简介.012.jpeg" style="display: block; margin: 10px auto; width: 800px; height: auto;" />
<img src="/blog/img/network-programming/网络编程模型简介.013.jpeg" style="display: block; margin: 10px auto; width: 800px; height: auto;" />
<img src="/blog/img/network-programming/网络编程模型简介.014.jpeg" style="display: block; margin: 10px auto; width: 800px; height: auto;" />
<img src="/blog/img/network-programming/网络编程模型简介.015.jpeg" style="display: block; margin: 10px auto; width: 800px; height: auto;" />
<img src="/blog/img/network-programming/网络编程模型简介.016.jpeg" style="display: block; margin: 10px auto; width: 800px; height: auto;" />
<img src="/blog/img/network-programming/网络编程模型简介.017.jpeg" style="display: block; margin: 10px auto; width: 800px; height: auto;" />
<img src="/blog/img/network-programming/网络编程模型简介.018.jpeg" style="display: block; margin: 10px auto; width: 800px; height: auto;" />
<img src="/blog/img/network-programming/网络编程模型简介.019.jpeg" style="display: block; margin: 10px auto; width: 800px; height: auto;" />
<img src="/blog/img/network-programming/网络编程模型简介.020.jpeg" style="display: block; margin: 10px auto; width: 800px; height: auto;" />
<img src="/blog/img/network-programming/网络编程模型简介.021.jpeg" style="display: block; margin: 10px auto; width: 800px; height: auto;" />
<img src="/blog/img/network-programming/网络编程模型简介.022.jpeg" style="display: block; margin: 10px auto; width: 800px; height: auto;" />
<img src="/blog/img/network-programming/网络编程模型简介.023.jpeg" style="display: block; margin: 10px auto; width: 800px; height: auto;" />
<img src="/blog/img/network-programming/网络编程模型简介.024.jpeg" style="display: block; margin: 10px auto; width: 800px; height: auto;" />


## 用C实现的goroutine

### main.cpp

```c++
//
//  main.cpp
//  coroutine
//
//  Created by liujia on 16/7/27.
//  Copyright © 2016年 liujia. All rights reserved.
//

#define _XOPEN_SOURCE
#include <iostream>
#include <memory>
#include <thread>
#include "coroutine.h"
#include "util.h"

util::ThreadLocalSingleton<SCHEDULER_PTR> t_scheduler;

void func_1(int i){
    printf("func_1(): enter, param : %d \n", i);
    t_scheduler.getThreadInstanceRef()->yield_running_coroutine();
    printf("func_1(): leave, param : %d \n", i);
}

void func_2(const char* s){
    printf("func_2(): enter, param : %s \n", s ? s : "null");
    t_scheduler.getThreadInstanceRef()->yield_running_coroutine();
    printf("func_2(): leave, param : %s \n", s ? s : "null");
}

void func_3(void){
    printf("func_3(): enter \n");
    printf("func_3(): leave \n");
}

void thread_func() {
    t_scheduler.getThreadInstanceRef().reset(new scheduler());
    
    int id1 = t_scheduler.getThreadInstanceRef()->create_coroutine(std::bind(&func_1, 9));
    int id2 = t_scheduler.getThreadInstanceRef()->create_coroutine(std::bind(&func_2, "I am a coroutine"));
    int id3 = t_scheduler.getThreadInstanceRef()->create_coroutine(func_3);
    
    printf("thread_func start\n");
    
    /*
    t_scheduler.getThreadInstanceRef()->resume_coroutine(id1);
    t_scheduler.getThreadInstanceRef()->resume_coroutine(id2);
    t_scheduler.getThreadInstanceRef()->resume_coroutine(id1);
    t_scheduler.getThreadInstanceRef()->resume_coroutine(id2);
    */
    
    for(;t_scheduler.getThreadInstanceRef()->has_coroutine();){
        t_scheduler.getThreadInstanceRef()->resume_one_coroutine();
    }
    
    printf("thread_func end\n");
}

int main(int argc, const char * argv[]) {
    printf("Hello, World!\n");
    
    std::thread t(thread_func);
    t.join();
    
    return 0;
}
```

### coroutine.h

```c++
//
//  coroutine.hpp
//  coroutine
//
//  Created by liujia on 16/7/27.
//  Copyright © 2016年 liujia. All rights reserved.
//

#ifndef coroutine_h
#define coroutine_h

#define _XOPEN_SOURCE

#include <ucontext.h>
#include <stdio.h>
#include <stddef.h>
#include <stdlib.h>
#include <string.h>
#include <assert.h>
#include <stdint.h>
#include <map>
#include <deque>
#include <memory>
#include <functional>

#include "util.h"

class coroutine;
typedef std::shared_ptr<coroutine> COROUTINE_PTR;

class scheduler;
typedef std::shared_ptr<scheduler> SCHEDULER_PTR;
typedef std::weak_ptr<scheduler> SCHEDULER_WEAK_PTR;

typedef std::function<void()> COROUTINE_FUNC;

static const size_t STACK_SIZE = 1024 * 1024;

enum STATUS{
    CLOSED = 0,
    READY = 1,
    RUNNING = 2,
    SUSPENDED = 3
};

class coroutine : public util::noncopyable
{
public:
    coroutine(int id, SCHEDULER_WEAK_PTR scheduler, COROUTINE_FUNC func) :
    _id(id),
    _scheduler(scheduler),
    _func(func),
    _status(READY),
    _stack_ptr(new char[STACK_SIZE]),
    _stack_size(STACK_SIZE)
    {}
    
    ~coroutine() {
        if(_stack_ptr){
            delete []_stack_ptr;
            _stack_ptr = nullptr;
        }
    }
    
    int get_id() const {
        return _id;
    }
    
    STATUS get_status() const {
        return _status;
    }
    
    bool is_running() const {
        return _status == RUNNING;
    }
    
    ucontext_t& get_context_ref() {
        return _context;
    }
    
    ucontext_t* get_context() {
        return &_context;
    }
    
    void* get_stack() const{
        return (void*)_stack_ptr;
    }
    
    size_t get_stack_size() const{
        return _stack_size;
    }
    
    void start() {
        _status = RUNNING;
        if(_func){
            _func();
        }
    }
    
    void yield() {
        if(_status != RUNNING){
            return;
        }
        
        _status = SUSPENDED;
    }
    
private:
    int _id;
    ucontext_t _context;
    const char* _stack_ptr;
    size_t _stack_size;
    STATUS _status;
    COROUTINE_FUNC _func;
    SCHEDULER_WEAK_PTR _scheduler;
};

class scheduler : public std::enable_shared_from_this<scheduler>
{
public:
    scheduler() :
    _running_id(-1),
    _atomic_id(new util::atomic_id())
    {}
    ~scheduler(){
        
    }
    
    int create_coroutine(COROUTINE_FUNC func) {
        COROUTINE_PTR routine = std::make_shared<coroutine>(_atomic_id->get_id(), shared_from_this(), func);
        if(!routine){
            return -1;
        }
        
        _routines.insert(std::make_pair(routine->get_id(), routine));
        _add_to_ready(routine);
        return routine->get_id();
    }
    
    void close_all() {
        std::for_each(_routines.begin(), _routines.end(), [](std::map<int, COROUTINE_PTR>::value_type each){
            if(each.second){
                each.second.reset();
            }
        });
        _routines.clear();
        _ready_routines.clear();
    }
    
    void remove_coroutine(int id){
        auto iter = _routines.find(id);
        if(iter == _routines.end()){
            return;
        }
        
        if(_running_id == id){
            _running_id = -1;
        }
        
        iter->second.reset();
        _routines.erase(id);
        _remove_from_ready(id);
    }
    
    void resume_coroutine(int id){
        COROUTINE_PTR routine = _find_by_id(id);
        if(!routine){
            return;
        }
        
        if(routine->get_status() == RUNNING){
            return;
        }
        
        if(routine->get_status() == READY){
            getcontext(routine->get_context());
            routine->get_context_ref().uc_stack.ss_sp = routine->get_stack();
            routine->get_context_ref().uc_stack.ss_size = routine->get_stack_size();
            routine->get_context_ref().uc_stack.ss_flags = 0;
            routine->get_context_ref().uc_link = &_context_main;

            _running_id = routine->get_id();
            _remove_from_ready(routine->get_id());
            makecontext(routine->get_context(), (void (*)(void))scheduler::func_wrapper, 1, (void*)(new SCHEDULER_PTR(shared_from_this())));
            swapcontext(&_context_main, routine->get_context());
        }else if(routine->get_status() == SUSPENDED) {
            _running_id = routine->get_id();
            _remove_from_ready(routine->get_id());
            swapcontext(&_context_main, routine->get_context());
        }else{
            assert(0);
        }
        return;
    }
    
    void resume_one_coroutine(){
        std::deque<COROUTINE_PTR>::iterator iter = std::find_if(_ready_routines.begin(), _ready_routines.end(), [](std::deque<COROUTINE_PTR>::value_type ptr){
            return ptr && (ptr->get_status() == READY || ptr->get_status() == SUSPENDED);
        });
        
        if(iter != _ready_routines.end()) {
            resume_coroutine((*iter)->get_id());
        }
    }
    
    void yield_running_coroutine() {
        yield_coroutine(get_running_id());
    }
    
    void yield_coroutine(int id){
        if(id == -1){
            return;
        }
        
        COROUTINE_PTR routine = _find_by_id(id);
        if(!routine){
            return;
        }
        
        if(routine->get_status() != RUNNING){
            return;
        }
        
        assert(routine->get_id() != -1);
        
        routine->yield();
        _running_id = -1;
        _add_to_ready(routine);
        swapcontext(routine->get_context(), &_context_main);
    }
    
    int get_running_id() const{
        return _running_id;
    }
    
    COROUTINE_PTR get_running_coroutine() const{
        return _find_by_id(get_running_id());
    }
    
    bool has_coroutine() const{
        return !_routines.empty();
    }
    
private:
    COROUTINE_PTR _find_by_id(int id) const{
        auto iter = _routines.find(id);
        if(iter == _routines.end()){
            return COROUTINE_PTR();
        }
        
        return COROUTINE_PTR(iter->second);
    }
    
    void _remove_from_ready(int id) {
        std::remove_if(_ready_routines.begin(), _ready_routines.end(), [id](COROUTINE_PTR ptr) -> bool{
            return ptr && ptr->get_id() == id;
        });
    }
    
    void _add_to_ready(COROUTINE_PTR ptr){
        if(!ptr){
            return;
        }
        if(_is_ready(ptr->get_id())){
            return;
        }
        _ready_routines.push_back(ptr);
    }
    
    bool _is_ready(int id) {
        return std::find_if(_ready_routines.begin(), _ready_routines.end(), [id](COROUTINE_PTR ptr) -> bool{
            return ptr && ptr->get_id() == id;
        }) != _ready_routines.end();
    }
    
    static void func_wrapper(void* param) {
        if(!param){
            return;
        }
        
        SCHEDULER_PTR* ptr = (SCHEDULER_PTR*)param;
        SCHEDULER_PTR sche = *ptr;
        delete ptr;
        if(!sche){
            return;
        }
        
        COROUTINE_PTR routine = sche->get_running_coroutine();
        if(!routine){
            return;
        }
        
        routine->start();
        sche->remove_coroutine(routine->get_id());
    }
    
private:
    ucontext_t _context_main;
    int _running_id;
    typedef std::map<int, COROUTINE_PTR> COROUTINES_MAP;
    COROUTINES_MAP _routines;
    std::deque<COROUTINE_PTR> _ready_routines;
    std::unique_ptr<util::atomic_id> _atomic_id;
};

#endif /* coroutine_h */
```

### util.h

```c++
//
//  util.h
//  coroutine
//
//  Created by liujia on 16/7/27.
//  Copyright © 2016年 liujia. All rights reserved.
//

#ifndef util_h
#define util_h

#include <memory>
#include <atomic>
#include <thread>
#include <mutex>
#include <iostream>
#include <pthread.h>

namespace util {

class noncopyable
{
protected:
    noncopyable() {}
    ~noncopyable() {}
private:
    noncopyable( const noncopyable& );
    const noncopyable& operator=( const noncopyable& );
};
    
class atomic_id
{
public:
    atomic_id() : _id(0)
    {}
    ~atomic_id()
    {}
    
    int get_id(){
        return ++_id;
    }
private:
    std::atomic<int> _id;
};

template <class T>
class ThreadLocalSingleton
{
public:
    static T* getThreadInstance(void)
    {
        pthread_once(&tlsKey_once, ThreadLocalSingleton::tls_make_key);
        T* instance = (T*)pthread_getspecific(tlsKey);
        if(!instance)
        {
            try
            {
                instance = new T;
                pthread_setspecific(tlsKey, instance);
            }
            catch (const char* ex)
            {
                printf("exception happens: %s\n",ex);
            }
        }
        return instance;
    }
    
    static T& getThreadInstanceRef(void)
    {
        return *getThreadInstance();
    }

private:
    static pthread_key_t tlsKey;
    static pthread_once_t tlsKey_once;
        
    static void tls_make_key()
    {
        (void)pthread_key_create(&ThreadLocalSingleton::tlsKey, ThreadLocalSingleton::tls_destructor);
    }
        
    static void tls_destructor(void* obj)
    {
        delete ((T*)obj);
        pthread_setspecific(tlsKey, NULL);
    }
};

template <class T>
pthread_key_t ThreadLocalSingleton<T>::tlsKey;
template <class T>
pthread_once_t ThreadLocalSingleton<T>::tlsKey_once = PTHREAD_ONCE_INIT;
 
}

#endif /* util_h */
```

