---
layout: default
title: How to use uvm factory
parent: UVM Randoms
grand_parent: My Randoms
description: How to use uvm factory
comments: true
share: true
tags: [uvm]
nav_order: 1
toc_en: true
---

# How to use uvm factory
Uvm factory is one of the most notable term when using uvm methodology.
It helps increase flexibility and resuability of our testbench.
{: .fs-5 .fw-500 }

---
## Uvm factory and it's benefit
Uvm factory allow us to replace an uvm object or component class with it's child class with minimum code modification.

Let take an example below:
* We have an uvm testbench and inside this testbench, we have an transaction item class to construct an Ethernet packet.
Usually when defining a transaction item class, we will randomize it's variables and define the randomization contraints.
Then we create our tests, with many different scenarios to generate and randomize several object of this Ethernet packet class.

* Later on, we want to run the same set of tests created above, using the same transaction item class but with different constraints.
And we also want to have the tests running with the existing contraints.

* Simply thinking, we will extend the existing class, and add new constraints to the new child class.
But to run all the tests with the new class is a real problem.
Modifying all the existing tests in order to construct this new class is too much labor intensive =D.

* uvm factory will rescue us in this case.
Using uvm factory, we just need to call a simple method,
then all the objects of the original class will be substitued by the new class when constructing.

---

## How to use uvm factory

### Uvm factory rules
To make sure the uvm object and uvm component can be substitued, we need to follow some rules below:
* Calling these macros below to register the class to uvm factory:
{% highlight verilog %}
//for uvm component
`uvm_component_utils(<class type>)
`uvm_component_param_utils(< class type >)

//for uvm object
`uvm_object_utils(<class type>)
`uvm_object_param_utils(<class type>)

//i.e:
class l2_layer_base extends uvm_object;
   `uvm_object_utils(l2_layer_base)
   ...
endclass

//i.e: parameterized object
class l2_layer_base #(int T=32) extends uvm_object;
   typedef l2_layer_base #(T) class_t;
   `uvm_object_param_utils(class_t)
   ...
endclass
{% endhighlight %}

* The constructor function must follow below format:
{% highlight verilog %}
//for uvm component
   function new(string name="<class name>", uvm_component parent = null);
      super.new(name, parent);
   endfunction

//for uvm object
   function new(string name="<class name>");
      super.new(name);
   endfunction
{% endhighlight %}

* Using type_id::create() to construct object
{% highlight verilog %}
//i.e for l2_layer_base#(T) uvm object
   l2_layer_base m_l2_obj = l2_layer_base#(64)::type_id::create("m_l2_obj");
//
{% endhighlight %}

### Uvm factory override
For any class which follow those rules above, we now can perform override that class with the child class.

To simply understand the uvm factory, let's just think of it as a lookup array,
with the lookup key is the original class type and the return value is the overridden class type of that class.

When constructing the object, the constructor will check that lookup array (uvm factory),
then construct the overridden class (if existed) instead of the original one.

<div class="code-highlight" markdown="1" >
The overridden class can only be the child class of the original one. <br>
The override method must be called before the object/component construction take place.
</div>


#### Override by type
Override by type means each time an object/component is constructed, the overridden class type is constructed inside of the original one.
{% highlight verilog %}
//i.e for l2_layer_base uvm object, override the original class with its child class l2_layer
   l2_layer_base::type_id::set_type_override(l2_layer::get_type())
{% endhighlight %}


#### Override by instance
Override by instance means we can override a specific instance of a uvm component in the uvm component hierarchy.
{% highlight verilog %}
//i.e for driver_a uvm component, override the original class with its child class driver_b
   driver_a::type_id::set_inst_override(driver_b::get_type(), "uvm_test_top.m_env.m_agent.m_driver");
{% endhighlight %}

**Only uvm component can have apply the overide by instance approach**
because the uvm object does not have a specific instance in the uvm testbench hierarchy.

#### Override from command line
From the command line, we can also override by type and instance by using built in plusargs.
However I have never used these ones.

{% highlight verilog %}
+uvm_set_type_override=<org_type>,<new_override_type>

+uvm_set_inst_override=<org_type>,<new_override_type>,<full_inst_path>
//
{% endhighlight %}

### Uvm factory debug
uvm factory class is actually a singleton class, and it has several method those will help us debugging:
{% highlight verilog %}
// to get the uvm_factory object handle
uvm_factory m_factory = uvm_core_service_t::get().get_factory(); //uvm1.2
uvm_factory m_factory = uvm_factory::get();  //uvm1.1d

//print the override information
m_factory.print();

m_factory.debug_create_by_type(l2_layer_base::get_type());

m_factory.debug_create_by_name("l2_layer_base"); //assuming the class was created with name "l2_layer_base"
//
{% endhighlight %}

---

## Finding more information
1. [ How uvm factory actually works ]({{ site.baseurl }}{% link _posts/x_myrandoms/uvm_randoms/2021-09-12-how-uvm-factory-actually-works.md %})
1. [ uvm cookbook ](https://verificationacademy.com/cookbook/factory)


