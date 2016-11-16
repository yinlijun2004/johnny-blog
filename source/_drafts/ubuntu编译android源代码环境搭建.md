---
title: ubuntu编译android源代码环境搭建
date: 2016-11-16 09:04:37
tags:
---

## 安装jdk
```bash
sudo apt-get install openjdk-7-jdk openjdk-7-jre 
```

### 配置jdk环境

#### 创建文件javacfg.sh并输入一下内容
```bash
#!/bin/bash
JDK_ARRAY={
    "/usr/lib/jvm/java-7-openjdk-amd64/",
    "/home/yinlijun/android_toolchain/jdk1.7.0_80",
    "/home/yinlijun/android_toolchain/jdk1.8.0_111"
}

length = ${#JDK_ARRAY[@]};
index = 0;
while [ $index -lt length ];
    do 
        echo JDK_ARRAY[$index];
    done

case $1 in
    help)
    echo ""
 ;;
    jdk6)
 export JAVA_HOME=/usr/lib/jvm/java-6-oracle/
 ;;
    jdk7)
 export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64/
 ;;
    *)
 export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64/
 ;;
esac

export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib

java -version

while getopts ":hs:" optname
  do
    case "$optname" in
      "p")
        for index in JDK_ARRAY

            echo "Option $optname is specified"
        ;;
      "q")
        echo "Option $optname has value $OPTARG"
        ;;
      "?")
        echo "Unknown option $OPTARG"
        ;;
      ":")
        echo "No argument value for option $OPTARG"
        ;;
      *)
      # Should not occur
        echo "Unknown error while processing options"
        ;;
    esac
    echo "OPTIND is now $OPTIND"
  done

JAVA_HOME="/opt/jdk1.6.0_37/bin"
cd $JAVA_HOME
for file in $(ls)
do 
	update-alternatives --install /usr/bin/$file $file $JAVA_HOME/$file 10000
done
```

#### OpenJDK和SunJDK有啥区别
[OpenJDK和SunJDK有啥区别？ - 回答作者: Aloys寒风](http://zhihu.com/question/19646618/answer/40621705)

[What is the difference between JVM, JDK, JRE & OpenJDK?](http://stackoverflow.com/questions/11547458/what-is-the-difference-between-jvm-jdk-jre-openjdk)

安装其他工具包
```
sudo apt-get install git gitg gnupg flex bison gperf build-essential  zip curl libc6-dev  libncurses5-dev:i386 x11proto-core-dev  libx11-dev:i386 libreadline6-dev:i386   libgl1-mesa-dev g++-multilib mingw32 tofrodos python-markdown	libxml2-utils xsltproc zlib1g-dev:i386 
```



make: *** [out/host/linux-x86/obj/EXECUTABLES/aidl_intermediates/aidl_language_y.cpp] 断开的管道

sudo apt-get install bison