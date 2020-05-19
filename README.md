# 1. Giới thiệu chung
Ansible là một công cụ cung cấp phần mềm open-source, quản lý cấu hình và công cụ triển khai ứng dụng. Nó chạy trên nhiều hệ thống  Unix cũng như Microsoft Windows. Nó bao gồm ngôn ngữ khai báo riêng để mô tả cấu hình hệ thống. Ansible được viết bởi `Michael DeHaan` và được `Red Hat` mua lại vào năm 2015. 
Ansible sử dụng kết nối từ xa thông qua SSH hoặc Windows Remote Management (cho phép thực thi PowerShell từ xa) để thực hiện các tác vụ của mình.
Sử dụng định dạng JSON để hiển thị thông tin và sử dụng YAML để xây dựng cấu trúc mô tả hệ thống.
## Đặc điểm
+ Không cần cài phần mềm lên các agent, chỉ cần cài đặt tại master.
+ Không service, daemon, chỉ thực thi khi được gọi
+ Bảo mật cao (do sử dụng giao thức SSH để kết nối)
+ Cú pháp đơn giản
## Yêu cầu cài đặt
+ Hệ điều hành: Linux (Redhat, Debian, Ubuntu, Centos, ...), Windows
+ Thư viện Jinja2: dùng để xây dựng template cấu hình
+ Thư viện PyYAML: hỗ trợ cấu trúc YAML
+ Python 2.4 trở lên
## Install
Để config PPA trên máy và install Ansible, ta chạy các lệnh:

```
$ sudo apt update
$ sudo apt install software-properties-common
$ sudo apt-add-repository --yes --update ppa:ansible/ansible
$ sudo apt install ansible
```
![install ansible](https://user-images.githubusercontent.com/61723456/82310600-ce433a80-99ee-11ea-8e29-0464bf831b3a.png)

Có thể tham khảo các hướng dẫn cài đặt [tại đây](https://docs.ansible.com/ansible/latest/installation_guide/index.html)
# 2. Mô hình
![kiến trúc](https://user-images.githubusercontent.com/61723456/82309515-6fc98c80-99ed-11ea-8493-310459b0e479.png)

+ Host inventory file là file chứa các thông tin của các remote server, nơi mà các công việc được thực thi.
+ Ansible config có thể được chỉnh sửa để phù hợp với các cài đặt trong các môi trường, nó sử dụng INI để lưu các config.
## playbook 
Là một file định dạng YAML chứa những miêu tả về các host nào được config và một danh sách các command mà ta muốn thực hiện trên host đó.

Một playbook có thể chứa nhiều công việc, và mỗi công việc phải chứa các thành phần như:
+ Danh sách các hosts để config 
+ Danh sách các tasks để thực thi trên các hosts kia.

Một số modules cơ bản trong ansible:
+ yum: dùng để cài đặt hoặc remove các package sử dụng apt package manager.
+ copy: thực hiện copy 1 file từ máy local tới các máy hosts.
+ file: set các thuộc tính cho 1 file, symlink hay 1 thư mục.
+ service: start, stop or restart 1 service.
+ template: generate 1 file từ 1 template và copy file đó tới các máy hosts.
+ command
+ debug

Một số option cơ bản
+ -i: inventory host. Load thư viện từ host
+ -m: gọi module của ansible
+ -a: command được gửi vào module
+ -u: user

Cấu trúc của 1 playbook
```
vim.yml
- hosts: all
  remote_user: root
  tasks:
    - name: Install vim
      apt: name=vim state=present 
```
Ở đây chỉ có một task, task này chỉ định một package tên là "vim" (name=vim) ở trạng thái "present" (state=present) trên máy đích. Sử dụng module "apt".
+ host: nơi liệt kê các host hoặc tên nhóm các hosts mà ta muốn thực hiện các task trong mục **tasks**.
+ tasks: là một danh sách các hành động mà ta muốn thực thi. Các thành phần trong task: tên task(name), các module sẽ được thực thi, và các tham số cần thiết cho các module đó.

Để chạy file playbook.yml ta thực hiện các câu lệnh:
```
ansible-playbook -i hosts vim.yml
```

## Inventory file
chứa các thông tin về hosts mà ta sẽ thực hiện các task trên đó, file host được thực hiện chính là một `inventory file`. Theo mặc định, ansible sẽ coi file `/etc/ansible/hosts` là một `inventory file` mặc định. Ta nên tạo một `inventory file` khác nhau cho mỗi project, sau đó ta có thể truyền vào khi chạy câu lệnh **ansible** với option `-i` như sau:
```
ansible all -i /path/to/inventory -m ping
```
hoặc câu lệnh **ansible-playbook** cùng vơi option `-i`.

`Inventory` là một file JSON hoặc INI file. Nhưng hầu hết các trường hợp thì ta thường dùng INI file vì nó có cấu trúc đơn giản. JSON chỉ dùng khi inventory file cần được tạo động.
ví dụ:
```
host1.example.com
host2.example.com
host3.example.com
192.168.0.3
```
nếu host có tên tương tự nhau thì ta có thể sử dụng:
```
host[1:3].example.com
```

### Các option config trong inventory file
Tham số cho toàn bộ các connections:

+ `ansible_host`: cho phép ta sử dụng 1 tên khác để làm việc với host, thay vì sử dụng ip hoặc hostname trong các playbook. Điều này giúp ta không phải thay đổi tên host trong các playbook khi ip hay hostsname của nó thay đổi. Ví dụ: `vm1 ansible_host=192.168.122.140`.
+ `ansible_user`: user để login vào remote host thông qua SSH. Ví dụ `ansible_user=vm1` sẽ tương đương với `ssh vm1@192.168.122.140`.
+ `ansible_port`: port mà SSH server lắng nghe trên remote host. Ta cũng có thể sử dụng thông qua bí danh bằng cách `hostname:port`.

Các tham số để chỉ định cho SSH connection:

+ `ansible_ssh_private_key_file`: SSH key file được sử dụng để log in. Ví dụ: `ansible_ssh_private_key_file=/path/to/id_rsa` tương đương với `ssh -i /path/to/id_rsa`
+ `ansible_ssh_pass`: nếu user mà ta đang sử dụng để connect tới remote host yêu cầu password. Nhưng sử dụng option này không đảm bảo tính an toàn, do đó bạn nên sử dụng SSH key hoặc sử dụng cờ ask-pass trong command line để chỉ cung cấp pass ở thời điểm run time.

Thêm quyền cho user connect:

+ `ansible_become`: tương đương với `ansible_sudo` hoặc `ansible_su`, cho phép user hiện tại sử dụng quyền `sudo` hoặc `su`.
+ `ansible_become_method`: method được dùng để chạy với quyền `superuser`. Mặc định là `sudo`, nhưng ta cũng có thể chỉnh lại thành `su`, `pbrun`, `pfexec`, hoặc `doas`.
+ `ansible_become_user`: mặc định become sẽ đưa bạn đến quyền root, tương đương với `ansible_sudo_user` hay `ansible_su_user`. Nhưng nếu bạn có 1 user khác có quyền để thực hiện task bạn đang muốn chạy và bạn muốn có quyền như user đó, thay vì root thì bạn có thể dùng `ansible_become_user`. Option này tương đương với việc sử dụng cờ `-u` khi run command line, ví dụ: `sudo -u myusser command`.
+ `ansible_become_pass`: cho phép set password để có quyền của user trong `ansible_become_user`, nhưng ta không nên sử dụng vì tính bảo mật.
Các paremeters để điều chỉnh remote host environment:

### Tạo groups inventory

```
[web]
host1.example.com
host2.example.com

[database]
db.example.com
```
ta có 2 host trong group **web** và 1 host trong group **database**
Ngoài ra ansible có hai group mặc định là:
+ all: chứa tất cả các host.
+ ungrouped: chứa tất cả các host không thuộc group nào.
## Ansible Galaxy
[Ansible Galaxy](https://galaxy.ansible.com/) là một trung tâm để tìm kiếm và chia sẻ nội dung Ansible. Là một website để chứa các roles, đồng thời nó cho phép người dùng có thể upload roles của mình lên để mọi người tham khảo và sử dụng. Các roles là 1 khái niệm trong core của ansible, chúng thực hiện giống như các core function.
