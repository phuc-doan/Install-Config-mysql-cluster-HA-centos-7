# Prepare: 🛠
- **1. Management Node - NDB_MGMD/MGM**

    The Cluster management server is used to manage the other node of the cluster. We can create and configure new nodes, restart, delete, or backup nodes on the cluster from the management node.

- **2. Data Nodes - NDBD/NDB**

    This is the layer where the process of synchronizing and data replication between nodes happens.

- **3. SQL Nodes - MySQLD/API**

    The interface servers that are used by the applications to connect to the database cluster.
    
 #### *Trong hướng dẫn này, tôi sẽ hướng dẫn bạn cài đặt và cấu hình MySQL Cluster với centOS 7. Chúng tôi sẽ cấu hình nút quản lý, hai nút dữ liệu và hai nút SQL 🎈🎈*
 
 ![image](https://user-images.githubusercontent.com/83824403/161948702-364dc533-365c-4b1f-a30c-4eb8e9072319.png)

# Excute: ✈

### STEP1: Cài Management node

- *note*: Bước đầu tiên là tạo "Nút quản lý" với CentOS 7 db1 và IP 192.168.1.120. Đảm bảo rằng bạn đã đăng nhập vào máy chủ db1 với tư cách là người dùng root.
- Tải gói cài đặt Mysql Cluster software

*Tôi sẽ tải xuống từ trang MySQL bằng wget. Tôi đang sử dụng "Red Hat Enterprise Linux 7 / Oracle Linux 7 (x86, 64-bit), Gói RPM" ở đây tương thích với CentOS 7. Sau đó giải nén tệp tar.*


```
cd ~
wget http://dev.mysql.com/get/Downloads/MySQL-Cluster-7.4/MySQL-Cluster-gpl-7.4.10-1.el7.x86_64.rpm-bundle.tar
tar -xvf MySQL-Cluster-gpl-7.4.10-1.el7.x86_64.rpm-bundle.tar
```
##### Kết quả như hình bên dưới:

![image](https://user-images.githubusercontent.com/83824403/161949368-a8581954-b58c-4133-b504-771316abe24e.png)

- Cài đặt và gỡ bỏ các gói

*note*: Trước khi cài đặt gói rpm cho MySQL Cluster, bạn cần cài đặt perl-Data-Dumper được yêu cầu bởi máy chủ MySQL-Cluster. Và bạn cần xóa mariadb-libs trước khi chúng tôi có thể cài đặt MySQL Cluster.

```
yum -y install perl-Data-Dumper
yum -y remove mariadb-libs
```

- Cài Mysql cluster: 
 
 *Chúng ta có thể tải ngay bằng rpm command hoặc có thể tự build, Ở đây mình tải luôn cho nhanh*
 
 ``` 
 cd ~
rpm -Uvh MySQL-Cluster-client-gpl-7.4.10-1.el7.x86_64.rpm
rpm -Uvh MySQL-Cluster-server-gpl-7.4.10-1.el7.x86_64.rpm
rpm -Uvh MySQL-Cluster-shared-gpl-7.4.10-1.el7.x86_64.rpm
```
 
##### *Hãy đảm bảo 100% quá trình trên ko lỗi lầm, nếu lỗi bước nào fix bước đó ngay tránh take time về sau*

- Cấu hình Mysql Cluster

1. tạo thư mục config. Ở đây tôi dùng /var/lib/mysql-cluster

```
mkdir -p /var/lib/mysql-cluster
```
2. Sau đó, tạo tệp cấu hình mới cho quản lý cụm có tên "config.ini" trong thư mục mysql-cluster


```
cd /var/lib/mysql-cluster
vi config.ini
```

3. hãy paste dòng sau

```
[ndb_mgmd default]
# Directory for MGM node log files
DataDir=/var/lib/mysql-cluster
 
[ndb_mgmd]
#Management Node db1
HostName=192.168.1.120
 
[ndbd default]
NoOfReplicas=2      # Number of replicas
DataMemory=256M     # Memory allocate for data storage
IndexMemory=128M    # Memory allocate for index storage
#Directory for Data Node
DataDir=/var/lib/mysql-cluster
 
[ndbd]
#Data Node db2
HostName=192.168.1.121
 
[ndbd]
#Data Node db3
HostName=192.168.1.122
 
[mysqld]
#SQL Node db4
HostName=192.168.1.123
 
[mysqld]
#SQL Node db5
HostName=192.168.1.124
```

- Bậ node quản lý này lên

- Có thể sử dụng tạm command này 


```
ndb_mgmd --config-file=/var/lib/mysql-cluster/config.ini
```

#### Kết quả đúng chắc chắn sẽ là

![image](https://user-images.githubusercontent.com/83824403/161950804-528b6def-78e5-4294-a7dd-5f98b027f5fb.png)
 
 *chúng ta sẽ sd lệnh **ndb-mgm** để giám sát thằng này*
 
 ```
 ndb_mgm
show
```

 
![image](https://user-images.githubusercontent.com/83824403/161951015-baecfe2f-5d05-4073-8601-e51e66b23380.png)

#### *Bạn có thể thấy nút quản lý đã được bắt đầu với: mysql-6.6 và ndb-7.4.*

### STEP2: Cài Mysql Cluster Data nodes

##### tôi sẽ sử dụng 2 máy chủ CentOS cho các nút dữ liệu.

- db2 = 192.168.1.121

- db3 = 192.168.1.122

**Đăng nhập với tư cách người dùng root và tải xuống phần mềm MySQL Cluster**

- Đăng nhập vào máy chủ db2 bằng ssh:

```
ssh root@192.168.1.121
```

- Tiếp theo vẫn là download và giải nén nó

```
cd ~
wget http://dev.mysql.com/get/Downloads/MySQL-Cluster-7.4/MySQL-Cluster-gpl-7.4.10-1.el7.x86_64.rpm-bundle.tar
tar -xvf MySQL-Cluster-gpl-7.4.10-1.el7.x86_64.rpm-bundle.tar
```
- Cài và gỡ bỏ các gói ( Giống phần trên )
- 
**Install perl-Data-Dumper and remove the mariadb-libs:**

```
yum -y install perl-Data-Dumper
yum -y remove mariadb-libs
```


-  Install MySQL Cluster

**Tiếp tục cài Mysql Cluster bằng rpm**

```
cd ~
rpm -Uvh MySQL-Cluster-client-gpl-7.4.10-1.el7.x86_64.rpm
rpm -Uvh MySQL-Cluster-server-gpl-7.4.10-1.el7.x86_64.rpm
rpm -Uvh MySQL-Cluster-shared-gpl-7.4.10-1.el7.x86_64.rpm
```
*Đảm bảo các bước trên không có lỗi gì*

- Cấu hình node dữ liệu

```
vi etc/my.conf/
```

- paste đoạn sau đây vào thư mục config

```
ndbcluster
ndb-connectstring=192.168.1.120     # IP address of Management Node
 
[mysql_cluster]
ndb-connectstring=192.168.1.120     # IP address of Management Node
```

- tạo Thư mục mới là mysql-cluster\

**Sau đó, tạo thư mục mới cho dữ liệu cơ sở dữ liệu mà chúng tôi đã xác định trong tệp cấu hình nút quản lý "config.ini".**

```
mkdir -p /var/lib/mysql-cluster
```

- Sau đó bật datanode/nbdb

```
ndbd
```
 - Kết quả sẽ như sau:


![image](https://user-images.githubusercontent.com/83824403/161952450-064da8c4-75f1-4608-88c9-6ab1b4ecea18.png)



![image](https://user-images.githubusercontent.com/83824403/161952496-7e7c315d-2025-4556-9c87-b8d77726e998.png)

#### Data Node db2 kết nối với nút quản lý ip 192.168.1.120.

#### Thực hiện lại bước trên máy chủ db3.

*Vì tôi sd 2 nút dữ liệu, vui lòng thực hiện lại các bước 2.A - 2.D trên nút dữ liệu thứ hai của tôi.*


### STEP3: Cài SQL node

- Đây là bước chứa thiết lập cho SQL Node cung cấp quyền truy cập ứng dụng vào cơ sở dữ liệu. Chúng tôi sử dụng 2 máy chủ CentOS cho các nút SQL:

**db4 = 192.168.1.123**

**db5 = 192.168.1.124**

- Bằng cách nào đó chúng ta vào server db4. ở đây tôi dùng ssh với root cho nhanh

```
ssh root@192.168.1.123
```

- Tải các gói Mysql Cluster packet

```
cd ~
wget http://dev.mysql.com/get/Downloads/MySQL-Cluster-7.4/MySQL-Cluster-gpl-7.4.10-1.el7.x86_64.rpm-bundle.tar
tar -xvf MySQL-Cluster-gpl-7.4.10-1.el7.x86_64.rpm-bundle.tar
```

- vẫn là remove các gói  conflict với mysql cluster

```
yum -y install perl-Data-Dumper
yum -y remove mariadb-libs
```
*Lưu ý:  install gói data dumper*

- Cài Mysql Cluster


- Cài đặt máy chủ, máy khách và gói chia sẻ MySQL Cluster bằng các lệnh rpm bên dưới:

```
cd ~
rpm -Uvh MySQL-Cluster-client-gpl-7.4.10-1.el7.x86_64.rpm
rpm -Uvh MySQL-Cluster-server-gpl-7.4.10-1.el7.x86_64.rpm
rpm -Uvh MySQL-Cluster-shared-gpl-7.4.10-1.el7.x86_64.rpm
```

- Cấu hình SQL node

- Tạo thư mục /etc/my.conf

```
vi /etc/my.conf
```
- Paste các dòng này vào thư mục vừa tạo

```
[mysqld]
ndbcluster
ndb-connectstring=192.168.1.120       # IP address for server management node
default_storage_engine=ndbcluster     # Define default Storage Engine used by MySQL
 
[mysql_cluster]
ndb-connectstring=192.168.1.120       # IP address for server management node
```
##### Giờ thì bật mysql thôi :))

```
systemctl start mysql
```

##### Lặp lại y hệt với server DB5

### STEP 4: Monitor chúng <333

- Để xem status cluster, chúng ta có thể đăng nhập vào db1/ db management node

```
ssh root@192.168.1.120
```


- Chúng ta có thể show từ commnand sau:

```
ndb_mgm
ndb_mgm> show
```
![image](https://user-images.githubusercontent.com/83824403/161954003-33f29532-8ed6-49e5-8e43-bf72b3cad7bf.png)


**Một số command nữa là**

```
ndb_mgm -e "all status"
ndb_mgm -e "all report memory"
```

### STEP 5: Kiểm tra nó thôi

- Để thực hiện kiểm tra trên Cụm MySQL mới của chúng tôi, chúng tôi phải đăng nhập vào máy chủ SQL Nodes db4 hoặc db5.

- Đăng nhập vào máy chủ db4:

```

ssh root@192.168.1.123
```

- Thay đổi mật khẩu MySQL mặc định được lưu trữ trong tệp ".mysql_secret" trong thư mục gốc:

```
cd ~

cat .mysql_secret
```

- đây là mẫu của tôi, nó sẽ hiện như này:

##### The random password set for the root user at Tue Mar 22 19:44:07 2016 (local time): qna3AwbJMuOnw23T
- Bây giờ thay đổi mật khẩu bằng lệnh dưới đây:

```

mysql_secure_installation
```


- Nhập mật khẩu mysql cũ của bạn và sau đó nhập mật khẩu mới, nhấn enter để xác nhận tất cả.

- Nếu tất cả đã xong, bạn có thể đăng nhập vào MySQL shell bằng mật khẩu của mình:

```
mysql -u root -p
```

- Sau khi bạn đăng nhập, hãy tạo một người dùng root mới với máy chủ lưu trữ "@", vì vậy chúng tôi sẽ có thể truy cập MySQL từ bên ngoài.

```
CREATE USER 'root'@'%' IDENTIFIED BY 'doanphuc298';
```


- Thay thế doanphuc298 bằng mật khẩu an toàn của riêng bạn! Bây giờ bạn có thể thấy người dùng root mới với máy chủ lưu trữ "@" trên danh sách người dùng MySQL:

```
select user, host, password from mysql.user;
```
- Và cấp cho người dùng gốc mới quyền truy cập đọc và ghi từ nút từ xa:

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY PASSWORD '*94CC7BF027327993D738E11...(Encrypted PASSWORD)' WITH GRANT OPTION;
```
![image](https://user-images.githubusercontent.com/83824403/161954783-fa8fa99f-576b-4fc9-9536-15408b2c2de6.png)

### Hehe WE all done!:

##### Bây giờ hãy thử tạo một cơ sở dữ liệu mới từ máy chủ db4 và bạn cũng sẽ thấy cơ sở dữ liệu trên db5.

![image](https://user-images.githubusercontent.com/83824403/161954978-7f7fb086-5050-43e9-9a87-bd411744131c.png)


## Good! Cụm MySQL đã được thiết lập thành công trên CentOS 7 với 5 nút máy chủ.


#### Phần kết luận

- MySQL Cluster là một công nghệ cung cấp tính khả dụng và dự phòng cao cho cơ sở dữ liệu MySQL. 
- Nó sử dụng NDB hoặc NDBCLUSTER làm công cụ lưu trữ và cung cấp tính năng phân nhóm không chia sẻ và tự động phân vùng cho cơ sở dữ liệu MySQL.
-  Để triển khai cluster, chúng ta cần 3 thành phần: Management Node (MGM), Data Nodes (NDB) và SQL Nodes (API). 
-  Mỗi nút phải có bộ nhớ và đĩa riêng. Không nên sử dụng lưu trữ mạng như NFS.
-   Để cài đặt MySQL Cluster trên hệ thống tối thiểu CentOS 7, chúng tôi phải xóa gói mariadb-libs, xung đột mariadb-libs với MySQL-Cluster-server và bạn phải cài đặt gói perl-Data-Dumper, gói này cần bởi MySQL-Cluster -máy chủ.
-    MySQL Cluster dễ cài đặt và cấu hình trên nhiều máy chủ CentOS.

# Hope you happy with my scripts
# *Author: Đoàn Văn Phúc ❤❤ *

