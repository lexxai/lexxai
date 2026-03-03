---
layout: post
title: "Prism. Code formatting in blogspot."
date: 2024-10-31 15:33:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2024/10/prism-code-formating-in-blogspot.html
---

[Prism](https://prismjs.com/#basic-usage)

[![](/assets/images/blog/%D0%97%D0%BD%D1%96%D0%BC%D0%BE%D0%BA%20%D0%B5%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-10-31%20%D0%BE%2017.36.47.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiJkh_MJ5D-2GGnYGSRgKlL4FpSqZYET0gHq8hmkSdnUorcIXUQNsUncBjl-X-zikGyegNlOXxlYMPmD6zF4oH7ovof3bvI2uouA8OvrPlt_gPqwUYi8Q6qXuUFll_iCGE12pcaC-QEZd6egsKiLKHh0URnAFQ2yCAD2_jdN2hN8j8LYkxkktcJsi2N4QIA/s1944/%D0%97%D0%BD%D1%96%D0%BC%D0%BE%D0%BA%20%D0%B5%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-10-31%20%D0%BE%2017.36.47.png)

### Example of use Prism:

```
# Your Python code goes here

from types import MethodType

class Person:
    def __init__(self, name):
        self.name = name

# Function to add dynamically as a method
def say_hello(self):
    return f"Hello, my name is {self.name}"

# Create an instance of Person
person = Person("Alice")# Bind the function say_hello to person instance as a method
person.say_hello = MethodType(say_hello, person)

# Call the new method on the instance
print(person.say_hello())  # Output: "Hello, my name is Alice"
```

[cdnjs links](https://cdnjs.com/libraries/prism)

```
https://cdnjs.cloudflare.com/ajax/libs/prism/9000.0.1/themes/prism-tomorrow.min.css
https://cdnjs.cloudflare.com/ajax/libs/prism/9000.0.1/themes/prism.min.css
https://cdnjs.cloudflare.com/ajax/libs/prism/9000.0.1/components/prism-python.min.js
```
