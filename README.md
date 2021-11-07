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
wget https://github.com/gperftools/gperftools/releases/download/gperftools-2.9/gperftools-2.9.1.tar.gz
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



