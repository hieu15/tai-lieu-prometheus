## Cài đặt windows-exporter
Để giảm sát được các server Windows, chúng ta cần phải cài đặt Wmi Exporter (Agent collectors) để Promethues có thể thu thập được metric từ các server Windows này.
### I/ Cài đặt windows_exporter
Links tham khảo : [https://github.com/prometheus-community/windows_exporter/releases](https://github.com/prometheus-community/windows_exporter/releases)
Bước 1: 
Download **windows_exporter**
Có 2 phiên bản Agent:
**wmi_exporter.exe** (click to run, dành cho các bạn nào chỉ cần các metric được enable sẵn).
**wmi_exporter.msi** (dùng để cài đặt thông qua CMD, enable các tính năng thu thập metric nâng cao).

Nếu các bạn chỉ đơn thuần download các agent này cài đặt vào máy chủ Windows thì bạn chỉ có thể thu thập metric ở mức cơ bản, như hình ở bên trên. Dưới dây là hướng dẫn bạn thu thập metric nâng cao cho service Windows. Ví dụ như: AD, DNS, SQL, IIS, …
Khi cài agent qua CMD sẽ giúp các bạn chọn lọc thu thập những loại metric nào cần thu thập, giảm các loại metric không cần thiết, có thể thu thập nhiều loại metric hơn mặc định.

Mở port 9182 trên server linux, và windows server.
Bước 2: Mở CMD với quyền Administrator.
Bước 3: Chạy comment msiexec với cú pháp như sau:
```bash
msiexec /i \\10.10.10.22\Softwares\Server_softwares\prometheus\windows_exporter-0.14.0-amd64.msi ENABLED_COLLECTORS="ad,dns,cpu,cs,logon,memory,logical_disk,os,service,system,process,tcp,net,textfile,thermalzone"
```
Trong đó C:\wmi_exporter-0.9.0-amd64.msi là đường dẫn chứa file wmi_exporter.msi.
ENABLED_COLLECTORS=”các loại metric cần thu thập, tên của loại metric trong hình bên trên”
Tùy theo từng loại windows server mà lựa chọn metric cho phù hợp.

