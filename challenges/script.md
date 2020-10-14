# 최종 Assessment - 사공태정 


### 접속

* Host1 : `ssh -i SKCC_Final.pem centos@3.35.99.0`

* Host2 : `ssh -i SKCC_Final.pem centos@52.78.113.180`

* Host3 : `ssh -i SKCC_Final.pem centos@52.78.121.88`

* Host4 : `ssh -i SKCC_Final.pem centos@52.78.15.180`

* Host5 : `ssh -i SKCC_Final.pem centos@52.78.186.31`


### 환경 세팅

* 비밀 번호 세팅

`passwd centos`

_password : centos_
 
`sudo vi /etc/ssh/sshd_config`

```
PasswordAuthentication yes
```

`sudo service sshd restart`

`ssh centos@[ip]`

* 호스트 세팅

  `sudo hostnamectl set-hostname [node-name].bdai.com`

  `sudo vi /etc/hosts`

  ```
  10.0.0.111  cm.bdai.com cm
  10.0.0.239  m1.bdai.com m1
  10.0.0.177   d1.bdai.com d1
  10.0.0.124  d2.bdai.com d2
  10.0.0.174  d3.bdai.com d3
  ```

  `getent hosts`
  
  세팅 후 재접속

# 1. Create a CDH Cluster on AWS

a. Linux setup
Add the following linux accounts to all nodes
 1. User training with a UID of 3800
 2. Set the password for user “training” to “training” 
 3. Create the group skcc and add training to it
 4. Give training sudo capabilities
List the your instances by IP address and DNS name (don’t use /etc/hosts for this)
List the Linux release you are using
List the file system capacity for the first node (master node)
List the command and output for yum repolist enabled
List the /etc/passwd entries for training (only in master name node) List the /etc/group entries for skcc (only in master name node)
List output of the flowing commands:
 1. getent group skcc
 2. getent passwd training