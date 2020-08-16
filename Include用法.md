[TOC]

## Include 模块(old version)

### 在tasks中使用include模块

通过include，我们可以在一个playbook中包含另一个文件。

例子：假设，我想要编写两个playbook，这两个playbook分别用于安装LAMP环境和LNMP环境，两个playbook的大致内容如下：

- 安装LAMP的playbook

```yml
# cat lamp.yml
---
- hosts: test70
  remote_user: root
  gather_facts: no
  tasks:
  - name: install mysql
    yum:
      name: mysql
      state: present
  - name: install php-fpm
    yum:
      name: php-fpm
      state: present
  - name: install apache
    yum: 
      name: httpd
      state: present
    
```

- 安装LNMP的playbook

```yml
# cat lnmp.yml
---
- hosts: test70
  remote_user: root
  gather_facts: no
  tasks:
  - name: install mysql
    yum:
      name: mysql
      state: present
  - name: install php-fpm
    yum:
      name: php-fpm
      state: present
  - name: install nginx
    yum:
      name: nginx
      state: present
```

无论在`lamp.yml`中还是在`lnmp.yml`中，安装mysql和php部分的任务完全相同的，所以，我们可以把这两个任务提取出来，作为一个逻辑单元。

- 把安装mysql和php部分的任务提取到`install_MysqlAndPhp.yml`文件中

```yml
# cat install_MysqlAndPhp.yml

- name: install mysql
  yum: 
    name: mysql
    state: present
    
- name: install php
  yum:
    name: php-fpm
    state: present
```

当需要安装mysql和php-fpm时，只需要调用此yml文件即可

- 使用include模块修改后的`lamp.yml`和`lnmp.yml`

```yml
# cat lamp.yml
---
- hosts: test70
  remote_user: root
  gather_facts: no
  tasks:
  - include: install_MysqlAndPhp.yml
  - name: install apache
    yum:
      name: httpd
      state: present

# cat lnmp.yml
---
- hosts: test70
  remote_user: root
  remote_user: root
  gather_facts: no
  tasks:
  - include: install_MysqlAndPhp.yml
  - name: install nginx
    yum: 
      name: nginx
      state: present
```

使用了`include模块`，引用了`install_MysqlAndPhp.yml`文件，当我们引用此文件时，`install_MysqlAndPhp.yml`文件中的tasks都会在被引用处执行，这就是include的用法

### handlers中使用include模块



### 使用include引用playbook

- `lamp.yml`的结尾引入了`lnmp.yml`，当我们在执行`lamp.yml`时，会先执行lamp相关的任务，然后再执行`lnmp.yml`中的任务。(**注意新版必须改成include_tasks**)

```yml
# cat lamp.yml
---
- hosts: test70
  remote_user: root
  gather_facts: no
  tasks:
  - include: install_MysqlAndPhp.yml
  - name: install apache
    yum:
      name: httpd
      state: present

- include: lnmp.yml
```



### 把vars传入include模块引用的文件中



### 用tags给include打标签



### 给include添加条件判断和循环

- 将需要循环调用的多个任务写入到一个yml文件中，然后使用include调用这个yml文件，再配合loop进行循环

```yml
# cat test_include1.yml
---
- hosts: test70
  remote_user: root
  gather_facts: no
  tasks:
  #给include添加条件判断
  - include: setup-RedHat.yml
    when: 
```





## include_tasks模块(new version)



### 在tasks中使用include_tasks

- include_tasks模块和include模块的用法完全相同。都是用来包含一个任务列表

例子：

```yml
# cat intest.yml
---
- hosts: web_server1
  remote_user: root
  gather_facts: no
  tasks:
  - debug:
      msg: "test tasks1"
  - include_tasks: in.yml
  - debug: 
      msg: "test task2"
      
# cat in.yml
- debug:
    msg: "task1 in in.yml"
- debug:
    msg: "task2 in in.yml"
```

输出结果：

```bash
[root@gateway ansible-learning]# ansible-playbook -i inventory.ini intest.yml 

PLAY [web_server1] *************************************************************

TASK [debug] *******************************************************************
ok: [web_server1] => {
    "msg": "test task1"
}

TASK [include_tasks] ***********************************************************
included: /root/ansible-learning/in.yml for web_server1

TASK [debug] *******************************************************************
ok: [web_server1] => {
    "msg": "task1 in in.yml"
}

TASK [debug] *******************************************************************
ok: [web_server1] => {
    "msg": "task2 in in.yml"
}

TASK [debug] *******************************************************************
ok: [web_server1] => {
    "msg": "test task2"
}

PLAY RECAP *********************************************************************
web_server1                : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

可以看出：控制台输出了`include_tasks`。

- `include_tasks模块`与`include模块`之间的区别：当我们使用`include_tasks`时，`include_tasks`本身会被当做一个**task**，这个**task**会把被include的文件的路径输出在控制台中。如果将上例中的`include_tasks`关键字替换成`include`，控制台中则不会显示上图中标注的任务信息。由此可见"include"是透明的，"include_tasks"是可见的，"include_tasks"更像是一个任务，这个任务包含了其他的一些任务。



### include_tasks模块中file参数和apply参数

#### file参数

- free_form形式包含文件

```
- include_tasks: in.yml
```

- 使用file参数包含文件

```yml
- include_tasks:
    file: in.yml
```

#### apply参数

- 背景：tags只能调用`include_tasks`本身，并不能调用对应文件中的任务

例子：

```yml
# cat intest.yml
---
- hosts: test70
  remote_user: root
  gather_facts: no
  tasks:
  - debug:
      msg: "test task1"
  - include_tasks:
      file: in.yml
    tags: t1
  - debug:
      msg: "test task2"
```

输出结果：

```bash
[root@gateway ansible-learning]# ansible-playbook -i inventory.ini intest.yml --tags t1

PLAY [web_server1] *************************************************************

TASK [include_tasks] ***********************************************************
included: /root/ansible-learning/in.yml for web_server1

PLAY RECAP *********************************************************************
web_server1                : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

从执行后的输出信息可以看出，当我们指定t1标签后，`include_tasks`这个任务本身被调用了，而`include_tasks`对应文件中的任务却没有被调用，所以我们可以得出结论，在使用tags时，`include_tasks`与`include`并不相同，标签只会对`include_tasks`任务本身生效，而不会对其中包含的任务生效。

- 使用apply参数来解决

不仅需要在apply参属下添加tags，而且需要在include_tasks下添加tags并标为always

```
---
- hosts: web_server1
  remote_user: root
  gather_facts: no
  tasks:
  - include_tasks:
      file: in.yml
      apply:
        tags:
        - t1
    tags: always
```

输出结果：

```bash
[root@gateway ansible-learning]# ansible-playbook -i inventory.ini  intest.yml --tags t1

PLAY [web_server1] *************************************************************

TASK [include_tasks] ***********************************************************
included: /root/ansible-learning/in.yml for web_server1

TASK [debug] *******************************************************************
ok: [web_server1] => {
    "msg": "task1 in in.yml"
}

TASK [debug] *******************************************************************
ok: [web_server1] => {
    "msg": "task2 in in.yml"
}

PLAY RECAP *********************************************************************
web_server1                : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

**注意**：在使用`include_tasks`时，不仅使用apply参数指定了tags，同时还使用tags关键字，对`include_tasks`本身添加了always标签。



