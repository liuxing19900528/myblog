**马哥的淘宝店:https://shop592330910.taobao.com/**


#ubuntu 安装gitlab
```
使用清华大学gitlab的镜像https://mirror.tuna.tsinghua.edu.cn/help/gitlab-ce/
```

```
curl https://packages.gitlab.com/gpg.key 2> /dev/null | sudo apt-key add - &>/dev/null
vi /etc/yum.repos.d/gitlab-ce.repo
```

```
[gitlab-ce]
name=gitlab-ce
baseurl=http://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7
repo_gpgcheck=0
gpgcheck=0
enabled=1
gpgkey=https://packages.gitlab.com/gpg.key
```


```
sudo yum makecache
sudo yum install gitlab-ce
```

```
然后打开/etc/gitlab/gitlab.rb,将external_url = 'http://xxx.xxx.com'
修改为自己的IP地址或者自己的域名，然后编译，这里会用chef来进行，sudo gitlab-ctl reconfigure
这样如果没有报错就是安装完成了
```

#配置ldap登录
```
### LDAP Settings
###! Docs: https://docs.gitlab.com/omnibus/settings/ldap.html
###! **Be careful not to break the indentation in the ldap_servers block. It is
###!   in yaml format and the spaces must be retained. Using tabs will not work.**
gitlab_rails['ldap_enabled'] = true
###! **remember to close this block with 'EOS' below**
gitlab_rails['ldap_servers'] = YAML.load <<-'EOS'
    main: # 'main' is the GitLab 'provider ID' of this LDAP server
     label: 'LDAP'
     host: 'xx.xx.xx.xx'
     port: 389
     uid: 'sAMAccountName'
     bind_dn: 'cn=username,cn=Users,dc=xxx,dc=com'
     password: 'password'
     encryption: 'plain' # "start_tls" or "simple_tls" or "plain"
     verify_certificates: true
     ca_file: ''
     ssl_version: ''
     active_directory: true
     allow_username_or_email_login: false
     block_auto_created_users: false
     base: 'dc=xxx,dc=com'
     user_filter: ''
     attributes:
       username: ['uid', 'userid', 'sAMAccountName']
       email:    ['mail', 'email', 'userPrincipalName']
       name:       'cn'
       first_name: 'givenName'
       last_name:  'sn'
#     ## EE only
#     group_base: ''
#     admin_group: ''
#     sync_ssh_keys: false
#
#   secondary: # 'secondary' is the GitLab 'provider ID' of second LDAP server
#     label: 'LDAP'
#     host: '_your_ldap_server'
#     port: 389
#     uid: 'sAMAccountName'
#     bind_dn: '_the_full_dn_of_the_user_you_will_bind_with'
#     password: '_the_password_of_the_bind_user'
#     encryption: 'plain' # "start_tls" or "simple_tls" or "plain"
#     verify_certificates: true
#     ca_file: ''
#     ssl_version: ''
#     active_directory: true
#     allow_username_or_email_login: false
#     block_auto_created_users: false
#     base: ''
#     user_filter: ''
#     attributes:
#       username: ['uid', 'userid', 'sAMAccountName']
#       email:    ['mail', 'email', 'userPrincipalName']
#       name:       'cn'
#       first_name: 'givenName'
#       last_name:  'sn'
#     ## EE only
#     group_base: ''
#     admin_group: ''
#     sync_ssh_keys: false
EOS
```

```
ldapsearch -vvv -LLL -H ldap://ldapip -b "dc=test,dc=com"  -D 'cn=账号,OU=系统账号,OU=xxxx科技有限公司,dc=test,dc=com' -w密码

xxxx科技有限公司  这个是在ad域里面分的一个目录，这个目录和默认的Users那个目录同级的。
系统账号  这个是 公司 下面分的一个子目录。
dc 一般是公司域名。
```

```
ldapsearch -vvv -LLL -H ldap://ldapip -b "dc=test,dc=com"  -D 'cn=账号,cn=Users,dc=test,dc=com' -w密码

这里的Users就是 在ad域里面的那个默认的组


```

#centos 安装gitlab

1. Install and configure the necessary dependencies

```
sudo yum install -y curl openssh-server openssh-clients cronie
sudo lokkit -s http -s ssh


```

2. Add the GitLab package server and install the package

```
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
sudo yum install -y gitlab-ce


```



3. Configure and start GitLab

```
sudo gitlab-ctl reconfigure

```
