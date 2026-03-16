### **Bài 2: TRIỂN KHAI DỊCH VỤ ACTIVE DIRECTORY DOMAIN SERVICES**

**Các nội dung chính:**
*   Nâng cấp máy chủ Windows Server 2019 lên Domain Controller.
*   Gia nhập (Join) máy trạm Windows 10 vào Domain.

---

### **2.1 Nâng cấp máy chủ lên Domain Controller và Join Domain**

**1. Mục tiêu**
*   Cài đặt và cấu hình dịch vụ Active Directory Domain Services (AD DS) trên máy chủ để quản lý một miền (domain) mới.
*   Cho phép máy trạm gia nhập vào miền để quản lý tập trung.

**2. Chuẩn bị**
*   **Máy chủ Domain Controller (DC):**
    *   Hệ điều hành: Windows Server 2019 Datacenter (Desktop Experience).
    *   Tên máy: `DC01-2019`.
    *   IP Address: `192.168.1.100`.
    *   Subnet Mask: `255.255.255.0`.
    *   **Preferred DNS Server:** `127.0.0.1` (Tự trỏ về chính nó).
*   **Máy trạm (Client):**
    *   Hệ điều hành: Windows 10 x64.
    *   Tên máy: `W10-CLIENT01`.
    *   IP Address: `192.168.1.110`.
    *   Subnet Mask: `255.255.255.0`.
    *   **Preferred DNS Server:** `192.168.1.100` (Trỏ về địa chỉ IP của máy DC).
*   **Tên miền (Domain Name):** `bkaptech.vn`

**Lưu ý quan trọng trước khi bắt đầu:**
1.  **Cấu hình DNS:** Đây là bước quan trọng nhất. Máy client **BẮT BUỘC** phải trỏ DNS về địa chỉ IP của máy chủ Domain Controller. Nếu sai bước này, máy client sẽ không thể tìm thấy và gia nhập domain.
2.  **Kết nối mạng:** Đảm bảo cả hai máy ảo được đặt trong cùng một dải mạng ảo (ví dụ: VMnet Host-only, LAN Segment) và có thể `ping` thấy nhau bằng địa chỉ IP.
3.  **Tường lửa (Firewall):** Để kiểm tra kết nối `ping` dễ dàng, bạn có thể tạm thời tắt Firewall trên cả hai máy.
    *   **Lệnh PowerShell để tắt Firewall:** `Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False`

**3. Hướng dẫn chi tiết**

#### **Phần A: Cài đặt Role Active Directory Domain Services trên Server**

**Cách 1: Sử dụng giao diện đồ họa (GUI)**

1.  Trên máy chủ `DC01-2019`, mở **Server Manager**.
2.  Chọn **Manage** -> **Add Roles and Features**.
3.  Trong cửa sổ **Add Roles and Features Wizard**, nhấn **Next** để bỏ qua màn hình chào mừng.
4.  Chọn **Role-based or feature-based installation** và nhấn **Next**.
5.  Chọn máy chủ hiện tại từ danh sách (Server Pool) và nhấn **Next**.
6.  Trong danh sách **Roles**, tích vào ô **Active Directory Domain Services**.
7.  Một cửa sổ pop-up sẽ hiện ra, yêu cầu thêm các feature cần thiết. Nhấn nút **Add Features**.
8.  Nhấn **Next** liên tục qua các màn hình Features và AD DS.
9.  Tại màn hình **Confirmation**, nhấn **Install** để bắt đầu cài đặt.

**Cách 2: Sử dụng Windows PowerShell**

Mở PowerShell với quyền quản trị (Admin) và chạy lệnh sau:
```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
```

---

#### **Phần B: Nâng cấp máy chủ lên Domain Controller (Promote)**

Sau khi cài đặt Role ở Phần A thành công, bạn cần thực hiện bước "nâng cấp" máy chủ này thành một máy điều khiển miền.

**Cách 1: Sử dụng giao diện đồ họa (GUI)**

1.  Trong **Server Manager**, bạn sẽ thấy một biểu tượng cảnh báo hình tam giác màu vàng ở góc trên bên phải. Nhấn vào đó và chọn **Promote this server to a domain controller**.
2.  Trong cửa sổ **Deployment Configuration**, chọn **Add a new forest**.
3.  Tại ô **Root domain name**, nhập tên miền của bạn: `bkaptech.vn`. Nhấn **Next**.
4.  Tại cửa sổ **Domain Controller Options**:
    *   **Forest functional level** và **Domain functional level**: Chọn **Windows Server 2016** (đây là mức cao nhất và tương thích tốt nhất cho các môi trường hiện đại).
    *   Nhập mật khẩu cho **Directory Services Restore Mode (DSRM)**. Đây là mật khẩu dùng để khôi phục AD khi có sự cố. *Lưu ý: Mật khẩu này nên khác mật khẩu Administrator*.
    *   Nhấn **Next**.
5.  Nhấn **Next** ở màn hình **DNS Options** (bỏ qua cảnh báo delegation).
6.  Hệ thống sẽ tự động tạo tên **NetBIOS** (BKAPTECH). Nhấn **Next**.
7.  Giữ nguyên đường dẫn mặc định cho database, log files, và SYSVOL. Nhấn **Next**.
8.  Xem lại các lựa chọn của bạn và nhấn **Next**.
9.  Hệ thống sẽ kiểm tra các điều kiện tiên quyết. Khi thấy thông báo "All prerequisite checks passed successfully", nhấn **Install**.
10. Máy chủ sẽ tự động cấu hình và khởi động lại.

**Cách 2: Sử dụng Windows PowerShell**

Mở PowerShell với quyền quản trị và chạy lệnh sau. Hệ thống sẽ yêu cầu bạn nhập mật khẩu DSRM một cách an toàn.

```powershell
Install-ADDSForest `
    -DomainName "bkaptech.vn" `
    -DomainNetbiosName "BKAPTECH" `
    -DomainMode Win2016 `
    -ForestMode Win2016 `
    -InstallDns `
    -Force
```
*Lưu ý: Dấu `` ` `` ở cuối mỗi dòng cho phép bạn ngắt lệnh ra nhiều dòng cho dễ đọc. Lệnh `-Force` sẽ tự động khởi động lại máy sau khi hoàn tất.*

---

#### **Phần C: Join máy trạm Windows 10 vào Domain**

**1. Cấu hình và kiểm tra máy trạm**
*   Trên máy `W10-CLIENT01`, đảm bảo bạn đã đặt IP tĩnh và **Preferred DNS Server** là `192.168.1.100`.
*   Mở Command Prompt (cmd) và ping thử đến DC bằng cả IP và tên miền để chắc chắn DNS hoạt động:
    *   `ping 192.168.1.100`
    *   `ping bkaptech.vn`
*   Nếu cả hai lệnh đều thành công, bạn có thể tiến hành join domain.

**Cách 1: Sử dụng giao diện đồ họa (GUI)**

1.  Chuột phải vào nút **Start** -> **System**.
2.  Kéo xuống và chọn **Advanced system settings**.
3.  Trong cửa sổ **System Properties**, chuyển qua tab **Computer Name**.
4.  Nhấn nút **Change...**.
5.  Trong mục **Member of**, chọn **Domain** và nhập `bkaptech.vn`. Nhấn **OK**.
6.  Một cửa sổ đăng nhập sẽ hiện ra. Nhập tài khoản và mật khẩu của người quản trị domain:
    *   User name: `Administrator`
    *   Password: (Mật khẩu bạn đã đặt cho tài khoản Administrator của máy DC).
7.  Nhấn **OK**. Một thông báo chào mừng bạn đến với domain sẽ xuất hiện.
8.  Nhấn **OK** và khởi động lại máy tính theo yêu cầu.

**Cách 2: Sử dụng Windows PowerShell**

Mở PowerShell với quyền quản trị trên máy `W10-CLIENT01` và chạy lệnh sau. Một cửa sổ sẽ hiện ra yêu cầu bạn nhập mật khẩu cho tài khoản Administrator của domain.

```powershell
Add-Computer -DomainName "bkaptech.vn" -Credential "bkaptech.vn\Administrator" -Restart
```
*Lệnh `-Restart` sẽ tự động khởi động lại máy tính sau khi join domain thành công.*

**4. Kiểm tra kết quả**
*   Sau khi máy `W10-CLIENT01` khởi động lại, tại màn hình đăng nhập, bạn sẽ có thể đăng nhập bằng tài khoản domain, ví dụ: `BKAPTECH\Administrator`.
*   Trên máy chủ `DC01-2019`, mở **Active Directory Users and Computers** (từ Server Manager -> Tools), vào mục **Computers**, bạn sẽ thấy đối tượng máy tính `W10-CLIENT01` đã xuất hiện.