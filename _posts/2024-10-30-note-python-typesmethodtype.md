---
layout: post
title: "Note. Python. types.MethodType"
date: 2024-10-30 14:51:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2024/10/note-python-typesmethodtype.html
---

From LinkedIn Learning.
[Pattern. Strategy](https://www.linkedin.com/learning/python-design-patterns-14304845/strategy-example): 

[![](/assets/images/blog/%D0%97%D0%BD%D1%96%D0%BC%D0%BE%D0%BA%20%D0%B5%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-10-30%20%D0%BE%2016.17.10.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg3tELycGTzk8Tn471cVw9r1QkHIKwzjgv3DCWd3xmp3BZvtbCZheRgnN0tqomndde1w4h7mQImc3lHRMdI4SXWshBWQWl13nY62QK6TWtGzmO72sSC1G1xBzADai7AVO0rD75NCsr8L91LL-blRXceY3X93iBw5pgbtCiET1xwrgVaoI6jnOYvw2WI5Dzd/s2880/%D0%97%D0%BD%D1%96%D0%BC%D0%BE%D0%BA%20%D0%B5%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-10-30%20%D0%BE%2016.17.10.png)

#### What it types.MethodType ?

types.MethodType in Python is a utility function from the types module that
allows you to bind a function as a method to an instance of a class. This can
be especially useful when you want to add or replace methods dynamically for
an object at runtime. Let's go over how it works and look at an example.

```
types.MethodType(function, instance)
```

* function: The function you want to bind to an instance.
* instance: The object (usually an instance of a class) to which you want to
  bind the function as a method.

#### When to Use types.MethodType

1. Adding new methods to existing instances: Useful in scenarios where you
   need to modify the behavior of a particular instance without changing the
   class itself.
2. Replacing methods dynamically: If you need to override or alter a method
   on a per-instance basis (e.g., for testing or specific instances only).
3. Creating flexible or extensible objects: This can help in creating objects
   that can adapt their behavior after they are instantiated.

#### Example Usage

Suppose you have a class Person but want to dynamically add a new method to
one specific instance:

### Explanation of Example

* We defined a regular function say\_hello outside the class Person.
* Using MethodType, we bind say\_hello to the person instance, making it a
  method for just that instance.
* After binding, we can call person.say\_hello() as if it was originally
  defined as part of the Person class, and it will have access to the self
  parameter (the person instance).

### Key Points

* MethodType does not add the method to the class but to the specific
  instance.
* The self argument works correctly because MethodType binds the function
  to the instance.

#### Advanced Example: Replacing an Existing Method

In this example, the original greet method is replaced only for person. If you
create another Person instance, it will still use the original greet
method.  
  
Using types.MethodType offers flexibility, especially for
cases where modifying the class itself isn’t desired or possible.

#### More info

* [types — Dynamic type creation and names for built-in types — Python
  3.13.0 documentation](https://docs.python.org/3/library/types.html)
* [types — Dynamic type creation and names for built-in types — Python
  3.13.0 documentation UA](https://docs.python.org/uk/3.13/library/types.html)
* [source code Lib/types.py](https://github.com/python/cpython/blob/3.13/Lib/types.py)
* [Python Examples of types.MethodType](https://www.programcreek.com/python/example/501/types.MethodType)
