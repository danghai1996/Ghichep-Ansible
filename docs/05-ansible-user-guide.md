# Ansible user guide

# 1. Các công cụ dòng lệnh Ansible
## 1.1. `ansible`
Là 1 cộng cụ đơn giản để thực hiện các công việc từ xa. Lệnh này cho phép xác định và chạy 1 tác vụ duy nhất dựa trên 1 tập các máy chủ.

Cú pháp
```
ansible <host-pattern> [options]
```

## 1.2. `ansible-config`
Cú pháp:
```
ansible-config [view|dump|list] [--help] [options] [ansible.cfg]
```

## 1.3. `ansible-console`
Cú pháp:
```
ansible-console [<host-pattern>] [options]
```

## 1.4. `ansible-doc`
**Ansible-doc** hiển thị thông tin về các module được cài đặt trong thư viện ansible. Nó hiển thị danh sách ngắn gọn về các plugin và mô tả ngắn của chúng, có thể tạo 1 đoạn ngắn có thể dán vào playbook.

Cú pháp:
```
ansible-doc [-l|-F|-s] [options] [-t <plugin type> ] [plugin]
```

## 1.5. `ansible-galaxy`
Lệnh `ansible-galaxy` quản lý các vai trò Ansible trong kho lưu trữ được chia sẻ, mặc định là Ansible Galaxy https://galaxy.ansible.com

Cú pháp:
```
ansible-galaxy [delete|import|info|init|install|list|login|remove|search|setup] [--help] [options] ...
```

## 1.6. `ansible-inventory`
Lệnh được sử dụng để hiển thị hoặc kết xuất khoảng không quảng cáo đã định cấu hình khi Ansible nhìn thấy nó.

Cú pháp:
```
ansible-inventory [options] [host|group]
```

## 1.7. `ansible-playbook`
Lệnh sử dụng để chạy các playbook ansible.

Cú pháp:
```
ansible-playbook [options] playbook.yml [playbook2 ...]
```

## 1.8. `ansible-pull`
Cú pháp:
```
ansible-pull -U <repository> [options] [<playbook.yml>]
```

## 1.9. `ansible-vault`
Sử dụng để mã hóa bất kỳ tệp dữ liệu có cấu trúc nào được Ansible sử dụng.

Cú pháp:
```
ansible-vault [create|decrypt|edit|encrypt|encrypt_string|rekey|view] [options] [vaultfile.yml]
```

# 2. Ad-hoc
Ad-Hoc command cho phép ta thực hiện các task đơn, các yêu cầu độc lập đến các máy được quản lý.

Nó giúp thực hiện nhanh, thực hiện hàng loạt các tác vụ như khởi động os, khởi động hoặc stop các dịch vụ, kiểm tra trạng thái của các dịch vụ, xem nội dung file hoặc cây thư mục.. trên các máy chủ được quản lý.

Cú pháp chung của ad-hoc command trong Ansible:
```
ansible [pattern] -m [module] -a "[module options]"
```

## 2.1. Chuyển tập tin
Ví dụ: Copy file `/etc/hosts` của các node thuộc group `ubuntu` sang thư mục `/tmp/hosts`

```
ansible ubuntu -m copy -a "src=/etc/hosts dest=/tmp/hosts"
```
OUTPUT

<img src="..\images\Screenshot_8.png">

Kiểm tra trên 2 node hosts:

<img src="..\images\Screenshot_9.png">
<img src="..\images\Screenshot_10.png">

## 2.2. Quản lý các gói
Cài đặt nhưng không cập nhật gói

CentOS:
```
# Cài đặt nhưng không update gói
ansible all -m yum -a "name=<tên gói> state=present"

# Cài đặt gói ở phiên bản mới nhất
ansible all -m yum -a "name=<tên gói> state=latest"

# Gỡ cài đặt gói
ansible all -m yum -a "name=<tên gói> state=absent"
```

Ubuntu:
```
# Cài đặt nhưng không update gói
ansible all -m apt -a "name=<tên gói> state=present"

# Cài đặt gói ở phiên bản mới nhất
ansible all -m apt -a "name=<tên gói> state=latest"

# Gỡ cài đặt gói
ansible all -m apt -a "name=<tên gói> state=absent"
```

**Chú ý:**
- `all` : có thể chỉ định node hoặc group inventory

### **Ví dụ:** Cài đặt apache2 trên các node Ubuntu
```
ansible ubuntu -m apt -a "name=apache2 state=latest"
```

OUTPUT:
```
[root@ansible-server ~]# ansible ubuntu -m apt -a "name=apache2 state=latest"
host2-ubuntu18 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    }, 
    "cache_update_time": 1614651605, 
    "cache_updated": false, 
    "changed": true, 
    "stderr": "", 
    "stderr_lines": [], 
    "stdout":
    ....

host3-ubuntu20 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    }, 
    "cache_update_time": 1614651644, 
    "cache_updated": false, 
    "changed": true, 
    "stderr": "", 
    "stderr_lines": [], 
    "stdout": 
    ...
```

Kiểm tra lại xem apache đã được cài lên nhóm `ubuntu` hay chưa:
```
[root@ansible-server ~]# ansible ubuntu -a "apache2 -v"
host3-ubuntu20 | CHANGED | rc=0 >>
Server version: Apache/2.4.41 (Ubuntu)
Server built:   2020-08-12T19:46:17
host2-ubuntu18 | CHANGED | rc=0 >>
Server version: Apache/2.4.29 (Ubuntu)
Server built:   2020-08-12T21:33:25
```

Có thể kiểm tra trên các node việc cài đặt `apache2`



## Đọc thêm:
- https://docs.ansible.com/ansible/latest/user_guide/command_line_tools.html