---
layout: default
title: std::randomize examples
parent: Systemverilog Randoms
grand_parent: My Randoms
description: std::randomize examples
tag: systemverilog
comments: true
toc_en: true
nav_order: 1
---
# std::randomize examples
Sometimes an example is just enough.
Honestly, I do search and copy a lot. It is simply because I cannot remember everything.
Sometimes I pay a lot of time to understand and get the code run perfectly and then completely forget about it after 1 year.
This post is to store some of Systemverilog std::randomize examples that I created/collected, just for me to copy later. :D
{: .fs-5 .fw-500 }
---

<div>  
<input type="text" class="tablefilterinput" id="FilterInput" onkeyup="tablefilter()" placeholder="Table filter input..." title="filter input">

<table id="myTable" >
   <tr>
      <th> Description </th>
      <th> Code </th>
      <th> Link </th>
   </tr>
   <tr>
      <td> systemverilog std::randomize array with constrains on unique element, number of elements, array sum, constrains for each element value using foreach</td>
      <td>
      <div class="code">
      {% highlight verilog %}
  int d_array[]; // also works with queue d_array[$]

  std::randomize(d_array) with {                                
    unique {d_array};
    d_array.size() == 10;
    d_array.sum    == 150;
    foreach (d_array[i]) {
      d_array[i] < 100;
      d_array[i] > 1;
      d_array[i] != i;
    } };
      {% endhighlight %}
      </div>
      </td>
      <td>
      <a href="https://www.edaplayground.com/x/5nYm" title="std::randomize example for array/queue">
      <svg width="25" height="25" viewBox="0 -0.1 2 2" class="customsvg"> <use xlink:href="#svg-edaplay"></use></svg></a>
      </td>
   </tr>

   <tr>
      <td> systemverilog std::randomize array using sum with</td>
      <td>
      <div class="code">
      {% highlight verilog %}
    int d_array[]; //also work with queue, try d_array[$]

    std::randomize(d_array) with {
      d_array.size() == 5;
      d_array.sum() with ( item > 10 ? item:0)  == 100;
      foreach (d_array[i]) {
        d_array[i] inside {[0:100]};
      }
    };
      {% endhighlight %}
      </div>
      </td>
      <td>
      <a href="https://www.edaplayground.com/x/5css" title="std::randomize example for array using sum with, and also inside">
      <svg width="25" height="25" viewBox="0 -0.1 2 2" class="customsvg"> <use xlink:href="#svg-edaplay"></use></svg></a>
      </td>
   </tr>

   <tr>
      <td> systemverilog std::randomize using implication constraints</td>
      <td>
      <div class="code">
      {% highlight verilog %}
    int d_array[]; 

    std::randomize(d_array) with {
      d_array.size() == 3;
      d_array[0] > 10 -> d_array[1] inside {0,1,2};
      d_array[0] < 10 -> d_array[1] inside {3,4,5}
                         & d_array[2] inside {6,7,8};
    };     
      {% endhighlight %}
      </div>
      </td>
      <td>
      <a href="https://www.edaplayground.com/x/39Hg" title="randomize example using implication constraint">
      <svg width="25" height="25" viewBox="0 -0.1 2 2" class="customsvg"> <use xlink:href="#svg-edaplay"></use></svg></a>
      </td>
   </tr>

   <tr>
      <td> systemverilog std::randomize using condition constraints</td>
      <td>
      <div class="code">
      {% highlight verilog %}
    int d_array[]; 

    std::randomize(d_array) with {
      d_array.size() == 3;
      d_array[0] inside {[0:15]};
      if (d_array[0] > 10) {
        d_array[1] inside {0,1,2};
        d_array[2] inside {10,11,12};
      }
      else if (d_array[0] > 5) {
        d_array[1] inside {3,4,5};
        d_array[2] inside {6,7,8};
      }
      else {
        d_array[1] inside {13,14,15};
        d_array[2] inside {16,17,18};
      }
    };
      {% endhighlight %}
      </div>
      </td>
      <td>
      <a href="https://www.edaplayground.com/x/dQx" title="randomize example using condition constraint">
      <svg width="25" height="25" viewBox="0 -0.1 2 2" class="customsvg"> <use xlink:href="#svg-edaplay"></use></svg></a>
      </td>
   </tr>

   <tr>
      <td> systemverilog std::randomize with multiple variables</td>
      <td>
      <div class="code">
      {% highlight verilog %}
  int d_array[]; 
  int num;

  std::randomize(d_array, num) with {
    num inside {[0:100]};
    d_array.size()  == 3;
    d_array.sum(item) with ( item > 10? item:0) == num;
    foreach (d_array[i]){
      d_array[i] inside {[0:100]};
    }
  };     
      {% endhighlight %}
      </div>
      </td>
      <td>
      <a href="https://www.edaplayground.com/x/f5g" title="std::randomize example with multiple variables">
      <svg width="25" height="25" viewBox="0 -0.1 2 2" class="customsvg"> <use xlink:href="#svg-edaplay"></use></svg></a>
      </td>
   </tr>


   <tr>
      <td> systemverilog std::randomize with weight distribution</td>
      <td>
      <div class="code">
      {% highlight verilog %}
  int d_array[]; 

  std::randomize(d_array) with {
    d_array.size()  == 1000;
    foreach (d_array[i]){
      d_array[i] dist { [0:50]:/80, [51:99]:/20 };
    }
  }; // 80% d_array element value will be in range [0:50]
      {% endhighlight %}
      </div>
      </td>
      <td>
      <a href="https://www.edaplayground.com/x/5HdN" title="std::randomize example with weight distribution">
      <svg width="25" height="25" viewBox="0 -0.1 2 2" class="customsvg"> <use xlink:href="#svg-edaplay"></use></svg></a>
      </td>
   </tr>


   <tr>
      <td> systemverilog std::randomize with solve .. before </td>
      <td>
      <div class="code">
      {% highlight verilog %}
  int d_array[]; 
  int num;

  std::randomize(d_array, num) with {
    solve num before d_array;
    num inside {[0:100]};
    d_array.size()  == 3;
    d_array.sum(item) with ( item > 10? item:0) == num;
    foreach (d_array[i]){
      d_array[i] inside {[0:100]};
    }
 };     
      {% endhighlight %}
      </div>
      </td>
      <td>
      <a href="https://www.edaplayground.com/x/3fJD" title="std::randomize example with solve before">
      <svg width="25" height="25" viewBox="0 -0.1 2 2" class="customsvg"> <use xlink:href="#svg-edaplay"></use></svg></a>
      </td>
   </tr>



   <tr>
      <td> systemverilog std::randomize with multi-dimensional array, constraint sum of all element, size for each dimension </td>
      <td>
      <div class="code">
      {% highlight verilog %}
  int d_array [][]; 

  std::randomize(d_array) with {
    d_array.sum() with (item.sum() with (item)) == 1000;  // total value of the array

    d_array.size() == 10; 

    foreach (d_array[i]) {
      d_array[i].size() inside {[2:10]};
    }

    foreach (d_array[i,j]) {
      d_array[i][j] inside {[0:100]};
    }

  };
      {% endhighlight %}
      </div>
      </td>
      <td>
      <a href="https://www.edaplayground.com/x/7XLu" title="std::randomize example with multi-dimensional array">
      <svg width="25" height="25" viewBox="0 -0.1 2 2" class="customsvg"> <use xlink:href="#svg-edaplay"></use></svg></a>
      </td>
   </tr>


   <tr>
      <td> systemverilog std::randomize with multi-dimensional array, 2nd example </td>
      <td>
      <div class="code">
      {% highlight verilog %}
    int d_array [5][]; 
  
    std::randomize(d_array) with {
          unique {d_array};
          foreach (d_array[i]) {
                  d_array[i].size() inside {[1:2]};
          }
          foreach (d_array[i,j]) {
                   d_array[i][j] inside {[0:16]};
          }
    };
      {% endhighlight %}
      </div>
      </td>
      <td>
      <a href="https://www.edaplayground.com/x/D4wU" title="std::randomize example with multi-dimensional array">
      <svg width="25" height="25" viewBox="0 -0.1 2 2" class="customsvg"> <use xlink:href="#svg-edaplay"></use></svg></a>
      </td>
   </tr>


   <tr>
      <td> systemverilog std::randomize with associative array, using enum as index </td>
      <td>
      <div class="code">
      {% highlight verilog %}

  typedef enum {red=1, green, blue, pink, yellow} color_e;
  int d_array[color_e] = '{red:10, green:20, blue:30, pink:40, yellow:50};
  //must construct the array element before randomizing.
  
  function void display();
    std::randomize(d_array) with {
      unique{d_array};
      foreach (d_array[i]) {
        d_array[i] < 50;
        d_array[i] > 0;
      }
    };
      {% endhighlight %}
      </div>
      </td>
      <td>
      <a href="https://www.edaplayground.com/x/mPVS" title="std::randomize example with associative array, using enum as index">
      <svg width="25" height="25" viewBox="0 -0.1 2 2" class="customsvg"> <use xlink:href="#svg-edaplay"></use></svg></a>
      </td>
   </tr>

   <tr>
      <td> systemverilog std::randomize with enum variable</td>
      <td>
      <div class="code">
      {% highlight verilog %}
  typedef enum {RED=11, GREEN=22, BLUE=33, PINK=44, YELLOW=55} color_e;
  color_e m_color; 
  color_e m_2nd_color;
  
  function void display();
    std::randomize(m_2nd_color, m_color) with {
      m_color     inside {RED,GREEN, YELLOW};

      // redundant code, no need to declare this below constraint
      // the constraint solver only selects the value inside the set of enum labels
      m_2nd_color inside {m_2nd_color};   
    };
              
      {% endhighlight %}
      </div>
      </td>
      <td>
      <a href="https://www.edaplayground.com/x/9SLg" title="std::randomize example with enum variable">
      <svg width="25" height="25" viewBox="0 -0.1 2 2" class="customsvg"> <use xlink:href="#svg-edaplay"></use></svg></a>
      </td>
   </tr>



   <tr>
      <td> systemverilog std::randomize with 2 variable of 2 difference objects </td>
      <td>
      <div class="code">
      {% highlight verilog %}
  Packet pkt1;
  Packet pkt2;
  
  initial begin
    pkt1 = new();
    pkt2 = new();
    
    std::randomize(pkt1.count, pkt2.count) with {
      pkt1.count >0;
      pkt2.count >0;
      (pkt1.count + pkt2.count) == 100;
    };
  end 

      {% endhighlight %}
      </div>
      </td>
      <td>
      <a href="https://www.edaplayground.com/x/UPkP" title="std::randomize with variable of 2 different obj">
      <svg width="25" height="25" viewBox="0 -0.1 2 2" class="customsvg"> <use xlink:href="#svg-edaplay"></use></svg></a>
      </td>
   </tr>


   <tr>
      <td> systemverilog std::randomize with this and local:: </td>
      <td>
      <div class="code">
      {% highlight verilog %}
TDB
    
TDB

      {% endhighlight %}
      </div>
      </td>
      <td>
      <a href="https://www.edaplayground.com/x/UPkP" title="std::randomize with variable of 2 different obj">
      <svg width="25" height="25" viewBox="0 -0.1 2 2" class="customsvg"> <use xlink:href="#svg-edaplay"></use></svg></a>
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
