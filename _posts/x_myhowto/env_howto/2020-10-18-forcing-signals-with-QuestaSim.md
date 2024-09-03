---
layout: default
title: Forcing Signals with Questasim/ModelSim
parent: Environment How to
grand_parent: My How to
description: Some commands to forcing signals with Questa Simulator (or ModelSim) EDA tool
comments: true
tags: [linux_env,eda_tool]
nav_order: 1
toc_en: true

---
# Forcing Signals with Questasim/ModelSim
I've been using vcs and questasim for verification for a while, and both of these tools allow us to interact with the simulator using their cli.
We can use them for debugging, create checking point, restore simulation, ... .
But mostly for my case, I use this cli for forcing signals.
And this method help me in many situations such as creating some error cases (for testing purpose), or forcing signals from the logic that has not been implements yet.
Let's go through some examples for forcing signals using Questasim (or another version of this eda tool is ModelSim).
{: .fs-5 .fw-500 }

---
## How to force signals
As mentioned, we force signals using the interactive mode of those EDA tool, so whenever we in the command line interface (aka cli) we can call the force commands to force signals.
Usually, I create a file containing forcing commands, then source this file during simulation like below.

<div class ="code" markdown="1" >
{% highlight bash %}
  vsim -do <force_file.tcl> <other simulation options>
{% endhighlight %}
</div>

For Questasim and VCS, these commands is the Tcl-based command, which means you will use tcl syntax for writing your force file.

---
## Some examples of forcing signals using Questasim
### First up
* Make sure your signals have correct path, you can find signal path by using Questasim cli command in cli mode:`find nets -recursive -ports /top/*TOP*/*MODULE_A*/*RESET*`
* Check your tcl syntax carefully.
* Careful with radix format. Tcl will understand this radix format `0b11`, but for Questasim, we need to write data value with this format: `'b11`, `'hfaab`, `'d1024`. So check the EDA user guide carefully.
* If you use force file like this: `vsim -do <force_file.tcl>`, considering using below template
<div class ="code" markdown="1" >
{% highlight bash %}
  #force_file.tcl
  echo "FORCE BEGIN"
  #your force commands
  echo "FORCE END"
  run -all
{% endhighlight %}
</div>

### Forcing Examples
<div>
<input type="text" class="tablefilterinput" id="FilterInput" onkeyup="tablefilter()" placeholder="Table filter input..." title="filter input">

<table id="myTable" >
   <tr>
      <th> Description </th>
      <th> Code </th>
   </tr>

   <tr>
      <td>
      forcing signals with fix value from certain simulation time
      </td>
      <td>
      <div class="code">
      {% highlight tcl %}
  when { $now > 1000ns } {
    force {/top/U_MODULE_A/I_SIGNAL_A[1:0]} 'h3           
    force /top/U_MODULE_A/I_SIGNAL_B        'b1
  }
      {% endhighlight %}
This means:
From 1000ns of simulation time, I_SIGNAL_A[1:0] will be 2'b11, I_SIGNAL_B will be 1'b1
      </div>
      </td>
   </tr>

   <tr>
      <td>
      forcing signal to be similar to other signal
      </td>
      <td>
      <div class="code">
      {% highlight tcl %}
  when { /top/U_MODULE_A/I_CLK'event and $now > 1000ns } {
    force /top/U_MODULE_A/I_SIGNAL_C [ examine /top/U_MODULE_A/I_CLK ]
  }
      {% endhighlight %}
This means:
From 1000ns of simulation time, at every event of I_CLK (rising edge, falling edge), get the value of I_CLK (using examine command), and force the I_SIGNAL_C with that value
      </div>
      </td>
   </tr>


   <tr>
      <td>
      forcing signal when another signal got x value
      </td>
      <td>
      <div class="code">
      {% highlight tcl %}
  when { /top/U_MODULE_A/I_SIGNAL_A'event and $now > 1000ns } {
    set signal_value "[ examine /top/U_MODULE_A/I_SIGNAL_A ]"
    set signal_val_tolow "[ string tolower $signal_val_tolow ]"
    if { [ string compare $signal_val_tolow "1'hx" ] == 0 } {
       force /top/U_MODULE_A/I_SIGNAL_C 0
    }
  }
      {% endhighlight %}
This means:
From 1000ns of simulation time, when I_SIGNAL_A change to X value, force the I_SIGNAL_C to be 0
      </div>
      </td>
   </tr>

   <tr>
      <td>
      forcing signal to be a clock
      </td>
      <td>
      <div class="code">
      {% highlight tcl %}
  when { top/U_MODULE_A/I_RESET_N'event and top/U_MODULE_A/I_RESET_N == 'b1  } {
     force top/U_MODULE_A/I_CLK_B 1 , 0 0.5 ns -r 1.0 ns
  }
      {% endhighlight %}
This means:
When I_RESET_N changes value and the I_RESET_N has 1'b1 value, force the I_CLK_B to be 1, after 0.5ns force the I_CLK_B to be 0.
After 1ns from the moment I_CLK_B first forced to be 1, repeat the same sequence. Then we'll has I_CLK_B forced as a clock with 1ns clock cycle.
      </div>
      </td>
   </tr>

   <tr>
      <td>
      forcing signal when another signal active 1 the first time
      </td>
      <td>
      <div class="code">
      {% highlight tcl %}
  set flag_m 1
  when { /top/U_MODULE_A/I_SIGNAL_E'event and /top/U_MODULE_A/I_SIGNAL_E == 1} {
    if { $flag == 1  } {
      force /top/U_MODULE_A/I_SIGNAL_D 1
      set flag_m 0
    }
  }
      {% endhighlight %}
This means:
When I_SIGNAL_E change value to 1 the first time, force the I_SIGNAL_D to 1.
      </div>
      </td>
   </tr>

</table>

    <script>
      function tablefilter() {
        var input, filter, table, tr, td, i, txtValue;
        input = document.getElementById("FilterInput");
        filter = input.value.toUpperCase();
        table = document.getElementById("myTable");
        tr = table.getElementsByTagName("tr");
        for (i = 0; i < tr.length; i++) {
          td = tr[i].getElementsByTagName("td")[0];
          if (td) {
            txtValue = td.textContent || td.innerText;
            if (txtValue.toUpperCase().indexOf(filter) > -1) {
              tr[i].style.display = "";
            } else {
              tr[i].style.display = "none";
            }
          }
        }
      }
    </script>
</div>

---
## Finding more information
Read the User Manual released by Questasim for more information and example on how to force signals.
The manual files usually can be found at the installation directory of each tool.
Besides, you could also refer the ModelSim PE manual. It's a free download tool from Mentor for education purpose and the commands are similar to Questasim.
