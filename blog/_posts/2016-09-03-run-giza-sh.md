---
#layout: post
title: Run GIZA faster with .sh parameter
description: >
  Tutorial.
author: author1
comments: true
---

Main Contributor: Nguyễn Ngọc Gia Hy

One of my reader sees that my running GIZA++ <a href="/blog/2016-01-27-install-giza-ubuntu/">method</a> is still slow and does not utilize all potential of .sh file. So he decided writes one himself and sent the instruction to me:

<ol> 
  <li>Download RunGIZA++.sh <a href="https://github.com/hovinh/giza">here</a> and put it in giza-pp (folder you got when downloading GIZA++ source code).</li>
  <li>Open Terminal, move to giza-pp folder.</li>
  <li>Type this line:</li>
</ol>

```powershell
$ sh RunGIZA++.sh <arg1> <arg2> <arg3> <arg4>
```
with:
  - \<arg1\>: link to folder containing corpus
  - \<arg2\>: source file’sname
  - \<arg3\>: target file’s name
  - \<arg4\>: link to folder containing result file
  
Example:
```powershell
sh RunGIZA++.sh '/home/vinh/Desktop/Corpus' Source Target '/home/vinh/Desktop/Corpus'
```

I have to confess, you don’t need to concern about long and confusing command lines anymore, only take a slip of coffee and wait for it to be done.

However. I also meet a few problems when try it the first time, hope you don’t make the same mistake:

- Give permission for RunGIZA++.sh file as **“Allow executing file as program“**
- Guarantee link to folder does exist.
- Name of 2 files Source and Target should not contain extension, or at least not the familiar one. If I name them as “Source.txt” and “Target.txt”, you will get an error like **“cannot open vocabulary files”**. On the other hand, if their names are “Source.en” và “Target.vn” or no extension, everything will be file.
- Name of file or folder in link should not contain space, or it can also lead to unexpected error.
I guess that GIZA automatically cut off familiar extension from file name and then get the remaining string to name output files. We can see this when looking at the Terminal.

Case 1:

```powershell
/home/vinh/Desktop/Try/Source.txt -> Source
/home/vinh/Desktop/Try/Target.txt -> Target
```

Case 2:
```powershell
/home/vinh/Desktop/Try/Source.en -> Source.en
/home/vinh/Desktop/Try/Target.vn -> Target.vn
```