I / Update system, cấu hình sync NTP Việt Nam và disable selinux trên CentOS
```
yum install ntpdate -y
ntpdate 1.ro.pool.ntp.org
vim /etc/sysconfig/selinux
```
Change “SELINUX=enforcing” to “SELINUX=disabled”.
Cài đặt iptables trên CentOS7
``````
systemctl stop firewalld
systemctl disable firewalld
systemctl mask --now firewalld
yum install iptables-services -y
systemctl start iptables
systemctl enable iptables
``````
Mở port trên iptables bằng cách thêm vào file config của iptables với nội dung sau
`
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 3000 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 9090 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 9116 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 9100 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 9182 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 9093 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 9087 -j ACCEPT
`
II/ Cài đặt Prometheus
Download source prometheus từ website
wget https://github.com/prometheus/prometheus/releases/download/v2.20.1/prometheus-2.20.1.linux-amd64.tar.gz
tar -xvzf prometheus-2.20.1.linux-amd64.tar.gz
mv prometheus-2.20.1.linux-amd64 /usr/local/prometheus
Tạo folder lưu trử cho data của prometheus
mkdir /usr/local/prometheus/data-prometheus
Tạo user để chạy service prometheus
sudo useradd --no-create-home --shell /bin/false prometheus
Phân quyền cho user vừa tạo là owner  của thư mục source prometheus
chown -R prometheus:prometheus /usr/local/prometheus/
Tạo service prometheus trong systemd
vi /etc/systemd/system/prometheus.service
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/prometheus/prometheus \
--config.file /usr/local/prometheus/prometheus.yml \
--storage.tsdb.path /usr/local/prometheus/data-prometheus \
--web.console.templates=/usr/local/prometheus/consoles \
--web.console.libraries=/usr/local/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
