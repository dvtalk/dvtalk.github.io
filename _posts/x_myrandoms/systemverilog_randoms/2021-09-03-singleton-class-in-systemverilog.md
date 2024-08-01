---
layout: default
title: Singleton class in Systemverilog
parent: Systemverilog Randoms
grand_parent: My Randoms
description: How to create a Singleton class and it's usage in Systemverilog
tags: [systemverilog, uvm]
comments: true
toc_en: true
nav_exclude: false
search_exclude: false
nav_order: 3
---

# Singleton class in Systemverilog
Singleton class is a type of class which only has one object. Once constructed, it will persist and can be accessed everywhere.
Singleton pattern is actually one of the simpliest [design pattern](https://refactoring.guru/design-patterns), and also pretty common in many programming languages.
{: .fs-5 .fw-500 }

---
## How to create a singleton class in Systemverilog
### Creating a protected constructor
To create a singleton class, firstly we need a protected constructor.
{% highlight verilog %}
class example_singleton_class;

   protected function new(string name = "example_singleton_class")
      super.new(name);
   endfunction

endclass
{% endhighlight %}
* By make the constructor protected, we have a full control of how this class is constructed,
because this constructor `new()` function can only be called inside this class.

### Adding static m_self variable and static get_inst() function
Since the constructor `new()` is protected, we need to create a function that acts as a contructor and return the handle to the object.
{% highlight verilog %}
class example_singleton_class;
   ...
   static protected example_singleton_class m_self;
   ...
   protected function new(string name = "example_singleton_class")
      super.new(name);
   endfunction

   static function example_singleton_class get_inst();
      if (m_self == null) begin
         m_self = new("example_singleton_class");
      end
      return m_self;
   endfunction

endclass

{% endhighlight %}
* The `m_self` will have the handle to the `example_singleton_class` object itself.
* The `get_inst()` will check if the `m_self` is null. If it is, we will construct a `example_singleton_class` object then assign the handle of that object to `m_self` variable.
If the `m_self` already has the handle of the singleton object, we just need to return that handle.
By doing this, we can make sure that the `new()` constructor will only called one time when the `get_inst()` is called the first time.
Eventually, only one `example_singleton_class` object will be created.
* We need to define the `get_inst()` function as static, so that we can called it by using the class name with scope operator `example_singleton_class::get_inst()`.
This provided the global access to the `get_inst()` function.
Also since it is `static`, the `get_inst()` cannot be defined as `virtual`.
* Because the `get_inst()` is static, the `m_self` must be static as well (The static function can only access to the static variable of the class).
* We also want to defined the `m_self` as protected, so that the only way to get the handle of this object is to call the `get_inst()` function.

### Constructing a singleton class
Now, when we need the access to the object of `example_singleton_class`, we just need to call the `get_inst()` function as below.
Since this function is static, we can call it everywhere in our environment.
{% highlight verilog %}
...
   example_singleton_class m_singleton = example_singleton_class::get_inst();
...
{% endhighlight %}

#### Eager initialization
We can have the singleton class to construct itself by calling the `get_inst()` when initialize the `m_self` variable
{% highlight verilog %}
...
   static protected example_singleton_class m_self = get_inst();
...
{% endhighlight %}

When doing this, the singleton class will be constructed at the very beginning of the simulation.
This way to constructing singleton object is called **eager initialization**.

---
## Is it any good?
### Which class can be singleton
From it properties, a class might need below requirements can be a candidate for creating singleton class:
* Need to be access from many unrelated part of our environment.
* Have the access to a shared resource.
* Have a single purpose which many classes needs.

This could be a class that provide the backdoor access to the memory of DUT, a class that handle all the plusargs that passed from the run time commands 
or a class to dynamic allocate/deallocate index number of data packets.



### Some uvm singleton classes
If using the uvm methodology, we can see that several classes are actually singleton:
1. uvm_root and uvm_top:
* uvm_root is the the singleton class, and the uvm_top has the handle to that singleton class.
* This class serves as a top for all of the uvm component, and it is contructed by default when we call `run_test()`.
1. uvm_core_service_t:
* From uvm1.2, the core services class will contains the method to access to handle of several uvm feature such as Factory, Report Server.
* To get the handle of this singleton class, we just call `uvm_core_service_t cs = uvm_core_service_t::get()`
1. uvm_factory:
* Factory is actually another type of design pattern, but we'll talk about it in another post.
* In uvm, uvm_factory allow us to substitute a class with its child class when constructing object.
* To have the handle to uvm_factory, we can call its static class as other singleton: `uvm_factory factory = uvm_factory::get();`
* There's another way to get uvm_factory singleton handle using core services: `uvm_factory factory = uvm_core_service_t::get().get_factory()`
* To understand how uvm works, read this post: [How uvm_factory acutally works]({{ site.baseurl }}{% link _posts/x_myrandoms/uvm_randoms/2021-09-12-how-uvm-factory-actually-works.md %})
1. uvm_pool:
* `uvm_pool` is a class which acts as an associative array.
* We can decide the type of key and data of this associative array by defining 2 parameters of `uvm_pool` class.
* This class is singleton class so it can be used to share data between components, objects in the environment.
* The 2 common `uvm_pool` built-in the uvm are [ `uvm_event_pool` ]({{ site.baseurl }}{% link _posts/x_myrandoms/uvm_randoms/2021-09-20-uvm-event.md %})
and [ `uvm_barrier_pool` ]({{ site.baseurl }}{% link _posts/x_myrandoms/uvm_randoms/2021-09-27-uvm-barrier.md %})
1. uvm_queue:
* Similar to `uvm_pool`, `uvm_queue` is a class which act as a Systemverilog queue.
* Defined as singleton class, this queue can be accessed anywhere to share data.

---
## Alternative solution: static method in package

---
## Finding more information
To have more understanding as well as more examples, check below references:
1. [Singleton Pattern - Creational Design Patterns](https://refactoring.guru/design-patterns/singleton)
1. [How uvm_factory acutally works]({{ site.baseurl }}{% link _posts/x_myrandoms/uvm_randoms/2021-09-12-how-uvm-factory-actually-works.md %})
1. [uvm_barrier_pool]({{ site.baseurl }}{% link _posts/x_myrandoms/uvm_randoms/2021-09-27-uvm-barrier.md %})
1. [uvm_event_pool]({{ site.baseurl }}{% link _posts/x_myrandoms/uvm_randoms/2021-09-20-uvm-event.md %})

