layout: post
title: 安装Scrapy
date: 2016-05-27 21:45:36
tags:
- Python
- Crawler
---
# 安装python环境 #

下载[python 2.7.11](https://www.python.org/ftp/python/2.7.11/Python-2.7.11.tgz)

    tar -zxvf Python-2.7.11.tgz

    cd Python-2.7.11

    ./configure --prefix=/usr/local

    make && make all && make install

<!--more-->

# 安装setuptools #

使用[ez_setup.py](bootstrap.pypa.io)安装：

    wget --no-check-certificate https://bootstrap.pypa.io/ez_setup.py
    python ez_setup.py install

# 安装pip #

使用[get-pip.py](bootstrap.pypa.io)安装：

    wget --no-check-certificate https://bootstrap.pypa.io/get-pip.py

    python get-pip.py

使用该方法会自动安装 setuptools 和 wheel

使用安装包： [pip-8.1.2](https://pypi.python.org/pypi/pip/)

    tar -zxvf pip-8.1.2.tar.gz

    cd pip-8.1.2

    python setup.py install

使用此方法安装必须先安装好setuptools

# 安装Scrapy #

    pip install Scrapy

---

遇到的问题： Failed building wheel for cryptography

cryptography安装不成功，这个包是有关https使用的，系统中要有支持的库，看到了这里[stackoverflow](http://stackoverflow.com/questions/22073516/failed-to-install-python-cryptography-package-with-pip-and-setup-py),里面通过安装以下包解决了问题

    sudo yum install gcc libffi-devel python-devel openssl-devel

使用rpm命令，检查发现缺少了libffi-devel

    yum install libffi-devel

安装之后顺利安装完成Scrapy

    cat > myspider.py <<EOF
    from scrapy import Spider, Item, Field

    class Post(Item):
      title = Field()

    class BlogSpider(Spider):
      name = 'kevin'blog'
      start_urls = ['dupengchuan.me']
      def parse(self, response):
        return [Post(title=e.extract()) for e in response.css("h2 a::text")]
    EOF

运行

    scrap runspider myspider

运行结果正常这里不再显示

***
OS： CentOS 6.5
