---
layout: default
title: Using Plusargs in UVM Test
parent: UVM How to
grand_parent: My How to
description: Using Plusargs in UVM Test
comments: true
share: true
tags: [uvm]
nav_order: 1
toc_en: true
---

# Using Plusargs in UVM Testbench
Test plusargs.
{: .fs-5 .fw-500 }

---
## Systemverilog system function for plusargs
Systemverilog has two system functions for retriving the arguments (aka plusargs) from the command line, these are `$test$plusargs` and `$value$plusargs`.

### $test$plusargs(string)
This system function receives a string as input argument, then search through the plusargs in the command line.
If the **prefix** of any of the plusargs is matched with the provided string of the system function, this function return **non-zero** value.
Usually, we will use this function as an expression in the consditional statement `if`-`else` as below example:

{% highlight verilog %}
// Simulator command line argument: +PLUSARGS_TEST

if($test$plusargs(PLUSARGS_TEST)) begin 
  $display("Found plusargs +PLUSARGS_TEST");
end

//
if($test$plusargs(PLUSARGS)) begin 
  $display("Found +PLUSARGS, substring of command line string ");
end

// output will be 
// Found plusargs +PLUSARGS_TEST
// Found +PLUSARGS, substring of command line string
//
{% endhighlight %}

### $value$plusargs("PLUSARGS=format string", var)
Similar to `$test$plusargs`, but the plusargs now comes with a value, and we can assign that value to a variable as below example.
{% highlight verilog %}
// Simulator command line argument: +PLUSARGS_TEST=20 +PLUSARGS_TEST2+100 +PLUSARGS_TEST3+300+400

int m_var;
int m_var2;

if($value$plusargs("PLUSARGS_TEST=%d", m_var )) begin 
  $display ("Found plusargs +PLUSARGS_TEST = %d", m_var);
end 

// notice we will use the "+", not "=" character
// we actually do not need any special character at all
if($value$plusargs("PLUSARGS_TEST2+%d", m_var )) begin    
  $display ("Found plusargs +PLUSARGS_TEST2 = %d", m_var);
end 

// we can even has more than one value passed from command line
if($value$plusargs("PLUSARGS_TEST3+%d+%d", m_var, m_var2 )) begin    
  $display ("Found plusargs +PLUSARGS_TEST3 = %d, %d", m_var, m_var2);
end 

// The output will be:
// Found plusargs +PLUSARGS_TEST = 20
// Found plusargs +PLUSARGS_TEST2 = 100
// Found plusargs +PLUSARGS_TEST3 = 300, 400
// 


{% endhighlight %}

#### string format for plusargs
We can use below format string when get the value from plusargs

|      | |
|:-----|:--------|
|%d    |decimal conversion|
|%o    |octal conversion|
|%h,%x |hexadecimal conversion|
|%b    |binary conversion|
|%e    |real exponential conversion|
|%f    |real decimal conversion|
|%g    |real decimal or exponential conversion|
|%s    |string (no conversion)|



---
## UVM built-in functions those help
Some uvm functions those make handling test plusargs easier

### Plusargs for enum variable
Assuming we have a enum type `sha_mode_e`, and the plusargs input value is stored in a string.
Instead of using `switch-case` to match the string to each enum name constant,
we can use `uvm_enum_wrapper#(<enum type>)::from_name()` function to cast the plusarg string to the enum variable.

{% highlight verilog %}
 //
 // Example simulator command line argument: +SHA_MODE=SHA_256
 //

 string     m_tmp_str;
 sha_mode_e m_sha_mode;

 if($value$plusargs("SHA_MODE=%s", m_tmp_str)) begin
    if (uvm_enum_wrapper#(sha_mode_e)::from_name(m_tmp_str, m_sha_mode)) begin
       `uvm_info("[PLUSARGS]", $psprintf("Sha mode: %s", m_sha_mode.name()), UVM_LOW )
    end
 end
{% endhighlight %}

### uvm_cmdline_processor sigleton class
This uvm class supports many functions that handle the plusargs.
There is a global variable of `uvm_cmdline_processor` class called `uvm_cmdline_proc` and can be used to access command line information.

Let's check some examples below.

#### plusarg containing a list of values
In this example we will use the `uvm_cmdline_proc.get_arg_value()` function to get the argument as a string.
Then use the `uvm_split_string()` to split this string to entries based on the sepapartor.

{% highlight verilog %}
 //
 // Example simulator command line argument: +QUEUE_EN_LIST=0,5,10,15
 //

 string queue_en_lst;

 if (uvm_cmdline_proc.get_arg_value("+QUEUE_EN_LIST=", queue_en_lst)) begin
    string queue_en_s[$];
    int    queue_en[$];  // list of queue id that will be enable

    uvm_split_string(queue_en, "," , queue_en_s);  // split the list with "," as separator
    foreach (queue_en_s[i]) begin
       queue_en[i] = queue_en_s[i].atoi();  // string to int
    end
 end
{% endhighlight %}


#### multiple plusargs with the same name, but different values
Assuming we need a plusarg with this format to configure one aes encryption operation : `+AES_OPR_CFG=<AES_MODE>,<KEY_LEN>`.

We also need to have multiple aes encryption operations in 1 test, each receiving plusarg will corresponding to 1 operation.

In this example, we will use the `uvm_cmdline_proc.get_arg_values()` function instead of `uvm_cmdline_proc.get_arg_value()`.

{% highlight verilog %}
 //
 // Example simulator command line argument: +AES_OBJ_CFG=AES_CBC,128 +AES_OBJ_CFG=AES_ECB,256
 // Expect 2 aes operations:
 //               1st aes operation: aes mode is AES_CBC, key length 128
 //               2nd aes operation: aes mode is AES_ECB, key length 256
 //

 string aes_opr_lst[$];

 if (uvm_cmdline_proc.get_arg_values("+AES_OPR_CFG=", aes_opr_lst)) begin
    // the aes_opr_lst will contain 2 entries:
    //  "AES_CBC,1" and "AES_ECB,256"

    foreach(aes_opr_lst[i]) begin
       aes_mode_e m_aes_mode;
       string     tmp_q[$];

       uvm_split_string(aes_opr_lst[i], ",", tmp_q);
       // tmp_q queue will have 2 entries, 
       // 1st entry is the string of AES_MODE
       // 2nd entry is the string of key length
       // now we need to cast these entries to aes_mode_e and int type respectively

       if (uvm_enum_wrapper#(aes_mode_e)::from_name(tmp_q[0], m_aes_mode)) begin
          $display("AES mode %s", m_aes_mode.name()));
       end 
       $display("Key length %d", tmp_q[1].atoi());
    end 
 end
{% endhighlight %}

---
## Finding more information
1. Systemverilog LRM, section 21.6 Command line input
1. [uvm_cmdline_processor](https://verificationacademy.com/verification-methodology-reference/uvm/docs_1.2/html/files/base/uvm_cmdline_processor-svh.html)


