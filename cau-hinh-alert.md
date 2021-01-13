### Cấu hình Alert trong prometheus
Bước 01: Download alert_manager về
```bash
wget https://github.com/prometheus/alertmanager/releases/download/v0.21.0/alertmanager-0.21.0.linux-amd64.tar.gz
```
Bước 02: Giải nén và copy vào thư mục /usr/local để dễ dàng quản lý
```bash
```bash
tar -xvzf alertmanager-0.21.0.linux-amd64.tar.gz
mv alertmanager-0.21.0.linux-amd64 /usr/local/alertmanager
```
Bước 03: Tạo user để chạy service alert_manager
```bash
sudo useradd --no-create-home --shell /bin/false alert_manager
```
Phân quyền cho user vừa tạo là owner của thư mục source alert_manager
```bash
chown -R alert_manager:alert_manager /usr/local/alertmanager
```
Tạo service trong systemd cho alertmanager
```bash
vi /etc/systemd/system/alertmanager.service
```
Nội dung file service như sau:
```bash
[Unit]
Description=AlertManager
Wants=network-online.target
After=network-online.target

[Service]
User=alert_manager
Group=alert_manager
Type=simple
ExecStart=/usr/local/alertmanager/alertmanager --config.file=/usr/local/alertmanager/alertmanager.yml -cluster.advertise-address=0.0.0.0:9093

[Install]
WantedBy=multi-user.target
```
=> Nếu đặt Alert manager là IP public phải thêm vào option --cluster.advertise-address=0.0.0.0:9093
Enable và start service 
```bash
systemctl daemon-reload
systemctl enable alertmanager
systemctl restart alertmanager
```
Bước 04: Cấu hình alert trong prometheus
```bash
vi /usr/local/prometheus/prometheus.yml
```
Tại targets: nhập thông tin ip, port , tại đây chính là ip của prometheus server và port 9093 (alertmanager dùng port này).
 
Bước 05: Tạo các rule alert trong prometheus. Để tạo alert trong prometheus, bạn cần phải định nghĩa ra các rule. Tạo các file rule trong thư mục promethues với format sau: rulename.yml
 
Nội dung file rule tùy các bạn định nghĩa. 
Tại đây mình tạo 1 rule alert cho windows host:  windows-rule.yml
```bash
vi /usr/local/prometheus/windows-rules.yml
```

Nội dung file rule như sau:
```go
############# Define Rule Alert ###############
# my global config
############# Define Rule Alert ###############
groups:
- name: Windows-alert
  rules:

################ Memory Usage High
  - alert: Memory Usage High
    expr: 100*(wmi_os_physical_memory_free_bytes) / wmi_cs_physical_memory_bytes > 90
    for: 1m
    labels:
      severity: warning
    annotations:
      summary: "Memory Usage (instance {{ $labels.instance }})"
      description: "Memory Usage is more than 90%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

################ CPU Usage High
  - alert: Cpu Usage High
    expr: 100 - (avg by (instance) (irate(wmi_cpu_time_total{mode="idle"}[2m])) * 100) > 80
    for: 1m
    labels:
      severity: warning
    annotations:
      summary: "CPU Usage (instance {{ $labels.instance }})"
      description: "CPU Usage is more than 80%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

################ Disk Usage
  - alert: DiskSpaceUsage
    expr: 100.0 - 100 * ((wmi_logical_disk_free_bytes{} / 1024 / 1024 ) / (wmi_logical_disk_size_bytes{}  / 1024 / 1024)) > 95
    for: 1m
    labels:
      severity: error
    annotations:
      summary: "Disk Space Usage (instance {{ $labels.instance }})"
      description: "Disk Space on Drive is used more than 95%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

################ ServiceStatus
  - alert: ServiceStatus
    expr: wmi_service_status{status="ok"} != 1
    for: 1m
    labels:
      severity: error
    annotations:
      summary: "Service Status (instance {{ $labels.instance }})"
      description: "Windows Service state is not OK\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

################ CollectorError
  - alert: CollectorError
    expr: wmi_exporter_collector_success == 0
    for: 1m
    labels:
      severity: error
    annotations:
      summary: "Collector Error (instance {{ $labels.instance }})"
      description: "Collector {{ $labels.collector }} was not successful\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```
Bước 06: Tiếp tục chúng ta sẽ khai báo tên của rule đó vào cấu hình của prometheus theo như hình bên dưới.
```go
vi /usr/local/prometheus/prometheus.yml
```
Bước 07: Kiểm tra lại rule đã tạo bằng lệnh sau
```go
./promtool check config  prometheus.yml
```
Bước 08: Restart prometheus service và kiểm tra lại kết quả. Tương tự bạn sẽ tạo nhiều rule cho nhiều  nhóm thiết bị khác nhau.

Bước 09: Ở bước này mình sẽ tạo 1 con telegram_bot sau đó gửi alert qua telegram. Download source code và build file binary
## TELEGRAM BOT 01
```bash
cd ${GOPATH-$HOME/go}/src/github.com
go get github.com/inCaller/prometheus_bot
cd ${GOPATH-$HOME/go}/src/github.com/inCaller/prometheus_bot
make clean
make
```
Copy bỏ vào thư mục /usr/local để dễ dàng quản lý.
```bash
mv /root/go/src/github.com/inCaller/prometheus_bot /usr/local
```
Đăng nhập vào telegram để tạo telegram bot và group nhận alert. 
Cách tạo telegram bot tham khảo tại đây
[https://www.teleme.io/articles/create_your_own_telegram_bot?hl=vi](https://www.teleme.io/articles/create_your_own_telegram_bot?hl=vi)

Khi tạo telegram bot mình sẽ ghi lại token của con bot 

Tiếp tục tạo group sau đó add bot vừa tạo vào, sử dụng telegram qua web bạn sẽ lấy dc tham số -chat-id

Chat-id chính là : g364942581
 

```bash
vi /usr/local/prometheus_bot/config.yaml
```
```bash
Nội dung file config.yaml như sau:
```bash
telegram_token: "997872129:AAEPKYz3nPwmFsgq6ao-MdPsC5fy5z376GQ"
#### ONLY IF YOU USING TEMPLATE required for test

template_path: "template.tmpl"
time_zone: "Asia/Ho_Chi_Minh"
split_token: "|"

##### ONLY IF YOU USING DATA FORMATTING FUNCTION, NOTE for developer: important or test fail
time_outdata: "02/01/2006 15:04:05"
split_msg_byte: 4000
```
Tiếp tục cấu hình alert manager
```bash
vi /usr/local/alertmanager/alertmanager.yml
```
thay đổi tham số url như sau: http://ipalertmanager:9087/alert/-chat_id
 
Thực hiện test với cú pháp sau:
```bash
export TELEGRAM_CHATID="-364942581"
make test
```
Kiểm tra thấy bot đã gửi được tin nhắn

## TELEGRAM BOT 02
```bash
git clone https://github.com/metalmatze/alertmanager-bot.git
cd alertmanager-bot
make
cp -R alertmanager-bot /usr/local
```
```bash
sudo useradd --no-create-home --shell /bin/false alertmanager-bot
chown -R alertmanager-bot:alertmanager-bot /usr/local/alertmanager-bot
```
```bash
vi /etc/systemd/system/alertmanager-bot
```

```bash
[Unit]
Description=Alert Manager Bot
Wants=network-online.target
After=network-online.target

[Service]
User=alertmanager-bot
Group=alertmanager-bot
Type=simple
ExecStart=/usr/local/alertmanager-bot/alertmanager-bot --store=bolt --alertmanager.url=http://10.10.10.26:9093 --bolt.path=/usr/local/alertmanager-bot/bot.db --listen.addr=0.0.0.0:8080 --telegram.admin=494696973 --telegram.token=997872129:AAEPKYz3nPwmFsgq6ao-MdPsC5fy5z376GQ --template.paths=/usr/local/alertmanager-bot/default.tmpl --log.level=debug

[Install]
WantedBy=multi-user.targe
```

```bash
sudo systemctl daemon-reload
sudo systemctl start alertmanager-bot
sudo systemctl status alertmanager-bot
sudo systemctl enable alertmanager-bot
```
