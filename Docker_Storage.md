# Docker Storage

Mặc định rằng tất cả những file khởi tạo bên trong container được lưu trữ trong một Writable Container Layer. Nó có nghĩa rằng:
    * Dữ liệu không tồn tại khi mà container không tồn tại và khó có thể lấy được dữ liệu ra ngoài nếu một process đang cần.
    * Writable Container Layer của một container được liên kết chạt chẽ với các host machine nơi mà container đang chạy. Bạn khó di chuyển dữ liệu đi nơi khác.
    * Viết vào bên trong Writable Container Layer của một container yêu cầu một Storage driver để quản lí filesystem. Storage driver cung cấp một hệ thống hợp nhất(union filesystem), sử dụng nhân Linux. 

Docker có 2 lựa chọn cho container lưu trữ file trên host. Vì vậy mà các phai đại diện cho từng container sau khi stops: _volumes_ và _bind mounts_. Nếu bạn không chạy docker trong linux thì bạn cũng có thể sử dụng một _tmpfs mounts_

## Tổng quan
### Chọn kiểu mount 

* **Volumes** được lưu trữ trong một phần của host file system - phân vùng quản lí của Docker _(/var/lib/docker/volumes/)_. Không nên có bất kì sửa chữa nào trong thư mục này ngoài các thao tác của Docker.
* **Bind mounts** có thể được lưu trữ tại bất cứ đâu trên host hệ thống. Chúng có thể là file hệ thống quan trọng hoặc những thư mục,...
* **tmpfs mounts** chỉ được lưu trữ trong host system’s memory và không bao giờ viết nó trong host system’s filesystem.

### Một số chi tiết về kiểu Mount
* **Volumes**: Được khởi tạo và quản lý bởi docker. Bạn có thể khởi tạo một volume bằng cách sử dụng lệnh  _docker volume create_.
* Khi bạn khởi tạo mộtvolumes  volume. Nó sẽ được lưu trữ trong mộ thư muc trong Docker host _(/var/lib/docker/volumes/)_. Khi bạn mount volume vào trong một container thì mục này là mục được mout vào bên trong docker. Điều này tương tự với **bind mounts**, ngoại trừ việc volumes được quản lý bởi docker và được isolated từ core functionality of host machine.
* Một volume có thể được mount vào bên trong nhiều container. Khi không có container nào đang chạy mà sử dụng volume, volume đó vẫn sẽ sẵn sàng và không lo bị xóa tự động. Bạn có thể xóa volume không sử dụng bằng lệnh _docker volume prune_.
* Khi bạn mount một volume. Nó có thể đã được đặt tên hoặc vô danh. Với  Volume  vô danh và các thông số thì sẽ được đặt tự động và là duy nhất. Các volume cũng được hỗ trợ sử dụng của _volume driver_ - cái mà chấp nhận để bạn lưu trữ dữ liệu cuả bạn bên trong một remote hosts hoặc cloud, ...

* **Bind mounts**: Khi bạn sử dụng bind mount, một file hoặc danh mục trong host machine được mount vào trong host machine. Địa chỉ file hoặc directory sẽ được tham chiếu bởi một đường dẫn đầy đủ. File và Directory không cần tồn tại trong Docker host vì nếu không có thì nó sẽ tự tạo. Bạn không thể sử dụng Docker CLI commands để quản lý trực tiếp các liên kết gắn kết.

* **tmpfs mounts**: Gắn kết tmpfs không được duy trì trên đĩa, trên máy chủ Docker hoặc trong một container. Nó có thể được sử dụng bởi một container trong suốt vòng đời của container, để lưu trữ trạng thái không liên tục hoặc thông tin nhạy cảm. Ví dụ, trong nội bộ, các dịch vụ swarm sử dụng các tmpfs mount để gắn kết các bí mật vào một container chứa dịch vụ.

### Khi nào sử dụng Volumes ?
* Chia sẻ dữ liệu thông qua nhiều container đang chạy. Nếu bạn không khởi tạo nó. Một Volume đã được khởi tạo lần đầu tiên sẽ được mount trong container. Khi container đó dừng hoặc bị xóa đi thì volume đó vẫn tồn tại. Nhiều Containers có thể mount vào một volume một cách đồng thời. Volumes chỉ được loại bỏ khi bạn xóa chúng.
* Khi máy chủ docker không chắc chắn có thư mục hay cấu trúc file, volumes giúp bạn phân tách cài đặt của máy docker khỏi môi trường chạy của container.
* Khi bạn muốn lưu trữ đữ liệu trong những container trong remote host hoặc cloud hơn là ở locally.
* Khi bạn cần backup, lưu trữ hoặc di chuyển sang một container khác. Volume là sự lựa chọn tốt nhất. Bạn có thể dừng container đang sử dụng volume đó. sau đó backup lại volume(Volume được lưu trong thư mục _/var/lib/docker/volumes/<tên volume>_ )

## Khi nào sử dụng bind mounts?
Thông thường việc sử dụng volumes là một sự lựa chọn tốt nếu có thể. Bind mounts phù hợp với những trường hợp sau:
* Chia sẻ configure từ máy thật sang container. Để làm việc này thì tìm hiểu thư mục _/etc/resolv.conf_.
* Chia sẻ mã nguồn hoặc xây dựng các tạo phẩm giữa một môi trường phát triển trên máy chủ Docker và một container. Chẳng hạn, bạn có thể gắn một thư mục _target/_ Maven vào một container và mỗi lần bạn xây dựng dự án Maven trên máy chủ Docker, container sẽ có quyền truy cập vào các tạo phẩm được xây dựng lại.
Nếu bạn sử dụng docker cho việc phát triển theo cách này. Sản phẩm từ Dockerfile của bạn sẽ copy từ trong image thay vì dựa vào một liên kết.
* Khi cấu trúc tệp hoặc thư mục của máy chủ Docker được đảm bảo phù hợp với các liên kết gắn kết mà các container yêu cầu.

### Khi nào sử dụng tmpfs mounts?
* tmpfs mount là sự lựa chọn tốt nhất trong một số trường hợp khi bạn không muốn dữ liệu của bạn tồn tại hoặc trong máy chủ hoặc container. Điều này có thể là vì lý do bảo mật hoặc để bảo vệ hiệu suất của container khi ứng dụng của bạn cần ghi một khối lượng lớn dữ liệu trạng thái không liên tục.

### Mẹo sử dụng bind mounts hoặc volumes.
Nếu bạn sử dụng bind mounts hoặc volumes. Hãy suy xét những điều kiện sau:
* Nếu bạn mount vào một volume rỗng bên trong một directory tồn tại trong docker. Những files hoặc directories này sẽ được sao chép vào ổ đĩa. Tương tự, nếu bạn bắt đầu một container và một volumes chưa tồn tại. Một empty volume ngay lập tức sẽ được khởi tạo cho bạn. 
* Nếu bạn **bind mount hoặc non-empy volume** vào bên trong một directory trong container mà nó tồn tại, thì những file hoặc directories. Giống như khi bạn lưu file vào bên trong _/mnt_ trong Linux host và sau đó bạn mounted một USB vào trong _/mnt_. Nội dung của / mnt sẽ bị che khuất bởi nội dung của ổ USB cho đến khi ổ USB không được kết nối. Các tập tin bị che khuất không bị xóa hoặc thay đổi, nhưng không thể truy cập được trong khi gắn kết hoặc âm lượng được gắn kết.

## Volumes
**Volumes** là cơ chế ưa thích cho việc lưu trữ dữ liệu bởi docker container. Trong khi **bind mounts** phụ thuộc vào cấu trúc directory của máy chủ. Volumes là được quản lí hoàn toàn bởi docker. **Volumes** có lợi thế hơn **bind mounts** như sau:
	* Volumes dễ dàng để backup hoặc di chuyển hơn bind mounts.
	* Bạn có thể quản lí các volumes bằng cách sử dụng Docker CLI Command hoặc Docker API.
	* Volumes làm việc trong cả windows và Linux.
	* Volumes có thể an toàn trong việc chia sẻ với nhiều containers.
	* Volume driver giúp bạn lưu trữ volumes trong những remote hosts hoặc cloud. Để mã hóa nội dung của volumes hoặc thêm những chức năng khác.
	* New volumes can have their content pre-populated by a container.
 
Ngoài ra, volumes thường sự lựa chọn tốt với những dữ liệu lưu trữ lâu dài bên trong container. Bởi vì một volumes không tăng size của containers mà nó sử dụng, và Nội dung của Volumes tồn tại bên ngoài của vòng đời của một container.
Nếu vùng chứa của bạn tạo dữ liệu trạng thái không liên tục, hãy cân nhắc sử dụng **tmpfs mounts** để tránh lưu trữ dữ liệu ở bất cứ đâu vĩnh viễn và để tăng hiệu suất của Container container bằng cách tránh ghi vào lớp có thể ghi vào container.

Volumes sử dụng ràng buộc _rprivate_ và nó không configurable cho volumes.



