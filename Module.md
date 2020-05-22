# Modules
Ansible cung cấp rất nhiều module. khi sử dụng playbook cũng như khi thực hiện thông qua dòng lệnh với option `-m`, ta chỉ cần khai báo tên module và tên dịch vụ trong các tasks, module được khai báo tự khắc sẽ nhận lệnh và thực thi theo yêu cầu. Nếu muốn tự tạo một module riêng thì ansible vẫn hỗ trợ và cho phép viết module để chạy trên python.
## Một số module
### 1. Commands - Thực thi các lệnh trên các targets
+ Module `command` lấy tên lệnh theo sau là các đối số được phân tách bằng dấu cách.
+ Lênh được gọi sẽ thực thi trên tất cả các node được chọn.
+ Các lệnh không được xử lý qua shell, nên các biến như `$Home` và các toán tử "<", ">", "|", ";" và "&" sẽ không khả dụng. Nếu muốn thực hiện các tính năng trên thì dùng module `shell`.
+ Để tạo các `command` dễ đọc hơn, các tasks sử dụng các đối số được phân tách bằng dấu cách, truyền tham số bằng cách sử dụng từ khóa `args` hoặc sử dụng tham số `cmd`.

**args**: Truyền lệnh dưới dạng một danh sách chứ không phải là một chuỗi. Sử dụng `argv` để tránh trích dẫn các giá trị yêu cầu sự chính xác cao (ví dụ "user name").
```
- name: Run command if /path/to/database does not exist (with 'args' keyword).
  command: /usr/bin/make_database.sh db_user db_name
  args:
    creates: /path/to/database
```

**cmd**: Lệnh thực thi.
```
- name: Run command if /path/to/database does not exist (with 'cmd' parameter).
  command:
    cmd: /usr/bin/make_database.sh db_user db_name
    creates: /path/to/database
```

### 2. File
#### copy – Copy files to remote locations
+ Module `copy` sao chép một tệp từ máy local hoặc remote đến một vị trí trên máy remote.
+ Sử dụng module `fetch` để sao chép các tệp từ các remote locations vào local box.
+ Nếu cần thêm biến trong các tệp được sao chép, hãy sử dụng `template`.

**dest** (path): Đường dẫn duy nhất tại remote, nơi tập tin sẽ được sao chép vào.
Nếu `src` là một thư mục, đây cũng phải là một thư mục.
Nếu `dest` là một đường dẫn không tồn tại và nếu  kết thúc bằng '/' hoặc `src` là một thư mục, thì `Dest` được tạo.
Nếu `src` và `Dest` là các tệp, thư mục cha của `dest` không được tạo và task sẽ fails nếu nó chưa tồn tại.
**src** (path): Local path đến một tệp để sao chép vào remote server.
Nếu đường dẫn là một thư mục, nó được sao chép đệ quy. Trong trường hợp này, nếu đường dẫn kết thúc bằng '/', chỉ bên trong nội dung của thư mục đó được sao chép đến đích. Mặt khác, nếu nó không kết thúc bằng '/', thì chính thư mục có tất cả nội dung sẽ được sao chép.

### 3. docker
#### docker_compose – Manage multi-container Docker applications with Docker Compose
+ Sử dụng Docker Compose để start, shutdown và scale dịch vụ.
+ Hoạt động với các compose version 1 và 2.
+ Cấu hình có thể được đọc từ tệp `docker-compose.yml` hoặc `docker-compose.yaml`.
+ Hỗ trợ chế độ kiểm tra.
+ Module này được gọi là docker_service trước Ansible 2.8. Cách sử dụng không thay đổi.

**project_src** (path): Đường dẫn đến thư mục chứa tệp docker-compose.yml hoặc docker-compose.yaml.

**project_name** (string): Cung cấp một tên dự án. Nếu không được cung cấp, tên dự án được lấy từ tên cơ sở của project_src.

**state** (string): Trạng thái mong muốn của project.
Chỉ định `present` giống như chạy `docker-compose up`. 
Chỉ định `absent` cũng giống như chạy `docker-compose down`.

```
- name: run prom
  hosts: localhost
  tasks:
    - name: docker compose
      docker_compose: 
        project_src: /home/toan/prometheus

```

#### docker_container – manage docker containers
+ Quản lý vòng đời của container docker.
+ Hỗ trợ chế độ kiểm tra. Chạy với `--check` và `--diff` để xem sự khác biệt về cấu hình và danh sách các hành động sẽ được thực hiện.

**command**: Lệnh thực thi khi container bắt đầu. Một lệnh có thể là một chuỗi hoặc một danh sách.

**hostname**: Tên container.

**image**: Đường dẫn của repo và tag được sử dụng để tạo container. Nếu một image không được tìm thấy hoặc pull được thỏa mãn, image sẽ được pull từ registry. Nếu không có tag được bao gồm, `latest` sẽ được sử dụng.
Cũng có thể là một image ID. Nếu đây là trường hợp, image được giả định là có sẵn tại local. Tùy chọn pull được bỏ qua cho trường hợp này.

**mounts**:  Chỉ định mounts được thêm vào container. Thay thế mạnh mẽ hơn cho volumes.

**network_mode**: Kết nối container với mạng. Lựa chọn là `bridge`, `host`, `none` hoặc `container:<name|id>`.

**state**: `absent` - Một container khớp với tên được chỉ định sẽ bị dừng và xóa. Sử dụng `force_kill` để kill container hơn là dừng nó. Sử dụng `keep_volume` để giữ lại volumes được liên kết với container bị xóa.
**present** - Xác nhận sự tồn tại của một container khớp với tên và mọi tham số cấu hình được cung cấp. Nếu không có container nào khớp với tên, một container sẽ được tạo. Nếu một container khớp với tên nhưng cấu hình được cung cấp không khớp, container sẽ được cập nhật, nếu có thể. Nếu không thể cập nhật, nó sẽ bị xóa và được tạo lại với cấu hình được yêu cầu.
**started** - Xác nhận rằng container có mặt lần đầu tiên và sau đó nếu container không chạy sẽ chuyển nó sang trạng thái chạy. Sử dụng khởi động lại để buộc một container phù hợp bị dừng và khởi động lại.
Đã dừng - Xác nhận rằng container hiện diện lần đầu tiên và sau đó nếu container đang chạy sẽ chuyển nó sang trạng thái dừng.Sử dụng tùy chọn tạo lại để luôn bắt buộc tạo lại một thùng chứa phù hợp, ngay cả khi nó đang chạy.

```
- name: Create a data container
  docker_container:
    name: mydata
    image: busybox
    volumes:
      - /data
```
