## DOCKER & CONTAINER
### Table of contents

[I. Introducton](#modau)

[II. Getting Start](#batdau)
- [1. Step 1](#step1)
- [2. Step 2](#step2)
- [3. Step 3](#step3)
- [4. Step 4](#step4)
- [5. Step 5](#step5)
- [6. Step 6](#step6)
- [7. Step 7](#step7)

[III. Summary](#Tongket)

===========================

<a name="Modau"></a>
## I. Introduction
- Bài viết này sẽ giới thiệu cho bạn kiến thức cơ bản về container và docker, ưu khuyết điểm, cách sử dụng và quản trị docker & container. 
- Trong bài viết này mình sẽ không nói tới Kubernetes (k8s) vì đây là bài cơ bản nhất cho bạn nào mới làm quen với Docker.

<a name="batdau"></a>
## II. Getting Start:

<a name="step1"></a>
## Container:
- Cơ bản, container là một dạng ảo hóa mà không có chứa kernel bên trong nó. Container chỉ chứa các library, software cần thiết để chạy application nào đó. 
Sự khác biệt lớn nhất giữa VM và container là kernel, về bản chất thì container không thể quản lý device, module nó chỉ chạy thuần application mà thôi. 
Khái niệm container này tồn tại khá lâu, nếu bạn từng xài Proxmox (chuẩn LXC) hoặc Parallel (chuẩn openVZ) thì sẽ gặp dạng này.

**Ưu điểm.**
- Nhẹ, phải nói là rất nhẹ do container ko chứa kernel, nó cũng không phải quản lý module nào nên thời gian boot là rất nhanh. 
- Hơn nữa đây là giải pháp dành cho service nào chỉ chạy application mà thôi. 
- Thời gian deploy nhanh. Với developer thì thời gian từ dep sang product là tối quan trọng. 

**Khuyết điểm.** 
- Do không có kernel nên với dân system thì rất khó cho việc deploy service có chạy TAP/TUN device. 
- Hầu như không thể dùng với device ngoài nào mặc dù device đã được attach directly thông qua USB.

<a name="step2"></a>
## Docker:
- Docker là một bộ tool set để quản lý các container, nó có nhiệm vụ tạo subnet, assign ip, nat giữa bên trong mạng của container ra ngoài. Để cho dễ hiểu thì docker giống như 1 con router để quản lý gói tin vào ra. 
- Hơn nữa, docker cũng quản lý luôn các image (có thể coi image là file VM mà có chứa sẵn một phần mềm cụ thể), người dùng chỉ cần download về và kích hoạt application đó mà thôi. 
- Ngoài ra docker cũng có nhiệm vụ quản lý storage của từng container, nó có thể cấp cho container một hoặc nhiều chỗ lưu trữ (được định nghĩa là các phân vùng đặc biệt, nó giống với file chứa VM - Volume) 
- Sự khác biệt giữa container (LXC/OpenVZ) và container (Docker). Tại đây, ta sẽ có sự khác biệt rất rõ giữa 2 nhóm trên.
  - LXC/OpenVZ tuy được gọi là container nhưng vẫn có chứa thêm service process để có thể link IP, gateway, ngoài ra nó còn có thể chứa nhiều service trên 1 container. 
  - Docker container chỉ kích hoạt đúng process của application, phần còn lại do docker xử lý (thành ra ở docker thì PID duy nhất mà bạn có thể thấy là 1). Docker container trên lý thuyết là chỉ sử dụng một service duy nhất là application.

<a name="step3"></a>
## Quản trị Docker.
- Dưới đây mình xin hướng dẫn cách sử dụng docker trên linux. Tất cả các function sẽ luôn được bắt đầu bởi docker.

> # docker <command> <option> 

`Ví dụ:` 
> # docker ps -a --> command hiện toàn bộ    các container đang có. 

**Quản lý container.**
Để hiện danh sách các container đang có trên máy 

> # docker ps -a 

- Để dừng một container 

> # docker stop <container name> 

- Và để xóa một container 

> # docker rm <container name> 

- Để chạy lại container đã bị 

> # docker start <container name> 

- Hoặc chạy lại container đang chạy hoặc update image mới vào container hiện tại. 
> # docker restrt <container name> 

- Để kiểm tra một container (xem configuration chẳn hạng). 
> # docker inspect <container name> 

- Để chạy container từ một image 
> # docker run <option> <image name> 

**Chú ý:** câu lệnh run thường sẽ có thêm nhiều option như là mount, name, publish v.v… mọi người nên chạy thêm câu lệnh sau để hiện ra đẩy đủ các option 
> # docker run --help 

Ví dụ, với câu lệnh: 
> # docker run --name some-nginx -v /some/content:/usr/share/nginx/html -d nginx 

`Giải thích:` 
- Ta sẽ kêu docker chạy 1 container với tên container là some-nginx 
- Sau đó link đường dẫn /some/content của máy hiện tại vào direcroty /usr/share/nginx/html của container, cơ bản là khi bạn add file nào vào directory /some/content thì file đó sẽ được bỏ vào directory 
/usr/share/nginx/html của container. 
- Kế tiếp là khi chạy container thì dcoker sẽ detach (-d) ra khỏi bash của container 
- Cuối cùng là tên image là nginx, nếu như image này không tồn tại thì docker sẽ tự động download từ docker hub. Bạn có thể tìm thêm danh sách các image đang có sẵn tại https://hub.docker.com 
Với các container đang chạy, ta có thể xem log của nó bằng câu lệnh sau. 

> # docker logs <container name> 
Trong một số trường hợp bạn cần truy cập vào bash của container đó, đây là câu lệnh bạn cần 
> # docker exec -ti <container name> bash

Ví dụ, để truy xuất vào bash của container “some-nginx” mà ta mới tạo, câu lệnh. 
> # docker exec -ti some-nginx bash 

Sau câu lệnh trên thì bạn sẽ truy cập trực tiếp vào bên trong container “some-nginx”. 
Quản lý image. 
Để liệt kê danh sách các image đã được download về máy, câu lệnh 
> # docker images 

Hoặc cũng có thể dùng 
> # docker image list 

Để xóa image đang có, ta dùng câu lệnh 
> # docker rmi <image id> 

Hoặc câu lệnh sau. 
> # docker image rm <image id> 

Chú ý, bạn nên list ra danh sách các image rồi mới có thể lấy được “image id” 
Để download image từ docker hub thì có thể dùng 
> # docker pull <image name> 

Hay có thể dùng 
> # docker image pull <image name> 

Để có thể biết được thông tin của image, thực hiện 
> # docker inspect <image id> 

Quản lý Volume (chỗ lưu trữ) 
Tương tự như image, ta cũng có thể làm các thao tác tạo, xóa, liệt kê và xem thông tin volume. 
> # docker volume create <volume name> 

> # docker volume rm <volume id> 

> # docker volume ls 

> # docker volume inspect <volume id> 
Ngoài ra còn các option khác, bạn có thể dùng help để xem cụ thể 
> # docker volume --help 

Quản lý Network. 
Network ở đây là tạo 1 subnet riêng cho từng container hoặc một nhóm container riêng biệt, mục đích là để tách riêng cho các container này ko liên lạc được vì an toàn thông tin chẳng hạn. 
Tương tự với Image và Volume, chúng ta cũng có thể tạo, xóa, liệt kê và xem thông tin từng network. 
> # docker network create <network name> 

> # docker network rm <network id/name> 

> # docker network ls 

> # docker network inspect <network id/name> 
Hoặc bạn có thể tìm hiểu thêm với help. 
> # docker network –help

<a name="step4"></a>
## Dockerfile 
Dockerfile là một file setup scripting, nó sẽ thực thi một loạt các cài đặt library hoặc các bước chuẫn bị cho application. Mục đích của file này là sau khi hoàn thành thì nó sẽ tạo ra một file image mới hoàn toàn theo ý của bạn. 
`Ví dụ:` bạn muốn có file image mà chạy cả postfix và dovecot chung 1 container, thì dockerfile sẽ là cái mà bạn cần lưu tâm. 
Dưới đây mình có gợi ý cơ bản về Dockerfile như sau. 
# Download image của debian 
FROM debian:jessie-slim 
# Chuẩn bị    environment cho app 
``` sh
ENV OE_USER='openerp'\ 
OE_PATH='/home/openerp'\ 
LANG='en_US.UTF-8'\ 
LANGUAGE='en_US:en'\ 
LC_ALL='en_US.UTF-8'\ 
DEBIAN_FRONTEND=noninteractive 
```
# Copy source openerp từ    máy chính vào container. 
> COPY ./openerp6.tar.gz /tmp 

# Thực hiện một loạt cài đặt, giải nén file v.v… 
``` sh
RUN set -x \ 
&& mkdir /usr/share/man/man1 \ 
&& apt-get update \ 
&& apt-get upgrade -y \ 
&& tar xzf /tmp/openerp6.tar.gz -C ${OE_PATH} \ 
```

# Khai báo các volume được phép link từ    máy chính 
> VOLUME ["/home/openerp/conf", "/var/log/openerp"] 

# Mở    port kết nối từ    ngoài vào 
> EXPOSE 8069 8070 

# Câu lệnh thực thi khi container bắt đầu chạy. 
> CMD ["supervisord", "-c", "/etc/supervisor/supervisord.conf"] 

Docker file trên là cơ bản cho hầu hết các image hiện nay. Bạn có thể tham khảo thêm trên github nơi có rất nhiều docker file cho bạn nghiên cứu. 
Sau khi hoàn tất file docker, bạn cho build image, đầu tiên đi vào đường dẫn mà đang đang chứa file có tên là Dockerfile, sau đó thực thi câu lệnh 
> # docker build . -t <tag name> 

Tag name là tên của image và version, VD: “openerp/1.0”, nếu không có tag name thì docker sẽ tự tạo image id. Có thể trong lúc tạo image, docker sẽ trả về error vì sai cấu trúc, bạn chỉ cần follow theo chỗ sai và sửa trên 
dockerfile. 
Sau khi hoàn thành thì bạn có thể dùng “# docker images” để liệt kê ra cái image mà bạn mới tạo.

<a name="tongket"></a>
## III. Summary:

Watch Video here: 

- [How to Install and configure Samba on Ubuntu 20.04 Part 1:  Public folder](https://youtu.be/2o5zgA8ml38)
- [How to configure and Install Samba on Ubuntu 20.04 Part2: Private Share](https://youtu.be/6s9ZEp3xS94)

**Contact me:**
- Email: manhhungbl@gmail.com
- Skype: spyerx3
- Youtube Channel: youtube.com/howtoused

Thank you very much!
