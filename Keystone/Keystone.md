Thông thường ở các phiên bản Openstack trước các **Service** Openstack như **Keystone**, **Nova**, **Glance**, **Neutron..** thường được gọi là các **Project**. Để tránh nhầm lẫn, trong ghi chú này các **service** sẽ được gọi là **module** trong openstack để tránh nhầm lẫn với khái niệm project trong keystone

### Keystone là gì
Keystone là một module trong openstack, **nhiệm vụ chính là để xác thực người dùng và quản lý các module khác** trong openstack.

### Chức năng
Từ định nghĩa phía trên thì có thể nhận thấy keystone có hai chức năng chính bao gồm **Quản lý người dùng, Quản lý các module khác**.

- Quản lý người dùng: Xác thực danh tính và quyền hạn của người dùng
- Quản lý các module khác: Cung cấp 1 catalog service của các module khác. Bao gồm service và endpoint service

--> openstack catalog list

### Các khái niệm cơ bản
1. **User**: User có thể là một cá nhân (con người) sử dụng openstack hoặc một service (module) khác - vì bản thân mỗi service đều có một user riêng.

2. **Credentials**: Là thông tin xác thực được danh tính của người dùng nó có thể tồn tại ở 3 dạng sau:
   - username / password
   - username và API Key (Secrect key)
   - Token được assigned bởi keystone (Token này chứa list các role của user)

   Trong đó phương thức 1 và 2 được sử dụng cho người dùng khi xác thực lần đầu tiên. Khi đã xác thực lần đầu và nhận được token thì từ đó về sau sử dụng token để xác thực tiếp.

3. **Authentication**: Là quá trình sử dụng Credentials để xác thực danh tính người dùng.

4. **Token**: Khi người dùng đã được xác thực thì người dùng sẽ được nhận một token. Token có các đặc điểm sau:
   - Có life span, mặc định sẽ expried trong 1 tiếng
   - Có 4 loại token chính (Kể từ Openstack Ocata thì có 2 loại token được support là UUID và Fernet)
      + UUID tokens
      + PKI and PKIZ tokens
      + Fernet tokens

5. **Role**: Là tập hợp các ACLs chỉ định vai trò của người dùng có 2 role mặc định là:
   - admin
   - member
 
   _Một user cần phải có một project đi kèm để xác định được role_

6. **Policy**: Là một file JSON chứa các bộ quyền nằm ở /etc/keystone/policy.json
 
   _Nếu cấu hình keystone sử dụng file này nó sẽ là các ACLs dùng để xác thực quyền hạn cho người dùng_

7. **Project**: Là tập hợp các tài nguyên thuộc sở hữu của một người dùng, một nhóm những người dùng hoặc một service
   - Một project có thể có một user hoặc có nhiều user, sử dụng tài nguyên tùy thuộc vào role của user đó
   - Trước khi một user có thể sử dụng tài nguyên của project thì user phải được liên kết với project đó và được khởi tạo role trong project

8. **Service: Service** - là một module trong openstack ví dụ như Glance, Nova, Neutron...

9. **Endpoint**
   - Endpoint là một API Url mà tại đó user có thể giao tiếp với service
   - Với các region khác nhau thì có các endpoint khác nhau
   - Có 3 loại endpoint là admin url, internal url, public url. Trên môi trường product, mỗi loại endpoint này nên đc gắn với 1 dải mạng riêng.
      + Admin có thể modify user, project.
      + Public - client là người dùng quản trị cloud server thông qua external network
      + Internal dùng để giao tiếp giữa các service 

10. **Group**: Group là một nhóm nhiều người dùng
    - Group phải thuộc một Domain
    - Group không phải là duy nhất trên toàn bộ hệ thống mà chỉ là duy nhất trong domain mà nó thuộc

11. **Domain** là một "container" chứa project, user, group.
    - Mỗi domain định nghĩa ra một namespace. API service cũng sẽ chứa thông tin về domain.
    - Keystone cung cấp một domain mặc định tên là "Default"
    - Domain là duy nhất trên toàn bộ hệ thống

** Ví dụ

** Thực hành

https://www.cnblogs.com/charles1ee/p/6293387.html
https://docs.openstack.org/security-guide/identity/tokens.html
https://docs.openstack.org/keystone/pike/getting-started/architecture.html
