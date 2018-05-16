---
title: "Movie Studio (in development)"
layout: post
date: 2018-04-30
tag:
- python
- flask
- sqlalchemy
- mysql
- blueprint

image:
headerImage: false
projects: true
hidden: true # don't count this post in blog pagination
description: "article management system"
jemoji: '<img class="emoji" title=":python:" alt=":python:" src="/assets/images/language_icon/python.png" height="20" width="20" align="absmiddle">'
author: Jiaqi Xu
externalLink: false
---

This project is in progress.
#### Environment Setting
* Mac OS 环境搭建
    1. python3.6： https://www.python.org/
    2. mysql5.6(集成工具MAMP): https://www.mamp.info<br>
       如果命令行找不到路径：添加环境变量到～／.bash_profile
       export PATH=$PATH:/Applications/MAMP/Library/bin
    3. pycharm
    4. 安装虚拟环境：pip3 install virtualenv

* Virtualenv的使用
    1. 创建虚拟环境：virtualenv -p /usr/local/bin/python3 movie_project （因为当时mac上既有python3，也有python2，该命令是为了创建python3的虚拟环境）
    2. 激活虚拟环境：source movie_project/bin/activate
    3. 退出虚拟化环境：deactivate<br>
    也可以在pycharm中手动创建项目的虚拟环境

* 在虚拟环境下使用pip安装packages
    1. 安装前检测: pip3 freeze
    2. 安装flask等包: pip3 install flask
    3. 安装后检测： pip3 freeze

* 本次项目所需要的packages

    可以使用pip install -r requirements.text安装所有包
    ```python
    appnope==0.1.0
    backcall==0.1.0
    click==6.7
    decorator==4.3.0
    Flask==0.12.2
    Flask-SQLAlchemy==2.1
    Flask-WTF==0.14.2
    ipdb==0.11
    ipython==6.3.1
    ipython-genutils==0.2.0
    itsdangerous==0.24
    jedi==0.12.0
    Jinja2==2.10
    MarkupSafe==1.0
    parso==0.2.0
    pexpect==4.5.0
    pickleshare==0.7.4
    Pillow==5.1.0
    prompt-toolkit==1.0.15
    ptyprocess==0.5.2
    Pygments==2.2.0
    PyMySQL==0.8.0
    simplegeneric==0.8.1
    six==1.11.0
    SQLAlchemy==1.2.6
    traitlets==4.3.2
    wcwidth==0.1.7
    Werkzeug==0.14.1
    WTForms==2.1
    ```

#### Movie Studio
   To be Written ...

#### Github
[Movie Studio](https://github.com/jiaqi-xu/movie_project)
