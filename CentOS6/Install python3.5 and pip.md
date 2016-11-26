## 准备编译环境

yum groupinstall 'Development Tools'

yum install zlib-devel bzip2-devel  openssl-devel ncurses-devel

## 下载

cd ~

wget  https://www.python.org/ftp/python/3.5.2/Python-3.5.2.tar.xz

## 编译

tar Jxvf  Python-3.5.2.tar.xz

cd Python-3.5.2

sudo ./configure --prefix=/usr/local/python3

sudo make

sudo make install

## 设置环境变量

echo 'export PATH=$PATH:/usr/local/python3/bin' >> ~/.bashrc

## 或者可以直接替换python2

rm   /usr/bin/python

ln -sv  /usr/local/python3/bin/python3.5 /usr/bin/python

## 替换pip

rm /usr/bin/pip

ln -sv /usr/local/python3/bin/pip3.5 /usr/bin/pip

## 更新yum配置

ll /usr/bin | grep python

这时/usr/bin目录下面是包含以下几个文件的（ll |grep python），其中有个python2.6，只需要指定yum配置的python指向这里即可

vim /usr/bin/yum

通过vim修改yum的配置

\#!/usr/bin/python改为#!/usr/bin/python2.6，保存退出。

完成了python3的安装。
