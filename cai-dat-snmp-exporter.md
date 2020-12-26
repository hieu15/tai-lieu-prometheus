## Cài đặt snmp-exporter
### I/ Cài đặt GoLang
```bash
wget https://golang.org/dl/go1.15.linux-amd64.tar.gz
tar -xvzf go1.15.linux-amd64.tar.gz
mv go /usr/local/
export GOROOT=/usr/local/go
export PATH=$GOROOT/bin:$PATH
```
Kiểm tra lại GoLang version
```bash
go version
go env
```
Mặc định GoLang chỉ tồn tại cho 1 shell, nếu muốn shell nào cũng chạy được phải thêm đoạn export GoLang vào **.bashrc**
```bash
export GOROOT=/usr/local/go
export PATH=$GOROOT/bin:$PATH
```
### II/ Cài đặt snmp-exporter
Link tham khảo : [https://github.com/prometheus/snmp_exporter](https://github.com/prometheus/snmp_exporter)
Cài các package cần thiết
```bash
sudo yum install git zip unzip gcc gcc-g++ make net-snmp net-snmp-utils net-snmp-libs net-snmp-devel # RHEL-based distros
```
Build Generator
```bash
go get github.com/prometheus/snmp_exporter/generator
cd ${GOPATH-$HOME/go}/src/github.com/prometheus/snmp_exporter/generator
go build
make mibs
```
Tạo file generator.yml
```bash
vi generator.yml
```
```yaml
modules:
########## Fortigate
  fortigate_snmp:
   walk:
   - ifXTable
   - fgVpn
   - fgSystem
   - fgIntf
   version: 2
   max_repetitions: 25
   retries: 3
   timeout: 10s
   auth:
    community: snmpmonitor
########### Cisco 
  cisco:
#   walk: [sysUpTime, interfaces, ifXTable]
   walk:
   - 1.3.6.1.2.1.2.2.1.1
   - 1.3.6.1.2.1.2.2.1.2
   - 1.3.6.1.2.1.2.2.1.10
   - 1.3.6.1.2.1.2.2.1.13
   - 1.3.6.1.2.1.2.2.1.14
   - 1.3.6.1.2.1.2.2.1.16
   - 1.3.6.1.2.1.2.2.1.19
   - 1.3.6.1.2.1.2.2.1.2
   - 1.3.6.1.2.1.2.2.1.20
   - 1.3.6.1.2.1.2.2.1.5
   - 1.3.6.1.2.1.2.2.1.7
   - 1.3.6.1.2.1.2.2.1.8
   - 1.3.6.1.2.1.31.1.1.1.1
   - 1.3.6.1.2.1.31.1.1.1.18
   - 1.3.6.1.4.1.9.9.48.1.1.1.5
   - 1.3.6.1.4.1.9.9.48.1.1.1.6
   - 1.3.6.1.4.1.9.2.1
   - 1.3.6.1.2.1.1.5
   lookups:
     - source_indexes: [ifIndex]
       lookup: ifAlias
     - source_indexes: [ifIndex]
       lookup: ifDescr
     - source_indexes: [ifIndex]
       # Use OID to avoid conflict with Netscaler NS-ROOT-MIB.
       lookup: 1.3.6.1.2.1.31.1.1.1.1 # ifName
   overrides:
     ifAlias:
       ignore: true # Lookup metric
     ifDescr:
       ignore: true # Lookup metric
     ifName:
       ignore: true # Lookup metric
     ifType:
       type: EnumAsInfo
   version: 2
   max_repetitions: 25
   retries: 3
   timeout: 10s
   auth:
    community: snmpmonitor
```
Download mib bỏ vào thư mục generator

<mark>**Phải bỏ tất cả các file MIB vào thư mục mibs. Nếu trong thư mục mibs có tồn tại thư mục khác ví dụ generator/cisco_v2 thì khi generate ra snmp.yml sẽ bị lổi vì không đọc được OID bên trong.**</mark>

Link mib cisco : [ftp://ftp.cisco.com](ftp://ftp.cisco.com/)

Đối với fortigate download tại system/snmp

<mark>Đối với các thiết bị mạng cần phải enable snmp.</mark>
Export đường dẫn MIB và generate file snmp.yml
```bash
export MIBDIRS=mibs
./generator generate
```
Cài đặt snmp_exporter. [Download tại đây](https://github.com/prometheus/snmp_exporter/releases)

```bash
wget https://github.com/prometheus/snmp_exporter/releases/download/v0.18.0/snmp_exporter-0.18.0.linux-amd64.tar.gz
tar -xvzf snmp_exporter*
mv snmp_exporter-0.18.0.linux-amd64 /usr/local/snmp_exporter
```

Tạo user để chạy service snmp_exporter
```bash
sudo useradd --no-create-home --shell /bin/false snmp_exporter
```
Phân quyền cho user vừa tạo là owner của thư mục source snmp_exporter
```bash
chown -R snmp_exporter:snmp_exporter /usr/local/snmp_exporter
```
```bash
vi /etc/systemd/system/snmp_exporter.service
```
```bash
[Unit]
Description=Snmp_exporter
Wants=network-online.target
After=network-online.target

[Service]
User=root
Group=root
Type=simple
ExecStart=/usr/local/snmp_exporter/snmp_exporter \
--config.file=/usr/local/snmp_exporter/snmp.yml

[Install]
WantedBy=multi-user.target
```
Copy file snmp.yml mà chúng ta đã generate vào thư mục của snmp_exporter
```bash
systemctl restart snmp_exporter.service
systemctl status snmp_exporter.service
systemctl enable snmp_exporter.service
```
### III/ Tạo job trong prometheuse
```bash
vi /usr/local/prometheus.yml
```

```yaml
################################ FORTIGATE
  - job_name: 'fortigate'
    static_configs:
      - targets:
        - 172.16.100.1 # fortigate device.
        labels:                           
         hostname: GROUP-FG
         device: fortigate
         company: LDG
    scrape_interval: 3m
    scrape_timeout : 3m
    metrics_path: /snmp
    params:
      module: [fortigate_snmp]
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 10.10.10.26:9116  # SNMP exporter.
################################ CISCO
  - job_name: 'cisco'
    static_configs:
      - targets: ['10.10.10.101']
         # Access switch.
        labels:                           
         hostname: A-SW1
         device: cisco
         company: DG
         
    scrape_interval: 3m
    scrape_timeout : 3m
    metrics_path: /snmp
    params:
      module: [cisco]
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 10.10.10.26:9116  # SNMP exporter.
```

