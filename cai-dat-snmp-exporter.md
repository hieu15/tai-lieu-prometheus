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
II/ Cài đặt snmp-exporter
Link tham khảo : [https://github.com/prometheus/snmp_exporter](https://github.com/prometheus/snmp_exporter)
Cài các package cần thiết
```bash
sudo yum install git gcc gcc-g++ make net-snmp net-snmp-utils net-snmp-libs net-snmp-devel # RHEL-based distros
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
    community: ldgsnmpmonitor
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
    community: ldgsnmpmonitor
```
Download mib bỏ vào thư mục generator
Link mib cisco : [ftp://ftp.cisco.com/pub/mibs/v1/OLD-CISCO-SYS-MIB.my](ftp://ftp.cisco.com/pub/mibs/v1/OLD-CISCO-SYS-MIB.my)
Đối với fortigate download tại system/snmp

<mark>Đối với các thiết bị mạng cần phải enable snmp.</mark>



