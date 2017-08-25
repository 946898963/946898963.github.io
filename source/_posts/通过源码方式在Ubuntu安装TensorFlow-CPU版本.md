---
title: 通过源码方式在Ubuntu安装TensorFlow(CPU版本)
date: 2017-01-05 10:35:30
categories:
- 技术
- 机器学习
- Tensorflow
tags:
- Tensorflow
---

最近要搞毕业设计，无奈之前跟了一个坑比导师，研究生之间帮他做的坑爹的东西Low,不能拿出来写论文，在隔壁曹老师的建议下，选择了一个图像识别的题目来做论文。最终决定用深度学习中的CNN来做图片识别，同时决定使用Tensorflow来训练模型。要使用Tensorflow肯定要先安装Tensorflow，结果本人在Linux上装Tensorflow一装就装了两天，期间各种问题，可以说一部安装史，就特么是一步血泪史，哭，当然也没有那么难，因为本人之前没用过Linux,所以这也是一个装起来很困难的愿意。好了废话不多说，接下来记录下艰辛的安装过程。以供后人参考。当然，由于本人是小白，所以遇到的问题，只能上网搜索，能解决问题，但是原因也不是很懂，所以各位大神见笑勒。转入正题。

擦，再废话几句。安装Tensorflow的方式很多，用Pip的方式比较简单。本人先是在师兄徐博的Linux操作系统的电脑上利用Pip的方式安装的Tensorflow，参考[真正从零开始，TensorFlow详细安装入门图文教程！ ][1]，很简单的几步就安装好了，但是本人由于需要使用谷歌官方提供的exmaple中的image_retraining来进行训练，发现目录下面跟找不到这个文件，身边的万大神跟我说他是用编译源码的方式安装的能找到image_retraining，并且能成功运行，并且本人在网上看到网友说有些功能确实是只有在编译源码的方式进行安装才比较好用，见[Tensorflow 的安装和用InceptionV3训练新的图像分类模型 ][2]，于是本人才决定用编译源码的方式来进行安装，但至于是不是只有用编译源码的方式安装Tensorflow，才能使用他的image_retraining来进行训练，本人就不知道了。接下来说说我是怎么用源码的方式在Linux系统上安装Tensorflow的。


首先是安装Linux操作系统（不要问我为啥不接着用师兄的电脑，内牛满面，下面会说的。），教程的话网上很多，自己上网上搜索下优盘方式安装Ubuntu就是了，很简单，安装过程就不录了。只是简单的说下自己感觉需要注意的，在安装的时候，可能会提示你为了获得更好的安装体验请保证电脑联网，本人刚开始以为必须得联网才行，傻逼兮兮的整了笔记本开了无线，让电脑接入。后来重新装的时候，才发现不用连接网络就行，而且不联网速度比链接网络的时候的速度快很多，因为不用安装许多的软件。当然有利有弊，本人重装的时候没选择联网，结果好多软件都没有安装，尤其是输入法超难用，所以自己有花了好长时间来安装了个搜狗输入法，哭，安装个输入法都这么难，没办法，不会用啊Linux。哦，在安装搜狗输入法之前先要让电脑联网，这特么又是个蛋疼的问题，接下来说说怎么解的。

因为本人在学校里面，登录校园网的话需要用锐捷客户端登录，但是本人木有找到Linux版本的锐捷客户端，宝宝。。。幸好有我大华科的大神校友，搞出来的MentoHust解决了我的问题,MentoHust是一个在Linux下与锐捷兼容性较好的认证客户端，方便使用Linux和锐捷的同学使用校园网。具体的安装过程我也不说了，附录上某位网友整理的日志， [Ubuntu下使用MentoHUST搞定 锐捷校园网认证网络][3] 在此感谢这位同仁。运行的MentoHust出问题的话第二个问题可以参考这篇[ Ubuntu12.04 用MentoHUST认证上网提示“打开libnotify失败，请检查是否已安装该库文件”解决方案 ][4]来解决。在此附录下MentoHust的相关指令：
```
查看详细配置                   ：sudo mentohust -h  
使用动态IP、DHCP方式登录校园网 ：sudo mentohust -u你的用户名 -p你的密码 -a1 -d2 -b2 -v4.10 -w 
退出认证使用                   ：sudo mentohust -k 或者pkill mentohust
再次认证时只需要键入           ：sudo mentohust  
```
就这么解决了电脑的联网问题之后，接下来就是刚才说的输入法的问题了，众所周知Linux上的软件比较少，好用的输入法好像也比较少，本人在知乎[Ubuntu 上有哪些好用的输入法可以推荐？][5]看到网友说Linux上的搜狗输入法可以用，就去网上搜怎么在Ubuntu安装搜狗输入法，结果发现直接在搜狗官网下载安装就是了[搜狗输入法 Linux][6]，还是比较方便的，对搜狗提供经Linux版本的输入法给个赞。电脑能连接上网了，也安装上好用的输入法了，接下来终于可以安装TensorFLow了，内牛满面。。。。欢天喜地，没想到，接下来才是特么的XXXXOOOO。



首先要从网上下载Tensorflow的源码，根据自己的习惯放在某个位置就好了。
```
git clone --recurse-submodules https://github.com/tensorflow/tensorflow
```
使用 --recurse-submodules 选项来获取 TensorFlow 需要依赖的 protobuf库文件。对了，需要安装git,在Bash里输入一下指令，就可以安装git了。
```
sudo apt-get install git
```
接下来需要安装Bazel,Bazel是一个编译用的工具，你编译Tensorflow的源码需要用到。安装bazel是本人安装过程中遇到的一个XXXOOOO的地方之一，期间遇到了各种问题，现在想想倒不是安装Bazel有多困难，而是因为网上的资料太多，而自己刚开始参考的博文都不太好，再加上本人是个菜逼，一安装出现问题就只能在网上搜答案，解决不了就GG，然后重装系统。。。。从头再来。。汗。。。接下来说怎么安装的bazel。

如果电脑的系统是Ubuntu Wily (15.10)以下，需要安装Oracle JDK 8。
首先
```
sudo add-apt-repository ppa:webupd8team/Java
```
如果出现没有找到任何绝对信任的密钥这个提示，不用管就可以了。
接着
```
sudo apt-get update
```
接着
```
sudo apt-get install oracle-java8-installer
```
这一步需要下载文件，所以需要等一会。
最后在
```
sudo apt-get update一下。
```
接下来输入。
```
echo "deb [arch=amd64] http://storage.googleapis.com/bazel-apt stable jdk1.8" | sudo tee /etc/apt/sources.list.d/bazel.list
```
然后
```
curl https://bazel.build/bazel-release.pub.gpg | sudo apt-key add -
```
接下来，安装更新bazel
```
sudo apt-get update && sudo apt-get install bazel
```
这样bazel应该就安装成功了。这种安装方式是通过custom APT repository进行安装的，推荐这种方式，本人用这种方式安装的很顺利，哭，用其他的方式快安装哭了。。。快把我安装哭的方式是Installer方式，接下来说下这种方式的正确的安装步骤。
首先安装JDK 8,
如果是Ubuntu Trusty (14.04 LTS)
```
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
```
如果是Ubuntu Wily (15.10)
```
sudo apt-get install openjdk-8-jdk
```
接下来安装其他的依赖
```
sudo apt-get install pkg-config zip g++ zlib1g-dev unzip
```
然后下载[Bazel][7]，执行
```
chmod +x 你下载的bazel的.sh文件
./你下载的bazel的.sh文件 --user
```
接下来，添加到路径
```
export PATH="$PATH:$HOME/bin"
```
这样就安装成功了。看着也挺简单，但是我最初安装的时候，安装完后总是提示找不到bazel，没有安装bazel，找了好多解决方案都没成功，因为最开始安装用的师兄的电脑，他电脑是Linux的，好多东西都装过了，不知道是不是某些东西设置的有问题，所以。。。。我才换了个电脑重新从装Linux开始，从头开始搞的。。佩服我吧。。哈哈(哭。。)。具体的没安装成功的原因至今仍然不知道。。。。关于安装Bazel推荐这篇[Installing Bazel][8]，通过安装Bazel我深有感触，技术开发看文档，还是看官方英文原版的比较好，图省事看中文的资料最后可能得不尝试，因为好多中国的好多的技术博客都是你抄我，我抄你，抄来抄去就抄出各种问题，nidaye的，还是耐下心来看官方文档吧，很严肃认真的告诫自己。

bazel安装成功之后，接下来可以说是最重要的一步了。。我在这一步又跪了。。哭。。这一步其实很简单，顺利的话就是三条指令，但是本人不顺利出了问题。。先说说，顺利的话，只需要执行的三条指令，创建 pip 包并安装。
```
bazel build -c opt //tensorflow/tools/pip_package:build_pip_package
```
```
bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg
```
```
pip install /tmp/tensorflow_pkg/xxxxx.whl
```
注意xxxx。whl是在/tmp/tensorflow_pkg下的你的tensorflow的.wl文件，不同的平台文件是不一样的，需要你去/tmp/tensorflow_pkg目录下查看你的平台下软件是什么名字。顺利的话，执行完这三步，就OK了。然而本人在第一步就出了问题。
执行完第一步后，我以为成功安装了pip了，但是当我执行第三步的时候，提示我出现了问题，提示
```
pip未安装。。。。
```
我曹，老子当时的心情是悲愤的，特么，好不容易安装成功了bazel，安装pip又特妹的出现了问题。。。这特么是教程啊，人家安装的时候怎么没有出现这种问题，不同人不同命啊，尼玛啊，老子真想砸电脑啊，当然只是想想，屌丝可没钱砸电脑。忍着砸电脑的悲愤心情，接着在网上搜索方案。各种尝试，然而。。当天特么并没有找到解决方案，当时已经晚上九点了，老子又装了一下午跟一晚上了，为了不让自己当天一点收货都没有，老子带着这种悲愤的心情，去做了其他事情。。然后带着这种悲愤的心情睡觉，第二天早上从悲愤中醒来，然后带着悲愤的心情吃了早饭，然后继续悲愤的搜索解决方案，就在某一刻哥各种灵魂附体（想想黄健翔的著名时刻），哥鬼使神差的去看/tensorflow/tensorflow/tools/pip_package路径下的文件，发现有个setup.py,然后想起昨晚好像用setup.py安装过某个软件，于是哥机智的重新找出了那个网页，然后按照那个网页上安装软件的方式，成功安装成功了pip。。。具体指令是这样的
```
python setup.py install
easy_install pip
```
具体有没有第二条指令记不清楚了。。如果各位也遇到跟我一样的问题，可以尝试下，还有就是如果easy_install pip 提示找不到pip的话可以换成easy_install build_pip_package.sh试试，因为/tensorflow/tensorflow/tools/pip_package有这个文件啊，可以尝试下。附上那个给我灵感的网页[Python 版本为2.6安装argparse ][9]，悲剧，记得在安装的过程中argparse也出现过问题。
当我bash中输入pip出现相关提示，意味着安装成功的时候，老子的心情。。特闷。。自己体会。当时以为，我曹，终于安装成功了，尼玛，然后怀着激动的心情，执行了
```
bazel build -c opt //tensorflow/tools/pip_package:build_pip_package
bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg
pip install /tmp/tensorflow_pkg/xxxxx.whl
```
然而，当我执行完第三条指令的时候，并没有出现successfully installed的提示语言，我就试着在tensorflow目录下输入python然后在Python环境下执行
```
import tensorflow as tf
```
然而bash里出现了错误，
```
Error importing tensorflow. Unless you are using bazel,
you should not try to import tensorflow from its source directory;
please exit the tensorflow source tree, and relaunch your python interpreter
from there.
```
哥赶脚是不能在当前目录下执行import tensorflow as tf，然后就去搜了下，果然是不能在tensorflow目录下输入进入python环境执行import tensorflow as tf，于是哥就怀着安装成功了心情，退出了当前的目录，然后在根目录下进入python环境，然后执行了import tensorflow as tf然后，特么竟然提示
```
no module .....
```
心情。。。。（自己想象吧）。继续网上搜方案，在StackOverFlow上看到相关的说可能是pip安装的tensorflow的版本过低，导致安装的识别不出来，于是，哥就尝试着从网上下载高版本的tensorflow的.whl文件进行安装，安装成功后，尝试着进入python环境，然后执行了import tensorflow as tf，提示成功。“大功告成了”（真的成功了！！）。。。。

然后尝试着去运行谷歌官网的Inceptionv3模型进行训练，关于InceptionV3,可以参考[Tensorflow 的安装和用InceptionV3训练新的图像分类模型 ][10]第二部分以及[如何训练Inception的最后一层新的类别？][11]。
首先在tensorflow的根目录下的命令行中输入编译retrian的命令：
```
bazel build tensorflow/examples/image_retraining:retrain
```
这一步执行成功，需要花费一定时间。接着
```
bazel-bin/tensorflow/examples/image_retraining/retrain --image_dir ~/XXX
```
这里一定要注意！！！image_dir ~/XXX是训练数据集的路径跟名称。。。详细的说，我的训练集图片是放在/home/cnmqsb/leaf文件夹下，那么指令就是
```
bazel-bin/tensorflow/examples/image_retraining/retrain --image_dir ~/leaf
```
其实到这里，就已经安装成功了，但是本人在这里犯了一个错误，导致本人接下来浪费了几个小时的时间，感兴趣的话，可以看下去。
当时我参考的博客[Tensorflow 的安装和用InceptionV3训练新的图像分类模型 ][12]里面是这样讲的
```image_dir ~/XXX是训练数据集的路径XXX是数据集的名称```，然后我就以为image_dir是路径，XXX是存放数据集的文件夹名称，于是把指令写成了这样
```
bazel-bin/tensorflow/examples/image_retraining/retrain --/home/cnmqsb/ ~/leaf
```
结果提示，如下错误
```
W tensorflow/core/platform/cpu_feature_guard.cc:95] The TensorFlow library wasn't compiled to use SSE4.2 instructions, but these are available on your machine and could speed up CPU computations.
W tensorflow/core/platform/cpu_feature_guard.cc:95] The TensorFlow library wasn't compiled to use AVX instructions, but these are available on your machine and could speed up CPU computations.
Looking for images in '_python_build'
No files found
Looking for images in 'build'
Traceback (most recent call last):
  File "/home/cnmqsb/tensorflow/bazel-bin/tensorflow/examples/image_retraining/retrain.runfiles/org_tensorflow/tensorflow/examples/image_retraining/retrain.py", line 1052, in <module>
    tf.app.run(main=main, argv=[sys.argv[0]] + unparsed)
  File "/home/cnmqsb/tensorflow/bazel-bin/tensorflow/examples/image_retraining/retrain.runfiles/org_tensorflow/tensorflow/python/platform/app.py", line 44, in run
    _sys.exit(main(_sys.argv[:1] + flags_passthrough))
  File "/home/cnmqsb/tensorflow/bazel-bin/tensorflow/examples/image_retraining/retrain.runfiles/org_tensorflow/tensorflow/examples/image_retraining/retrain.py", line 774, in main
    FLAGS.validation_percentage)
  File "/home/cnmqsb/tensorflow/bazel-bin/tensorflow/examples/image_retraining/retrain.runfiles/org_tensorflow/tensorflow/examples/image_retraining/retrain.py", line 140, in create_image_lists
    file_list.extend(gfile.Glob(file_glob))
  File "/home/cnmqsb/tensorflow/bazel-bin/tensorflow/examples/image_retraining/retrain.runfiles/org_tensorflow/tensorflow/python/lib/io/file_io.py", line 269, in get_matching_files
    compat.as_bytes(filename), status)]
  File "/usr/lib/python2.7/contextlib.py", line 24, in __exit__
    self.gen.next()
  File "/home/cnmqsb/tensorflow/bazel-bin/tensorflow/examples/image_retraining/retrain.runfiles/org_tensorflow/tensorflow/python/framework/errors_impl.py", line 469, in raise_exception_on_not_ok_status
    pywrap_tensorflow.TF_GetCode(status))
tensorflow.python.framework.errors_impl.NotFoundError: build
```
看错误提示分成两部分，一部分是
```
W tensorflow/core/platform/cpu_feature_guard.cc:95] The TensorFlow library wasn't compiled to use SSE4.2 instructions, but these are available on your machine and could speed up CPU computations.
W tensorflow/core/platform/cpu_feature_guard.cc:95] The TensorFlow library wasn't compiled to use AVX instructions, but these are available on your machine and could speed up CPU computations.
```
第二部分是
```
Looking for images in '_python_build'
No files found
Looking for images in 'build'
Traceback (most recent call last):
  File "/home/cnmqsb/tensorflow/bazel-bin/tensorflow/examples/image_retraining/retrain.runfiles/org_tensorflow/tensorflow/examples/image_retraining/retrain.py", line 1052, in <module>
    tf.app.run(main=main, argv=[sys.argv[0]] + unparsed)
  File "/home/cnmqsb/tensorflow/bazel-bin/tensorflow/examples/image_retraining/retrain.runfiles/org_tensorflow/tensorflow/python/platform/app.py", line 44, in run
    _sys.exit(main(_sys.argv[:1] + flags_passthrough))
  File "/home/cnmqsb/tensorflow/bazel-bin/tensorflow/examples/image_retraining/retrain.runfiles/org_tensorflow/tensorflow/examples/image_retraining/retrain.py", line 774, in main
    FLAGS.validation_percentage)
  File "/home/cnmqsb/tensorflow/bazel-bin/tensorflow/examples/image_retraining/retrain.runfiles/org_tensorflow/tensorflow/examples/image_retraining/retrain.py", line 140, in create_image_lists
    file_list.extend(gfile.Glob(file_glob))
  File "/home/cnmqsb/tensorflow/bazel-bin/tensorflow/examples/image_retraining/retrain.runfiles/org_tensorflow/tensorflow/python/lib/io/file_io.py", line 269, in get_matching_files
    compat.as_bytes(filename), status)]
  File "/usr/lib/python2.7/contextlib.py", line 24, in __exit__
    self.gen.next()
  File "/home/cnmqsb/tensorflow/bazel-bin/tensorflow/examples/image_retraining/retrain.runfiles/org_tensorflow/tensorflow/python/framework/errors_impl.py", line 469, in raise_exception_on_not_ok_status
    pywrap_tensorflow.TF_GetCode(status))
tensorflow.python.framework.errors_impl.NotFoundError: build
```
根据第一部分的提示，我猜想是不是因为我安装的tensorflow不是自己本地用bazel编译的出来进行安装的，所以才会出错（前面说过了用自己本地编译出来安装的tensorflow版本过低，识别不出来，所以从网上重新下载了tensorflow来进行安装），这时候我就纠结了，这该怎么办，本地编译出来的版本过低，识别不出来，但是用从网上下载的高版本的tensorflow又不能运行InceptionV3，这该怎么办，在网上搜索一一番没有结果之后。。。。哥就继续怀着悲愤的心情离开了实验室，去吃午饭了。。吃午饭的时候，一直提醒自己不能再安装了，再装下去估计一天又过去了，嗯，不装了。结果吃完午饭回到实验室，又特么开始装，午觉都不睡，在网上搜不到关于
```
W tensorflow/core/platform/cpu_feature_guard.cc:95] The TensorFlow library wasn't compiled to use SSE4.2 instructions, but these are available on your machine and could speed up CPU computations.
W tensorflow/core/platform/cpu_feature_guard.cc:95] The TensorFlow library wasn't compiled to use AVX instructions, but these are available on your machine and could speed up CPU computations.
```
错误提示的解决方案，于是我只好抱着死马当活马医的试试看的想法，又重新用本地编译出来的tensorflow进行安装。安装完后，退出tensorflow的根目录，进入Python环境执行，import tensorflow as tf,这时候奇迹出现了，竟然没出错，提示如下：
```
>>> import tensorflow as tf
W tensorflow/core/platform/cpu_feature_guard.cc:95] The TensorFlow library wasn't compiled to use SSE4.2 instructions, but these are available on your machine and could speed up CPU computations.
W tensorflow/core/platform/cpu_feature_guard.cc:95] The TensorFlow library wasn't compiled to use AVX instructions, but these are available on your machine and could speed up CPU computations.
>>> 
```
这说明tensorflow安装成功了，只不过上面出现过的第一部分的提示，出现在了这里。。。。原来用自己从网上下载的高版本的tensorflow安装完后，在python环境下执行import的操作，是没有这个提示的。不管了，我重新回到tensorflow根目录下面执行
```
bazel-bin/tensorflow/examples/image_retraining/retrain --/home/cnmqsb/ ~/leaf
```
妈的，还是报错，报错就是上面的第二部分。
```
Looking for images in '_python_build'
No files found
Looking for images in 'build'
Traceback (most recent call last):
  File "/home/cnmqsb/tensorflow/bazel-bin/tensorflow/examples/image_retraining/retrain.runfiles/org_tensorflow/tensorflow/examples/image_retraining/retrain.py", line 1052, in <module>
    tf.app.run(main=main, argv=[sys.argv[0]] + unparsed)
  File "/home/cnmqsb/tensorflow/bazel-bin/tensorflow/examples/image_retraining/retrain.runfiles/org_tensorflow/tensorflow/python/platform/app.py", line 44, in run
    _sys.exit(main(_sys.argv[:1] + flags_passthrough))
  File "/home/cnmqsb/tensorflow/bazel-bin/tensorflow/examples/image_retraining/retrain.runfiles/org_tensorflow/tensorflow/examples/image_retraining/retrain.py", line 774, in main
    FLAGS.validation_percentage)
  File "/home/cnmqsb/tensorflow/bazel-bin/tensorflow/examples/image_retraining/retrain.runfiles/org_tensorflow/tensorflow/examples/image_retraining/retrain.py", line 140, in create_image_lists
    file_list.extend(gfile.Glob(file_glob))
  File "/home/cnmqsb/tensorflow/bazel-bin/tensorflow/examples/image_retraining/retrain.runfiles/org_tensorflow/tensorflow/python/lib/io/file_io.py", line 269, in get_matching_files
    compat.as_bytes(filename), status)]
  File "/usr/lib/python2.7/contextlib.py", line 24, in __exit__
    self.gen.next()
  File "/home/cnmqsb/tensorflow/bazel-bin/tensorflow/examples/image_retraining/retrain.runfiles/org_tensorflow/tensorflow/python/framework/errors_impl.py", line 469, in raise_exception_on_not_ok_status
    pywrap_tensorflow.TF_GetCode(status))
tensorflow.python.framework.errors_impl.NotFoundError: build
```
吐血了，第一部分的错误提示是没有了，但是第二部分的错误提示依旧存在，吐了一会血，吐完血没办法，还得接着搞，看错误提示，我想是不是我的tensorflow的源码编译出了问题，于是我重新执行了
```
bazel build -c opt //tensorflow/tools/pip_package:build_pip_package
```
尴尬，我以为这是编译tensorflow，但其实好像不是，汗。执行完了之后重新运行
```
bazel-bin/tensorflow/examples/image_retraining/retrain --/home/cnmqsb/ ~/leaf
```
结果发现还是不行，于是我就怀疑是不是我的路径设置的有问题，于是我重新搜索其他的介绍关于怎么用InceptionV训练模型的博客，结果在[如何训练Inception的最后一层新的类别？][13]中发现了这么一句话：
```
xxxxxxx把物类根目录作为image_dir的参数xxxxxxxx
```
。。。。我当时就(ˇˍˇ) 想，特码的，不会吧，原来是老子把路径设置错了，于是我就把指令改成了这样：
```
bazel-bin/tensorflow/examples/image_retraining/retrain --image_dir ~/home/cnmqsb/leaf
```
但是还是出现了如下错误提示：
```
Image directory '/home/cnmqsb/home/tmp/leaf' not found.
Traceback (most recent call last):
  File "/home/cnmqsb/tensorflow/bazel-bin/tensorflow/examples/image_retraining/retrain.runfiles/org_tensorflow/tensorflow/examples/image_retraining/retrain.py", line 1052, in <module>
    tf.app.run(main=main, argv=[sys.argv[0]] + unparsed)
  File "/home/cnmqsb/tensorflow/bazel-bin/tensorflow/examples/image_retraining/retrain.runfiles/org_tensorflow/tensorflow/python/platform/app.py", line 44, in run
    _sys.exit(main(_sys.argv[:1] + flags_passthrough))
  File "/home/cnmqsb/tensorflow/bazel-bin/tensorflow/examples/image_retraining/retrain.runfiles/org_tensorflow/tensorflow/examples/image_retraining/retrain.py", line 775, in main
    class_count = len(image_lists.keys())
AttributeError: 'NoneType' object has no attribute 'keys'
```
看第一句代码，发现识别的路径是‘/home/cnmqsb/home/tmp/leaf’于是我就知道了，~/应该对应/home/cnmqsb/于是我把指令写成下面这样：
```
bazel-bin/tensorflow/examples/image_retraining/retrain --image_dir ~/leaf
```
然后，然后，就出了激动人心的画面。。自行想像，截图就不附录了。。。。哭。。。。安装了两天。终于在第二天的下午四点多安装成功了。老子的心情是日狗的。。。。关于为什么刚开始安装编译出来的tensorflow不能识别，但是后来安装了从网上下载的高版本的tensorflow后，重新安装会编译出来的低版本tensorflow能正确识别，原因是啥我也不知道。。。。可能是老天被偶感动了吧。。哭。好了关于源码方式安装tensorflow就写到了这里，因为当时安装的过程中，有些错误没有截图，写总结的时候，忘记了安装过程中的某些问题，所以在这里安装过程中出现的问题并没有一一列出。有些遗憾，以后在开发的过程中，遇到问题，一定要静下心来一一的截图保留，方便自己做开发日志。此外就是上面写到，做开发还是参考资料还是参考官网原版的英文的比较好，还有就是吐槽下，垃圾百度。。。是真搜不出什么东西来，做开发还是用谷歌，本人因为不知道怎么在Ubuntu上翻墙（当时没有心情搞），所以只能一边骂一遍用百度，哎中国开发者的悲哀。哎，佩服苦逼的自己哦，安装了两天，还特么在新年的第二天就打破了自己立的新的一年规范作息不熬夜的FLAG....下面把安装过程中的一些参考的网上资料附录上(上面附录的下面就不附录了),，如果各位遇到问题，可以先从下面的网页看看有没有解决办法。

[Tensorflow下载与安装 ][14]   推荐，这是极客学院里各位大神们根据英文官方教程翻译的，赞~\(≧▽≦)/~。

[tensorflow源代码方式安装][15]

[TensorFlow 研究实践笔记][16]

[apt-get update 和 upgrade 的区别 ][17]

[python - 升级所有已安装的第三方包][18]

[ubuntu14.04安装setuptools ][19]

[ubuntu如何卸载apt-get install安装的软件][20]

[ubuntu14.10下卸载安装在python上的模块 ][21]

[Python安装模块出错（ImportError: No module named setuptools）解决方法][22]

[No module named setuptools 解决方案 ][23]

最后记录下**时间线**：

2017.1.2  下午                   在徐博电脑安装Tensorflow失败，卡在bazel安装上。
2017.1.2  晚上-2017.1.3凌晨七点  自己装操作系统，从头开始装，卡在pip安装上。
2017.1.3  下午-晚上              解决pip安装问题，卡在了tensorflow的安装上。
2017.1.4  上午-中午              安装tensorflow成功，但是图片路径设置错误，导致出错。
2017.1.4  下午                   找出了路径设置错误，成功运行 image_retraining example。

**2017年1月4日晚记录于武汉华中科技大学国家防伪中心识别实验室,完成于1月5号早上。**

 


  [1]: http://www.leiphone.com/news/201606/ORlQ7uK3TIW8xVGF.html
  [2]: http://blog.csdn.net/qq_25231283/article/details/52700394
  [3]: http://www.linuxidc.com/Linux/2013-10/91157.htm
  [4]: http://blog.csdn.net/chen_blog/article/details/20727345
  [5]: https://www.zhihu.com/question/19694153
  [6]: http://pinyin.sogou.com/linux/
  [7]: https://github.com/bazelbuild/bazel/releases
  [8]: https://bazel.build/versions/master/docs/install.html
  [9]: https://www.janecc.com/python-26-argparse.html
  [10]: http://blog.csdn.net/qq_25231283/article/details/52700394
  [11]: http://www.bubuko.com/infodetail-1490939.html
  [12]: http://blog.csdn.net/qq_25231283/article/details/52700394
  [13]: http://www.bubuko.com/infodetail-1490939.html
  [14]: http://wiki.jikexueyuan.com/project/tensorflow-zh/get_started/os_setup.html
  [15]: http://www.cnblogs.com/simplelovecs/p/5150114.html
  [16]: http://www.linuxidc.com/Linux/2016-07/133228.htm
  [17]: http://blog.csdn.net/duyiwuer2009/article/details/26983267
  [18]: http://www.cnblogs.com/huangjacky/p/4124819.html
  [19]: https://my.oschina.net/deanzhao/blog/317603
  [20]: http://www.linuxdiyf.com/linux/14109.html
  [21]: http://blog.csdn.net/cryhelyxx/article/details/42737985
  [22]: http://blog.sina.com.cn/s/blog_3fe961ae0100zgav.html
  [23]: http://blog.chinaunix.net/uid-12014716-id-3989658.html