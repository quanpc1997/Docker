# Docker Storage

Mặc định rằng tất cả những file khởi tạo bên trong container được lưu trữ trong một Writable Container Layer. Nó có nghĩa rằng:
    * Dữ liệu không tồn tại khi mà container không tồn tại và khó có thể lấy được dữ liệu ra ngoài nếu một process đang cần.
    * Writable Container Layer của một container được liên kết chạt chẽ với các host machine nơi mà container đang chạy. Bạn khó di chuyển dữ liệu đi nơi khác.
    * Viết vào bên trong Writable Container Layer của một container yêu cầu một Storage driver để quản lí filesystem. Storage driver cung cấp một hệ thống hợp nhất(union filesystem), sử dụng nhân Linux. 

Docker có 2 lựa chọn cho container lưu trữ file trên host. Vì vậy mà các phai đại diện cho từng container sau khi stops: _volumes_ và _bind mounts_. Nếu bạn không chạy docker trong linux thì bạn cũng có thể sử dụng một _tmpfs mounts_

## Chọn kiểu mount 

* **Volumes** được lưu trữ trong một phần của host file system - phân vùng quản lí của Docker _(/vả/lib/docker/volumes/)_. Không nên có bất kì sửa chữa nào trong thư mục này ngoài các thao tác của Docker.
* **Bind mounts** có thể được lưu trữ tại bất cứ đâu trên host hệ thống. Chúng có thể là file hệ thống quan trọng hoặc những thư mục,...
* **tmpfs mounts** chỉ được lưu trữ trong host system’s memory và không bao giờ viết nó trong host system’s filesystem.

