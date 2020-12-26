## Cài đặt node-exporter
Để giám sát các ESXi host hoặc Vcenter chúng ta cần cài vmware_exporter trên prometheus server.
[https://github.com/pryorda/vmware_exporter](https://github.com/pryorda/vmware_exporter)

Requires Python >= 3.6
Bước 01:  Cài đặt Python 3.6
```bash
wget https://www.python.org/ftp/python/3.6.4/Python-3.6.4.tar.xz
tar -xJf Python-3.6.4.tar.xz
cd Python-3.6.4
./configure
make
make install
pip3 install --upgrade pip
```
Bước 02: Cài đặt vmware_exporter
```bash
pip3 install vmware_exporter
```
Sau khi cài đặt xong, đường dẫn lưu trử tại đây
```bash
/usr/local/lib/python3.6/site-packages/vmware_exporter
```
Ngoài ra có thể tìm bằng comment sau:
```bash
find / -name "vmware_exporter*"
```
Bước 03: Tạo user read-only dùng để monitor trên vcenter hoặc ESXi host.
Bước 04: Tạo file cấu hình cho vmware_exporter 
```bash
cd /usr/local/lib/python3.6/site-packages/vmware_exporter 
vi config.yml
```
Nội dung file config như sau:
```json
default:
    vsphere_host: 10.10.10.43
    vsphere_user: 'monitor'
    vsphere_password: 'password
    ignore_ssl: True
    specs_size: 5000
    collect_only:
        vms: True
        vmguests: False
        datastores: True
        hosts: True
        snapshots: True
esxi02:
    vsphere_host: 10.10.10.41
    vsphere_user: 'monitor'
    vsphere_password: 'password'
    ignore_ssl: True
    specs_size: 5000
    collect_only:
        vms: True
        vmguests: False
        datastores: True
        hosts: True
        snapshots: True
```
Bước 05: Tạo service trong systemd cho vmware_exporter 
Tạo user : 
```bash
sudo useradd --no-create-home --shell /bin/false vmware_exporter
chown -R vmware_exporter /usr/local/bin/vmware_exporter
```
```bash
vi /etc/systemd/system/vmware_exporter.service
```

Nội dung file như sau: 
```bash
Unit]
Description=Prometheus VMWare Exporter
After=network.target

[Service]
User=vmware_exporter
Group=vmware_exporter
ExecStart=/usr/bin/python3 /usr/local/bin/vmware_exporter -c /usr/local/lib/python3.6/site-packages/vmware_exporter/config.yml
Type=simple

[Install]
WantedBy=multi-user.target
```
Bước 06: Tạo job trong prometheus

```bash
- job_name: 'ESXI03'
  metrics_path: '/metrics'
  static_configs:
    - targets: 
      - 10.10.10.43
      labels:
       hostname: ESXI03
       device: VMWARE
       company: XXX
  params:
    section: [default]
  relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: instance
    - target_label: __address__
      replacement: 10.10.10.26:9272

- job_name: 'ESXI02'
  metrics_path: '/metrics'
  static_configs:
    - targets:
      - 10.10.10.41
      labels:
       hostname: ESXI02
       device: VMWARE
       company: XXX
  params:
    section: [esxi02]
  relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: instance
    - target_label: __address__
      replacement: 10.10.10.26:9272
```




