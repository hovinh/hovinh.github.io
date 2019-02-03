---
#layout: post
title: Install SRILM on Windows
description: >
  Tutorial.
author: author1
comments: true
---

SRILM is popular toolkit in training n-gram language model. There are 2 operating systems I have tried:
- Linux: Check <a href="/blog/2016-04-22-install-srilm-ubuntu">here</a>.
- Window: using cyqwin to install after 4 to 5 attemps. OS: Window 10 .

This post will help you install cyqwin (Linux simulation) to run SRILM on Window. These are requirement materials:
<ol>
  <li>Download <a href="http://www.speech.sri.com/projects/srilm/download.html">SRILM</a>.</li>
  <li>Download <a href="https://cygwin.com/install.html">Cyqwin</a>, careful on choosing  32 or 64 bit.</li>
  <li>Disk C should have 1GB free (just in case, it only consumes about 500MB)</li>
  <li>Disk D/E with 1GB free( where all packages of cyqwin downloaded)</li>
</ol>

By default, I choose Dish D is where I download Cyqwin (32 bit) and its packages. You will see file named setup-x86.exe. This is the one to help you install cyqwin. In the future, if you want to install other packages, just run it and do the samething.

## Download Cyqwin and Packages

- Choose a download source: Install from Internet.
- Root Directory, by default is C:/cyqwin. You can change the address if you want.
- Local Package Directory: where contains downloaded package. I choose disk D.
- Select Your Internet Connection: Direct Connection.
- Choose a download site: each site will have different default packages. I recommend http://mirrors.kernel.org.
- Select Packages: You need to choose the following packages: binutils, bzip2, cyqwin, cyqwin-devel, gawk, gcc-core, gcc-g++, gzip, libargp, libcharset1, libiconv, libiconv-devel, libiconv2, make, tar, tcl, tcl-devel, tcl-tk, tcl-tk-devel, tcsh, xz, zlib-devel, zib0. If they are choosed, the version name will show up.
- Resolve Dependencies: If the package you chose need to install other packages so it can run, this window will show up. Make sure you tick on "Select required packages(RECOMMEND)".
- Get 1 book to read and wait for them all downloaded.
- Choose create shortcut icon.

## Setup Cyqwin

- Open Cygwin Terminal. It will generate '.bash_profile', '.bashrc', '.inputrc'  in dir 'C:\cyqwin\home\yourname\'
- Type these commands:
```powershell
$ cd /
$ mkdir srilm  // create a folder named 'srilm'
$ cd srilm
```

- Copy file 'srilm-1.7.1.tar.gz' into 'C:\cyqwin\srilm'. Extracting it with:
```powershell
$ tar zxvf srilm-1.7.1.tar.gz
```
- In 'C:\cyqwin\home\yourname\.bashrc', add these line on the top of the file and save:
```powershell
export SRILM=/srilm
export MACHINE_TYPE=cyqwin
export PATH=$PATH:$pwd:$SRILM/bin/cygwin
export MANPATH=$MANPATH:$SRILM/man
```

- Restart Cyqwin.
In the official instruction, when downloading package, Cyqwin automatically install them and you can jump to step 4. If you didn't make it, come back to step 3.

## Install packages manually:

In the folder containing downloaded packages, there is a folder name somehow look like this: 'http%3a%2f%2fmirrors.kernel.org%2fsourceware%2fcygwin%2f', click on and find the packages have the name below. A small note here: you may find your file have diffrent version but don't worry about it, these are only the latest version at the moment I write this post:

- binutils-2.25-4.tar.xz
- gawk-4.1.3-1.tar.xz
- gcc-core-4.9.3-1.tar.xz
- gcc-g++-4.9.3-1.tar.xz
- gzip-1.6-1.tar.xz
- make-4.1-1.tar.xz
- tcl-8.5.18-1.tar.xz
- tcl-devel-8.5.18-1.tar.xz
- tcl-tk-8.5.18-1.tar.xz
- tcl-tk-devel-8.5.18-1.tar.xz
- tcsh-6.19.00-2.tar.xz

Open Cyqwin and do this for each file:

```powershell
$ cd \
$ tar xvf binutils-2.25-4.tar.xz
```

Restart Cyqwin.

## Install SRILM

In 'C:\cyqwin\srilm', open 'Makefile', add:
```powershell
SRILM = /srilm
```

In Cyqwin, move to srilm and install it:

```powershell
$ cd /srilm
$ make World
$ make all
$ make cleanest
```

In the line 'make World', if there is a bug show up, be patient and check the notification, the error usually lie in line 5 or 6 counted from below, with format like  'Command not found' or 'tcl.h do not exist'. Google it name to find the packages you should install and run setup file in step 2, try it again. If not, come to step 3.

Change cyqwin's maximum memory: Type this command in Cyqwin:

```powershell
$ regtool -i set /HKLM/Software/Cygnus\ Solutions/Cygwin/heap_chunk_in_mb 2048
```

## Execute SRILM

You open this <a href="https://github.com/hovinh/srilm">link</a> to download corpus and relating code. I test on English language, if you want to do it with Chinese for example, check the file in Reference part.

Copy 2 file corpus.txt and vocab.txt(list of words in corpus) into 'C:\cyqwin\srilm'.

```powershell
$ ngram-count -vocab vocab.txt -text corpus.txt -order 3 -write count.txt -unk
$ ngram-count -vocab vocab.txt -read count.txt -order 3 -lm lm.lm -gt1min 3 -gt1max 7 -gt2min 3 -gt2max 7 -gt3min 3 -gt3max 7
```
Open lm.lm and check it.

## Reference:
<ol>
  <li>http://www.cs.brandeis.edu/~cs114/CS114_docs/SRILM_Tutorial_20080512.pdf</li>
  <li>A lot of FAQ of StackOverFlow, Cyqwin, Google, AskUbuntu, Youtube.</li>
</ol>

