# Một số khái niệm trong Ansible

## Control node & Managed nodes: 
Control node là một máy tính chạy Ansible, phải có ít nhất 1 control node. Managed node là bất kỳ thiết bị nào đang được quản lý bởi control node. 

## Inventory: 
Danh sách các node được quản lý trong file inventory dạng INI. Bạn có thể chỉ định thông tin của node như IP, hostname, ...

Vị trí mặc định cho inventory là 1 file có tên `/etc/ansible/hosts`.

## Collections:
Collections là 1 định dạng phân phối cho nội dung Ansible có thể bao gồm playbooks, roles, modules và plugins. Bạn có thể cài đặt và sử dụng collection thông qua Ansible Galaxy.

## Modules: 
Mỗi module có 1 mục đích sử dụng cụ thể, từ quản lý người dùng trên 1 loại cơ sở dữ liệu cụ thể đến quản lý các giao diện VLAN trên 1 loại thiết bị mạng cụ thể. Bạn có thể gọi 1 module với 1 nhiệm vụ hoặc gọi nhiều module khác nhau trong 1 playbook.

## Playbook: 
là các tệp trong đó mã Ansible được viết. Playbook được viết ở định dạng YAML. Chúng giống như một danh sách việc cần làm của Ansible,  chứa một danh sách các task để thực hiện.

```
Có thể hiểu playbook là 1 vở kịch trong đó sẽ có các “role” là các vai diễn trong vở kịch đó, mỗi vai diễn sẽ thực hiện các nhiệm vụ (task) tương ứng với vai diễn đó.
```

## Ansible role: 
Được sử dụng để đơn giản hóa playbook. Nghĩa là khi ta làm việc với Ansible playbook, ta có thể dễ dàng phân chia các nhiệm vụ thành các role khác nhau. Ngoài ra, những role này có thể được sử dụng lại trong tương lai.

Khi tạo ra 1 role, sẽ có các thư mục tương ứng được tạo ra như: `defaults`, `handlers`, `meta`, `molecule`, `tasks`, `templates`, `vars`

- `defaults`: các biến mặc định cho role, các biến này có độ ưu tiên thấp nhất trong số các biến có sẵn, và có thể dễ dàng bị ghi đè bởi bất kỳ biến nào khác, bao gồm các biến inventory

- `handlers`: trình xử lý, có thể được sử dụng trong hoặc ngoài role

- `meta`: Siêu dữ liệu cho role, bao gồm cả phần phụ thuộc role.

- `tasks`: Danh sách các nhiệm vụ chính mà role thực thi

- `templates`: Nó chứa các mẫu mà ta có thể triển khai thông qua role này. Nó không có file `main.yml`

- `vars`: Nó lưu trữ các biến được sử dụng trong các vai trò. Nó có mức độ ưu tiên cao nhất và chúng ta chỉ có thể ghi đè bằng cách chuyển các biến qua CLI. Nó cũng có 1 file `main.yml`

## Task: 
Các đơn vị hành động trong Ansible. Bạn có thể thực thi 1 nhiệm vụ duy nhất bằng cách sử dụng `ad-hoc command`

## Handler: 
Đôi khi bạn muốn 1 tác vụ chỉ chạy khi một thay đổi được thực hiện trên máy. Ví dụ: bạn muốn khởi động lại 1 service khi 1 tác vụ cập nhật cấu hình của service đó, nhưng sẽ không nếu cấu hình không thay đổi. Ansible sử dụng handler để giải quyết trường hợp này. 