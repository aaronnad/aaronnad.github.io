---
title: Hello Homelab
date: 2022-12-31 18:40:00 -0000
categories: [Homelab]
tags: [dell,test,portainer]
---

# Welcome

This is the first initial entry to the bullhosting site

## The second heading of the page

This might contain an unordered list

* this
* looks
* a
* little
* something
* like 
* this

### for heading 3 we might choose to do some code entries

```
look at this
```

but we could change this to be language specific

```python
import os, time

path = r"PathGoesHere"
now = time.time()

for filename in os.listdir(path):
    if os.path.getmtime(os.path.join(path, filename)) < now - 7 * 86400:
        if os.path.isfile(os.path.join(path, filename)):
            print(filename)
            os.remove(os.path.join(path, filename))
```
```javascript
console.log("Hellow World)
```

## This is how we add an image

![img-description](/assets/img/italy/IMG_4495.JPG)

_Italy Photo example_

## This is what happens when i update the site

I didn't see anything update??