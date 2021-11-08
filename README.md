# **Cách cài đặt Snort 3 trên Server Ubuntu 20.04**

**Bước 1 : Cập nhật server**



```
sudo apt update
sudo apt upgrade
```



**Bước 2 : Cài đặt thư viện bắt buộc**

Cài đặt các thư viện phụ thuộc của Snort 
```
sudo apt install build-essential libpcap-dev libpcre3-dev libnet1-dev zlib1g-dev luajit hwloc libdnet-dev libdumbnet-dev bison flex liblzma-dev openssl libssl-dev pkg-config libhwloc-dev cmake cpputest libsqlite3-dev uuid-dev libcmocka-dev libnetfilter-queue-dev libmnl-dev autotools-dev libluajit-5.1-dev libunwind-dev
```

Tạo thư mục để lưu trữ Snort
```
mkdir snort-source-files
cd snort-source-files
```

Cài đặt phiên bản mới nhất của thư viện Snort Data Acquisition (LibDAQ)
```
git clone https://github.com/snort3/libdaq.git
cd libdaq
./bootstrap
./configure
make
sudo make install
```
Cài đặt Tcmalloc để tối ưu hóa việc phân bổ bộ nhớ và cung cấp khả năng sử dụng bộ nhớ tốt hơn.


```
cd ../
wget https://github.com/gperftools/gperftools/releases/download/gperftools-2.9.1/gperftools-2.9.1.tar.gz
tar xzf gperftools-2.9.1.tar.gz 
cd gperftools-2.9.1/
./configure
make 
sudo make install
```
Bước 3: Cài đặt Snort 3 trên Ubuntu 20.04
1. Clone git snort về 
```
cd ../
git clone git://github.com/snortadmin/snort3.git
```
2. Vào thư mục Snort3
```
cd snort3/
```
3. Cấu hình và kích hoạt Tcmalloc
```
./configure_cmake.sh --prefix=/usr/local --enable-tcmalloc
``` 
4. Cài đặt Snort3 
```
cd build
make 
sudo make install
```
5. Cập nhật thư viện được chia sẻ 
```
sudo ldconfig
```
6. Kiểm tra xem đã cài được chưa 
```
snort -V
```

Output 
```
nhatnguyennhn@snortanninhmang:~/snort-source-files/snort3/build$ snort -V

   ,,_     -*> Snort++ <*-
  o"  )~   Version 3.1.16.0
   ''''    By Martin Roesch & The Snort Team
           http://snort.org/contact#team
           Copyright (C) 2014-2021 Cisco and/or its affiliates. All rights reserved.
           Copyright (C) 1998-2013 Sourcefire, Inc., et al.
           Using DAQ version 3.0.5
           Using LuaJIT version 2.1.0-beta3
           Using OpenSSL 1.1.1f  31 Mar 2020
           Using libpcap version 1.9.1 (with TPACKET_V3)
           Using PCRE version 8.39 2016-06-14
           Using ZLIB version 1.2.11
           Using LZMA version 5.2.4
```

**Cấu hình Card mạng**

Bật chế độ promiscuous mode để xem lưu lượng mạng gửi đến Snort
```
ip link set dev eth0 promisc on
```
Xác minh 
```
ip add sh eth0
```

Output 
```
nhatnguyennhn@snortanninhmang:~$ ip add sh eth0
2: eth0: <BROADCAST,MULTICAST,PROMISC,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0d:3a:82:5e:ea brd ff:ff:ff:ff:ff:ff
    inet 10.1.0.6/24 brd 10.1.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::20d:3aff:fe82:5eea/64 scope link
       valid_lft forever preferred_lft forever
```
Vô hiệu hóa  Offloading mode để ngăn Snort 3 cắt bớt các gói tin lớn, tối đa là 1518 byte. Kiểm tra xem tính năng này có được bật hay không bằng lệnh sau.
```
ethtool -k eth0 | grep receive-offload
```
Nếu nhìn thấy Output này, tức là GRO được bật trong khi LRO được sửa hoặc LRO được bật.

```
generic-receive-offload: on
large-receive-offload: on
```
Tắt bằng lệnh
```
ethtool -K eth0 gro off lro off
```
Tạo  systemd service 
```
sudo nano /etc/systemd/system/snort3-nic.service
```
Copy cấu hình sau vào file vừa tạo
```
[Unit]
Description=Set Snort 3 NIC in promiscuous mode and Disable GRO, LRO on boot
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/sbin/ip link set dev eth0 promisc on
ExecStart=/usr/sbin/ethtool -K eth0 gro off lro off 
TimeoutStartSec=0 
RemainAfterExit=yes

[Install]
WantedBy=default.target
```
Reload lại  systemd service :
```
sudo systemctl daemon-reload
```
Cho systemd service khởi động chung với server 
```
sudo systemctl enable --now snort3-nic.service
```
Output 
```
Created symlink /etc/systemd/system/default.target.wants/snort3-nic.service → /etc/systemd/system/snort3-nic.service.
```
Xác minh snort3-nic.service
```
sudo systemctl status snort3-nic.service
```
Output 
```
nhatnguyennhn@snortanninhmang:~$ sudo systemctl status snort3-nic.service
● snort3-nic.service - Set Snort 3 NIC in promiscuous mode and Disable GRO, LRO on boot
     Loaded: loaded (/etc/systemd/system/snort3-nic.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sun 2021-11-07 02:36:52 UTC; 1 day 10h ago
   Main PID: 51883 (code=exited, status=0/SUCCESS)
      Tasks: 0 (limit: 4107)
     Memory: 0B
     CGroup: /system.slice/snort3-nic.service

Nov 07 02:36:52 snortanninhmang systemd[1]: Starting Set Snort 3 NIC in promiscuous mode and Disable GRO, LRO on boot...
Nov 07 02:36:52 snortanninhmang systemd[1]: Finished Set Snort 3 NIC in promiscuous mode and Disable GRO, LRO on boot.
```
**Cài đặt rule**

Tạo thư mục rule 
```
mkdir /usr/local/etc/rules
```
Tải rule 
```
wget https://www.snort.org/downloads/community/snort3-community-rules.tar.gz
```
Giải nén rule
```
tar xzf snort3-community-rules.tar.gz -C /usr/local/etc/rules/
```
Snort 3 khác Snort 2 ở chỗ nó không có file snort.conf mà thay vào đó là 2 file snort_defaults.lua và snort.lua .

Tệp snort.lua chứa cấu hình chính của Snort.

Tệp snort_defaults.lua chứa các giá trị mặc định như url đến rule

Mở snort.lua 
```
sudo nano /usr/local/etc/snort/snort.lua
```
Thay đổi HOME_NET thành IP server bạn cài snort. Còn  EXTERNAL_NET  thành HOME_NET
```
-- HOME_NET and EXTERNAL_NET must be set now
-- setup the network addresses you are protecting
HOME_NET = '13.70.2.162/32'

-- set up the external network addresses.
-- (leave as "any" in most situations)
EXTERNAL_NET = 'any'
EXTERNAL_NET = '!$HOME_NET'

include 'snort_defaults.lua'
include 'file_magic.lua'
```
Thay đổi url rule 
```
ips = 
{     
-- use this to enable decoder and inspector alerts     
--enable_builtin_rules = true,     

-- use include for rules files; be sure to set your path     
-- note that rules files can include other rules files     
include = '/usr/local/etc/rules/snort3-community-rules/snort3-community.rules' 
}
```
**Chạy Snort**

Tạo tài khoản  non-login system user 
```
sudo useradd -r -s /usr/sbin/nologin -M -c SNORT_IDS snort
```
Tạo  systemd service để Snort chạy với tư cách user snort
```
sudo nano /etc/systemd/system/snort3.service
```
Copy dòng dưới bỏ vào
```
[Unit]
Description=Snort 3 NIDS Daemon
After=syslog.target network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/snort -c /usr/local/etc/snort/snort.lua -s 65535 -k none -l /var/log/snort -D -i eht0 -m 0x1b -u snort -g snort

[Install]
WantedBy=multi-user.target
```
Reload lại 
```
sudo systemctl daemon-reload
```
Thêm quyền sở hữu với cho phép vào log file
```
sudo chmod -R 5775 /var/log/snort
sudo chown -R snort:snort /var/log/snort
```
Reload lại để snort khởi động cùng server :
```
sudo systemctl enable --now snort3
```
Kiểm tra xem snort có chạy không
```
sudo systemctl status snort3
```
Output :
```
nhatnguyennhn@snortanninhmang:~$ sudo systemctl status snort3
● snort3.service - Snort 3 NIDS Daemon
     Loaded: loaded (/etc/systemd/system/snort3.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Mon 2021-11-08 07:33:47 UTC; 5h 29min ago
    Process: 67190 ExecStart=/usr/local/bin/snort -c /usr/local/etc/snort/snort.lua -s 65535 -k none -l /var/log/snort -D -i eht0 -m 0x1b -u snort -g snort (>
   Main PID: 67190 (code=exited, status=0/SUCCESS)

Nov 08 07:33:46 snortanninhmang snort[67190]: --------------------------------------------------
Nov 08 07:33:46 snortanninhmang snort[67190]: Module Statistics
Nov 08 07:33:46 snortanninhmang snort[67190]: --------------------------------------------------
Nov 08 07:33:46 snortanninhmang snort[67190]: Summary Statistics
Nov 08 07:33:46 snortanninhmang snort[67190]: --------------------------------------------------
Nov 08 07:33:46 snortanninhmang snort[67190]: timing
Nov 08 07:33:46 snortanninhmang snort[67190]:                   runtime: 00:00:00
Nov 08 07:33:46 snortanninhmang snort[67190]:                   seconds: 0.018443
Nov 08 07:33:46 snortanninhmang snort[67190]: o")~   Snort exiting
Nov 08 07:33:46 snortanninhmang systemd[1]: snort3.service: Succeeded.
```
