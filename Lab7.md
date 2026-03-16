### **Bài 7: CÀI ĐẶT VÀ CẤU HÌNH DỊCH VỤ DHCP**

**Các nội dung chính:**
*   Cài đặt và cấu hình DHCP Server tích hợp với Active Directory.
*   Cấu hình Scope, Exclusion, và các tùy chọn (Options).
*   Thiết lập cấp phát IP cố định (Reservation).
*   Sao lưu và khôi phục dịch vụ DHCP.
*   Cài đặt và cấu hình DHCP Relay Agent.

---

### **6.1 Cài đặt và cấu hình DHCP Server tích hợp AD**

**1. Mục tiêu**
Cài đặt dịch vụ DHCP trên một máy chủ thành viên (Member Server) để tự động cấp phát địa chỉ IP và các thông tin cấu hình mạng cho các máy trạm trong miền.

**2. Yêu cầu chuẩn bị**
*   **Máy chủ Domain Controller:** `DC01-2019` (IP: `192.168.1.2`, quản lý miền `bkaptech.vn`).
*   **Máy chủ DHCP:** Một máy Windows Server 2019 mới, đặt tên `SRV01-MEMBER`, đã Join vào miền `bkaptech.vn` với IP tĩnh là `192.168.1.3`.
*   **Máy trạm:** `W10-CLIENT01` và `W10-CLIENT02`, đã Join vào miền. **Quan trọng:** Cấu hình card mạng của các máy này ở chế độ "Obtain an IP address automatically".

---

#### **Phần A: Cài đặt và Ủy quyền (Authorize) DHCP Server**

**Bước 1: Cài đặt DHCP Role trên `SRV01-MEMBER`**

**Cách 1: Sử dụng giao diện đồ họa (GUI)**

1.  Trên máy chủ `SRV01-MEMBER`, mở **Server Manager**.
2.  Chọn **Manage** -> **Add Roles and Features**.
3.  Nhấn **Next** cho đến màn hình **Server Roles**.
4.  Tích vào ô **DHCP Server**. Một cửa sổ pop-up sẽ hiện ra, nhấn **Add Features**.
5.  Nhấn **Next** liên tục và cuối cùng nhấn **Install**.

**Cách 2: Sử dụng Windows PowerShell**

Mở PowerShell với quyền quản trị (Admin) trên `SRV01-MEMBER` và chạy lệnh:
```powershell
Install-WindowsFeature -Name DHCP -IncludeManagementTools
```

**Bước 2: Hoàn tất cấu hình và Ủy quyền DHCP trong Active Directory**

Để DHCP Server được phép hoạt động trong môi trường Domain, nó phải được "ủy quyền".

1.  Sau khi cài đặt xong, trong **Server Manager**, nhấn vào biểu tượng lá cờ cảnh báo màu vàng và chọn **Complete DHCP configuration**.
2.  Trong cửa sổ wizard, nhấn **Next**.
3.  Chọn **Use the following user's credentials** và đảm bảo tài khoản là Administrator của domain (ví dụ: `BKAPTECH\Administrator`). Nhấn **Commit**.
4.  Nhấn **Close**. Server DHCP của bạn đã được ủy quyền.

---

#### **Phần B: Cấu hình Dải cấp phát (Scope)**

**Mục tiêu:** Tạo một Scope để định nghĩa dải IP sẽ được cấp cho máy trạm.

**Cách 1: Sử dụng giao diện đồ họa (GUI)**

1.  Trên `SRV01-MEMBER`, mở **Server Manager** -> **Tools** -> **DHCP**.
2.  Trong cửa sổ DHCP, mở rộng tên máy chủ, chuột phải vào **IPv4** và chọn **New Scope...**.
3.  **New Scope Wizard** sẽ khởi động:
    *   **Scope Name:** Nhập `[192.168.1.0] DHCP - LANA`.
    *   **IP Address Range:**
        *   Start IP address: `192.168.1.1`
        *   End IP address: `192.168.1.254`
        *   Subnet mask: `255.255.255.0`
    *   **Add Exclusions and Delay:** Thêm dải IP không cấp phát.
        *   Start IP address: `192.168.1.1`
        *   End IP address: `192.168.1.20`
        *   Nhấn **Add**.
    *   **Lease Duration:** Giữ mặc định 8 ngày.
    *   **Configure DHCP Options:** Chọn **Yes, I want to configure these options now**.
    *   **Router (Default Gateway):** Nhập `192.168.1.1` và nhấn **Add**.
    *   **Domain Name and DNS Servers:**
        *   Parent domain: `bkaptech.vn`
        *   IP address (DNS Server): `192.168.1.2` (địa chỉ IP của DC). Nhấn **Add**.
    *   **WINS Servers:** Nhấn **Next** để bỏ qua.
    *   **Activate Scope:** Chọn **Yes, I want to activate this scope now**.
4.  Nhấn **Finish** để hoàn tất.

**Cách 2: Sử dụng Windows PowerShell**

```powershell
# Tạo Scope
Add-DhcpServerv4Scope -Name "[192.168.1.0] DHCP - LANA" -StartRange 192.168.1.1 -EndRange 192.168.1.254 -SubnetMask 255.255.255.0 -State Active

# Thêm dải loại trừ (Exclusion)
Add-DhcpServerv4ExclusionRange -ScopeId 192.168.1.0 -StartRange 192.168.1.1 -EndRange 192.168.1.20

# Cấu hình các tùy chọn cho Scope
Set-DhcpServerv4OptionValue -ScopeId 192.168.1.0 -OptionId 3 -Value 192.168.1.1   # Option 3: Router
Set-DhcpServerv4OptionValue -ScopeId 192.168.1.0 -OptionId 6 -Value 192.168.1.2   # Option 6: DNS Servers
Set-DhcpServerv4OptionValue -ScopeId 192.168.1.0 -OptionId 15 -Value "bkaptech.vn" # Option 15: Domain Name
```

---

#### **Phần C: Cấu hình Cấp phát IP Cố định (Reservation)**

**Mục tiêu:** Đảm bảo máy trạm `W10-CLIENT02` luôn nhận được một địa chỉ IP cố định là `192.168.1.50`.

**Bước 1: Lấy địa chỉ MAC của `W10-CLIENT02`**
Trên máy `W10-CLIENT02`, mở Command Prompt và gõ `ipconfig /all`. Tìm dòng `Physical Address` và ghi lại địa chỉ này (ví dụ: `00-0C-29-7D-63-B2`).

**Bước 2: Tạo Reservation**

**Cách 1: Sử dụng giao diện đồ họa (GUI)**
1.  Trong công cụ DHCP trên `SRV01-MEMBER`, điều hướng đến **Scope** -> **Reservations**.
2.  Chuột phải vào **Reservations** và chọn **New Reservation...**.
3.  Điền các thông tin:
    *   **Reservation name:** `W10-CLIENT02`
    *   **IP address:** `192.168.1.50`
    *   **MAC address:** Nhập địa chỉ MAC bạn đã ghi lại (không có dấu gạch ngang).
    *   **Description:** `May tram cua Ke toan`.
4.  Nhấn **Add**.

**Cách 2: Sử dụng Windows PowerShell**
```powershell
Add-DhcpServerv4Reservation -ScopeId 192.168.1.0 -IPAddress 192.168.1.50 -ClientId "00-0C-29-7D-63-B2" -Name "W10-CLIENT02" -Description "May tram Ke toan"
```
*Thay đổi địa chỉ MAC cho đúng với máy ảo của bạn.*

**Kiểm tra:**
*   Khởi động máy trạm `W10-CLIENT01`, dùng `ipconfig /all` để kiểm tra. Máy sẽ nhận được một IP động trong dải `192.168.1.21` trở đi.
*   Khởi động máy trạm `W10-CLIENT02`, dùng `ipconfig /all`. Máy sẽ luôn nhận được IP `192.168.1.50`.

---

#### **Phần D: Sao lưu và Khôi phục DHCP Server**

**Cách 1: Sử dụng giao diện đồ họa (GUI)**

1.  **Sao lưu (Backup):**
    *   Trên `SRV01-MEMBER`, tạo thư mục `C:\DHCP_Backup`.
    *   Trong công cụ DHCP, chuột phải vào tên máy chủ và chọn **Backup...**.
    *   Trỏ đến thư mục `C:\DHCP_Backup` và nhấn **OK**.
2.  **Khôi phục (Restore):**
    *   Để giả lập sự cố, hãy xóa Scope `[192.168.1.0] DHCP - LANA`.
    *   Chuột phải vào tên máy chủ và chọn **Restore...**.
    *   Trỏ đến thư mục `C:\DHCP_Backup` và nhấn **OK**. Dịch vụ sẽ được yêu cầu khởi động lại.
    *   Sau khi khởi động lại, Scope sẽ xuất hiện trở lại.

**Cách 2: Sử dụng Windows PowerShell**

```powershell
# Sao lưu
Backup-DhcpServer -Path "C:\DHCP_Backup" -ComputerName "SRV01-MEMBER.bkaptech.vn"

# Khôi phục
# Đầu tiên, dừng dịch vụ
Stop-Service DHCPServer
# Thực hiện khôi phục
Restore-DhcpServer -Path "C:\DHCP_Backup" -ComputerName "SRV01-MEMBER.bkaptech.vn"
# Khởi động lại dịch vụ
Start-Service DHCPServer
```