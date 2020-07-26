# playbook的任务控制

[TOC]

## 任务控制

这里主要来介绍playbook中的任务控制。

任务控制类似于编程语言中的if...、for...等逻辑控制语句。

这里我们给出一个实际应用案例去说明在playbook中，任务控制如何应用。

### Example

在下面的playbook中，我们创建了tomcat、www和mysql三个用户。

安装了







### 任务控制-判断

- 添加了register，debug。并为启动nignx设置了条件。当满足when的条件时，才会执行systemd

```yml
---
- name: task control playbook example
  hosts: webservers
  
  tasks:
    - name: add nginx repo
      yum_repository: name=nginx baseurl='' description=''
    - name: yum nginx webserver
      yum: name=nginx state=present
    - name: add virtualhost config
      copy: src=nginx.conf dest=/etc/nginx
    - name: check nginx syntax
      shell: /usr/sbin/nginx -t
      register: nginxsyntax
    - name: print nginx syntax
      debug: var=nginxsyntax
    - name: start nginx server
      systemd: name=nginx state=started
      when: nginxsyntax.rc == 0
```



### 控制任务——循环


#### 使用循环创建三个用户

- 使用with_items对应一个列表createuser，每次执行user模块，变量item从createuser中获取。

```yml
hosts: webservers
vars: 
  createuser:
    - tomcat
    - www
    - mysql
tasks:
   - name: create user
     user: name="{{item}}" state=present
     with_items: "{{createuser}}"
```
**注意**：变量必须加引号，否则会被当作是一个字典而无法识别。



#### 使用loop关键字，循环控制的例子

```yml
- name: loop item
  hosts: all
  gather_facts: no
  vars:
    some_list:
      - a
      - b
      - c
  tasks:
    - name: show item
      debug:
        var: "{{item}}"
      loop: "{{some_list}}"
```

#### loop循环加条件判断

- 只有当when:item>3时，才会输出num_list结果

```yml
- name: loop item
  hosts: all
  gather_facts: no
  vars:
    some_list:
      - a
      - b
      - c
    num_list:
      - 1
      - 2
      - 3
      - 4
      - 5
  tasks:
    - name: show item
      debug:
        var: "{{item}}"
      loop: "{{some_list}}"
    - name: show item
      debug:
        var: "{{item}}"
      loop: "{num_list}}"
      when: item > 3	
```



## Tags属性

考虑这样一个情况：

若更新了Nginx的配置文件后，我们需要通过playbook将新的配置发布到生产服务器上，然后再重新加载我们的nginx服务。但以现在的playbook来说，每次更改nginx配置文件后虽然可以通过它发布到生产，但整个playbook都要执行一次，这样无形中扩大了变更范围和变更风险。

下面的Tags属性就可以解决目前playbook变更而导致的扩大变更范围和变更风险的问题。

