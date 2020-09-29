## Cài đặt node-exporter
Để giảm sát được các server Linux, chúng ta cần phải cài đặt node-exporter (agent trên Linux) để Promethues có thể thu thập được metric từ các server Linux này.
### I/ Cài đặt node_exporter
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


