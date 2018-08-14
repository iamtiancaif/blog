---
layout: post
title:  "Java Fixed Size Thread Pool Executor Example"
categories: computer
tags:  computer copy java
author: "Lokesh Gupta"
source: "https://howtodoinjava.com/core-java/multi-threading/java-fixed-size-thread-pool-executor-example/"
---

* content
{:toc}


In previous tutorial, we learned about [basic thread pool executor](https://howtodoinjava.com/core-java/multi-threading/java-thread-pool-executor-example/ "Java Thread Pool Executor Example") with unlimited possible number of threads into the pool and it’s example usage. Now lets look example of fixed size thread pool executor which will help in improved performance and better system resource utilization by limiting the maximum number of threads in thread pool.

1) Create a task to execute
---------------------------

Obviously, first step is to have a task which you would like to execute using executors.

	class Task implements Runnable {
		private String name;

		public Task(String name)  {
			this.name = name;
		}

		public String getName() {
			return name;
		}

		@Override
		public void run() {
			try {
				Long duration = (long) (Math.random() * 10);
				System.out.println("Doing a task during : " + name);
				TimeUnit.SECONDS.sleep(duration);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}

2) Execute tasks using Executors
--------------------------------

Now all you have to do is to create an instance of `ThreadPoolExecutor` with fixed size and pass the tasks to be executed into it’s `execute()` method.

	package com.howtodoinjava.demo.multithreading;

	import java.util.concurrent.Executors;
	import java.util.concurrent.ThreadPoolExecutor;
	import java.util.concurrent.TimeUnit;

	public class FixedThreadPoolExecutorExample
	{
		public static void main(String[] args)
		{
			ThreadPoolExecutor executor = (ThreadPoolExecutor) Executors.newFixedThreadPool(4);
			for (int i = 0; i < 10; i++)
			{
				Task task = new Task("Task " + i);
				System.out.println("A new task has been added : " + task.getName());
				executor.execute(task);
			}
			System.out.println("Maximum threads inside pool " + executor.getMaximumPoolSize());
			executor.shutdown();
		}
	}


Output:

	A new task has been added : Task 0
	A new task has been added : Task 1
	A new task has been added : Task 2
	A new task has been added : Task 3
	A new task has been added : Task 4
	A new task has been added : Task 5
	A new task has been added : Task 6
	A new task has been added : Task 7
	Doing a task during : Task 0
	Doing a task during : Task 2
	Doing a task during : Task 1
	A new task has been added : Task 8
	Doing a task during : Task 3
	A new task has been added : Task 9
	 
	Maximum threads inside pool 4
	 
	Doing a task during : Task 4
	Doing a task during : Task 5
	Doing a task during : Task 6
	Doing a task during : Task 7
	Doing a task during : Task 8
	Doing a task during : Task 9

Important Points:
-----------------

1) `newFixedThreadPool()` method creates an executor with a maximum number of threads at any time. If you send more tasks than the number of threads, the remaining tasks will be blocked until there is a free thread to process them This method receives the maximum number of threads as a parameter you want to have in your executor. In your case, you have created an executor with four threads.

2) The `Executors` class also provides the `newSingleThreadExecutor()` method. This is an extreme case of a fixed-size thread executor. It creates an executor with only one thread, so it can only execute one task at a time.

Happy Learning !!







