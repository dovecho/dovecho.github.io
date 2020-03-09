---
layout: post
title:  "Synology NAS 配置Django的失败经历 "
date: 8/3/2020 22:58:00 PM
---

## Django
[Django](https://www.djangoproject.com/) 是一个开放源代码的Web应用框架，由Python写成。

尝试服务器端采用Synology的DS918+，安装Python+PHP+MariaDB+Django，开发在客户端Windows系统中，采用VS-Code开发。

关于Django安装在WIndows端还是NAS端，还没想清楚，先把安装问题搞定吧。

## NAS的DSM配置

在NAS的桌面管理程序DSM中，安装以下套件，并进行功能配置

### 安装套件
- 安装套件：php3.5
- 安装套件：MariaDB
- ……

### 配置
- 开放SSH远程连接

## SSH登录到NAS端

并用管理员登录

### 安装PIP

```shell
wget --no-check-certificate https://bootstrap.pypa.io/get-pip.py
python3 get-pip.py
```

安装完PIP之后，可能提示Warning，执行```pip/pip3```会报错。这主要是路径问题。第一想到的是，在已有```PATH```里面增加一些链接，这也是现在的系统默认的方法，以达到安全和隔离的作用。我建议，还是这样做更为稳妥

```
sudo ln -s  /volume2/@appstore/py3k/usr/local/bin/pip ./pip
sudo ln -s  /volume2/@appstore/py3k/usr/local/bin/pip3 ./pip3
sudo ln -s  /volume2/@appstore/py3k/usr/local/bin/pip3.5 ./pip3.5
sudo ln -s /var/services/homes/dovecho/.local/bin/easy_install ./easy_install
sudo ln -s /var/services/homes/dovecho/.local/bin/easy_install-3.5 ./easy_install-3.5
```

### 配置PIP源

建立目录```.pip```，并新建一个文件```pip.conf```

```
mkdir -p .pip
vi pip.conf
```

在其中，增加如下内容

```
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple

[install]
trusted-host = https://pypi.tuna.tsinghua.edu.cn
```

### 安装Django

```
YOURUSERNAME@ShangyuanNAS:~$ pip3 install Django
Defaulting to user installation because normal site-packages is not writeable
Collecting Django
  Downloading Django-2.2.11-py3-none-any.whl (7.5 MB)
     |████████████████████████████████| 7.5 MB 786 kB/s
Collecting pytz
  Downloading pytz-2019.3-py2.py3-none-any.whl (509 kB)
     |████████████████████████████████| 509 kB 252 kB/s
Collecting sqlparse
  Downloading sqlparse-0.3.1-py2.py3-none-any.whl (40 kB)
     |████████████████████████████████| 40 kB 33 kB/s
Installing collected packages: pytz, sqlparse, Django
  WARNING: The script sqlformat is installed in '/var/services/homes/YOURUSERNAME/.local/bin' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
  WARNING: The script django-admin is installed in '/var/services/homes/YOURUSERNAME/.local/bin' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
Successfully installed Django-2.2.11 pytz-2019.3 sqlparse-0.3.1
```

同样，显示的Warning需要处理。后来考虑，干脆直接就改```$PATH$```好了，虽然并不是系统推荐的方法，只是偷懒罢了

### 修改```$PATH$```

Bash中执行下列命令，修改```$PATH$```
```
sudo vim /etc/profile
```

在```export PATH```前，```PATH=XXX:/var/services/homes/YOURUSERNAME/.local/bin```
如此，就OK了。

### 配置Django

 sudo ln -s ~/.local/bin/django-admin /usr/local/bin/django-admin



## 启动Django

找一个合适的目录

```
django-admin startproject testdj
cd testdj
python3 manage.py runserver
```
结果出来一大堆提示
```
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).

You have 17 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.

March 05, 2020 - 16:59:57
Django version 2.2.11, using settings 'testdj.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```
于是按照提示，运行程序
```
 python3 manage.py migrate
```
接着有了这么多提示
```
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying auth.0010_alter_group_name_max_length... OK
  Applying auth.0011_update_proxy_permissions... OK
  Applying sessions.0001_initial... OK
```
回头再跑一圈刚才的```runserver```，又有了很多提示，显示正常工作啦~
```
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).
March 05, 2020 - 17:01:57
Django version 2.2.11, using settings 'testdj.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```



## 配置Web Station

目前，发现确实有问题：

- NAS默认的网络配置，需要通过Web Station进行，配置好之后，Web Station会将相关设置同步到Apache或者NginX。

- 已经找到Apache的配置文件，如virtual host之类，但是修改之后，需要重启Apache Server。这时有两个方法：一是重启Apache,会导致启动不成功，考虑后台有交叉校验的操作；而是重启Web Station，此时配置文件被重写……

## 解决之道

恐怕，还是需要安装Docker，在Docker里面在折腾相关的东西。