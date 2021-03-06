# C++ 多线程 #

多线程是多任务处理的一种特殊形式，而多任务处理是一种让你的电脑能并发运行两个或两个以上程序的特性。一般有两种类型的多任务处理：基于进程的和基于线程的。

基于进程的多任务处理处理的是并发执行的程序。基于线程的多任务处理处理的是并发执行的程序的一部分。

多线程程序包含了可以并发运行的两个或更多个程序部分。这样程序中的每个部分称为一个线程，并且每个线程都定义了一个单独的执行路径。

C++ 不包含对多线程应用程序的任何嵌入式支持。相反，它完全依赖于操作系统来提供此项功能。

本教程假设您正在使用的是 Linux 操作系统，我们将要使用 POSIX 编写 C++ 多线程程序。 POSIX 线程，或称 Pthreads，它提供了在许多类 Unix 的 POSIX 系统（如 FreeBSD，NetBSD，GNU/Linux，Mac OS X和 Solaris）中可用的 API。

## 创建线程： ##

我们使用下面的函数来创建一个 POSIX 线程：

    #include <pthread.h>
    pthread_create (thread, attr, start_routine, arg)

这里的 **pthread_create** 创建了一个新线程，并使其可执行。这个函数可以在代码中的任意位置调用任意次。

下面是详细的参数说明：

<table>
<tr>
<th align="left">参数</th>
<th align="left">描述</th>
</tr>
   <tr>
      <td>thread</td>
      <td>新线程的不透明、唯一的标识符，它由子函数返回。</td>
   </tr>
   <tr>
      <td>attr</td>
      <td>一个不透明的属性对象,可用于设置线程属性。你可以指定一个线程的属性对象，默认值为 NULL。</td>
   <tr>
      <td>start_routine</td>
      <td>C++ 例程，线程一旦创建将会被执行。</td>
   </tr>
   <tr>
      <td>arg</td>
      <td>一个传递给 start_routine 的参数。它必须传递一个 void 类型指针的引用。如果没有参数传递，默认值为 NULL。</td>
   </tr>
</table>


一个进程可创建的最大线程数是依赖实现决定的。线程一旦创建，它们之间是对等的，而且也有可能创建其它的线程。线程之间没有隐含的层次或依赖关系。

## 终止线程： ##

我们使用下面的函数来终止一个 POSIX 线程：

    #include <pthread.h>
    pthread_exit (status)

此处的 **pthread_exit** 用于显式的退出一个线程。通常在线程已完成了其工作，并且没有存在的必要的时候，调用 pthread_exit（）函数。

如果 main（）在其创建的线程之前终止，并且使用了 pthread_exit（） 来退出线程，那么其线程将会继续执行。否则，当 main（） 终止后，这些线程将会自动终止。

## 例子： ##

下面简单的样例代码，用 pthread_create（） 函数创建了5个线程。每个线程均打印 “Hello World！”，然后调用 pthread_exit（） 函数终止了线程。

    #include <iostream>
    #include <cstdlib>
    #include <pthread.h>
    
    using namespace std;
    
    #define NUM_THREADS 5
    
    void *PrintHello(void *threadid)
    {
       long tid;
       tid = (long)threadid;
       cout << "Hello World! Thread ID, " << tid << endl;
       pthread_exit(NULL);
    }
    
    int main ()
    {
       pthread_t threads[NUM_THREADS];
       int rc;
       int i;
       for( i=0; i < NUM_THREADS; i++ ){
      cout << "main() : creating thread, " << i << endl;
      rc = pthread_create(&threads[i], NULL, 
      PrintHello, (void *)i);
      if (rc){
     cout << "Error:unable to create thread," << rc << endl;
     exit(-1);
      }
       }
       pthread_exit(NULL);
    }


使用 -lpthread 库编译上面的程序，如下所示：

    $gcc test.cpp -lpthread

现在执行上面的程序，将会产生如下的结果：

    main() : creating thread, 0
    main() : creating thread, 1
    main() : creating thread, 2
    main() : creating thread, 3
    main() : creating thread, 4
    Hello World! Thread ID, 0
    Hello World! Thread ID, 1
    Hello World! Thread ID, 2
    Hello World! Thread ID, 3
    Hello World! Thread ID, 4

## 传递参数给线程： ##

下面的例子展示了如何通过一个结构体传递多个参数。你可以在一个线程回调中传递任何数据类型，这是因为它指向 void 类型。
下面的例子解释了这一点：

    #include <iostream>
    #include <cstdlib>
    #include <pthread.h>
    
    using namespace std;
    
    #define NUM_THREADS 5
    
    struct thread_data{
       int  thread_id;
       char *message;
    };
    
    void *PrintHello(void *threadarg)
    {
       struct thread_data *my_data;
    
       my_data = (struct thread_data *) threadarg;
    
       cout << "Thread ID : " << my_data->thread_id ;
       cout << " Message : " << my_data->message << endl;
    
       pthread_exit(NULL);
    }
    
    int main ()
    {
       pthread_t threads[NUM_THREADS];
       struct thread_data td[NUM_THREADS];
       int rc;
       int i;
    
       for( i=0; i < NUM_THREADS; i++ ){
      cout <<"main() : creating thread, " << i << endl;
      td[i].thread_id = i;
      td[i].message = "This is message";
      rc = pthread_create(&threads[i], NULL,
      PrintHello, (void *)&td[i]);
      if (rc){
     cout << "Error:unable to create thread," << rc << endl;
     exit(-1);
      }
       }
       pthread_exit(NULL);
    }


当上述代码编译和执行后，将会有以下的结果：
    
    main() : creating thread, 0
    main() : creating thread, 1
    main() : creating thread, 2
    main() : creating thread, 3
    main() : creating thread, 4
    Thread ID : 3 Message : This is message
    Thread ID : 2 Message : This is message
    Thread ID : 0 Message : This is message
    Thread ID : 1 Message : This is message
    Thread ID : 4 Message : This is message


## 连接和分离线程： ##

下面的两个函数，我们可以用它们来连接或分离线程：

    pthread_join (threadid, status) 
    pthread_detach (threadid) 

pthread_join（）子例程会阻塞调用它的线程，一直等到其指定的 threadid 的线程结束为止。当一个线程创建后，它的属性决定了它是否是可连接的或可分离的。只有创建时属性为可连接的线程才可以连接。如果创建的是一个可分离的线程，那么它永远不能连接。

下面的例子演示了如何使用 pthread_join 函数来等待一个线程结束。

    #include <iostream>
    #include <cstdlib>
    #include <pthread.h>
    #include <unistd.h>
    
    using namespace std;
    
    #define NUM_THREADS 5
    
    void *wait(void *t)
    {
       int i;
       long tid;
    
       tid = (long)t;
    
       sleep(1);
       cout << "Sleeping in thread " << endl;
       cout << "Thread with id : " << tid << "  ...exiting " << endl;
       pthread_exit(NULL);
    }
    
    int main ()
    {
       int rc;
       int i;
       pthread_t threads[NUM_THREADS];
       pthread_attr_t attr;
       void *status;
    
       // Initialize and set thread joinable
       pthread_attr_init(&attr);
       pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_JOINABLE);
    
       for( i=0; i < NUM_THREADS; i++ ){
      cout << "main() : creating thread, " << i << endl;
      rc = pthread_create(&threads[i], NULL, wait, (void *)i );
      if (rc){
     cout << "Error:unable to create thread," << rc << endl;
     exit(-1);
      }
       }
    
       // free attribute and wait for the other threads
       pthread_attr_destroy(&attr);
       for( i=0; i < NUM_THREADS; i++ ){
      rc = pthread_join(threads[i], &status);
      if (rc){
     cout << "Error:unable to join," << rc << endl;
     exit(-1);
      }
      cout << "Main: completed thread id :" << i ;
      cout << "  exiting with status :" << status << endl;
       }
    
       cout << "Main: program exiting." << endl;
       pthread_exit(NULL);
    }


当上述代码编译和执行后，将产生以下的结果：

    main() : creating thread, 0
    main() : creating thread, 1
    main() : creating thread, 2
    main() : creating thread, 3
    main() : creating thread, 4
    Sleeping in thread
    Thread with id : 0 .... exiting
    Sleeping in thread
    Thread with id : 1 .... exiting
    Sleeping in thread
    Thread with id : 2 .... exiting
    Sleeping in thread
    Thread with id : 3 .... exiting
    Sleeping in thread
    Thread with id : 4 .... exiting
    Main: completed thread id :0  exiting with status :0
    Main: completed thread id :1  exiting with status :0
    Main: completed thread id :2  exiting with status :0
    Main: completed thread id :3  exiting with status :0
    Main: completed thread id :4  exiting with status :0
    Main: program exiting.