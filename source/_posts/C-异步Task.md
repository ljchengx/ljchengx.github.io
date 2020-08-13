---
title: 'C#异步Task'
date: 2020-01-21 14:22:46
tags:
- 异步
categories:
- C#
---



## 目前项目中使用最多的两种异步Task

以下方式是在项目中 主要处理服务事件的多线程异步方案
主要完成的单个服务 变成异步多线程 提升效率的写法 可以给需要的朋友提供一下思路


第一种是一个轮询服务，主要涉及一些定时的任务、轮询处理的问题，可以自己定义阀门和时间点

### 关于轮询异步Task

```C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading;
using System.Threading.Tasks;

namespace ConsoleApplication5
{
    class Program
    {
        public static bool _go = true;
        public static int currentTeacherCount = 0;
        public static int maxTask = 15;
        public static object obj = new object();
        public static bool _go_LogStastic = true; //阀门

        //轮询的多线程

        static void Main(string[] args)
        {
            while (_go)
            {
                if (_go_LogStastic)
                {
                    _go_LogStastic = false;
                    var task = new Task(delegate()
                    {
                        Log("1");

                    });
                    task.Start();
                    task.ContinueWith(q =>
                    {
                        _go_LogStastic = true;
                    });
                }
            }      
            Console.WriteLine("所有线程结束！");
            Console.ReadLine();
        }
        public static void Log(String str)
        {
            Console.WriteLine("开始处理第" + str + "个");
        }
    }
}

```

第二种是循环的情景，主要有单一处理方案但是总数据过多，需要嵌套循环处理的方式。

可以根据当前环境设定最大线程数来控制，并且支持原子计数，方便日志打印进行跟踪。


### 关于循环异步Task

```C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading;
using System.Threading.Tasks;

namespace ConsoleApplication5
{
    class Program
    {
        public static bool _go = true;
        public static int currentTeacherCount = 0;
        public static int maxTask = 15;
        public static object obj = new object();
        static void Main(string[] args)
        {
            List<Task> tasks = new List<Task>();
            List<String> list = new List<string>();
            list.Add("1");
            list.Add("2");
            list.Add("3");
            list.Add("4");
            foreach (var item in list)
            {
                tasks.Add(Task.Factory.StartNew(() =>
                {
                    Log(item);
                }).ContinueWith(t =>
                {
                    lock (obj)
                    {
                        Interlocked.Increment(ref currentTeacherCount);
                        Console.WriteLine("第" + currentTeacherCount + "数据同步完成,共" + list.Count() + "数据需要处理");
                    }
                }));
                if (tasks.Count >= maxTask)
                {
                    Task.WaitAny(tasks.ToArray());
                    tasks = tasks.Where(t => t.Status == TaskStatus.Running).ToList();
                }
            }
            Task.WaitAll(tasks.ToArray());
            Console.WriteLine("所有线程结束！");
            Console.ReadLine();
        }
        public static void Log(String str)
        {
            Console.WriteLine("开始处理第" + str + "个");
        }
    }
}

```