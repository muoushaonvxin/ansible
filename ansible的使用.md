
先相互配置好密钥认证:
 
ansible 的好处
 
1.轻量级
2.不用安装客户端，以ssh互信的方式工作
 
 
 
ansible的一些常用内置模块的操作：
 
-m modele_name   表示所使用的模块名称。
 
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
 
 
 
 
 
 
 
配置模版文件,模版文件的位置可以随便定义 /etc/ansible/roles/templates
 
vim  www.servera.com.conf
 
<VirtualHost *:80>
         ServerName www.{{ ansible_hostname }}.com
         DocumentRoot /var/www/html/{{ ansible_hostname }}.com
</VirtualHost>
 
