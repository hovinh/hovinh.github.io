---
#layout: post
title: Install SRILM on Ubuntu
description: >
  Tutorial.
author: author1
comments: true
---

Installing SRILM on Ubuntu is much simpler than on Windows.

- Download the  latest version of <a href="http://www.speech.sri.com/projects/srilm/download.html">SRILM</a> (current version is srilm-1.7.1), move downloaded file to "/Home".
- Open Terminal, type below commands (default directory is  "/usr/share/srilm", in case you want to change, then replace it with equivalent link):
```powershell
$ mkdir /usr/share/srilm
$ mv srilm-1.7.1.tar.gz /usr/share/srilm/
$ cd /usr/share/srilm
$ tar xvf srilm-1.7.1.tar.gz
```
- Open Makefile file
```powershell
$ sudo gedit Makefile
```

- Change Makefile: In line 7, you look for line with content looks like  "# SRILM = /home/speech/stolcke/project/srilm/devel". Remove "#" and replace with this line:
```powershell
$ SRILM = /usr/share/srilm
```

- Save and close file. Go back to  Terminal, use superuser permission. If you meet error "tcsh: command not found" then type "sudo apt-get install tcsh" before try it again.If yours is  Ubuntu 32 bit:
```powershell
$ sudo tcsh
$ sudo make NO_TCL=1 MACHINE_TYPE=i686-gcc4 World
$ sudo ./bin/i686-gcc4/ngram-count -help
```

- Or if it is Ubuntu 64 bit:
```powershell
$ sudo tcsh
$ sudo make NO_TCL=1 MACHINE_TYPE=i686-m64 World
$ sudo ./bin/i686-m64/ngram-count -help
```

## Execute SRILM:

Access this <a href="https://github.com/hovinh/srilm">link</a> to download related corpus and code. Copy 2 file corpus.txt and vocab.txt(vocabulary of words in corpus) into '\usr\share\srilm\bin\i686-gcc4'(for 32 bit) or "\usr\share\srilm\i686-m64"(for 64 bit). Manual copy might do not work, you could use Terminal, go to folder containing those files and type:
```powershell
$ sudo cp vocab.txt '/usr/share/srilm/bin/i686-m64'
$ sudo cp corpus.txt '/usr/share/srilm/bin/i686-m64'
``` 
Now move to above folder and run the program
```powershell
$ cd '/usr/share/srilm/bin/i686-m64'
$ sudo ./ngram-count -vocab vocab.txt -text corpus.txt -order 3 -write count.txt -unk
$ sudo ./ngram-count -vocab vocab.txt -read count.txt -order 3 -lm lm.lm -gt1min 3 -gt1max 7 - gt2min 3 - gt2max 7 -gt3min 3 - gt3max 7
```
If you do it correctly, you will get 2 new files named "count.txt" and "lm.lm". Have a look at them to see what had happened :))

>**Note**:
In command lines, you might get in trouble with errors such  as "Permission denied", then you should add "sudo" ahead, which gives you superuser permission. For example if line 4 gets error, you should change it to "sudo tar xvf srilm-1.7.1.tar.gz".
{:.lead}

## Reference:

<ol>
  <li>http://www.cs.brandeis.edu/~cs114/CS114_docs/SRILM_Tutorial_20080512.pdf</li>
  <li>http://askubuntu.com/questions/507659/how-do-i-install-srilm-on-ubuntu-14-04</li>
</ol>

