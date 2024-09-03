---
layout: post
title: A Design Verification Engineer Blog
nav_order: 0
description: "I'm a design verification engineer and this is where I keep my notes, my thoughts, my experiences or anything interesting that I think should belong here."
comments: false
share: false
permalink: /
---

# A Design Verification Engineer Blog
I'm a design verification engineer and this is where I keep my notes, my thoughts, my experiences or anything interesting that I think should belong here. :D
{: .fs-5 .fw-500 }

---
## Who am I
I'm a design verification engineer and I've been in this industry for quite a while. This dvtalk is my personal blog, where I'll keep notes of what I know, and also problems big and small which I faced. My main task is verification so I work with systemverilog, uvm everyday and these are what I know the most. However, now and then, I also find myself struggling with formal verification, design, and also scripting. Talking about scripting, I have been using perl for a long time before switching to python recently. And also I'm using vim with spaces. :D

---
## What you'll find here
* You'll find my randoms.
Let's just say my randoms are just a bit better organized piece of my notes since I note a lot. This will be where I keep them. They will be about systemverilog or uvm or vim. Just some random interesting things that I really don't now where to post.
* You'll find my how to.
These will be guides to do specific things. Mostly they will be about systemverilog and uvm and vim also, I guess.



---
OK, that's it mates.
Use the search box if you need anything.
And if you cannot find what you need, well, later then. :D

---
Posts:
<ul>
{% for post in site.posts %}
  {% if post.comments == true %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endif %}
{% endfor %}
</ul>

---
{% capture temptags %}
  {% for tag in site.tags %}
    {{ tag[1].size | plus: 1000 }}#{{ tag[0] }}#{{ tag[1].size }}
  {% endfor %}
{% endcapture %}
{% assign sortedtemptags = temptags | split:' ' | sort | reverse %}
<nobr>
{% for temptag in sortedtemptags %}
  {% assign tagitems = temptag | split: '#' %}
  {% capture tagname %}{{ tagitems[1] }}{% endcapture %}
  <a href="/tag/{{ tagname }}" class="btn btn-dawn mr-2">{{ tagname }}</a>
{% endfor %}
</nobr>
