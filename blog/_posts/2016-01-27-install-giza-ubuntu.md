---
#layout: post
title: Use GIZA++ to align words on Ubuntu
description: >
  Tutorial.
author: author1
comments: true
---

If you are used to with GIZA, you should change to <a href="/blog/2016-04-29-install-mgiza-ubuntu">MGIZA</a> for faster performance.

This post is based on tutorial from Luong Viet Thang 's <a href="http://lngvietthang.github.io/">blog</a>.

GIZA++ is popular toolkit for aligning bilingual corpus, preprocessing step for Statistical Machine Translation. You can easily look for online tutorials, however most of them are quite confusing in setup and using. For that reason,with creating .sh file approach, my goals in this post are:

- Installing GIZA++ easily.
- Flexible in using GIZA++ without wasting time in solving complex paremeter in command lines.

## Install GIZA++
### Pre-installed following programs:

- GNU GCC, GNU G++ : Compile C/C++ source code.
- Git : clone source code from Github.
- GNU Make: compile program.

You can check whether they are installed by typing this line in terminal: --version.

```powershell
$ gcc --version
```

If you have not installed or want to update, google <a href="https://www.google.com/search?q=Install+GNU+GCC+and+GNU+G%2B%2B+on+Ubuntu&oq=Install+GNU+GCC+and+GNU+G%2B%2B+on+Ubuntu&aqs=chrome..69i57.464j0j1&sourceid=chrome&es_sm=122&ie=UTF-8">“Install GNU GCC and GNU G++ on Ubuntu"</a>.

### Installation:

Opens Terminal (Ctrl+Alt+T), type below lines.

```powershell
$ git clone https://github.com/lngvietthang/giza-pp.git
$ cd giza-pp
$ make
```

I have tested and giza main downloading page also works perfectly, so you can use this <a href="https://github.com/moses-smt/giza-pp">link</a> instead.

Their purpose is downloading GIZA++ source code from github, move to folder **giza-pp** and compile the program. In order to check, go to **/home/username/giza-pp**. In subfolder **GIZA++-v2** there are executing files: *GIZA++*, *plain2snt.out*, *snt2cooc.out* and *snt2plain.out*. Similarly, there is *mkcls* in *mkcls-v2*.

## Execute GIZA++
### Preparation:

- Two files contain processed bilingual corpus. I have 90.000 English-Vietnamese parallel sentences. Between each sentence in file, there must be line break. Sample format:

English:

> 1 December has been nominated as the day of the election .   
1,000, 100 and 10 are the antilogarithms of 3, 2 and 1.  
10 Downing Street is the British Prime Minister ' s official residence .
{:.lead}

Vietnamese:

> Ngày 1 tháng chạp dương lịch đã được chính thức chọn là ngày bầu cử .  
1.000, 100 và 10 là những số đối loga của 3, 2 và 1.  
Số 10 đường Downing là dinh chính thức của Ngài Thủ Tướng Anh .
{:.lead}

- GIZA++.

### Execution:

>**NOTE**: When knowing how to use GIZA, you can take a look at this post for a faster way.
{:.lead}

You should follow below instruction, any difficulty can take a look at Note part.

- Create folder on Desktop containing 2 bilingual files. Rename to **Source** and **Target** (there is no extension, chosing name based on you, which will affect the result. Should run them both for clearer explanation).
- Create a file named **run\_giza.sh** and saved at **/home/username** (same level with **giza_pp**). Copy below lines into file, press **Ctrl+H**, replace all words into the directory to folder containing bilingual files.
```powershell
cd giza-pp/GIZA++-v2
./plain2snt.out '/Source' '/Target'
cd ../mkcls-v2
./mkcls -p'/Source' -V '/Source.vcb.classes'
./mkcls -p'/Target' -V '/Target.vcb.classes'
cd ../GIZA++-v2
./snt2cooc.out '/Source.vcb' '/Target.vcb' '/Source_Target.snt' > '/Source_Target.cooc'
./GIZA++ -S '/Source.vcb' -T '/Target.vcb' -C '/Source_Target.snt' -CoocurrenceFile '/Source_Target.cooc' -o Result -outputpath ''
```

- Right click on **run_giza.sh**, choose **Properties**, tab **Permissions**, tick **Allow executing file as program**.
- Open Terminal, type:
```powershell
$ ./run_giza.sh
```

- After it's done, there will be many newly created files, you only need to care about  Result.A3.final. Result has following format:
> \# Sentence pair (1) source length 17 target length 12 alignment score : 2.42851e-27  
\+ ' Nếu điều_đó giữ_được hạnh_phúc gia_đình thì vẫn_còn hời đấy !  
NULL ({ 8 11 }) + ({ 1 }) ' ({ 2 }) It ({ }) ’ ({ }) ll ({ }) be ({ }) cheap ({ 9 }) at ({ }) the ({ }) price ({ 10 }) if ({ 3 }) it ({ 4 }) keeps ({ 5 }) the ({ }) family ({ 7 }) happy ({ 6 }) ! ({ 12 })
{:.lead}

Congratulation, you have success in installing and using GIZA++. You can do similar thing in Window with Cyqwin or VMWare. Thank you brother Thắng and askubuntu.com for your sponsorship :))

>**NOTE**: In case there are bugs, you can check below or send me an email.
- The GIZA++ is downloaded from git.hub of brother Thắng, so it may not be latest one. When using different versions, please check the tutorial for minor fixing before you can install the tool. Folder name may be vary like **mkcls-v2** instead of **mkcls**.
- This is my file for you to compare the .sh file, alternate string for replacing is **/home/vinh/Desktop/Corpus**
- Link should not contain folder or file name with space in its name.
```powershell
cd giza-pp/GIZA++-v2
./plain2snt.out '/home/vinh/Desktop/Corpus/Source' '/home/vinh/Desktop/Corpus/Target'
cd ../mkcls-v2
./mkcls -p'/home/vinh/Desktop/Corpus/Source' -V'/home/vinh/Desktop/Corpus/Source.vcb.classes'
./mkcls -p'/home/vinh/Desktop/Corpus/Target' -V'/home/vinh/Desktop/Corpus/Target.vcb.classes'
cd ../GIZA++-v2
./snt2cooc.out '/home/vinh/Desktop/Corpus/Source.vcb' '/home/vinh/Desktop/Corpus/Target.vcb' '/home/vinh/Desktop/Corpus/Source_Target.snt' > '/home/vinh/Desktop/Corpus/Source_Target.cooc'
./GIZA++ -S '/home/vinh/Desktop/Corpus/Source.vcb' -T '/home/vinh/Desktop/Corpus/Target.vcb' -C '/home/vinh/Desktop/Corpus/Source_Target.snt' -CoocurrenceFile '/home/vinh/Desktop/Corpus/Source_Target.cooc' -o Result -outputpath '/home/vinh/Desktop/Corpus'
```
- If you want to look for the directory to a folder, open it and move any files in it and drop to Terminal. Delete file name and double quotes.
- If there is any folder in directory contain in its name a space, this may lead to an error. For example: **/home/ho vinh/Desktop/Corpus**.
- If you are using dual boot OS, do not save any relevant files in shared partition. The safest way is saved them all in Desktop.
- The running time is depend on the size of data (not linearly). Be patient, my friend.