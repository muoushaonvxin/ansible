
## Ansible相互互信

<br>

编辑用户自定义管理配置文件
```shell
[root@zhangyz ~]# vim /etc/ansible/hosts                          
[test]
192.168.5.201
192.168.5.200

[nginx]
10.203.206.235
10.203.206.236
```

指定组名字然后对其进行测试
```shell
[root@zhangyz ~]# ansible nginx -m ping
10.203.206.235 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
10.203.206.236 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```

在平常的情况下ansible使用root的权限来进行管理
```shell
[webservers]
site01  ansible_ssh_user=root
site02  ansible_ssh_user=daniel
site01-dir  ansible_ssh_host=site01.dir ansible_ssh_port=65422

[production]
site01
site02
db01  ansible_ssh_user=fred
ansible_ssh_private_key_file=/home/fred/ssh/id_rsa  bastion

[mfs:children]
mfs_master
mfs_logger
mfs_node
mfs_client

[mfs_master]
172.16.2.202
[mfs_logger]
172.16.5.201
[mfs_node]
172.16.5.200
[mfs_client]
172.16.2.190
```


对于单个主机进行操作

ansible mfs_master -m  ping

ansible mfs_node -m ping



## 先相互配置好密钥认证ansible的好处
1) 轻量级
2) 不用安装客户端以ssh互信的方式工作
 
ansible的一些常用内置模块的操作

--------------------------------------------
| 命令参数 | 命令参数表示的作用                |
--------------------------------------------
-m modele_name 表示所使用的模块名称。
 
-a argument      跟在模块之后进行调用，使用命令参数来进行配置。
 
ansible web -m ping
 
 
 
ansible-doc  表示查看ansible的文档
 
ansible-doc -l
 
ansible-doc -s ping
 
 
 
 
 
指定主机并且在主机上创建用户
 
         ansible web -m user -a 'name=carol state=present shell=/sbin/nologin'
 
 
指定主机并且删除机器上的用户
 
         ansible web -m user -a 'name=carol state=absent'
 
 
指定主机并且创建文件
 
         ansible web -m file -a 'path=/tmp/testfile state=present owner=student group=root mode=0666'
 
         ansible web -m file -a 'path=/tmp/testfile state=touch owner=student group=root mode=0666'
 
 
copy 模块的作用
 
         ansible web -m copy -a 'dest=/srv/carolfile src=/tmp/carolfile'
 
         ansible web -m copy -a 'dest=/tmp/testdir src=/tmp/testdir directory_mode'
 
 
把目录传输过去
 
         ansible web -m copy -a 'dest=/tmp/ src=/tmp/testdir directory_mode'
 
 
 
command 模块的作用
 
         ansible web -m at -a '"command=tar -czf /tmp/test.tgz /etc" state=present count=5 units=minutes'
 
 
 
at 模块的作用
 
         ansible web -m at -a 'command="tar -czf /tmp/test.tgz /etc" state=present count=5 units=minutes'
 
 
 
cron 模块的作用
 
         ansible web -m cron -a 'name=test state=present job="touch /tmp/testfie1" weekday=5 hour=15 minute=0 day=* month=*'
 
 
 
service  模块的作用
 
         ansible web -m service -a 'name=vsftpd state=started enabled=true'
 
 
 
    *.*  在使用自动化工具的时候，会涉及到非常多的参数，这些参数如果用脑子记，没有多少人能够记住，
所以，只能 ansible-doc -s [[module_name]] 进行帮助。.
 
    *.*  只要是执行成功，但是，最终没有结果了，一定是少加了个参数，没有其它的。
Ansible-PlayBooks
 




ansible mfs_node -a 'ls /usr/local/src/'

对文件进行压缩
ansible mfs_node -a 'tar zcf /usr/local/src/ass.tar.gz /usr/local/src/jdk-6u45-linux-amd64.rpm'

ansible mfs_node -a 'ls /usr/local/src/'


ansible mfs_node -a 'chdir=/usr/local/src/ tar zcf bbb.tar.gz jdk-6u45-linux-amd64.rpm'


============================================================================================================

ansible ansible_node -m shell -a 'ps -ef | grep rpcbind'

ansible ansible_node -m raw -a 'ps -ef | grep rpcbind'

--------------------------------------------------------------------------------------------------------------------------------------------

ansible mfs_node -m service -a 'name=nginx state=started enabled=yes'

ansible mfs_node -m raw -a 'ps -ef | grep nginx'

ansible mfs_node -m service -a 'name=nginx state=stopped'

ansible mfs_node -m raw -a 'ps -ef | grep nginx'

ansible mfs_node -m service -a 'name=nginx state=restarted'


ansible   mfs_node   -m  service  -a    'name=nginx     state=restarted  sleep=3'

ansible     mfs_logger   -m   cron   -a    'name="reboot   system"   hour=2  user=root   job="/sbin/reboot"'


ansible   mfs_logger   -m  cron    -a   'name="check home directory"  minute=*/3  job="ls  -lh   /home" '

ansible     mfs_logger   -m   command   -a   'crontab -l'


ansible   mfs_logger   -m  cron  -a  'name="check home directory"    special_time=reboot   job="echo reboot" '

ansible    mfs_logger   -m   command  -a  'crontab  -l'


ansible    mfs_logger   -m  cron   -a   'name="check  home  directory"   state=absent '


ansible   mfs_logger    -m   command   -a   'ls   /etc/yum.repos.d/'

ansible    mfs_logger   -m  file   -a   'path=/etc/yum.repos.d/163.repo   state=absent'

ansible     mfs_logger   -m  command  -a  'ls   /etc/yum.repos.d/'


ansible    all    -m    copy  -a  'src=/etc/yum.repos.d/163.repo     dest=/etc/yum.repos.d/163.repo'



安装软件

ansible   mfs_logger   -m yum   -a   'name=httpd   state=installed'

ansible   mfs_logger   -m   service   -a  'name=httpd  state=started'

ansible    mfs_logger    -m shell   -a  'ps  -ef  | grep httpd'



创建一个用户

ansible    mfs_logger   -m   command  -a   'useradd  -s  /sbin/nologin   -M  user1'

ansible    mfs_logger   -m   user   -a   'createhome=yes   home=/home/user2    password=123456   name=user2   state=present    shell=/bin/bash'


删除用户
ansible    mfs_logger    -m   user    -a  'remove=yes   state=absent  name=user2'

========================================================================================================


同步模块

ansible   mfs_logger  -m  rynchronize   -a   'src=/tmp/helloworld    dest=/var/ww'

ansible   mfs_logger   -m   file  -a 'path=/var/www/'


 
 
 
通过命令行的某些操作一次只能做一件事情，ansible-playbook 来进行多个事情，并且重复。
 
剧本： 文件
 
yaml的语法格式：
 
xxx.yml 作为结尾
 
yaml文件当中有几个重要的组成部分：
 
缩进一定要符合其要求
 
主机组
 
remote_user： 用户身份
 
变量部分
 
任务部分 tasks
 
后续组成部分 handlers
 
 
vim test.yml
 
- hosts: web
  remote_user: root
  tasks:
 
  - name: install apache
    yum: name=httpd state=present
 
  - name: create user
    user: state=present name=tom
 
  
 
 
 
执行该配置文件：
 
ansible-playbook test.yml
 
 
 
 
 
 
 
 
 
 
 
vim test.yml
 
 
- hosts: web
  remote_user: root
  vars:
        server_name: www.servera.com
  tasks:
 
  - name: install httpd
    yum: name=httpd state=present
  
  - name: start httpd
    service: name=httpd state=started enabled=true
 
  - name: copy config file
    template: src=/etc/ansible/roles/templates/{{ server_name }}.conf dest=/etc/httpd/conf.d/www.{{ ansible_hostname }}.conf
    
  - name: create directory
    file: state=directory dest=/var/www/html/{{ ansible_hostname }}.com owner=apache group=apache
 
    notify:
          - restart httpd
 
  handlers:
  - name: restart httpd
    service: name=httpd state=restarted
 
 
 
- hosts: db
  remote_user: root
  tasks:
  
  - name: install mariadb-server
    yum: name=mariadb-server state=present
 
  - name: start mariadb
    service: name=mariadb state=started enabled=true
 
 
 

PlayBook  是由一个或多个  play  组成的列表，  play 的主要功能在于将事先归并为一组的主机装扮成事先
通过ansible 中的task 定义好的角色，  从根本上来讲，  所谓tash 无非是调用 ansible的 一个module， 将多个
play组织在一个playbook 中   即可以让他们联合起来按事先编排的机制完成某一任务


ansible  的配置文件

playbook  --------------   管理主机  ----------------- yaml 语法格式

modules  来完成模块

- hosts: mfs_node
user: root
vars:
- motd_warning:  'WARNING: Use by ACME Employees ONLY'
tasks:
- name: setup a MOTD
copy:  dest=/etc/motd  content="{{ motd_warning }}"
notify: say something
handlers:
- name: say something
command: echo "copy OK"



Target  section
定义将要执行playbook 的远程主机组

Variable section
定义playbook  运行时需要使用的变量

Task section
定义将要在远程主机上执行的任务列表

Handler section
定义task执行完成以后需要调用的任务





hosts； 定义远程的主机组
user： 执行该任务组的用户
remote_user：  与user 相同
sudo： 如果设置为 yes  ，  执行该任务组的用户在执行任务的时候， 获取root权限
sudo_user：   如果你设置 user 为 tom， sudo 为yes ， sudo_user 为 jerry  ， 则tom用户则会
获取 jerry 用户的权限
connection：   通过什么方式连接到远程主机， 默认为ssh
gather_facts：  除非你明确说明不需要在远程主机上执行 setup 模块， 否则默认为自动执行。 如果你确实不需要
setup模块所传递过来的变量， 你可以启用该选项。


=================================================================================

cd  /etc/ansible/

ansible   mfs_node  -m   setup                   //   收集远程主机的一些信息


========================================================

存在 playbook  当中的变量

vim variable
name: httpd
port: 80

vim test.yaml

vars:
- hosts: all
user:  root
vars_files:
- variables

tasks:
- home:  print IP
template:  src=files/test1.txt   dest=/tmp/test1.txt


rm  -rf      /tmp*

==============================================================================

ansible-playbook   test1.yaml


vars_prompt                当需要和用户进行交互的时候， 需要通过交互的方式来输入其中的值

---

- host: all
vars_prompt:
- name: http
prompt:  'please  enter  something: '
private: yes

tasks:
- name:  just   a test
template:   src=files/test.txt    dest=/tmp/test.txt       




可以取出大小
ansible_device.sda.partition.sda1.size


==========================================================

tasks:
- name: install apache
action:  yum  name=httpd  state=installed        # 第一种方法

- name:  configure  apache
copy:  src=files/httpd.conf     dest=/etc/httpd/conf/httpd.conf  #  第二种方法

- name:   restart   apache
service:
name:  httpd
state:   restarted





---

- hosts:  mfs_node
user:  root
vars_files:
- variables:

tasks:
- name:  print IP
template:   src=files/test1.txt   dest=/tmp/test1.txt



===================================================================
-hosts:  all
user: root

tasks:
- name:  template  configuration file
template:  src=template.j2   dest=/etc/foo.conf
notify:
- restart  memcached
- restart  apache

handlers:
- name:  restart  memcached
service:  name=memcached   state=restarted
- name:   restart  apache
service:   name=httpd   state=restarted


 
 
配置模版文件,模版文件的位置可以随便定义 /etc/ansible/roles/templates
 
vim  www.servera.com.conf
 
<VirtualHost *:80>
         ServerName www.{{ ansible_hostname }}.com
         DocumentRoot /var/www/html/{{ ansible_hostname }}.com
</VirtualHost>
 


#### PlayBook 中的模块



PlayBook  Modules     

在playbook  中使用模块跟在命令行中使用有一点点不一样， 这主要是因为在playbooks 中有许多
从setup 模块或者之前的模块中获取的数据要处理， 有些模块在命令行中无法使用， 是因为他们
需要访问变量，  还有一些可以在命令行中使用的模块在 playbooks 中使用，拥有了更加强大的功能


template 


add_host  添加主机模块是playbook  中一个强大的模块，它可以让你动态的添加受管的主机到
一个play 中， 我们可以使用 uri  模块从 CMDB 中获得主机信息然后添加他们，  它还可以将主机添加
到组里面，  如果组不存在的话还会自动创建它， 这个模块仅需要主机名和组名 2 个参数，  跟主机库存
清单的配置一样， 我们还可以添加扩展的参数像  ansible_ssh_user   ansible_ssh_port  


---
-name: Create operating  system group
hosts: all

tasks:
- group_by:  key=os_{{ ansible_distribution }}

- name:  Run on CentOS  hosts only
hosts  os_CentOS

tasks:
- name:  installed  Apache
yum:  name=httpd   state=latest


- name:  Run on Ubuntu  hosts  only
hosts:   os_Ubuntu

tasks:
- name:  install Apache
apt:    pkg=apache2   state=lated



---------------------------------------------------------------------------------------------------------------------------------------------



在大型复杂架构中， 你第一个要面对的问题就是不断增长的playbooks文件大小，一个很大的playbooks 很难去理解和维护，解决的办法就是
使用include   将你的plays 分解成为不同的段， 然后在其他的plays 中包含他们，  不同的段根据不同目的分类，全部包含在主play 中。

共有四种类型的包含：

变量包含：允许你将变量存在外部 YAML文件
playbook包含： 一个大型项目中可以包含多个plays
任务包含：将任务放到普通文件中， 当需要的时候包含他们
Handler包含： 允许你将所有的handlers 处理程序放到一个地方



vars_file:
- vars
- vars1

tasks:
- include:  tasks/foo.yml

handlers:
- include:

===============================================================

---
- hosts:  mfs_node
vars_files:
- vars.yml
- vars1.yml

tasks:
- include:  task.yml

handlers:
- include:  handler.yml

----------------------------------------------------------------------------------------------------------------------------





#### ansible variable 



什么时候定义ansible 的变量？

Variable Defined  in  Inventory

Variable  Defined  in   a   Playbook

Variables  Defined  in  Comandline

Registered  Variables


[web_servers]
app1_server   http_port=80

[webservers:vars]
some_server=foo.southeast.example.com
halon_system_timeout=30
self-destruct_countdown=60
escape_pods=2
/etc/ansible/host_vars/app1_server
/etc/ansible/group_vars/webservers


-hosts:  mfs_node
user: "{{ uservar }}"
tasks:
- shell: echo "{{ echovar }}"

ansible-playbook  command_vars.yml   -e  'uservar="root"  echovar="hello world" '
ansible-playbook  command_vars.yml   -e  '{"uservar":"root", "echovar":"hello world"}'
ansible-playbook  command_vars.yml   -e  '@test.json'

cat  test.json
uservar: root
echovar:  hello world



- hosts:  mfs_node
tasks:
-shell:  echo "5"
register:  result
ignore_errors:  True

-debug:  msg="it  failed"
when:  result | failedx

-debug:  msg="{{result.stdout}}"


-shell:  /usr/bin/bar
when:  result.rc == 5



