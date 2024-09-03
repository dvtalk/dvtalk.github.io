---
layout: default
title: Differences between uvm test and uvm testbench top
parent: UVM How to
grand_parent: My How to
description: Differences between uvm test and uvm testbench top 
comments: true
share: true
tags: [uvm]
nav_order: 1
toc_en: true
---

# Differences between uvm test and uvm testbench top 
Back in the early days of my journey with uvm, the concept of uvm test and testbench top really got me. Before learning to use systemverilog and uvm, I only knew about a basic testbench, where the top would handle everything. So when studying a uvm testbench, it took me quite sometime to absorb how uvm test and testbench top hierarchy are related. Here it is:
{: .fs-5 .fw-500 }

---
## What is uvm testbench top
uvm testbench top is like the top of the tradition testbench. This is where these things happen:
* The DUT will be instantiated.
* Clock will be generated and supplied to the DUT.
* The interface object will be constructed, and connect to the DUT ports. Also the interface object handle will be registered to the `uvm_config_db`.
* The `run_test()` method will be called.
* Also in this top, we must have this line: `import uvm_pkg::*;`

The testbench top will be passed to the EDA tool as top module in the compiling step. Then all the hdl path of your dut will start from this tb top and look like this: `tb_top.dut.sub_module1.signal_a` (where `tb_top` is the module name of the testbench top module)

---

## What is uvm test
uvm test will be a place where the uvm components such as `uvm_env`, `uvm_agent`, `uvm_scoreboard` will be constructed to create the testbench hierarchy.
A lot of things will occur inside the uvm test, but commonly these steps will be performed:
* Construct an `uvm_env`.
* Create the configuration for env and sub-configuration of sub-component (such as agent, monitor, etc).
* Get the interface handle from `uvm_config_db`.
* etc.

Usually in a uvm testbench, all those step above will be added to a test base, then all other tests will just need to extends to this test base as below:

<div class ="code" markdown="1" >
{% highlight verilog %}
   class test_base extends uvm_test;
   class test_no1 extends test_base;
{% endhighlight %}
</div>

---

## How uvm test and uvm testbench top are related?
uvm test and testbench top are two basic items of a uvm testbench, where everything begins. To distinguish these two, we should better understand how they related to each other.


### DUT and driver/monitor connections
* As described above, in the testbench top module, the DUT will be connect to the interface object. This interface object's handle will be register to the `uvm_config_db`.
* So in the uvm test side, to give the driver the ability to drive the DUT signals, as well as the monitor to sample the DUT signals, we will get the handle of this interface object (which connected to DUT in top module) from the `uvm_db_config`.
![testbench top and uvm test connection relation](https://jqqyra.by.files.1drv.com/y4mpOHaCB4faTDr78inMH3usW3dm9jyIGVN8J2MmWVJcChnz2PPDye3_drhatqEiIHoS4Y9TPr84wSMoLV3e0qAeo5DYOsJq5l_dfINp9Gl1plKkpfOpkWuwTG7MT63cxMXLAwiBJl8qwISYg0K_x0gNkfoj9gDilFbHWjs1DunMaCnuodN3kTl5kLP3Ay7I7rzjPDlhAgB9Qq91O0yVowknA)

### Testbench construction
* Everything in a uvm testbench begins in the testbench top, where the `run_test()` method is called. This method is actually a task defined in the `uvm_globals.svh` which is imported by this statement `import uvm_pkg::*`.
* Then inside this `run_test()` we will call the `uvm_top.run_test()`. The `uvm_top.run_test()` will get the name of the uvm test class (usually passed in as the command line argument `+UVM_TESTNAME=YOUR_TEST_NAME` ), then it will use the uvm factory to find that test class and construct it (the test class must be registered to the uvm factory by using `uvm_component_utils()` ).
* After that, still in the `uvm_top.run_test()`, the uvm phases will be started. First phase will be `build_phase`, where the uvm hierarchy will be constructed top down (from env to agent, then driver/monitor,...).
Refer to the figure below from [uvm_cookbook](https://verificationacademy.com/cookbook/testbench/build)
![testbench top and uvm test build flow figure](https://by3302files.storage.live.com/y4mEu1I9nAsQbg6vvTfyIQbEJ4v1P73CM0ISP066beWvKlA5Rp3hmucZ0esvu0fuvIlPrHiWVpBrkzN14ybN-hYLcei_h7qLrSnHUMOlLIaWbd4Hhxnl6ZCPr1gzAadmdK9PxQZaJ3VcpAQ4Hk6WMrem8bS2kQyycUTEPuMsiBCuvDE83Fk4bHQSN_4Ew-ZiHQCk4CPYgxB9l4p8okAe5oBUg/2020_08_09_uvm_test_vs_uvm_top_1.png?psid=1&width=772&height=553)

---

## Don't be confused

### uvm_top
The uvm testbench top different with `uvm_top`. The `uvm_top` is a global variable that hold the handle of `uvm_root` object inside `uvm_pkg` (which we import in the uvm testbench top). When we import the uvm package (`import uvm_pkg::*`) in the testbench top, then call the `run_test()` method, the `uvm_root` object is constructed (`uvm_root` is a singleton class). Check this code below in the [uvm_root.svh](http://www.studio-muzzi.com/project/docs/UVMdocs_smu/uvm-1.1d/uvm__root_8svh_source.html)

<div class ="code" markdown="1" >
{% highlight verilog %}
   const uvm_root uvm_top = uvm_root::get();
{% endhighlight %}
</div>


### uvm_test_top
We have `uvm_test_top`. It's actually the default variable containing the object of uvm test that we discussed above. When the `run_test()` is called, the uvm test class (which is passed in as cli argument with `+UVM_TESTNAME=YOUR_TEST_NAME`) will be constructed and the handle will be stored in the `uvm_test_top` variable. It also is the top hierarchy of our uvm testbench. When running the uvm testbench, we will see testbench hierarchy path like this `uvm_test_top.env.agent_1.driver_1`. Check this code below in the [uvm_root.svh](http://www.studio-muzzi.com/project/docs/UVMdocs_smu/uvm-1.1d/uvm__root_8svh_source.html)

<div class ="code" markdown="1" >
{% highlight verilog %}
$cast(uvm_test_top, factory.create_component_by_name(test_name, "", "uvm_test_top", null));
{% endhighlight %}
</div>


