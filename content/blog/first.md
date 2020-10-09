+++
categories = ["python", "interview"]
comments = true
date = "2020-10-02T15:59:13-04:00"
draft = false
showpagemeta = true
showcomments = true
slug = ""
tags = ["python", "interview", "data-structure"]
title = "Obvious Questions, Hidden Details"
description = "How to know if someone is good at Python"

+++

You want to know how good is someone at Python? You don't need to ask strange obscure questions. Just ask for simple questions which has some hidden technology details behind it.

Like : how do you do a membership test in python?!

Now everyone knows how to do a membership test! You just check if a collection of items contain a specific item.
Something like ;

```python
if number in list_of_numbers:
    # Do something
```
But what not everybody knows is the time complexity of membership testing.
lookup time in a list has O(n) time complexity. But lookup time in a set however has a constant O(1) time complexity.

So, the question is really simple to answer, but what answer you get back is entirely on how much knowledge someone has on Python! :)

For instance the difference between a membership test in a set vs a list can be `100 000` times faster! That could be huge!

##### Technical details
Sets are implemented using hash tables. So whenever you try to test if some value is in the set or not, 
it just needs to check whether the object is at the position determined by its hash.
