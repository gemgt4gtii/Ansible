# Ansible—(LAMP)

# 安裝(Centos.7)
    sudo yum -y install ansible
# 設定配置
## 添加主機： /etc/ansible/hosts
    sudo vim /etc/ansible/hosts 
![](https://d2mxuefqeaa7sj.cloudfront.net/s_974313649A1C19777D197E006FCDD42FC3EAFDC9D8A1D52C04E4CD1A249374BD_1553742992339_image.png)


文件格式

    [   ]      自定義名稱
    主機IP:PORT 連接參數

參數說明

    ansible_ssh_host         將要連線的遠端主機名.與你想要設定的主機的別名不同的話,可通過此變數設定.
    ansible_ssh_port         ssh埠號.如果不是預設的埠號,通過此變數設定.
    ansible_ssh_user         預設的 ssh 使用者名稱
    ansible_ssh_pass         ssh 密碼(這種方式並不安全,我們強烈建議使用 --ask-pass 或 SSH 金鑰)
    ansible_sudo_pass        sudo 密碼(這種方式並不安全,我們強烈建議使用 --ask-sudo-pass)
    ansible_sudo_exe (new in version 1.8)sudo 命令路徑(適用於1.8及以上版本)
    ansible_connection       與主機的連線型別.比如:local, ssh 或者 paramiko. Ansible 1.2 以前預 設使用 paramiko.1.2 以後預設使用 'smart','smart' 方式會根據是否支援 ControlPersist, 來判斷'ssh' 方式是否可行.
    ansible_ssh_private_key_file   ssh 使用的私鑰檔案.適用於有多個金鑰,而你不想使用 SSH 代理的情況.
    ansible_shell_type       目標系統的shell型別.預設情況下,命令的執行使用 'sh' 語法,可設定為 'csh' 或 'fish'.
    ansible_python_interpreter     目標主機的 python 路徑.適用於的情況: 系統中有多個 Python, 或者命令路徑不是"/usr/bin/python",比如  \*BSD, 或者 /usr/bin/python

測試連線：

    ansible web(定義的名稱) -m ping -o
![](https://paper-attachments.dropbox.com/s_974313649A1C19777D197E006FCDD42FC3EAFDC9D8A1D52C04E4CD1A249374BD_1553744801844_image.png)

## 設定檔： /etc/ansible/ansible.cfg
    sudo vim /etc/ansible/ansible.cfg
## 同步時間：
     ansible all -m shell -a 'echo "TZ='Asia/Shanghai'; export TZ" > /etc/profile '
![](https://paper-attachments.dropbox.com/s_974313649A1C19777D197E006FCDD42FC3EAFDC9D8A1D52C04E4CD1A249374BD_1553745122218_image.png)

## 定期同步時間
    ansible all -m cron -a "minute=*/3 job='/usr/sbin/ntpdate ntp1.aliyun.com &> /dev/null' name=dateupdate"
![](https://paper-attachments.dropbox.com/s_974313649A1C19777D197E006FCDD42FC3EAFDC9D8A1D52C04E4CD1A249374BD_1553747145493_image.png)

## 關閉firewalld和SELinux
    ansible all -m shell -a 'systemctl stop firewalld; systemctl disable firewalld; setenforce 0'
![](https://paper-attachments.dropbox.com/s_974313649A1C19777D197E006FCDD42FC3EAFDC9D8A1D52C04E4CD1A249374BD_1553747295187_image.png)

## 建立金鑰
    ssh-keygen -t rsa -N ''
    
    ssh-copy-id -i ~/.ssh/id_rsa.pub root@ip
# 安裝LNMP

將主機IP添加至目錄底下的 hosts

    [名稱]
    IP

修改項目根目錄下的main.yml，來確定MySQL，nginx，PHP的版本

    ```bash
    ＃示例：安裝mysql5.5.49版本和nginx1.8.0版本，nginx180配置來自於group_vars / all文件
    ---
    - hosts: lnmp
      remote_user: root
    
      roles:
        - {role: env}
        # - {role: mysql, mysql: "{{ mysql5549 }}"}
        - {role: nginx, nginx: "{{ nginx180 }}"}

如有必要,修改group_vars/all文件来修改

    # 範例:nginx1.8.0版本的相關配置
    nginx180:
      version: 1.8.0
      configure_args: >
        --user=$DAEMON_USER --group=$DAEMON_USER --prefix=$BASEDIR --with-http_stub_status_module --with-http_ssl_module --with-pcre --with-http_realip_module
    
    nginx_base_dir: /usr/local/nginx
    nginx_web_dir: /data/web/www 
    nginx_log_dir: /data/web/log
    nginx_daemon_name: nginxd
    nginx_daemon_user: nginx
## 執行安裝
    ansible-playbook -i hosts main.yml

