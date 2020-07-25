## yaml语法
[菜鸟教程yaml](https://www.runoob.com/w3cnote/yaml-intro.html)

### 使用pyyaml来验证语法是否正确
- 安装pyyaml
```
pip3 install pyyaml
```
- 创建`myyaml.yml`文件
```
//正确的情况
# cat myyaml.yml
---
- red
- blue
- green
...
```
- 使用`pyyaml`验证
```
#python3 -c 'import yaml,sys;print(yaml.safe_load(sys.stdin))'< myyaml.yml

['red', 'blue', 'green']
```
这里调用了sys包，为的是使用标准输入

- 错误的情况，将yaml文件写错
```
# cat myyaml.yml
---
- red
- green
-blue
...
#python3 -c 'import yaml,sys;print(yaml.safe_load(sys.stdin))'<myyaml.yml
Traceback (most recent call last):
  File "<string>", line 1, in <module>
  File "/usr/lib/python3/dist-packages/yaml/__init__.py", line 162, in safe_load
    return load(stream, SafeLoader)
  File "/usr/lib/python3/dist-packages/yaml/__init__.py", line 114, in load
    return loader.get_single_data()
```

## Playbook的编写
### 1.play的定义
- 1、每一个play都是以短横杠开始的
- 2、每一个play都是一个yaml字典格式

根据上面两条play的规则，一个假想的play应该是如下的样子
```
---
- key1: value1
  key2: value2
  key3: value4
...
```

由于一个playbook是由一个或多个play组成，那么一个含有多个play的playbook结构上应该是这个样子。

```
---
#一个含有3个play的伪playbook构成结构
- key1: value1
  key2: value2
  key3: value3
- key4: value4
  key5: value5
  key6: value6
- key7: value7
  key8: value8
  key9: value9
...
```
### 2、play属性
play中的每一个key，比如key1,key2等；这些key在playbook中被定义为play的属性。
接下来就看看ansible本身都支持哪些play属性。

**常用属性**
- name属性，每个play的名字。（用来说明某个play是干什么）
- hosts属性，每个play涉及的被管理服务器，同ad-hoc中的资产选择器
- tasks属性，每个play中具体要完成的任务，以列表的形式表达
- become属性，如果需要提权(su提权，sudo提权等)，则加上become相关属性
- become_user属性，若提权的话，提权到哪个用户上
- remote_user属性，指定连接被管理节点的用户，就是在远程服务器上执行具体操作的用户。若不指定，则默认使用当前执行ansible playbook的用户。

### 3、一个完整的playbook
palybook里面url必须要''但是不需要转义字符
```

---
- name: the first play example
  hosts: all
  remote_user: root
  tasks:
    - name: add nginx repo source
      yum_repository: name=nginx baseurl='http://nginx.org/packages/centos/$releasever/$basearch/' description='nginx stable repo'
    - name: install nginx package
      yum: name=nginx state=present
    - name: copy nginx.conf to remote server
      copy: src=nginx.conf dest=/etc/nginx/nginx.conf
    - name: start nginx server
      systemd: 
        name: nginx 
        enableed: true
        state: started
```
### 4、tasks属性中任务的多种写法
####  以启动nginx服务，并增加开机启动为例
- 一行的模式
```
systemd: name=nginx enabled=true state=started
```
- 多行的模式
```
systemd: name=nginx
         enabled=true
         state=started
```
- 多行写成字典模式
```
systemd:
  name: nginx
  enabled: true
  state: started
```
### 5、多个play的playbook
```
---
- name: manage web_servers 
  hosts: web_servers
  remote_user: root
  tasks: 
    - name: add nginx repo source
      yum_repository: name=nginx baseurl='http://nginx.org/packages/centos/$releasever/$basearch/' description='nginx stable repo'
    - name: install nginx package
      yum: name=nginx state=present
    - name: copy nignx 
      copy: src=nginx.conf dest=/etc/nginx/nginx.conf
    - name: start nginx server
      systemd: name=nignx enabled=true state=started

- name: manager dbservers
  hosts: dbservers
  tasks:
    - name: update database config
      copy: src=my.cnf dest=/etc/my.cnf
 
 
```
