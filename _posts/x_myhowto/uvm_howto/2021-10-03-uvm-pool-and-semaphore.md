---
layout: default
title: uvm_pool example with Systemverilog semaphore
parent: UVM How to
grand_parent: My How to
description: uvm_pool example with Systemverilog semaphore 
comments: true
share: true
tags: [uvm]
nav_order: 1
toc_en: true
---

# uvm_pool example with Systemverilog semaphore
Uvm support `uvm_pool`, which can be understood as a global associative array.
We can define the the type of index and value of this array.
{: .fs-5 .fw-500 }

By default, uvm has two pools using string as index for `uvm_event` and `uvm_barrier`, which are `uvm_event_pool` and `uvm_barrier_pool`, respectively.
In this post, let's go through examples of using `uvm_pool` for Systemverilog semaphore.
{: .fs-5 .fw-500 }

---
## uvm_pool and semaphore 101
First up, let's briefly cover the semaphore and `uvm_pool`.

### What is uvm_pool
1. `uvm_pool` allow us to store any type of data with a key as index, similar to an associative array.
1. It is singleton class. 
1. Must define a pool before use: `typedef uvm_pool(.KEY(int),.T(semaphore)) semaphore_pool`
1. To get the handle of uvm_pool singleton object, use static method `semaphore_pool::get_global_pool()`
1. Use public method `add(KEY key, T item)`,`delete(KEY key)`,`get(KEY key)`, to add/get object from pool
1. Use static method `semaphore_pool::get_global(KEY key)` to get obj from the pool.

### Systemverilog semaphore
Semaphore is a class in the built-in `std` package of Systemverilog.
As other languages, the semaphore is a bucket with a specific number of keys.
When all the keys are get from the bucket, other proccesses wishing to get the key must wait until there's a key return to the bucket.

Semaphore most well-known application is to manage the access to the common resource.

For example, if there are only 5 slots in the memory to store data, then we create a semaphore with 5 keys to control the access to these slots.
Any process wanting to grant a slot must get the key from the semaphore bucket, and return the key to the bucket when it finishes.
If there is no key in the bucket, meaning all the slots are occupied, 
the process must wait until other processes done using the resource and return the key.

Basic methods of Systemverilog semaphore are:
* *`new(int keyCount=0)`*: Construct a semaphore object with a specific number of keys.
* *`put(int keyCount=1)`*: Add keys to the bucket.
* *`get(int keyCount=1)`*: Get keys from the bucket. When there are not enough keys in the bucket,
this task will **block** the process until the keys are available.
* *`try_get(int keyCount=1)`*: Similar to `get()` method, but this method will **not block** the process.
If there is not enough keys as requested, this method will return 0.

#### Disadvantages of Systemverilog semaphore:
There are some points which I considered as disadvantages of Systemverilog semaphore:
1. Since semaphore is well-known for manage the access to common resource, it need to be accessed from many places in the environment.
1. We can put more keys to the bucket, using the same function `put()` which is used to return the key.
Therefore we might accidentally increase the total number of keys of the bucket, causing the conflict, since there are more keys than actual resources.
1. There is no built-in method for debugging.

These issues can be solved by creating a class wrapping around the Systemverilog semaphore with some custom methods
and using the `uvm_pool`, which will be shown in below examples.

---
## Example
### Using uvm_pool with Systemverilog semaphore directly
Firstly, let's remember the `std::semaphore` in Systemverilog is a class.
Therefore so we can construct and passing it's object handle as normal.

To create a pool of Systemverilog semaphore, we just need to define our pool for semaphore as below:
{% highlight verilog %}
   typedef uvm_pool #(.KEY(string), .T(semaphore)) uvm_semaphore_pool;
{% endhighlight %}

Then we contruct a pool then add/get semaphore object to this pool.
{% highlight verilog %}
   ...
   // To use this uvm_semaphore_pool, we must first construct it.
   // This pool is a singleton class, 
   // which will be constructed only one time by calling get_global_pool();
   uvm_semaphore_pool m_pool = uvm_semaphore_pool::get_global_pool();

   ...
   // Now we can add/get semaphore object to the pool

   //create semaphore with 2 keys
   semaphore m_port_avail_sem = new(2);

   //add this semaphore to pool with lookup name "port_avail_sem"
   uvm_semaphore_pool::get_global_pool().add("port_avail_sem", m_port_avail_sem);

   ...
   // Get 1 key of the semaphore "port_avail_sem" from the pool
   uvm_semaphore_pool::get_global("port_avail_sem").get(1);

   //
{% endhighlight %}


### Creating uvm_semaphore to use with uvm_object_string_pool
We can create our own `uvm_semaphore` class, wrap around the `semaphore` of Systemverilog.
This allow us having more control over the semaphore, also more information when debugging.

Besides, since we extend `uvm_semaphore` from `uvm_object`, 
we can use it with the `uvm_object_string_pool`, which is easier to use compare to the `uvm_pool`.

Let's create the `uvm_semaphore` class as below. You can get the full class here: [ uvm_semaphore ]( https://gist.github.com/dvtalk/692f45bba567aaeae98f61f63d867058 )
{% highlight verilog %}
   class uvm_semaphore extends uvm_object;
      protected semaphore m_semaphore;

      function new (string name="", int keyCount=0);
         super.new(name);
         m_semaphore = new(keyCount);
      endfunction

      virtual function void put(int keyCount=1);
         m_semaphore.put(keyCount);
         ...
      endfunction

      virtual task get(int keyCount=1);
         m_semaphore.get(keyCount);
         ...
      endtask
      
      virtual function void set_keyCount(int keyCount=0);
         ...
      endfunction
      ...
   endclass
{% endhighlight %}

Then we can use this `uvm_semaphore` with `uvm_object_string_pool`, similar to the [ uvm_event_pool ]({{ site.baseurl }}{% link _posts/x_myrandoms/uvm_randoms/2021-09-20-uvm-event.md %})
and [ uvm_barrier_pool ]({{ site.baseurl }}{% link _posts/x_myrandoms/uvm_randoms/2021-09-27-uvm-barrier.md %}).
{% highlight verilog %}
   typedef uvm_object_string_pool #(uvm_semaphore) uvm_semaphore_pool;
{% endhighlight %}

It will be a little more simple when using this pool:
* We can call the `get_global()` function to get the semaphore in the pool, 
and if the semaphore object does not exist, the pool will automatically construct a semaphore object for us.
* Also, we do not need to construct the pool manually, it will be construct automatically when we call `get_global()` the first time.

{% highlight verilog %}
   // When we call this function get_global("port_avail_sem").put(2) the first time:
   //    the semaphore pool will be constructed automatically. 
   //    the semaphore object associated with key name "port_avail_sem" will be constructed automatically.
   //    the keyCount of the object might be set as default to 0 (depend on your uvm_semaphore class), 
   //       we might put more key to the bucket to use.
   //    put 2 more keys in semaphore bucket.
   uvm_semaphore_pool::get_global("port_avail_sem").put(2); 
   
   ...
   // Get 1 key from the "port_avail_sem" semaphore in the pool
   uvm_semaphore_pool::get_global("port_avail_sem").get(1); //get key
   
{% endhighlight %}

---
## Finding more information
1. Systemverilog LRM, section 15.3 Semaphores
1. Systemverilog LRM, Annex G Std package
1. [ uvm_pool.svh ](https://verificationacademy.com/verification-methodology-reference/uvm/docs_1.2/html/files/base/uvm_pool-svh.html)


