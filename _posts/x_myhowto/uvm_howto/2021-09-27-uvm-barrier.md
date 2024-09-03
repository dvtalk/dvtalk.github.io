---
layout: default
title: How to use uvm_barrier and uvm_barrier_pool
parent: UVM How to
grand_parent: My How to
description: How to use uvm_barrier and uvm_barrier_pool
comments: true
share: true
tags: [uvm]
nav_order: 1
toc_en: true
---

# How to use uvm_barrier and uvm_barrier_pool
`uvm_barrier` is added alongside `uvm_event` for synchronization.
This class allow us to block the specific processes until an expected number of processes reach a defined point.
After that, all the pending processes will be resumed at the same time, similar to lifting the barrier.
{: .fs-5 .fw-500 }

---
## How to use uvm_barrier and uvm_barrier_pool

### uvm_barrier class basic functions
The `uvm_barrier` class supports several basic methods as below:
* *`new()`*: Creates a new barrier object.
* *`set_threshold(int threshold)`*: Set the number of waiting processes threshold. If the number of waiting processes is greater or equal to this number, the barrier will be lifted
* *`get_threshold()`*: Return the current threshold of the barrier.
* *`wait_for()`*: Blocking the current process, increase the number of waiting processes of the current uvm_barrier object.
* *`reset(bit wakeup=1)`*: Reset the counter back to zero. If `wakeup` argument is 1'b1, all the waiting processes will resume. If `wakeup` is 1'b0, all the waiting processes will be kept waiting.
* *`set_auto_reset(bit value=1)`*: Allow the barrier to autoreset after the threshold is reach.
* *`get_num_waiters()`*: Return the number of waiting processes.
* *`cancel()`*: Decrease the number of waiting processes by one.

Let's take an example below:
{% highlight verilog %}
...
   uvm_barrier m_bar = new();

   // Set the threshold to 2,
   // Means the barrier will be lifted when the number of processes waiting is 2.
   m_bar.set_threshold(2);

...
   fork
      begin
         $display("Process 1");
         #1ns;
         m_bar.wait_for();  // suspend the process, increase the barrier counter by 1
         $display("Process 1 resumes");
      end

      begin
         $display("Process 2");
         #3ns;
         m_bar.wait_for();  // increase the barrier counter by 1, reach threshold value
         $display("Process 2 resumes");
      end
   join_none

//
// after 1ns, the process 1 is suspended, the counter of `m_bar` barrier will be 1. 
//            the process 1 will wait for the counter to reach the threshold value (which is 2).
//
// after 3ns, in the process 2, the `m_bar.wait_for()` will increase the barrier counter by 1,
//            so the counter will be 2, equal to the threshold value, then the barrier is lifted.
//            eventually both of process 1 and 2 will be resumed after 3ns.
//

{% endhighlight %}


### uvm_barrier_pool
Similar to [ uvm_event ]({{ site.baseurl }}{% link _posts/x_myrandoms/uvm_randoms/2021-09-20-uvm-event.md %}), we has this `uvm_barrier_pool`, which is a `uvm_pool` for `uvm_barrier` object.
And we use it exactly like the `uvm_event_pool`.

{% highlight verilog %}
typedef uvm_object_string_pool #(uvm_barrier) uvm_barrier_pool;
{% endhighlight %}

Let's think of this pool as a global associative array
where the keys are strings of barrier names, and the values are the `uvm_barrier` objects.

Also, `uvm_pool` is a [ singleton class ]({{ site.baseurl }}{% link _posts/x_myrandoms/systemverilog_randoms/2021-09-03-singleton-class-in-systemverilog.md %}),
that explains why it has global access.

`uvm_barrier_pool` support several methods, but the most commonly used is `get_global()`.
* *`get_global(<string key>)`*: Return the uvm barrier object that stored in `uvm_barrier_pool` with `<string key>`.
If no item exist by the given input string, a new `uvm_barrier` object will be created for that key.

So to create/get an `uvm_barrier` object which is shared globally, we just need to call:
{% highlight verilog %}
   // if there is no uvm_barrier stored under name "test_finish_bar",
   // a new uvm_barrier object will be created by the pool
   uvm_barrier m_finish_bar = uvm_barrier_pool::get_global("test_finish_bar");

   m_finish_bar.set_threshold(3);

   // Or we can directly use the event as below:
   uvm_barrier_pool::get_global("test_finish_bar").set_threshold(3);
   ...
   uvm_barrier_pool::get_global("test_finish_bar").wait_for();
{% endhighlight %}

---
## Example
### Using uvm_barrier to wait for a number of signal pulse
{% highlight verilog %}
   ...
   virtual status_interface m_sts_if;
   ...

   // Set the threshold to 5, 
   // If there is 5 "wait_for()" statements blocking their processes, the barrier will be lifted
   uvm_barrier_pool::get_global("wait_done_count").set_threshold(5);

   ...
   uvm_event m_done_count_ev = new();

   fork
      forever begin
         @(posedge m_sts_if.op_done_signal_a);
         fork 
            begin
               uvm_barrier_pool::get_global("wait_done_count").wait_for();
               m_done_count_ev.trigger();
            end 
         join_none
      end 
   join_none

   m_done_count_ev.wait_on();
   $display("op_done_signal_a toggle 5 times");

   ...
{% endhighlight %}
* In above example code, each time the `op_done_signal_a` asserts, we create a process which has a `wait_for()` statement.
This statement will increase the counter of the barrier `wait_done_count` by one.
Also, since this process is put in `fork`-`join_none`, the forever loop will continue immediately and wait for the next rise edge of the `op_done_signal_a`.
* After 5 pulses of `op_done_signal_a`, there will be 5 processes blocked by `wait_for()` statement.
Since the threshold is 5, the barrier will be lifted, all 5 processes will be resumed and execute the next statement, trigger the event `m_done_count_ev`.
* So using `uvm_barrier` and an event, we can detect when the signal has asserted 5 times.

---
## Finding more information
1. [ uvm_barrier.svh ](https://verificationacademy.com/verification-methodology-reference/uvm/docs_1.2/html/files/base/uvm_barrier-svh.html)
1. [ uvm_pool.svh ](https://verificationacademy.com/verification-methodology-reference/uvm/docs_1.2/html/files/base/uvm_pool-svh.html)


