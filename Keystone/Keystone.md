_Thông thường ở các phiên bản Openstack trước các **Service** Openstack như **Keystone**, **Nova**, **Glance**, **Neutron..** thường được gọi là các **Project**. Để tránh nhầm lẫn, trong ghi chú này các **service** sẽ được gọi là **module** trong openstack để tránh nhầm lẫn với khái niệm project trong keystone_

## Keystone là gì ?
Keystone là một module trong openstack, **nhiệm vụ chính là để xác thực người dùng và quản lý các module khác** trong openstack.

![Keystone_function](https://raw.githubusercontent.com/huytm/Openstack-Queen-reseach/master/Keystone/image/keystone_f.png)

## Chức năng
Từ định nghĩa phía trên thì có thể nhận thấy keystone có hai chức năng chính bao gồm **Quản lý người dùng, Quản lý các module khác**.

- Quản lý người dùng: Xác thực danh tính và quyền hạn của người dùng
- Quản lý các module khác: Cung cấp 1 catalog service của các module khác. Bao gồm service và endpoint service
   
```shell
openstack catalog list
```
![Catalog](https://raw.githubusercontent.com/huytm/Openstack-Queen-reseach/master/Keystone/image/catalog.png)

## Các khái niệm cơ bản

1. User: User có thể là một cá nhân (con người) sử dụng openstack hoặc một service (module) khác - vì bản thân mỗi service đều có một user riêng.

2. Credentials: Là thông tin xác thực được danh tính của người dùng nó có thể tồn tại ở 3 dạng sau:

- username / password
- username và API Key (Secrect key)
- Token được assigned bởi keystone (Token này chứa list các role của user)

   _Trong đó phương thức 1 và 2 được sử dụng cho người dùng khi xác thực lần đầu tiên. Khi đã xác thực lần đầu và nhận được token thì từ đó về sau sử dụng token để xác thực tiếp._

3. Authentication: Là quá trình sử dụng Credentials để xác thực danh tính người dùng.

4. Token: Khi người dùng đã được xác thực thì người dùng sẽ được nhận một token. Token có các đặc điểm sau:

- Có life span, mặc định sẽ expried trong 1 tiếng
- Có 4 loại token chính (Kể từ Openstack Ocata thì có 2 loại token được support là UUID và Fernet)
    + UUID tokens
    + PKI and PKIZ tokens
    + Fernet tokens

5. Role: Là tập hợp các ACLs chỉ định vai trò của người dùng có 2 role mặc định là:

- admin
- member
 
   _Một user cần phải có một project đi kèm để xác định được role_

6. Policy: Là một file JSON chứa các bộ quyền nằm ở `/etc/keystone/policy.json`
 
   _Nếu cấu hình keystone sử dụng file này nó sẽ là các ACLs dùng để xác thực quyền hạn cho người dùng_

7. Project: Là tập hợp các tài nguyên thuộc sở hữu của một người dùng, một nhóm những người dùng hoặc một service

- Một project có thể có một user hoặc có nhiều user, sử dụng tài nguyên tùy thuộc vào role của user đó
- Trước khi một user có thể sử dụng tài nguyên của project thì user phải được liên kết với project đó và được khởi tạo role trong project

8. Service: Service - là một module trong openstack ví dụ như Glance, Nova, Neutron...

9. Endpoint

- Endpoint là một API Url mà tại đó user có thể giao tiếp với service
- Với các region khác nhau thì có các endpoint khác nhau
- Có 3 loại endpoint là admin url, internal url, public url. Trên môi trường product, mỗi loại endpoint này nên đc gắn với 1 dải mạng riêng.
    + Admin có thể modify user, project.
    + Public - client là người dùng quản trị cloud server thông qua external network
    + Internal dùng để giao tiếp giữa các service 

10. Group: Group là một nhóm nhiều người dùng

- Group phải thuộc một Domain
- Group không phải là duy nhất trên toàn bộ hệ thống mà chỉ là duy nhất trong domain mà nó thuộc

11. Domain là một "container" chứa project, user, group.

- Mỗi domain định nghĩa ra một namespace. API service cũng sẽ chứa thông tin về domain.
- Keystone cung cấp một domain mặc định tên là "Default"
- Domain là duy nhất trên toàn bộ hệ thống

## Ví dụ
**Ví dụ 1:**

User muốn tạo máy ảo

![Create_vm](https://raw.githubusercontent.com/huytm/Openstack-Queen-reseach/master/Keystone/image/keystone_create_vm.jpg)

1. User sẽ xác thực với keystone
2. Keystone sẽ xác thực người dùng và trả cho người dùng 1 token và catalog service
3. Người dùng sử dụng token này để yêu cầu tạo VM đến endpoint Nova service
4. Nova service sẽ validate lại token này với keystone xem có hợp lệ không và xác định được role của user
5. Nếu hợp lệ thì Nova sẽ sử dụng token này để yêu cầu 1 image để boot máy đến Glance
6. Glance tương tự, sẽ vidate lại token này với keystone và nếu hợp lệ sẽ trả lại cho Nova 1 image
7. Nova tiếp tục sử dụng token user và request network cho VM đến Neutron
8. Neutron validate lại token với keystoen và trả lại phần network cho VM
9. Tùy vào luồng xử lý bên trong, nếu đủ các thông tin cần thiết để tạo một VM rồi thì Nova sẽ trả lại cho người dùng 1 response thành công.

**Ví dụ 2:**
Quá trình xác thực trong Openstack

![Keystoneflow](https://raw.githubusercontent.com/huytm/Openstack-Queen-reseach/master/Keystone/image/keystone.png)


1. User gửi credentials cho keystone. Nếu thành công, Keystone sẽ trả lại cho User 1 token và một catalog endpoint các service.
2. User dùng token này để xác định với keystone các project mà mình sở hữu, Keystone sẽ trả về 1 danh sách các project mà user sở hữu.
3. User gửi thông tin project muốn sử dụng đến keystone, Keystone trả lại một list các service user có thể sử dụng, dựa vào role của user đối với Project đó và cung cấp một token khác là tenant token. 

>Token đầu tiên là để xác định User có được phép giao tiếp với keystone khôn.
>Token thử hai là để xác minh rằng User có quyền truy cập đến các service khác không.

4. User sử dụng tenant token (token thứ hai) để yêu cầu tạo vm đến nova service - ở trong hình là Endpoint,  Service nhận được token này sẽ verify lại với keystone xem có hợp lệ không và có được sử dụng resource của serive hay không?
5. Keystone sẽ xác thực lại token - role của user và trả lại thông báo success cho service
6. Service thực hiện request.
7. Sevice trả về thông báo tạo vm thành công cho user

## Thực hành

[Note thực hành](./Practice.md)


## Tài liệu tham khảo
https://www.cnblogs.com/charles1ee/p/6293387.html
https://docs.openstack.org/security-guide/identity/tokens.html
https://docs.openstack.org/keystone/pike/getting-started/architecture.html
