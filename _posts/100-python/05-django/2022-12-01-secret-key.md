---
layout: post
title: "Django - How to keep the secret key?"
date: 2022-12-01 12:00:00 +0100
categories: python django
permalink: django/secret-key
---

# How to keep the secret key?
The secret key and database settings should be kept separate from the code project.


## Solution
- Create `local_settings.py` at the root of the project and add it to `.gitignore`.
- To import settings from `local_settings.py` 
you need to add this code at the very end of the `settings.py`

```python
try:
    from local_settings import *
except ImportError:
    pass
```

