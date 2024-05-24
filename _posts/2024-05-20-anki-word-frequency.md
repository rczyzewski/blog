---
title: 'Adding a required new line characters'
subtitle: "editing test files"
author:
  - rczmdp
tags: [editing, vim]
description: |
 Programming is all about editing text files, so let's master this skill.   
---
Once upon a time I have read that programing is just about editing a text files.
For sure there is much more for development than just editing text files, 
but from other perspective it's also a part of the process. 
Data from the internet that I wanted to use, were coming in an incorrect format: there was missing 5k new line characters.

Original format:
```1234 abc 1211 cda 1120 rpm```
What I needed to enrich a script, was a different format of the file:
```
1234 abc 
1211 cda 
1120 rpm
```

## intellij

To avoid doing this manually, I have been trying to use IntelliJ `Replace in Files` which supports multi-line replacements.
So all I needed to do, to change the format of the file, was to write a basic regular expression.   
```regexp
(\d\d*) 
```
When transaforming a file, it should be preceed with a new line character, followed by ecnountered value - thats why `$1` is used.
![PyCharm - Replace in Files](/assets/posts/multiline_replacement.png)


## shell
The very same thing can be done with an old good `sed` shell command:
```shell
rcz@rcz % cat abc.txt
1234 abc 1211 cda 1120 rpm
rcz@rcz % cat abc.txt | sed -r -e 's/([0-9]+)/\n\1/g'  | egrep -v '^\w*$'
1234 abc 
1211 cda 
1120 rpm
```

After a short thought, the same can be done without the following grep, which has been used to remove the first empty line:
```shell 
rcz@rcz % cat abc.txt | sed -r -e 's/([0-9]+) ([^0-9]*)/\1 \2\n/g'
1234 abc
1211 cda
1120 rpm
```

## macros in Vim
A different way of doing the same thing, is to use macros, available in vim. 
```vim
qqwwi\nq
// qq - start recording a macro with the name q
ww - move to the begining of the next word
i - insert new line character
q - stop recording macro
10000@q   - execute the macro 10000 times
```



