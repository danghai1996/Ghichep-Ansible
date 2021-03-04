# Một số module cơ bản

Đầu tiên khi tiếp cận với Ansible, ta sẽ sử dụng các module ở chế độ tương tác, tức là ta sẽ thử chạy các lệnh của từng module 1 và phân biệt được cách sử dụng của các module đó.

Ví dụ: Ta có file `/etc/ansible/hosts` như sau:
```ini
[centos]
host1-centos7 ansible_host=10.10.35.131 ansible_port=22 ansible_user=root

[ubuntu]
host2-ubuntu18 ansible_host=10.10.35.132 ansible_port=22 ansible_user=root
host3-ubuntu20 ansible_host=10.10.35.133 ansible_port=22 ansible_user=root
```

# 1. Module `command`
Về cơ bản, có thể hiểu module command giúp ta thực hiện các lệnh linux từ xa. Ta có thể thực hiện các lệnh trên 1 node nhất định, 1 group, cũng có thể là toàn bộ các node đã được khai báo.

**Ví dụ 1:** Thực hiện lệnh `ls` trên các host

Thực hiện trên tất cả các node đã khai báo:
```
[root@ansible-server ~]# ansible all -m command -a "ls"
host1-centos7 | CHANGED | rc=0 >>
anaconda-ks.cfg
cent7-64bit
centos7.data
original-ks.cfg
host3-ubuntu20 | CHANGED | rc=0 >>
file_ubuntu20
hostubuntu20
host2-ubuntu18 | CHANGED | rc=0 >>
hello
u18-dir
u18.txt
```

Trên 1 node chỉ định:
```
[root@ansible-server ~]# ansible host2-ubuntu18 -m command -a "ls"
host2-ubuntu18 | CHANGED | rc=0 >>
hello
u18-dir
u18.txt
```

Trên các node trong group `ubuntu`:
```
[root@ansible-server ~]# ansible ubuntu -m command -a "ls"
host3-ubuntu20 | CHANGED | rc=0 >>
file_ubuntu20
hostubuntu20
host2-ubuntu18 | CHANGED | rc=0 >>
hello
u18-dir
u18.txt
```

**Ví dụ 2:** Check uptime các node
```
[root@ansible-server ~]# ansible all -m command -a "uptime"
host3-ubuntu20 | CHANGED | rc=0 >>
 10:45:03 up 23:44,  2 users,  load average: 0.00, 0.00, 0.00
host1-centos7 | CHANGED | rc=0 >>
 03:45:03 up 1 day,  1:13,  2 users,  load average: 0.00, 0.01, 0.03
host2-ubuntu18 | CHANGED | rc=0 >>
 10:45:03 up 1 day,  1:13,  2 users,  load average: 0.00, 0.00, 0.00
```

# 2. Module `setup`
Ta có thể sử dụng module `setup` để kiểm tra các thông tin tổng quát về hệ điều hành của các node, ví dụ kiểm tra phiên bản, kiểm tra thông tin card mạng, tên host, thông số về phần cứng ….

**Ví dụ:** kiểm tra distro của các node host:
```
ansible all  -m setup -a 
```
OUTPUT
```jso
[root@ansible-server ~]# ansible all  -m setup -a 'filter=ansible_distribution'
host1-centos7 | SUCCESS => {
    "ansible_facts": {
        "ansible_distribution": "CentOS", 
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false
}
host3-ubuntu20 | SUCCESS => {
    "ansible_facts": {
        "ansible_distribution": "Ubuntu", 
        "discovered_interpreter_python": "/usr/bin/python3"
    }, 
    "changed": false
}
host2-ubuntu18 | SUCCESS => {
    "ansible_facts": {
        "ansible_distribution": "Ubuntu", 
        "discovered_interpreter_python": "/usr/bin/python3"
    }, 
    "changed": false
}
```


**Ví dụ 2:** Kiểm tra thông tin về Network trên các node
```
ansible all  -m setup -a 'filter=ansible_default_ipv4'
```
OUTPUT
```json
[root@ansible-server ~]# ansible all  -m setup -a 'filter=ansible_default_ipv4'
host1-centos7 | SUCCESS => {
    "ansible_facts": {
        "ansible_default_ipv4": {
            "address": "10.10.35.131", 
            "alias": "eth0", 
            "broadcast": "10.10.35.255", 
            "gateway": "10.10.35.1", 
            "interface": "eth0", 
            "macaddress": "52:54:00:7f:08:ed", 
            "mtu": 1500, 
            "netmask": "255.255.255.0", 
            "network": "10.10.35.0", 
            "type": "ether"
        }, 
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false
}
host2-ubuntu18 | SUCCESS => {
    "ansible_facts": {
        "ansible_default_ipv4": {
            "address": "10.10.35.132", 
            "alias": "eth0", 
            "broadcast": "10.10.35.255", 
            "gateway": "10.10.35.1", 
            "interface": "eth0", 
            "macaddress": "52:54:00:e8:c0:eb", 
            "mtu": 1500, 
            "netmask": "255.255.255.0", 
            "network": "10.10.35.0", 
            "type": "ether"
        }, 
        "discovered_interpreter_python": "/usr/bin/python3"
    }, 
    "changed": false
}
host3-ubuntu20 | SUCCESS => {
    "ansible_facts": {
        "ansible_default_ipv4": {
            "address": "10.10.35.133", 
            "alias": "eth0", 
            "broadcast": "10.10.35.255", 
            "gateway": "10.10.35.1", 
            "interface": "eth0", 
            "macaddress": "52:54:00:d4:e7:01", 
            "mtu": 1500, 
            "netmask": "255.255.255.0", 
            "network": "10.10.35.0", 
            "type": "ether"
        }, 
        "discovered_interpreter_python": "/usr/bin/python3"
    }, 
    "changed": false
}
```


