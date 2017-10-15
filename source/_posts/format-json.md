title: format_json
tags:
  - vim
  - json
date: 2017-10-15 11:35:45
---

Put the following snip to your vimrc, whenever you open a JSON file, you can format it with hotkey fj in normal mode.

```
function FmtJSON(...) 
  let code="\"
        \ var i = process.stdin, d = '';
        \ i.resume();
        \ i.setEncoding('utf8');
        \ i.on('data', function(data) { d += data; });
        \ i.on('end', function() {
        \     try {
        \       JSON.parse(d) 
        \     } catch(e) { return console.log(d); }
        \     console.log(JSON.stringify(JSON.parse(d), null, 
        \ " . (a:0 ? a:1 ? a:1 : 2 : 2) . "));
        \ });\""
  execute "%! node -e " . code 
endfunction

nmap fj :<C-U>call FmtJSON(v:count)<CR>
```
The main part of function FmtJSON is copied from somewhere, but if the JSON file is malformed you probably override the original file unexpectedly, so I added try-catch in the end event handler.
