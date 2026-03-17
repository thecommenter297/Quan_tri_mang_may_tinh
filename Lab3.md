### **Bài 3: CẤU HÌNH CÁC ĐỐI TƯỢNG TRONG ACTIVE DIRECTORY**

**Các nội dung chính:**
*   Tạo và cấu trúc các đơn vị tổ chức (Organizational Unit - OU).
*   Tạo và quản lý tài khoản nhóm (Group) và người dùng (User).
*   Thiết lập các thuộc tính và chính sách cho tài khoản.
*   Cấu hình ủy quyền quản trị trên OU.

**1. Yêu cầu chung**

Tổ chức và cấu hình các đối tượng người dùng, nhóm trong miền `bkaptech.vn` theo cơ cấu phòng ban của một công ty giả định tại Hà Nội.

**2. Chuẩn bị**
*   Máy chủ `DC01-2019` đã được nâng cấp lên Domain Controller (hoàn thành Lab 2).
*   Máy trạm `W10-CLIENT01` đã gia nhập vào miền `bkaptech.vn` (hoàn thành Lab 2).

---

### **Phần A: Tạo cấu trúc Organizational Unit (OU)**

OU được dùng để sắp xếp các đối tượng trong Active Directory (như người dùng, máy tính, nhóm) theo phòng ban, vị trí địa lý, giúp cho việc quản lý và áp đặt chính sách trở nên dễ dàng hơn. Chúng ta sẽ tạo một cấu trúc OU như sau:
*   HANOI
    *   Technical
    *   Sale
    *   Marketing

**Cách 1: Sử dụng giao diện đồ họa (GUI)**

1.  Trên máy chủ `DC01-2019`, mở **Server Manager** -> **Tools** -> **Active Directory Users and Computers**.
2.  Chuột phải vào tên miền `bkaptech.vn`, chọn **New** -> **Organizational Unit**.
3.  Trong cửa sổ **New Object - Organizational Unit**, nhập tên là `HANOI`.
4.  Đảm bảo tùy chọn **Protect container from accidental deletion** được chọn. Nhấn **OK**.
5.  Chuột phải vào OU `HANOI` vừa tạo, chọn **New** -> **Organizational Unit**.
6.  Nhập tên `Technical` và nhấn **OK**.
7.  Lặp lại bước 5 và 6 để tạo thêm các OU `Sale` và `Marketing` bên trong OU `HANOI`.

**Cách 2: Sử dụng Windows PowerShell**

Mở PowerShell với quyền quản trị (Admin) trên `DC01-2019` và thực thi các lệnh sau:

```powershell
# Tạo OU cha HANOI
New-ADOrganizationalUnit -Name "HANOI" -Path "DC=bkaptech,DC=vn"

# Tạo các OU con bên trong HANOI
New-ADOrganizationalUnit -Name "Technical" -Path "OU=HANOI,DC=bkaptech,DC=vn"
New-ADOrganizationalUnit -Name "Sale" -Path "OU=HANOI,DC=bkaptech,DC=vn"
New-ADOrganizationalUnit -Name "Marketing" -Path "OU=HANOI,DC=bkaptech,DC=vn"
```

---

### **Phần B: Tạo Nhóm (Groups) và Người dùng (Users)**

**Yêu cầu:** Tạo các nhóm và người dùng sau, đặt trong các OU tương ứng.

*   **OU Technical:**
    *   **Group:** `GG_S_Technicals`
    *   **Users:** `Nguyễn Quốc Hưng` (hungnq), `Chu Hồng Quân` (quanch)
*   **OU Sale:**
    *   **Group:** `GG_S_Sales`
    *   **Users:** `Lưu Văn Trưởng` (truonglv), `Lưu Văn Nghĩa` (nghialv)
*   **OU Marketing:**
    *   **Group:** `GG_S_Marketings`
    *   **User:** `Nguyễn Tiến Cường` (cuongnt)

**Cách 1: Sử dụng giao diện đồ họa (GUI)**

1.  **Tạo Group:**
    *   Trong **Active Directory Users and Computers**, chuột phải vào OU `Technical`.
    *   Chọn **New** -> **Group**.
    *   **Group name:** `GG_S_Technicals`.
    *   **Group scope:** `Global`, **Group type:** `Security`.
    *   Nhấn **OK**.
    *   Làm tương tự để tạo các group còn lại trong OU `Sale` và `Marketing`.
2.  **Tạo User:**
    *   Chuột phải vào OU `Technical`, chọn **New** -> **User**.
    *   Điền thông tin cho người dùng `Nguyễn Quốc Hưng`:
        *   First name: `Nguyen Quoc`
        *   Last name: `Hung`
        *   User logon name: `hungnq`
    *   Nhấn **Next**.
    *   Đặt mật khẩu, ví dụ: `P@ssw0rd123`. Bỏ chọn **User must change password at next logon** và chọn **Password never expires**. Nhấn **Next**, rồi **Finish**.
3.  **Thêm User vào Group:**
    *   Chuột phải vào user `hungnq` vừa tạo, chọn **Add to a group...**.
    *   Nhập tên group `GG_S_Technicals` và nhấn **Check Names**, sau đó nhấn **OK**.
4.  Lặp lại các bước trên để tạo tất cả người dùng và thêm họ vào các nhóm tương ứng.

**Cách 2: Sử dụng Windows PowerShell**

Đây là cách hiệu quả nhất để tạo hàng loạt. Mở PowerShell (Admin) và chạy script sau:

```powershell
# Định nghĩa mật khẩu mặc định
$Password = ConvertTo-SecureString "P@ssw0rd123" -AsPlainText -Force

# ---- PHÒNG BAN TECHNICAL ----
# Tạo Group
New-ADGroup -Name "GG_S_Technicals" -GroupScope Global -Path "OU=Technical,OU=HANOI,DC=bkaptech,DC=vn"
# Tạo Users
New-ADUser -Name "Nguyen Quoc Hung" -SamAccountName "hungnq" -UserPrincipalName "hungnq@bkaptech.vn" -Path "OU=Technical,OU=HANOI,DC=bkaptech,DC=vn" -AccountPassword $Password -Enabled $true -PasswordNeverExpires $true
New-ADUser -Name "Chu Hong Quan" -SamAccountName "quanch" -UserPrincipalName "quanch@bkaptech.vn" -Path "OU=Technical,OU=HANOI,DC=bkaptech,DC=vn" -AccountPassword $Password -Enabled $true -PasswordNeverExpires $true
# Thêm Users vào Group
Add-ADGroupMember -Identity "GG_S_Technicals" -Members "hungnq", "quanch"

# ---- PHÒNG BAN SALE ----
# Tạo Group
New-ADGroup -Name "GG_S_Sales" -GroupScope Global -Path "OU=Sale,OU=HANOI,DC=bkaptech,DC=vn"
# Tạo Users
New-ADUser -Name "Luu Van Truong" -SamAccountName "truonglv" -UserPrincipalName "truonglv@bkaptech.vn" -Path "OU=Sale,OU=HANOI,DC=bkaptech,DC=vn" -AccountPassword $Password -Enabled $true -PasswordNeverExpires $true
New-ADUser -Name "Luu Van Nghia" -SamAccountName "nghialv" -UserPrincipalName "nghialv@bkaptech.vn" -Path "OU=Sale,OU=HANOI,DC=bkaptech,DC=vn" -AccountPassword $Password -Enabled $true -PasswordNeverExpires $true
# Thêm Users vào Group
Add-ADGroupMember -Identity "GG_S_Sales" -Members "truonglv", "nghialv"

# ---- PHÒNG BAN MARKETING ----
# Tạo Group
New-ADGroup -Name "GG_S_Marketings" -GroupScope Global -Path "OU=Marketing,OU=HANOI,DC=bkaptech,DC=vn"
# Tạo User
New-ADUser -Name "Nguyen Tien Cuong" -SamAccountName "cuongnt" -UserPrincipalName "cuongnt@bkaptech.vn" -Path "OU=Marketing,OU=HANOI,DC=bkaptech,DC=vn" -AccountPassword $Password -Enabled $true -PasswordNeverExpires $true
# Thêm User vào Group
Add-ADGroupMember -Identity "GG_S_Marketings" -Members "cuongnt"
```

---

### **Phần C: Cấu hình các thuộc tính và chính sách đặc biệt**

Áp dụng các yêu cầu sau cho các đối tượng đã tạo.

1.  **Tất cả người dùng trên là thành viên của nhóm `Backup Operators`:**
    *   **GUI:** Giữ phím Ctrl và chọn tất cả các tài khoản người dùng vừa tạo, chuột phải, chọn **Add to a group...**, nhập `Backup Operators` và nhấn **OK**.
    *   **PowerShell:**
        ```powershell
        Add-ADGroupMember -Identity "Backup Operators" -Members "hungnq", "quanch", "truonglv", "nghialv", "cuongnt"
        ```

2.  **Tạm khóa tài khoản `hungnq`:**
    *   **GUI:** Chuột phải vào user `hungnq`, chọn **Disable Account**.
    *   **PowerShell:**
        ```powershell
        Disable-ADAccount -Identity "hungnq"
        ```

3.  **Người dùng `quanch` không được phép đổi mật khẩu:**
    *   **GUI:** Mở **Properties** của user `quanch`, qua tab **Account**, trong **Account options**, tích chọn **User cannot change password**.
    *   **PowerShell:**
        ```powershell
        Set-ADUser -Identity "quanch" -CannotChangePassword $true
        ```

4.  **Tài khoản `quanch` hết hạn vào ngày 31/12/2026:**
    *   **GUI:** Mở **Properties** của user `quanch`, qua tab **Account**, trong **Account expires**, chọn **End of:** và chọn ngày.
    *   **PowerShell:**
        ```powershell
        Set-ADUser -Identity "quanch" -AccountExpirationDate "2026-12-31"
        ```

5.  **Nhóm `GG_S_Technicals` chỉ được đăng nhập từ 7h sáng đến 9h tối (21h), từ thứ 2 đến Chủ nhật:**
    *   **GUI:**
        *   Chọn 2 user `hungnq` và `quanch`, chuột phải chọn **Properties**.
        *   Trong cửa sổ **Properties for Multiple Items**, qua tab **Account**.
        *   Nhấn nút **Logon Hours...**.
        *   Kéo chuột để chọn toàn bộ bảng, sau đó nhấn vào ô **Logon Denied** (tô xanh toàn bộ).
        *   Kéo chuột chọn vùng thời gian từ **7 AM** đến **9 PM** cho tất cả các ngày trong tuần. Nhấn vào ô **Logon Permitted** (tô xanh vùng này).
        *   Nhấn **OK** 2 lần.
    *   **PowerShell:** Cấu hình Logon Hours qua PowerShell phức tạp, sử dụng GUI là phương pháp được khuyến nghị và trực quan nhất cho tác vụ này.

---

### **Phần D: Cấu hình ủy quyền quản trị (Delegate Control)**

**Yêu cầu:** Cấp cho tài khoản `hungnq` quyền được phép tạo, xóa và quản lý các tài khoản người dùng khác, nhưng **chỉ trong phạm vi OU Technical**.

1.  **Kích hoạt lại tài khoản `hungnq`** để thực hiện việc đăng nhập kiểm tra quyền.
    *   **PowerShell:** `Enable-ADAccount -Identity "hungnq"`
2.  **Thực hiện ủy quyền:**
    *   Trong **Active Directory Users and Computers**, chuột phải vào OU `Technical` và chọn **Delegate Control...**.
    *   Trong Wizard, nhấn **Next**.
    *   Nhấn **Add...**, nhập `hungnq`, nhấn **Check Names** và **OK**. Nhấn **Next**.
    *   Trong màn hình **Tasks to Delegate**, chọn **Delegate the following common tasks:** và tích vào ô **Create, delete, and manage user accounts**.
    *   Nhấn **Next**, sau đó nhấn **Finish**.

---

### **Phần E: Kiểm tra ủy quyền từ máy trạm Windows 10**

**Yêu cầu:** Đăng nhập vào máy client bằng tài khoản `hungnq`, sử dụng công cụ quản trị từ xa (RSAT) để tạo một tài khoản mới tên `Vũ Văn Cường (cuongvv)` trong OU `Technical`.

**1. Cài đặt RSAT trên máy `W10-CLIENT01`:**
*   Trên Windows 10 (phiên bản 1809 trở lên), RSAT được cài như một tính năng tùy chọn.
    *   **GUI:** Mở **Settings** -> **Apps** -> **Optional features** -> **Add a feature**. Tìm và cài đặt **RSAT: Active Directory Domain Services and Lightweight Directory Services Tools**.
    *   **PowerShell (trên máy Windows 10, chạy với quyền Admin):**
        ```powershell
        Add-WindowsCapability -Online -Name "Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0"
        ```
> P/S: Thực ra khi máy client đã vào mạng nội bộ nó sẽ mất kết nối Internet nên sẽ không tải được như trên. Nên ta sẽ mượn tạm Internet:
> * Vào phần cài đặt máy ảo (VMware/VirtualBox), đổi Card mạng của máy Win 10 từ LAN Segment (hoặc Internal) sang NAT.
> * Vào phần IP trong Win 10, chỉnh lại thành Obtain an IP address automatically (để nó nhận IP từ modem ảo và có mạng).
> * Tiến hành cài đặt RSAT như hướng dẫn (lúc này nó sẽ tải được từ internet).
> * Sau khi cài xong, bạn chỉnh ngược lại: Đổi Card mạng về LAN Segment, đặt lại IP tĩnh .10 và DNS .100 để tiếp tục làm Lab.

**2. Kiểm tra:**
1.  Đăng xuất khỏi tài khoản Administrator trên máy `W10-CLIENT01`.
2.  Đăng nhập bằng tài khoản `bkaptech.vn\hungnq` với mật khẩu đã tạo.
3.  Vào **Start**, gõ "Administrative Tools" và mở nó.
4.  Mở **Active Directory Users and Computers**.
5.  **Thử tạo User trong OU Technical:**
    *   Điều hướng đến OU **HANOI** -> **Technical**.
    *   Chuột phải vào khoảng trống, chọn **New** -> **User**.
    *   Tạo tài khoản `Vũ Văn Cường` (cuongvv). Quá trình sẽ thành công.
6.  **Thử tạo User trong OU Sale:**
    *   Điều hướng đến OU **HANOI** -> **Sale**.
    *   Chuột phải, bạn sẽ thấy tùy chọn **New** -> **User** bị mờ đi hoặc khi tạo sẽ báo lỗi không có quyền.
7.  **Kết quả:** Điều này chứng tỏ việc ủy quyền đã thành công. `hungnq` chỉ có quyền quản trị user trong phạm vi OU `Technical` mà thôi.
