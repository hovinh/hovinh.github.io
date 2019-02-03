---
#layout: post
title: Install MGIZA on Ubuntu
description: >
  Tutorial.
author: author1
comments: true
---

MGIZA stands for Multi-threaded GIZA, show up the faster ability in compare to  GIZA by using parallel computing advantage. Installing steps in MGIZA is simpler than GIZA, most of them due to helpful config file of referent author.

## Preparation

```powershell"]
$ sudo apt-get install cmake
$ sudo apt-get install libboost-all-dev
```

## Installing

```powershell
$ git clone https://github.com/moses-smt/mgiza.git
$ cd mgiza/mgizapp
$ cmake .
$ make
$ make install
```

Go to folder "manual-compile", open file compile.sh and edit: SRC_DIR (link to  source code file of MGIZA), BOOST_ROOT(Private folder for installing boost - Check the Note section below), BOOST_INCLUDE, BOOST_LIBRARYDIR. For example:

```powershell
SRC_DIR = /home/phdlab/Desktop/HoXuanVinh/Tool/mgiza/mgizapp/src
BOOST_ROOT = /home/phdlab/boost_1_60_0
BOOST_INCLUDE = $BOOST_ROOT/include
BOOST_LIBRARYDIR = $BOOST_ROOT/lib64
```

Start to compile

```powershell
$ manual-compile/compile.sh
```

## Execution

In order to run MGIZA, we need:

- All files in folder mgizapp/bin
- File merge_alignment.py( in "mgizapp/inst/scripts")

Copy all files into 1 same folder(My folder is "/home/phdlab/Tool/Mgiza/bin"). For this demo, I copy my 2 corpus "data.en" and "data.vn" also to this folder. Open Terminal and move to that folder with "cd" command.

The following steps wotk with assumption that 2 input data named "data.en" and "data.vn", you need to change a little bit to fix your own data.

```powershell
./mkcls -n10 -pdata.en -Vdata.en.vcb.classes
./mkcls -n10 -pdata.vn -Vdata.vn.vcb.classes
./plain2snt data.en data.vn
./snt2cooc data.en_data.vn.cooc data.en.vcb data.vn.vcb data.en_data.vn.snt
```

Download file "configfile" at <a href="https://pastebin.com/b1ksHtUy">here</a> and save in same folder.

You can choose number of threads for MGIZA by change value of variable "ncpus". For example, your computer has  4 cores then double it and type "ncpus 8". You need to edit  "corp.tok.low.src" into "data.en" and "corp.tok.low.trg" into "data.vn". Save it.

Now type this line:

```powershell
$ ./mgiza configfile
```

GIZA save result into file named "A3.final",  to Mgiza, the more threads you choose, the more files you have, yet the name stays the same. Your work is merging them into 1 file.

>**Note**:
- If file's size is too small, maybe the program will crash, then solution is moving to GIZA++.
- In the last line of installing section, you need Boost to compile.  Do the following steps:Boost has version could crash if compile. I suggest version 1.60. The decision is  yours, but if compile unsuccessfule, the high risk might be incompatible version of Boost. The purpose of using many lines  for boost is to install in private folder, not in system folder. So you don't need superuser authority to run.
```powershell
$wget http://downloads.sourceforge.net/project/boost/boost/1.60.0/boost_1_60_0.tar.gz
$tar zxvf boost_1_60_0.tar.gz
$cd boost_1_60_0/
$./bootstrap.sh
$./b2 -j4 --prefix=$PWD --libdir=$PWD/lib64 --layout=system link=static install || echo FAILURE
```
After running, if it says SUCCESS then OK, otherwise FAILURE. However, if it is FAILURE, you just keep doing the next step to see if something goes wrong. If so, come back to reinstall Boost with different version :))
- If there is any error in running commands in Terminal, try superuser authority with keyword "sudo".
Reference:
{:.lead}

<ol>
  <li>http://www.statmt.org/moses/?n=Moses.ExternalTools#ntoc3</li>
  <li>https://fabioticconi.wordpress.com/2011/01/17/how-to-do-a-word-alignment-with-giza-or-mgiza-from-parallel-corpus/</li>
  <li>http://www.statmt.org/moses/?n=Development.GetStarted</li>
</ol>