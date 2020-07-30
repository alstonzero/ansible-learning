# 变量

## 一、Ansible变量介绍

playbook类比成了Linux中的shell。那么它作为一门ansible特殊的语言，肯定要涉及到变量定义、控制结构的使用等特性。在这一节中主要讨论变量的定义和使用。

## 二、变量命名规则

变量的名字由字母、下划线和数字组成，必须以字母开头。

如下变量命名为正确：

```
good_a
ok_b
```

如下变量命名为错误：

```
_aaa
2_bb
```

**保留关键字不能作为变量名称**

## 三、变量类型

根据变量的作用范围大体的将变量分为：

- 全局变量
- 剧本变量
- 资产变量

但只是一个比较粗糙的划分，不能囊括ansible中的所有变量。下面将分别从这三种变量入手，去介绍变量的使用。

### 1、全局变量

全局变量，是我们使用ansible或使用ansible-playbook时，手动通过-e参数传递给ansible的变量。

通过ansible或ansible-playbook的help帮助，可以获取具体格式使用方式：

```yml
# ansible -h |grep var
 - e EXTRA_VARS, --extra-vars=EXTRA_VARS 
                        set additional variables as
                    
```

#### Example

传递普通的key=value的形式

```shell
# ansible all -i localhost, -m debug -a "msg='my key is {{key}}'" -e "key=value"
```

传递一个YAML/JSON的形式（注意不管是YAML还是JSON，它们的最终格式一定要是一个字典）

```shell
# cat a.json
{"name":"qfedu","type":"school"}
```

```shell
# ansible all -i localhost, -m debug -a "msg='name is {{name}},type is {{type}}'" -e @a.json
```

```yaml
# cat a.yml
---
name: qfedu
type: school
...
```

```yml
# ansible all -i localhost, -m debug -a "msg='name is {{name}}',type is {{type}}" -e @a.yml
```

### 2.playbook(剧本)变量

此种变量和playbook有关，定义在playbook中的。它的定义方式有多种，我们这里介绍两种最常用的定义方式。

#### 通过play属性vars定义

```yml
---
- name: test play vars
  hosts: all
  vars:
    user: lilei
    home: /home/lilei
...
```

#### 通过play属性，var_files定义

当通过vars属性定义的变量很多时，这个play就会感觉特别臃肿。此时我们可以将变量单独从play中抽离出来。形成单独的yaml文件。

```yml
---
- name: test play vars
  hosts: all
  vars_files:
    - vars/users.yml
...
```

```yml
# cat vars/users.yml
---
user: lilei
home: /home/lilei
...
```

#### 如何在playbook中使用这些变量

在playbook中使用变量时，使用{{变量名}}来使用变量。

```yaml
---
- name: test play vars
  hosts: all
  vars:
    user: lilei
    home: /home/lilei
  tasks:
    - name: create the user {{user}}
      user:
        name: "{{user}}"
        home: "{{home}}"
```

### 在playbook中使用变量的注意点

```yml
---
# 这里我们将上面的playbook中引用变量的部分进行修改，去掉了双引号。
- name: test play vars
  hosts: all
  vars:
    user: lilei
    home: /home/lilei
  tasks:
    - name: create the user {{user}}
      user:
        # 注意这里将"{{user}}"改成了{{user}}
        name: {{user}}
        home: "{{home}}"
```

执行以上的playbook时，会出现以下错误。

```


```

这样错误的主要原因是playbook是yaml的文件格式，当ansible分析yaml文件时，有可能误以为类似

name:{{user}}是一个字典的开始。因此加针对变量的使用，加上了双引号，避免ansible错误解析。

### 3、资产变量

在之前的课程中学习了资产。资产共分为静态资产和动态资产。
这一节中学习的资产变量，就是和资产紧密相关的一种变量。
资产变量分为**主机变量**和**组变量**，分别针对资产中的单个主机和主机组。
#### 3.1 主机变量
添加到主机名或主机IP后面。例子：在web_servers添加变量user和port
```
[web_servers]                                                                              
111.171.216.69 user=alston port=2222
 
[db_servers]
111.171.208.123
 
[all_servers]
[all_servers:children]
web_servers
db_servers
 
[all_servers:vars]
ansible_port=22 
ansible_user=root 
ansible_ssh_private_key_file=/home/alston/.ansible/cp/id_rsa

```
#### 验证
- 获取定义的变量值
```
#ansible web_servers -i inventory.ini -m debug -a "msg='user:{{user}} port:{{port}}'"
111.171.216.69 | SUCCESS => {
    "msg": "user:alston port:2222"
}

```
- 未获取到定义的变量值，因为user这个变量对于111.171.208.123无效
```
111.171.208.123 | FAILED! => {
    "msg": "The task includes an option with an undefined variable. The error was: 'user' is undefined"
}

```
#### 3.2主机组变量
以下inventory中，定义了一个组变量home，此变量将针对webservers这个主机组中的所有的服务器有效。
```
[web_servers:vars]
home="home/alston"
```

#### 3.3 变量的优先级
主机变量优先级高于组变量

#### 3.4变量的继承
例子：all_servers组的成员是从web_servers和db_servers中继承过来的。
```
[all_servers]
[all_servers:children]
web_servers
db_servers
```
### 3.5inventory内置变量的说明
内置变量几乎都以`ansible_`为前缀

### Facts变量
Facts变量不包含在前文中介绍的全局变量、剧本变量及资产变量之内。
Facts变量不需要我们人为去声明变量名及赋值。
它的声明和赋值完全由ansible中setup模块帮我们完成。
它收集了有关被管理服务器的操作系统版本、服务器IP地址、主机名，磁盘的使用情况、CPU个数、内存大小等等有关被管理服务器的私有信息。
在每次playbook运行的时候都会发现在playbook执行前都会有一个Gathering Facts的过程。这个过程就是收集被管理服务器的Facts信息过程。
#### 4.1手动收集Facts变量
```
ansible all -i inventory.ini -m setup 
```
由于输出的信息实在太多。因此，需要过滤进行处理。
#### 4.2 filter参数去过滤信息
- 比如过滤出内存的信息
```
ansible all -i inventory.ini - m setup -a "filter=*memory*"
```
- 仅获取服务器的磁盘挂载情况
```
ansible all -i inventory.ini -m setup -a "filter=*mount*" 
```
#### 4.3在playbook中去使用Facts变量
默认情况下，在执行playbook的时候，它会去自动的获取每台被管理服务器的facts信息。用变量获取即可。
```
# cat testFacts.yml 
---
- name: a play example
  hosts: all
  remote_user: root
  tasks:
    - name: print facts variable
      debug:
        msg: "The default IPV4 address is {{ansible_default_ipv4.address}}"
```
输出结果
```
# ansible-playbook -i inventory.ini testFacts.yml 

PLAY [a play example] *********************************************************************

TASK [Gathering Facts] ********************************************************************
ok: [111.171.208.123]
ok: [111.171.216.69]

TASK [print facts variable] ***************************************************************
ok: [111.171.216.69] => {
    "msg": "The deafult IPV4 address is 111.171.216.69"
}
ok: [111.171.208.123] => {
    "msg": "The deafult IPV4 address is 111.171.208.123"
}

PLAY RECAP ********************************************************************************
111.171.208.123            : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
111.171.216.69             : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
#### 在playbook中去关闭Facts变量的获取
- 若在整个playbook的执行过程中，完全未使用过Facts变量，此时我们可以将其关闭`gather_facts:no`，以加快playbook的执行速度。
```
---
- name: a play example
  hosts: all
  gather_facts: no
  remote_user: root
  tasks:
    - name: print hello world
      shell: echo "Hello World"
```

### 5.注册变量
往往用于保存一个task任务的执行结果,以便于debug时使用。
或者将此次task的结果作为条件，去判断是否去执行其他task任务
注册变量在playbook中通过`registor`键字去实现。
```
---
- name: install a package and print the result
  hosts: web_servers
  remote_user: root
  tasks:
    - name: install nginx package
      yum: name=nginx state=present
      register: install_result
    - name: print result
      debug: var=install_result
```
#### 执行
```
# ansible-playbook myplaybook3.yml

```
### 6.变量优先级
目前介绍了
其中Facts变量不需要人为去声明、赋值；注册变量只需要通过关键自register去声明，而不需要赋值。
而全局变量、剧本变量及资产变量则完全需要人为的去声明、赋值。
变量的优先权讨论，也将着重从这三类变量去分析。
假如在使用过程中，我们同时在全局变量、剧本变量及资产变量声明了同一个变量名，那么哪一个优先级最高呢？
下面我们将以实验的形式去验证变量的优先级。
#### 环境准备
- 1、定义一份资产、而定义了资产变量user
```
[db_servers]

[web_servers]

```

```
