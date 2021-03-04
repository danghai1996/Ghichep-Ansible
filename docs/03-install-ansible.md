# Cài đặt, triển khai Ansible

Ở đây, ta sẽ chuẩn bị 1 node Ansible control và 3 node Manage (dùng các hệ điều hành Linux: CentOS-7, Ubuntu 18, Ubuntu 20). Các bạn hoàn toàn có thể dựng thêm các node với phiên bản Linux khác.

**Lưu ý:** Các thao tác thực hiện dưới quyền `root`

# Mô hình lab
Mô hình lab:

<img src="..\images\Screenshot_1.png">

IP Planning:

<img src="..\images\Screenshot_2.png">

# 1. Thiết lập IP, hostname
- Thực hiện đặt IP và hostname của các node theo mô hình lab đã đề ra.

# 2. Cài đặt Ansible
> ## Thực hiện trên node Ansible server
Update gói và cài đặt Ansible:
```
yum install -y epel-release 
yum update -y

yum install -y ansible
```

Kiểm tra phiên bản Ansible vừa cài đặt:
```
[root@ansible-server ~]# ansible --version
ansible 2.9.17
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Nov 16 2020, 22:23:17) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]
```

Vậy ở đây, mình đã cài đặt Ansible phiên bản 2.9.17.

# 2. Cấu hình SSH Key và khai báo file inventory
Ansible hoạt động theo cơ chế agentless, có nghĩa là không cần cài agent vào các máy client để điều khiển, thay vào đó ansible sẽ sử dụng việc điều khiển các client thông qua SSH. Do vậy, tới bước này ta có thể dùng 2 cách để ansible có thể điều khiển được các máy client.

- **Cách 1:** Sử dụng usename, port của ssh để khai báo trong inventory. Các này không được khuyến cáo khi dùng trong thực tế vì việc password dạng clear text sẽ hiện thị, hoặc nếu dùng cách này thì cần phải secure cho file inventory này bằng `ansible-vault`

- **Cách 2:** Sử dụng ssh keypair. Có nghĩa là ta sẽ tạo ra private key và public key trên node AnisbleServer và copy chúng sang các node client (hay còn gọi là các host).

Tuy nhiên, nếu sử dụng ssh thông thường sẽ dễ bị lộ mật khẩu vì password sẽ được khai báo dưới dạng cleartext. Do vậy, trong bài lab này mình sẽ sử dụng ssh key để ssh vào và monitor các host.

## 2.1. Cấu hình SSH Key
> ### Thực hiện trên node Ansible Server
Tạo ssh key và copy nó sang các node client (host) còn lại.

Tạo key:
```
ssh-keygen
```

Sau khi có SSH Key, ta thực hiện copy file key sang các node còn lại:
```
[root@ansible-server ~]# ssh-copy-id root@10.10.35.131
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
The authenticity of host '10.10.35.131 (10.10.35.131)' can't be established.
ECDSA key fingerprint is SHA256:2RsQ05j8u5MIHuTChlfiQveNSexhMR8crU3+7XoyRfA.
ECDSA key fingerprint is MD5:30:cd:f4:db:44:d3:36:81:99:f5:6c:f5:be:d0:da:1f.
Are you sure you want to continue connecting (yes/no)? yes
Please type 'yes' or 'no': yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@10.10.35.131's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@10.10.35.131'"
and check to make sure that only the key(s) you wanted were added.
```

Tương tự như vậy, ta tiến hành ssh key vào các host còn lại.
```
ssh-copy-id root@10.10.35.132
ssh-copy-id root@10.10.35.133
```

Sau khi copy key xong, kiểm tra lại bằng cách đứng từ node Ansible server ssh đến các node Client. Nếu không bị hỏi mật khẩu thì ta đã sử dụng ssh key thành công.

**Lưu ý:** Nhớ `exit` sau khi ssh đến từng node.

## 2.2. Khai báo Inventory
> ### Thực hiện trên node Ansible Server
Mặc định thì danh sách các host mà AnsibleServer điều khiển sẽ nằm ở file `/etc/ansible/host`. File mặc định này sẽ chứa các khai báo mẫu, ta sẽ thực hiện sao lưu lại và khai báo file mới theo các thông tin của các node.

Backups file cấu hình:
```
mv /etc/ansible/hosts /etc/ansible/hosts.bak
```

Tạo mới file cấu hình để khai báo host:
```
vi /etc/ansible/hosts
```
Nội dung:
```ini
[centos7]
10.10.35.131

[ubuntu18]
10.10.35.132

[ubuntu20]
10.10.35.133
```

**Lưu ý**: có thể sử dụng cặp thẻ [ ] để khai báo các group. Các group này sẽ do ta quy hoạch sao cho phù hợp với hệ thống, với ứng dụng của chúng ta.

Kiểm tra danh sách host đã được khai báo trong file inventory:
```
ansible all --list-hosts
```
**Trong đó:**
- `all` là tùy chọn của lệnh, để liệt kê tất cả hosts trong file inventory.

OUTPUT:
```
[root@ansible-server ~]# ansible all --list-hosts
  hosts (3):
    10.10.35.131
    10.10.35.133
    10.10.35.132
```

Nếu chỉ muốn liệt kê các host trong 1 group, ta thực hiện theo cú pháp sau:
```
ansible <Tên group> --list-hosts
```

Ví dụ:
```
[root@ansible-server ~]# ansible centos7 --list-hosts
  hosts (1):
    10.10.35.131
```

Như trên mới là việc khai báo danh sách các host.

Ta cần khai báo thêm các tùy chọn về mật khẩu, về port, thậm chí cả về user và Ansible Server sẽ sử dụng để điểu khiển các host.

Sửa file `/etc/ansible/host`
```ini
[centos7]
host1-centos7 ansible_host=10.10.35.131 ansible_port=22 ansible_user=root

[ubuntu18]
host2-ubuntu18 ansible_host=10.10.35.132 ansible_port=22 ansible_user=root

[ubuntu20]
host3-ubuntu20 ansible_host=10.10.35.133 ansible_port=22 ansible_user=root
```

Trong đó:
- `host1-centos7`, `host2-ubuntu18`, `host3-ubuntu20`: Tương ứng là các hostname của các node

- `ansible_host`: Địa chỉ IP của node client tương ứng

- `ansible_port`: Port của SSH phía client, nếu ta thay đổi thì sẽ chỉnh lại cho đúng.

- `ansible_user`: Là username của client mà AnsibleServer sẽ dùng để tương tác, trong bước trên tôi sử dụng là user `root` và thông qua SSH Key.

## 2.3. Kiểm tra kết nối các host
Sử dụng ping để kiểm tra các kết nối từ ansible control đến các host. Kết quả trả về: 
- `SUCCESS`: tức là các host đang hoạt động
- `UNREACHABLE!`: tức là các host đó không hoạt động hoặc không thể kết nối tới host đó.

Trước khi kiểm tra, ta thực hiện tắt 1 node `host3-ubuntu20`.

Kiểm tra ping tới các host đã khai báo:
```
ansible all -m ping
```
- `all` : tất cả các hosts đã khai báo
- `-m` : module. Sau đó sẽ là module sử dụng. Ở đây là `ping`

<img src="..\images\Screenshot_3.png">

Kết quả như ví dụ trên cho thấy `host1-centos7` và `host2-ubuntu18` phản hồi về trạng thái màu xanh (`SUCCESS`) tức là từ Ansible Server đã ping được tới 2 host đó. Còn lại, `host3-ubuntu20` có màu đỏ (`UNREACHABLE!`) tức là từ Ansible Server không tìm thấy `host3-ubuntu20`

Bật lại `host3-ubuntu20`, sau đó kiểm tra lại:
```
ansible all -m ping
```

<img src="..\images\Screenshot_4.png">

=> Xanh hết :D . Vậy là các host khai báo trong file inventory đã được "thông".

Ngoài ra thay vì check toàn bộ, ta có thể check theo group hoặc host chỉ định:

Check theo group:

<img src="..\images\Screenshot_5.png">

Check theo host:

<img src="..\images\Screenshot_6.png">

**Lưu ý:** Nếu tồn tại tên group và tên host giống nhau, ta sẽ thấy có cảnh báo như sau:

<img src="..\images\Screenshot_7.png">