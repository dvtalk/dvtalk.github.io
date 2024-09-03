---
layout: default
title: How uvm factory actually works
parent: UVM How to
grand_parent: My How to
description: How uvm factory actually works
comments: true
share: true
tags: [uvm]
nav_order: 1
toc_en: true
---

# How uvm factory actually works
This post will go into the detailed implementation of the uvm factory.
If you have not known what uvm factory is yet, check this one:
[How to use uvm factory]({{ site.baseurl }}{% link _posts/x_myrandoms/uvm_randoms/2021-09-11-how-to-use-uvm-factory.md %}).
{: .fs-5 .fw-500 }

---
## Things to know before diving in
* Class scope resolution operator `::`.
1. We use the class scope resolution operator to gain access to an element inside a class.
1. A class scope resolution operator can be used for all static element: static class variables, static function/task, parameters, local parameters, enums, unions, constraints, nested class declaritions and *typedefs*.

* Abstract class
1. Abstract class is like a template of other classes, cannot be instantiated directly. [Virtual class and pure virtual methods]({{ site.baseurl }}{% link _posts/x_myrandoms/systemverilog_randoms/2021-08-26-summarize-virtual-in-systemverilog.md %})

* Parameterized class
1. Known as generic class in other programming languages.
1. The important point when using parameterized class is that the class type is only valid when parameters are provided. And each set of parameters will create a different class type.
1. For example, given the parameterized class `class bus_monitor #(int ADDR_BUS_WIDTH);`, the `bus_monitor` is not a valid class type. `bus_monitor#(32)` and `bus_monitor#(64)` are valid, and these are two different class type.

* Singleton class
1. This is a type of class that can only has one single instance.
1. Check this post for detailed informations [ Singleton class in Systemverilog  ]({{ site.baseurl }}{% link _posts/x_myrandoms/systemverilog_randoms/2021-09-03-singleton-class-in-systemverilog.md %})

---
## Registering the uvm obj/component to uvm factory
Firstly, when defining a class, we must register that class to the uvm factory.
This step is simpified by using the uvm macro: `uvm_object_utils` or `uvm_component_utils`.

### Object/Component registry class
The macro will actually create a typedef of type_id as below:
{% highlight verilog %}
class l2_layer extends uvm_object;
   `uvm_object_utils(l2_layer)

//-->
//generated code from macros
   typedef uvm_object_registry #(l2_layer, "l2_layer") type_id;
   ...
endclass
{% endhighlight %}

The `uvm_object_registry#(l2_layer, "l2_layer")` class will be constructed and register it's instance handle to uvm factory by itself.
{% highlight verilog %}
class uvm_object_registry #(type T=uvm_object, string Tname="") extends uvm_object_wrapper;
   //1. factory register code
   typedef uvm_object_registry #(T,Tname) this_type;
   local static this_type me = get();
   static function this_type get();
      if (me == null) begin
         uvm_factory factory = uvm_coreservice_t::get().get_factory();
         me = new;
         factory.register(me);  //register to uvm factory
      end
      return me;
   endfunction
endclass
{% endhighlight %}
* We can see that the `uvm_object_registry#(l2_layer, "l2_layer")` is actually a singleton class.
Also, thanks to the initialization of the static variable `static this_type me = get()`, this class will be constructed automatically at the very beginning, before any execution.
This is actually called **eager initialization**, where the instance of singleton class is created thanks to the initialization of static variable.

* Also, in the `get()` function, this singleton class `uvm_object_registry#(l2_layer, "l2_layer")` will be registered to uvm factory using this statement `factory.register(me)`

* So, when define a class `l2_layer`, by using `uvm_object_utils(l2_layer)` macro, we actually will create a singleton object.
In `l2_layer` class, this singleton object `uvm_object_registry#(l2_layer, "l2_layer") has a type `type_id`.
This `type_id` object is registered to the uvm factory automatically.
Also, this singleton object will contain the class type `l2_layer` and the class name string as its parameters.
Those are the information will be used by the uvm factory for searching and constructing class instance.

---
## Constructing the new object
The other code convention when using uvm factory is that to construct the obj, instead of using the `new()` constructor directly,
we must use the `create()` method inside the `type_id` singleton object:

`l2_layer m_l2_obj = l2_layer::type_id::create("l2_layer");`
### Constructing object
Still in the `uvm_object_registry` class, we will have this `create()` method.

{% highlight verilog %}
class uvm_object_registry #(type T=uvm_object, string Tname="") extends uvm_object_wrapper;
   ...
   //1. singleton object construction and factory register code
   ...

   //2. create() function, called by user
   static function T create (string name="", uvm_component parent=null, string contxt="");
      uvm_object  obj;
      uvm_factory factory = uvm_coreservice_t::get().get_factory();

      //uvm factory will construct and return the obj
      //the uvm factory actually will construct the override child class of T
      obj = factory.create_object_by_type(get(),contxt,name);
      ...
   endfunction

endclass
{% endhighlight %}

* The `create()` is a static function, so we can access this function using class scope resolution operator `l2_layer::type_id::create()`
* In this method, we actually ask the uvm factory to construct the object using this statement `obj = factory.create_object_by_type(get(), contxt, name)`.
In uvm factory, there is an array to stored the original class type and it's overriden class type.
If the `l2_layer` is overriden by it's child class, this child class will be constructed instead.

Example of creating `l2_layer` object in a sequence class:
{% highlight verilog %}
class sequence_a extends uvm_sequence;
...
   l2_layer::type_id::set_type_override(l2_layer_mac::get_type()); // where the l2_layer_mac is extended from l2_layer
...
   l2_layer m_l2_obj = l2_layer::type_id::create("l2_layer");  // the l2_layer_mac object will be constructed
...
endclass
{% endhighlight %}
* In the above example, the `l2_layer` is overriden by its child class `l2_layer_mac`.
* So after constructing object using uvm factory, the `m_l2_obj` variable will point to an object of `l2_layer_mac`.

---
## Inside the uvm_factory
So, when defining the class with uvm factory we need:
1. Firstly, we will register the class into the `uvm_factory` using the uvm macro.
1. Then when constructing the obj, we will ask the `uvm_factory` to construct and return the expected obj.
If the class has been overriden, then the return obj will be the object of the overriden child class instead of the original one.

Let see how the `uvm_factory` can do that

### uvm_factory class
In uvm1.2, The `uvm_factory` is actually a abstract class.
The `uvm_default_factory` is the extended class of the `uvm_factory` and it's instance can be retrieved through a `uvm_coreservice_t` singleton class as below:

`uvm_factory factory = uvm_core_service_t::get().get_factory()`

### Object/Component wrapper class
The `uvm_object_registry` actually has another function called `create_object()`.
This function will be called by the `uvm_factory` to create an object of the registered class.

Also the `uvm_object_registry` is extended from the `uvm_object_wrapper` class, which is simply an abstract class.
{% highlight verilog %}
virtual class uvm_object_wrapper;
   virtual function uvm_object create_object (string name="");
      return null;
   endfunction

   pure virtual function string get_type_name();
endclass

class uvm_object_registry #(type T=uvm_object, string Tname="") extends uvm_object_wrapper;
   //1. factory register code
   ...
   factory.register(me);  //register to uvm factory
   ...

   //2. create() function, called by user

   //3. create_object() function, called by uvm_factory
   virtual function uvm_object create_object(string name="");
      T obj;
      obj = new(name);
      return obj;
   endfunction

   const static string type_name = Tname;
   virtual function string get_type_name();
      return type_name;
   endfunction
endclass
{% endhighlight %}
* The `create()` method will be called by user. As explained in the section above, this function will return the handle of the overriden class object.
* The `create_object()` method will be called by `uvm_factory`. This function will construct and return the object of the register class.

Example:
{% highlight verilog %}
...
   l2_layer::type_id::set_type_override(l2_layer_mac::get_type()); // where the l2_layer_mac is extended from l2_layer
...
   l2_layer m_l2_obj = l2_layer::type_id::create("l2_layer");  // the l2_layer_mac object will be constructed
...
{% endhighlight %}
* In the above example, inside the `uvm_object_registry#(l2_layer, "l2_layer")`,
the `create()` method will construct the `l2_layer_mac` object thanks to uvm factory, and the `create_object()` will construct the `l2_layer` object.

### Factory register() function
As above, to register a class to the factory, we call `factory.register(me)` inside the `get()` function of the `uvm_object_registry`.
Where `me` has the handle to the object of singleton class `uvm_object_registry#(l2_layer, "l2_layer")`

Let's take a look at the `register()` function
{% highlight verilog %}
class uvm_default_factory extends uvm_factory;

   protected bit                  m_types[uvm_object_wrapper];
   protected uvm_object_wrapper   m_type_names[string];

   function void register(uvm_object_wrapper obj);
      m_type_names[obj.get_type_name()] = obj;
      ...
      m_types[obj] = 1;
      ...
   endfunction
endclass
{% endhighlight %}

* In uvm factory, we have options for overriding the original class, either by type or by name.
The two above associative arrays `m_types` and `m_type_names` serve that purpose.
They will store the class registered to factory using either class type or class name string as lookup key.
* Before perform override function or class contructor function on a class,
the factory will check these 2 arrays to make sure that class has already been registered in the factory.

### Factory override function
Let just cover the override by type here.
The override by inst just operates in the same fashion as the by type override.

We has a queue to store the overriden information named `m_type_overrides`.
This queue store the information of original class and it's overriden one.
The `uvm_factory_override` class object is an object that store those information when we call the `set_type_override_by_type()` function.
{% highlight verilog %}
class uvm_default_factory extends uvm_factory;
   protected uvm_factory_override m_type_overrides[$];

   function void set_type_override_by_type (uvm_object_wrapper original_type,
                                            uvm_object_wrapper override_type,
                                            bit replace=1);

      uvm_factory_override override;
      override = new(.orig_type(original_type),
                     .orig_type_name(original_type.get_type_name()),
                     .full_inst_path("*"),
                     .ovrd_type(override_type));
      m_type_overrides.push_back(override);
   endfunction
endclass
{% endhighlight %}

### Factory create object function
Finaly, when create and object using the `create()` function as below:

`l2_layer m_l2_obj = l2_layer::type_id::create("l2_layer");`

We will call the function `create_object_by_type()` in uvm factory
{% highlight verilog %}
function uvm_object uvm_default_factory::create_object_by_type (uvm_object_wrapper requested_type,  
                                                        string parent_inst_path="",  
                                                        string name=""); 

   requested_type = find_override_by_type(requested_type, full_inst_path);

   return requested_type.create_object(name);
endfunction
{% endhighlight %}

* The `find_override_by_type()` will search the latest overriden class type in the `m_type_overrides[$]`, then return the class type that needed to construct.
* Then we will construct the object using the `create_object()` function inside the `uvm_object_registry#()` class.
* For example, if the `l2_layer` is an original class type, and the `l2_layer_mac` is the overriden class type. The `create_object()` of `uvm_object_registry#(l2_layer_mac, "l2_layer_mac")` will be call, and the `l2_layer_mac` object will be constructed eventually.

---
## Finding more information
1. [ uvm_factory.svh ](https://verificationacademy.com/verification-methodology-reference/uvm/docs_1.1a/html/files/base/uvm_factory-svh.html)


