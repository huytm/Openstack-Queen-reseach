
### 1. Tạo sửa xóa project bằng CLI
># openstack project --help

**Tạo project**
`# openstack project create huy-test-project --domain default`

```
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description |                                  |
| domain_id   | default                          |
| enabled     | True                             |
| id          | 1352f79aecb240afa531922e8771cb20 |
| is_domain   | False                            |
| name        | huy-test-project                 |
| parent_id   | default                          |
| tags        | []                               |
+-------------+----------------------------------+
```

**Sửa project**

Show list project -> Chọn project cần sửa 
```
# openstack project list 
# openstack project set --name huytest123 --description 'this is my project' 1352f79aecb240afa531922e8771cb20
# openstack project show 1352f79aecb240afa531922e8771cb20
```
```
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | this is my project               |
| domain_id   | default                          |
| enabled     | True                             |
| id          | 1352f79aecb240afa531922e8771cb20 |
| is_domain   | False                            |
| name        | huytest123                       |
| parent_id   | default                          |
| tags        | []                               |
+-------------+----------------------------------+
```

**Xóa Project**
```
# openstack project delete 1352f79aecb240afa531922e8771cb20
# openstack project show 1352f79aecb240afa531922e8771cb20
No project with a name or ID of '1352f79aecb240afa531922e8771cb20' exists.
```

## 2. Tạo, sửa, xóa user

**Tạo user**
`# openstack user create --project 1352f79aecb240afa531922e8771cb20 --password passworddaivakho huytm-user`
```
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| default_project_id  | 1352f79aecb240afa531922e8771cb20 |
| domain_id           | default                          |
| enabled             | True                             |
| id                  | ee398851a436479cadac9690180bd94e |
| name                | huytm-user                       |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

**Sửa thông tin user**
```
# openstack user set --name huytm-user1 ee398851a436479cadac9690180bd94e
# openstack user show ee398851a436479cadac9690180bd94e
```
```
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| default_project_id  | 1352f79aecb240afa531922e8771cb20 |
| domain_id           | default                          |
| enabled             | True                             |
| id                  | ee398851a436479cadac9690180bd94e |
| name                | huytm-user1                      |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

**Gán role cho user**
Check danh sách các role hiện tại
`# openstack role list`
```
+----------------------------------+-------+
| ID                               | Name  |
+----------------------------------+-------+
| abeec726dac143c885c7e8a757db805c | admin |
| b0d29b776dd6444c842004daabac1ab9 | user  |
+----------------------------------+-------+
```
Gán role:
```
# openstack role add --user huytm-user --project 1352f79aecb240afa531922e8771cb20 admin
# openstack role assignment list --user huytm-user --project 1352f79aecb240afa531922e8771cb20
```

```
+----------------------------------+----------------------------------+-------+----------------------------------+--------+-----------+
| Role                             | User                             | Group | Project                          | Domain | Inherited |
+----------------------------------+----------------------------------+-------+----------------------------------+--------+-----------+
| abeec726dac143c885c7e8a757db805c | ee398851a436479cadac9690180bd94e |       | 1352f79aecb240afa531922e8771cb20 |        | False     |
+----------------------------------+----------------------------------+-------+----------------------------------+--------+-----------+

```

**Xóa user**

```
# openstack user delete ee398851a436479cadac9690180bd94e
# openstack user show ee398851a436479cadac9690180bd94e
No user with a name or ID of 'ee398851a436479cadac9690180bd94e' exists.
```

## Show Database của keystone và các bảng cơ bản
```
# mysql -uroot -p
MariaDB [keystone]> show databases;
MariaDB [keystone]> use keystone
MariaDB [keystone]> show tables;
MariaDB [keystone]> select * from role
MariaDB [keystone]> select * from user;
MariaDB [keystone]> select * from project
MariaDB [keystone]> select * from password;
MariaDB [keystone]> select * from service;
```

## Tìm dòng cấu hình kiểu token của keystone
`# cat /etc/keystone/keystone.conf | grep 'token' -C 5 | grep provider --color`

## Logging

File log của keystone nằm ở `/var/log/keystone/keystone.log`

**Ánh xạ việc tạo, sửa, xóa user vào file log**
_Tạo user_ method **PUT**
```
2019-01-07 23:26:30.159 3672 INFO keystone.common.wsgi [req-823e54b1-ee6c-4b25-89ea-4515b96f69fe 21b515c1a0b04576bbf0a66dd89e5aec ad27d3d1462b45fa94ff42c7d68d352a - default default] PUT http://10.10.10.175:5000/v3/projects/1352f79aecb240afa531922e8771cb20/users/ee398851a436479cadac9690180bd94e/roles/abeec726dac143c885c7e8a757db805c
```

_Sửa user_ method **PATCH**

```
2019-01-07 23:28:57.299 3671 INFO keystone.common.wsgi [req-a26c21d4-1cbb-4acd-b5ea-0cde73e5a3cd 21b515c1a0b04576bbf0a66dd89e5aec ad27d3d1462b45fa94ff42c7d68d352a - default default] PATCH http://10.10.10.175:5000/v3/users/ee398851a436479cadac9690180bd94e
```

_Xóa user_ method **DELETE**

```
2019-01-07 23:38:10.546 3670 INFO keystone.common.wsgi [req-967b269d-c96e-45a9-bbdf-6fab6c3506f4 21b515c1a0b04576bbf0a66dd89e5aec ad27d3d1462b45fa94ff42c7d68d352a - default default] DELETE http://10.10.10.175:5000/v3/users/ee398851a436479cadac9690180bd94e
```
**Ánh xạ việc tạo, sửa, xóa project vào file log**
_Tạo project_ method **PUT**
```
2019-01-07 23:26:30.159 3672 INFO keystone.common.wsgi [req-823e54b1-ee6c-4b25-89ea-4515b96f69fe 21b515c1a0b04576bbf0a66dd89e5aec ad27d3d1462b45fa94ff42c7d68d352a - default default] PUT http://10.10.10.175:5000/v3/projects/1352f79aecb240afa531922e8771cb20/users/ee398851a436479cadac9690180bd94e/roles/abeec726dac143c885c7e8a757db805c
```

_Sửa project_ method **PATCH**

```
2019-01-07 23:28:57.299 3671 INFO keystone.common.wsgi [req-a26c21d4-1cbb-4acd-b5ea-0cde73e5a3cd 21b515c1a0b04576bbf0a66dd89e5aec ad27d3d1462b45fa94ff42c7d68d352a - default default] PATCH http://10.10.10.175:5000/v3/users/ee398851a436479cadac9690180bd94e
```

_Xóa project_ method **DELETE**

```
2019-01-07 23:39:56.457 3674 INFO keystone.common.wsgi [req-7c813d1f-1207-4589-b114-51e8181e33b5 21b515c1a0b04576bbf0a66dd89e5aec ad27d3d1462b45fa94ff42c7d68d352a - default default] DELETE http://10.10.10.175:5000/v3/projects/1352f79aecb240afa531922e8771cb20
```
