## Cài đặt node-exporter
Để giảm sát được các server Linux, chúng ta cần phải cài đặt node-exporter (agent trên Linux) để Promethues có thể thu thập được metric từ các server Linux này.
### I/ Cài đặt node_exporter trên CentOS, Ubuntu
Links tham khảo : https://prometheus.io/download/#node_exporter [https://prometheus.io/download/#node_exporter](https://prometheus.io/download/#node_exporter)

Nhớ mở port 9100
Bước 1: 
Download **node_exporter**
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.0.1/node_exporter-1.0.1.linux-amd64.tar.gz
```
Bước 02: Giải nén source code và copy đến đường dẫn sau /usr/local/node_exporter
```bash
tar -xvzf node_exporter-1.0.1.linux-amd64.tar.gz
mkdir /usr/local/node_exporter
mv node_exporter-1.0.1.linux-amd64 /usr/local/node_exporter
```
Bước 03: Tạo user dùng để run service
```bash
sudo useradd --no-create-home --shell /bin/false node_exporter
```
Phân quyền cho user vừa tạo là owner của thư mục source snmp_exporter
```bash
chown -R node_exporter:node_exporter /usr/local/node_exporter
```
Bước 03: Tạo service trong systemd cho node_exporter
```bash
vim /etc/systemd/system/node_exporter.service
```

Nội dung trong file như sau:
```bash
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
ExecStart=/usr/local/node_exporter/node_exporter

[Install]
WantedBy=default.target
```
Bước 04: Restart và enable service
```bash
systemctl daemon-reload
systemctl start node_exporter
systemctl enable node_exporter
```

### I/ Cài đặt node_exporter trên FreeBSD  
Pfsense là firewall mã nguồn mở chạy trên nền tảng FreeBSD. Chính vì thế chúng ta sẽ cài đặt node_exporter trên Pfsense để có thể giám sát được nó.\
Bước 01: SSH vào  Pfsense, chọn 8 để vào shell\
Bước 02: Cài đặt node_exporter
```bash
pkg add http://pkg.freebsd.org/FreeBSD:11:amd64/release_2/All/node_exporter-0.15.2.txz
```
Hoặc edit thành "YES"
```bash
vi /usr/local/etc/pkg/repos/FreeBSD.conf
vi /usr/local/etc/pkg/repos/pfSense.conf               
vi /etc/pkg/FreeBSD.conf
```
```bash
pkg install node_exporter
```
Bước 03: Hiệu chỉnh các service trong node_exporter
```bash
vi /usr/local/etc/rc.d/node_exporter
```
Tìm đến các tham số sau và chỉnh lại như bên dưới:
```bash
: ${node_exporter_enable:="YES"}
: ${node_exporter_user:="root"}
: ${node_exporter_group:="root"}
```
Bước 04: Enable và start node_exporter 
```bash
/usr/local/etc/rc.d/node_exporter enabled
/usr/local/etc/rc.d/node_exporter start
```
Bước 05: Tạo job trong prometheus để giám sát Pfsense với nội dung job sau:
```json
- job_name: 'pfsense'
  static_configs:
  - targets: ['10.10.10.54:9100']
    labels:
     hostname: LDG-VPN-01
     type: pfsense
     company: LDG
```