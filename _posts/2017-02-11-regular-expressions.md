---
author: kimmy
layout: post
title:  "Regular expressions"
date:   2017-02-11 19:03:03 +08:00
categories: Python
tags:
- Python
- Notes
- Trivia
- Regex
---

* content
{:toc}



## `str` Versus `bytes` in Regular Expressions


```python
# reference: Fluent Python Chapter 4

import re

re_numbers_str = re.compile(r'\d+')
re_words_str = re.compile(r'\w+')
re_numbers_bytes = re.compile(rb'\d+')
re_words_bytes = re.compile(rb'\w+')

text_str = ("Ramanujan saw \u0be7\u0bed\u0be8\u0bef"  #1
            " as 1729 = 1³ + 12³ = 9³ + 10³.")
text_bytes = text_str.encode('utf-8')

print('Text', repr(text_str), sep='\n ')
print('Numbers')
print(' str :', re_numbers_str.findall(text_str))
print(' bytes:', re_numbers_bytes.findall(text_bytes))
print('Words')
print(' str :', re_words_str.findall(text_str))
print(' bytes:', re_words_bytes.findall(text_bytes))
```

    Text
     'Ramanujan saw ௧௭௨௯ as 1729 = 1³ + 12³ = 9³ + 10³.'
    Numbers
     str : ['௧௭௨௯', '1729', '1', '12', '9', '10']
     bytes: [b'1729', b'1', b'12', b'9', b'10']
    Words
     str : ['Ramanujan', 'saw', '௧௭௨௯', 'as', '1729', '1³', '12³', '9³', '10³']
     bytes: [b'Ramanujan', b'saw', b'as', b'1729', b'1', b'12', b'9', b'10']


### KEYNOTE
---
1. This string is joined to the previous one at compile time.
2. you can use regular expressions
    on `str` and `bytes`, but in the second case bytes outside of the **ASCII** range are treated
    as **nondigits** and **nonword** characters
