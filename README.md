

- ansible_password 使用Vault加密存放
```
# password123 为vault的密码
# abcd1234 为要加密的string
echo "password123" >> .vault_host_ssh_pass_staging.txt

ansible-vault encrypt_string 'abcd1234' \
--name ansible_password \
--vault-id staging@.vault_host_ssh_pass_staging.txt

#结果
Encryption successful
ansible_password: !vault |
          $ANSIBLE_VAULT;1.2;AES256;staging
          66626461313830636534333261633466663332336338633134633334333439326433343335346666
          3232613766613039616330653363666339616339373262610a303030663034353763646333623138
          34396431393162383931313332363038383039653665396333313961333936366633306639383232
          3762363266383438660a396661326139656338323530626162623130373234646638306663623561
          3130


# 将 ansible_password 放在 group_vars/host_vars 看自己情况
###
notes: 
1.ansible不支持在同一个inventory下children有同名的，
    如下，都叫 web1 是不被允许的
        web:
            web1
        db:
            web1 

2. group_vars下的文件名必须要和对应的group相同
all:
  children:
    web:
      children:
        market:  --> 同这里的名称
          hosts:
            nginx1: --> 其他的hosts不能再叫这个名字
              ansible_host: 192.168.3.191
              ansible_user: HK-STAGING-ANS
        hotfix:
          hosts:
            nginx2: --> 不能同其他的hosts同名
              ansible_host: 192.168.3.192
              ansible_user: HK-STAGING-ANS

所以group_vars/market.yaml

3. Ansible ssh 过去是默认进去用户的家目录

###
# 调用
ansible-playbook webservers.yml \
-i inventories/staging/hosts.yaml \
-e target_hosts=market \
--vault-id staging@.vault_host_ssh_pass_staging.txt


ansible-playbook when.yaml -i inventories/staging/hosts.yaml -e target_group=market -e ansible_password=abcd1234
```









# 常见错误
1. to use the 'ssh' connection type with passwords or pkcs11_provider, you must install the sshpass program
```
tar -xvf sshpass-1.10.tar.gz &&\
cd sshpass-1.10 &&\
./configure && make && make install
```
2. Using a SSH password instead of a key is not possible because Host Key checking is enabled and sshpass does not support this.  
Please add this host's fingerprint to your known_hosts file to manage this host.
```
# 1. 获取配置文件路径
[root@localhost ansible]# ansible --version
ansible [core 2.18.6]
  config file = None

#2. 用户可以修改一下配置文件来修改设置,他们的被读取的顺序如下:
ANSIBLE_CONFIG (一个环境变量)
ansible.cfg (位于当前目录中)
.ansible.cfg (位于家目录中)
/etc/ansible/ansible.cfg

#3. 没有的话生成一个.直接放在项目的根目录下
ansible-config init --disabled > ansible.cfg

#4. 修改如下设置. 不检查known_hosts
host_key_checking = False

```

3. Could not import the dnf python module using /usr/bin/python3 (3.12.1 (main, Jun  3 2025, 08:22:40) [GCC 8.5.0 20210514 (Red Hat 8.5.0-4)]). 
Please install `python3-dnf` package or ensure you have specified the correct ansible_python_interpreter. 
(attempted ['/usr/libexec/platform-python', '/usr/bin/python3', '/usr/bin/python'])", "results": []