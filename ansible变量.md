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

