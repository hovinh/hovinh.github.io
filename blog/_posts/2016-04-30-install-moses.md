---
#layout: post
title: Install MOSES on Ubuntu
description: >
  Tutorial.
author: author1
comments: true
---

Moses is popular system in statistical machine translation, but quite hard to install. This post helps amateur to get used to with Moses for the first time, enough to use without caring about complex parameters. But if you want to improve your system, more understanding about **tuning** is required to achieve best result.

## Preparation
- Install aligment tool: <a href="/blog/2016-01-27-install-giza-ubuntu">GIZA</a> or <a href="/blog/2016-04-29-install-mgiza-ubuntu">MGIZA</a>.
- Install language model: <a href="/blog/2016-04-22-install-srilm-ubuntu">SRILM</a>.
- Install Ubuntu package
```powershell
$ sudo apt-get install build-essential git-core pkg-config automake libtool wget zlib1g-dev python-dev libbz2-dev
```

## Installation

Open Terminal:
```powershell
$ git clone https://github.com/moses-smt/mosesdecoder.git
$ cd mosesdecoder
```
To compile you need Boost, however there is version you could crash your compile process if use it.  The person who is in charge of Moses(Mr. Hieu Hoang) suggests using 1.55, I am fond of 1.60. So if anything goes wrong, the problem might be Boost version is incompatible. The purpose of using many lines  for boost is to install in private folder, not in system folder. So you don’t need superuser authority to run.

```powershell
$ wget http://downloads.sourceforge.net/project/boost/boost/1.60.0/boost_1_60_0.tar.gz
$ tar zxvf boost_1_60_0.tar.gz
$ cd boost_1_60_0/
$ ./bootstrap.sh
$ ./b2 -j4 --prefix=$PWD --libdir=$PWD/lib64 --layout=system link=static install || echo FAILURE
```

After running, if it says SUCCESS then OK, otherwise FAILURE. However, if it is FAILURE, you just keep doing the next step to see if something goes wrong. If so, come back to reinstall Boost with different version :))

Now compile  Moses

```powershell
$ ./bjam --with-boost=~/workspace/temp/boost_1_60_0 -j4
```
If success, there will be a "SUCCESS" line on Terminal.

## Test
You will have data of step n-1, now try to run nth step to see if the result as we expect.

```powershell
$ cd ~/mosesdecoder
$ wget http://www.statmt.org/moses/download/sample-models.tgz
$ tar xzf sample-models.tgz
$ cd sample-models
$ ~/mosesdecoder/bin/moses -f phrase-model/moses.ini
```

Go to "/mosesedecoder/sample-models/out". If 2 lines below show up, then congratulate, you have successfuly installed Moses.

>This is a small house  
This is a small house

## Configuring for real run:

This step is extremely complicated. Fortunately, we have a shortcut, which is EMS.

### Installing some packages:

```powershell
$ sudo apt-get update
$ sudo apt-get install imagemagick --fix-missing
$ sudo apt-get install graphviz
$ sudo apt-get install gv
```

### Create folder:

I choose aligment tool is GIZA++ and language model is SRILM.

- FolderA contains corpus, including: <a href="https://gist.github.com/lngvietthang/907b74187b88994cb6ee77820a9bdf6d">convTxt2Sgm.py</a>, weight.ini(copy from "/mosesdecoder/scripts/ems/example/data"), data.en, data.vn, test.en, test.vn(These 4 files actually are 2 billingual corpus, but split in ratio 9:1 for training and testing, .data for training and .test for testing. You can also create validate file for tuning). Directory: "/home/phdlab/Desktop/HoXuanVinh/Corpus".
- FolderB contains GIZA. Including: GIZA++, mkcls and snt2cooc.out. Directory: "/home/phdlab/mosesdecoder/tools"
- FolderC(do not need to create) contains "experiment.perl". Directory: "/home/phdlab/mosesdecoder/scripts/ems"
- FolderD is working directory, all results when running are saved here. Directory: "/home/phdlab/Desktop/HoXuanVinh/Vinh"

### Preparing data for running in FolderA:

Assumes you had 2 files "source.txt" and "target.txt", you will use code here to split them into trainining set và test set. If we have 2 billingual corpus English-Vietnamese, then their names will be:

- data.en: English corpus for training
- data.vn: Vietnamese corpus for training
- test.en: English corpus for testing
- test.vn: Vietnamese corpus for testing

Then convert .test file as format of Moses.

Open Terminal, move to FolderA and type:

```powershell
python convTxt2Sgm.py -src test.en -ref test.vn
```

### Configuration:

Go to "/home/phdlab/mosesdecoder/scripts/ems/example", open file config.toy and edit.

Line 9(FolderD):
```powershell
working-dir = /home/phdlab/Desktop/HoXuanVinh/Vinh
```

Line 13-15:
```powershell
input-extension = en
output-extension = vn
pair-extension = en-vn
```

Line 20(folder contains moses), 29(folder containes executive files of GIZA/MGIZA):
```powershell
moses-scr-dir = /home/phdlab/mosesdecoder
external-bin-dir = $moses-src-dir/tools
```

Line 32 (if choose srilm then comment out # irstlm, randlm):
```powershell
working-dir = /home/phdlab/Desktop/HoXuanVinh/Vinh
```

Line 42(FolderA):
```powershell
toy-data = /home/phdlab/Desktop/HoXuanVinh/Corpus
```

Line 92(number of cores):
```powershell
cores = 8
```

Line 141, 142, 150(configure parameter for SRILM and comment out other language model):
```powershell
lm-training = $srilm-dir/ngram-count
setting = "-interpolate -kndiscount -unk"
order = 5
```

Line 198 - 220, 486-513 (if  Tuning then change here, otherwise pass).

Line 292. If use MGIZA, add parameter you want:
```powershell
training-options = -mgiza -mgiza-cpus 8 -sort-parallel 8 -cores 8 -parallel
```

Line 317. Uncomment.

Line 657, 665:
```powershell
input-sgm = $toy-data/test.$input-extension.sgm
reference-sgm = $toy-data/test.$output-extension.sgm
```

### Running:

Open Terminal, move to folderC. Type:

```powershell
./experiment.perl -config example/config.toy -exec
```

When running, a graph will shows you which module Moses is running, if it turns red then there is error. Keep calm and open "FolderD/steps". You will see folder numbered from 1. The number tells the numerical order of the time you run Moses. If you meet error in the first time run, go to  folder "1". Look for error module with extension .STDERR, open and check the bug.

![Moses's graph](/assets/blog/2016-04-30/moses_graph.jpg){:data-width="1440" data-height="836"}
Execute Graph and the red module shows something is wrong.
{:.figure}


At the end of the process, open FolderD, folder evaluation, look for "test.output" file(See test result) and "test.nist-bleu" (to check BLEU score and alignement result). Good luck.

>Note:
- weight.ini is initialised weigth file of model. If know how to Tuning, then you can get the weight from there.
- Original Website says that with  130.000 pair of sentences, alignment result is quite low,  which mean you need larger and clean corpus for better result.
- Whenever you get an error and confusing, open "train-model.perl" in "mosesdecoder/scripts/training" to check argument and error message in SDTERR file..


## Reference
<ol>
  <li>http://www.statmt.org/moses/?n=Moses.Baseline</li>
  <li>http://www.statmt.org/moses/?n=Development.GetStarted</li>
  <li>http://www.statmt.org/moses/?n=FactoredTraining.EMS</li>
  <li>https://gist.github.com/lngvietthang/907b74187b88994cb6ee77820a9bdf6d</li>
</ol>