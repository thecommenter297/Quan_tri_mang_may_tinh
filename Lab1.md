### **Bài 1: TRIỂN KHAI VÀ CÀI ĐẶT HỆ ĐIỀU HÀNH WINDOWS SERVER 2019**

**Các nội dung chính:**
*   Cài đặt hệ điều hành Windows Server 2019 Datacenter (Desktop Experience).
*   Cài đặt hệ điều hành Windows Server 2019 Datacenter (Server Core) (Tham khảo).
*   Cấu hình NIC Teaming trên Windows Server 2019.

---

### **1.1 Cài đặt Windows Server 2019 Datacenter (Desktop Experience)**

**1. Mục tiêu**
Cài đặt và cấu hình cơ bản cho một máy chủ Windows Server 2019 phiên bản có giao diện đồ họa (Desktop Experience) và một máy trạm Windows 10 trong môi trường máy ảo.

**2. Chuẩn bị**
*   **Máy chủ (Server):**
    *   Hệ điều hành: Windows Server 2019 Datacenter
    *   Tên máy: `DC-2019`
    *   IP Address: `192.168.1.4`
    *   Subnet Mask: `255.255.255.0` (hoặc Prefix Length: `24`)
    *   Default Gateway: `192.168.1.1`
    *   Preferred DNS Server: `192.168.1.2`
*   **Máy trạm (Client):**
    *   Hệ điều hành: Windows 10 x64
    *   Tên máy: `W10-CLIENT01`
    *   IP Address: `192.168.1.10`
    *   Subnet Mask: `255.255.255.0`
    *   Default Gateway: `192.168.1.1`
    *   Preferred DNS Server: `192.168.1.2`
*   **Yêu cầu cấu hình tối thiểu cho máy ảo:**
    *   Kiến trúc: x86-64
    *   Tốc độ xử lý: 1.4 GHz
    *   RAM: 4 GB
    *   Dung lượng ổ cứng: 32 GB
*   **Phần mềm:**
    *   VMware Workstation (phiên bản 15 trở lên).
    *   File ISO cài đặt Windows Server 2019.
    *   File ISO cài đặt Windows 10 x64.

**3. Mô hình Lab**
(Mô hình mạng tương tự như file gốc, chỉ thay đổi tên máy và phiên bản hệ điều hành).
*   Máy chủ `DC-2019` (192.168.1.4) kết nối vào một switch ảo.
*   Máy trạm `W10-CLIENT01` (192.168.1.10) kết nối vào cùng switch ảo đó.

**4. Hướng dẫn chi tiết**

#### **Phần A: Tạo và cài đặt máy ảo Windows Server 2019**

1.  Mở VMware Workstation, chọn **File** -> **New Virtual Machine** (hoặc Ctrl + N).
2.  Trong cửa sổ New Virtual Machine Wizard, chọn **Custom (advanced)** và nhấn **Next**.
3.  Để nguyên Hardware compatibility mặc định, nhấn **Next**.
4.  Tại màn hình **Guest Operating System Installation**, chọn **Installer disc image file (iso)**, nhấn **Browse...** và trỏ đến file ISO Windows Server 2019 của bạn. Nhấn **Next**.
5.  Tại màn hình **Select a Guest Operating System**, chọn **Microsoft Windows** và Version là **Windows Server 2019**. Nhấn **Next**.
6.  Đặt tên máy ảo (ví dụ: `DC-2019`) và chọn vị trí lưu trữ. Nhấn **Next**.
7.  Tại **Firmware Type**, chọn **BIOS**. Nhấn **Next**.
8.  Cấu hình bộ xử lý (Processor Configuration), để mặc định (1 processor, 1 core) hoặc tăng lên tùy theo cấu hình máy thật. Nhấn **Next**.
9.  Cấu hình bộ nhớ (Memory), chọn **4 GB (4096 MB)**. Nhấn **Next**.
10. Tại **Network Type**, chọn **Use host-only networking** để tạo một mạng riêng ảo giữa máy thật và các máy ảo, thuận tiện cho việc thực hành. Nhấn **Next**.
11. Các bước tiếp theo (I/O Controller, Disk Type, Select a Disk), giữ nguyên các tùy chọn được đề xuất (Recommended) và nhấn **Next**.
12. Tại **Specify Disk Capacity**, đặt dung lượng ổ cứng là **60 GB** và chọn **Store virtual disk as a single file**. Nhấn **Next**.
13. Nhấn **Next** để xác nhận tên file disk.
14. Tại cửa sổ cuối cùng, xem lại cấu hình và nhấn **Finish**. Máy ảo sẽ tự động khởi động và boot vào bộ cài Windows.

#### **Phần B: Các bước cài đặt hệ điều hành**

1.  Tại màn hình **Windows Setup** đầu tiên, giữ nguyên ngôn ngữ và nhấn **Next**.
2.  Nhấn **Install now**.
3.  Tại màn hình **Select the operating system you want to install**, chọn **Windows Server 2019 Datacenter (Desktop Experience)**. Đây là phiên bản có giao diện đồ họa đầy đủ. Nhấn **Next**.
    *   *Lưu ý: Phiên bản không có chữ "(Desktop Experience)" là phiên bản Server Core, không có giao diện đồ họa.*
4.  Đánh dấu vào ô **I accept the license terms** và nhấn **Next**.
5.  Chọn **Custom: Install Windows only (advanced)**.
6.  Chọn ổ đĩa chưa được phân vùng (Drive 0 Unallocated Space) và nhấn **Next** để bắt đầu quá trình cài đặt.
7.  Sau khi cài đặt hoàn tất và máy khởi động lại, màn hình **Settings** sẽ hiện ra. Nhập mật khẩu cho tài khoản Administrator (ví dụ: `P@ssw0rd123`) và nhấn **Finish**.
8.  Tại màn hình đăng nhập, nhấn **Ctrl + Alt + Delete** (trên VMware, nhấn **Ctrl + Alt + Insert**) và đăng nhập bằng mật khẩu vừa tạo. **Server Manager** sẽ tự động khởi chạy.

#### **Phần C: Cấu hình cơ bản sau cài đặt (Đổi tên máy và đặt IP tĩnh)**

**Cách 1: Sử dụng giao diện đồ họa (GUI)**

1.  **Đổi tên máy:**
    *   Trong **Server Manager**, chọn **Local Server**.
    *   Nhấn vào tên máy tính hiện tại (một chuỗi ký tự ngẫu nhiên) trong mục **Computer name**.
    *   Trong cửa sổ **System Properties**, nhấn nút **Change...**.
    *   Nhập `DC-2019` vào ô **Computer name** và nhấn **OK**.
    *   Chọn **Restart Now** để khởi động lại máy chủ.
2.  **Đặt địa chỉ IP tĩnh:**
    *   Sau khi máy khởi động lại, đăng nhập và mở lại **Server Manager** -> **Local Server**.
    *   Nhấn vào dòng chữ **IPv4 address assigned by DHCP...** cạnh mục **Ethernet0**.
    *   Cửa sổ **Network Connections** sẽ mở ra. Chuột phải vào card mạng (**Ethernet0**) và chọn **Properties**.
    *   Chọn **Internet Protocol Version 4 (TCP/IPv4)** và nhấn **Properties**.
    *   Chọn **Use the following IP address** và điền các thông số:
        *   IP address: `192.168.1.4`
        *   Subnet mask: `255.255.255.0`
        *   Default gateway: `192.168.1.1`
    *   Chọn **Use the following DNS server addresses** và điền:
        *   Preferred DNS server: `192.168.1.2`
    *   Nhấn **OK** hai lần để đóng các cửa sổ.

**Cách 2: Sử dụng Windows PowerShell**

Mở PowerShell với quyền quản trị (chuột phải vào nút Start -> Windows PowerShell (Admin)).

1.  **Đổi tên máy:**
    ```powershell
    Rename-Computer -NewName "DC-2019" -Restart
    ```
    Máy sẽ tự động khởi động lại sau khi thực thi lệnh.

2.  **Đặt địa chỉ IP tĩnh:**
    Sau khi khởi động lại, mở lại PowerShell (Admin).
    *   **Bước 2.1: Xem thông tin card mạng để lấy InterfaceIndex.**
        ```powershell
        Get-NetAdapter
        ```
        Ghi lại số `ifIndex` (ví dụ: 12) của card mạng có tên là `Ethernet0`.
    *   **Bước 2.2: Xóa các cấu hình IP cũ (nếu có).**
        ```powershell
        Remove-NetIPAddress -InterfaceIndex <ifIndex_number> -Confirm:$false
        Remove-NetRoute -InterfaceIndex <ifIndex_number> -Confirm:$false
        ```
        *Thay `<ifIndex_number>` bằng `ifIndex` bạn đã ghi lại.*
    *   **Bước 2.3: Thiết lập IP và Default Gateway.**
        ```powershell
        New-NetIPAddress -InterfaceIndex <ifIndex_number> -IPAddress "192.168.1.4" -PrefixLength 24 -DefaultGateway "192.168.1.1"
        ```
        *`PrefixLength 24` tương đương với Subnet Mask `255.255.255.0`.*
    *   **Bước 2.4: Thiết lập DNS Server.**
        ```powershell
        Set-DnsClientServerAddress -InterfaceIndex <ifIndex_number> -ServerAddresses "192.168.1.2"
        ```
*Lưu ý: Lặp lại các bước trên để cài đặt và cấu hình máy trạm Windows 10 với thông tin tương ứng.*

---

### **1.2 Cài đặt Windows Server 2019 (Server Core) (Tham khảo)**

**1. Mục tiêu**
Cài đặt và cấu hình cơ bản cho Windows Server 2019 phiên bản không có giao diện đồ họa, sử dụng hoàn toàn dòng lệnh.

**2. Hướng dẫn**

*   Thực hiện các bước tạo máy ảo và cài đặt tương tự **Phần 1.1**, nhưng ở bước chọn hệ điều hành, hãy chọn **Windows Server 2019 Datacenter** (phiên bản không có chữ "Desktop Experience").
*   Sau khi cài đặt xong, hệ thống sẽ khởi động vào màn hình dòng lệnh (Command Prompt hoặc PowerShell).

**3. Cấu hình cơ bản trên Server Core**
Sau khi đăng nhập, bạn có thể dùng công cụ `sconfig` hoặc các lệnh PowerShell để cấu hình.

**Cách 1: Sử dụng Sconfig**

1.  Trong cửa sổ dòng lệnh, gõ `sconfig` và nhấn Enter.
2.  Một giao diện menu sẽ hiện ra.
3.  Chọn tùy chọn **2) Computer Name** để đổi tên máy.
4.  Chọn tùy chọn **8) Network Settings** để cấu hình địa chỉ IP. Làm theo các hướng dẫn trên màn hình để chọn card mạng, đặt IP, Subnet Mask, Default Gateway và DNS.
5.  Chọn **4) Configure Remote Management** và chọn **Enable** để cho phép quản trị từ xa.

**Cách 2: Sử dụng Windows PowerShell**

(Các lệnh hoàn toàn tương tự như khi cấu hình trên phiên bản Desktop Experience).

1.  **Đổi tên máy:**
    ```powershell
    Rename-Computer -NewName "CORE-2019" -Restart
    ```
2.  **Đặt IP tĩnh (Giả sử ifIndex là 12):**
    ```powershell
    # Xem thông tin card mạng
    Get-NetAdapter
    
    # Thiết lập IP và Gateway
    New-NetIPAddress -InterfaceIndex 12 -IPAddress "192.168.1.9" -PrefixLength 24 -DefaultGateway "192.168.1.1"

    # Thiết lập DNS
    Set-DnsClientServerAddress -InterfaceIndex 12 -ServerAddresses "192.168.1.9"
    ```

---

### **1.3 Cấu hình NIC Teaming trên Windows Server 2019**

**1. Mục tiêu**
Gộp hai hoặc nhiều card mạng vật lý thành một card mạng luận lý duy nhất để tăng băng thông và khả năng chịu lỗi.

**2. Chuẩn bị**
*   Máy chủ `DC-2019` đã cài đặt ở phần 1.
*   Thêm một card mạng thứ hai vào máy ảo `DC-2019`.
    *   Tắt máy ảo `DC-2019`.
    *   Chuột phải vào máy ảo, chọn **Settings**.
    *   Nhấn nút **Add...**, chọn **Network Adapter**, nhấn **Finish**.
    *   Chọn card mạng mới và cấu hình mạng là **Host-only**.
    *   Khởi động lại máy ảo.
*   Thông tin IP sẽ được đặt cho card NIC Teaming sau khi tạo:
    *   IP Address: `192.168.1.100`
    *   Subnet Mask: `255.255.255.0`
    *   Default Gateway: `192.168.1.1`
    *   DNS Server: `192.168.1.2`

**3. Hướng dẫn chi tiết**

**Cách 1: Sử dụng giao diện đồ họa (GUI)**

1.  Đăng nhập vào máy chủ `DC-2019`.
2.  Mở **Server Manager**, chọn **Local Server**.
3.  Trong phần **Properties**, tìm mục **NIC Teaming** và nhấn vào chữ **Disabled**.
4.  Trong cửa sổ **NIC Teaming**, tại mục **TEAMS**, nhấn vào **TASKS** và chọn **New Team**.
5.  Cửa sổ **New team** hiện ra:
    *   **Team name**: Đặt tên cho team, ví dụ: `NetworkTeam`.
    *   **Member adapters**: Tích chọn hai card mạng có sẵn (`Ethernet0` và `Ethernet1`).
    *   Mở rộng **Additional properties**, giữ nguyên các thiết lập mặc định (Teaming mode: Switch Independent, Load balancing mode: Dynamic).
    *   Nhấn **OK**.
6.  Chờ một vài giây để team được tạo và có trạng thái **Online**.
7.  Bây giờ, quay lại cửa sổ **Network Connections** (`ncpa.cpl`). Bạn sẽ thấy một card mạng mới có tên là `NetworkTeam`.
8.  Chuột phải vào card `NetworkTeam`, chọn **Properties** -> **Internet Protocol Version 4 (TCP/IPv4)** -> **Properties** và cấu hình IP tĩnh như trong phần yêu cầu chuẩn bị (`192.168.1.100`).
9.  Từ máy trạm Windows 10, mở Command Prompt và thực hiện lệnh `ping 192.168.1.100` để kiểm tra kết nối.

**Cách 2: Sử dụng Windows PowerShell**

Mở PowerShell với quyền quản trị.

1.  **Xem tên các card mạng có sẵn:**
    ```powershell
    Get-NetAdapter
    ```
    Ghi lại tên của 2 card mạng, ví dụ `Ethernet0` và `Ethernet1`.

2.  **Tạo NIC Team:**
    ```powershell
    New-NetLbfoTeam -Name "NetworkTeam" -TeamMembers "Ethernet0", "Ethernet1"
    ```
    *Thay "Ethernet0", "Ethernet1" bằng tên chính xác của các card mạng trên máy của bạn.*

3.  **Cấu hình IP cho NIC Team vừa tạo:**
    *   **Bước 3.1: Lấy InterfaceIndex của team mới.**
        ```powershell
        Get-NetAdapter
        ```
        Tìm adapter có tên `NetworkTeam` và ghi lại số `ifIndex`. Giả sử là `15`.
    *   **Bước 3.2: Thiết lập IP và Gateway.**
        ```powershell
        New-NetIPAddress -InterfaceIndex 15 -IPAddress "192.168.1.100" -PrefixLength 24 -DefaultGateway "192.168.1.1"
        ```
    *   **Bước 3.3: Thiết lập DNS.**
        ```powershell
        Set-DnsClientServerAddress -InterfaceIndex 15 -ServerAddresses "192.168.1.2"
        ```
4.  Kiểm tra lại bằng lệnh `ping` từ máy trạm Windows 10.
