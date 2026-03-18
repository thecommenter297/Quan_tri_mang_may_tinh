### **Bài 5: PHÂN QUYỀN VÀ CHIA SẺ DỮ LIỆU**

**Các nội dung chính sẽ được đề cập:**
*   Cấu hình và phân quyền chia sẻ dữ liệu qua mạng (Share & NTFS Permissions).
*   Cấu hình Shadow Copies để phục hồi phiên bản cũ của tệp tin.
*   Sử dụng Windows Server Backup để sao lưu và phục hồi dữ liệu.
*   Cấu hình Offline Files để cho phép truy cập dữ liệu khi không có mạng.

---

### **9.1 Cấu hình và Phân quyền Chia sẻ Dữ liệu**

**1. Mục tiêu**
Tạo các thư mục chia sẻ cho phòng ban IT và Sale trên một máy chủ tập trung (File Server). Phân quyền truy cập để đảm bảo chỉ các thành viên của phòng ban tương ứng mới có thể truy cập và chỉnh sửa dữ liệu của mình.

**2. Yêu cầu chuẩn bị**
*   **Máy chủ Domain Controller:** `DC01-2019` (đã cấu hình từ Lab 2).
*   **Máy chủ File Server:** Một máy Windows Server 2019 mới, đặt tên là `FILESRV01`, IP: `192.168.1.101`, đã gia nhập vào miền `bkaptech.vn`.
*   **Máy trạm:** `W10-CLIENT01` (đã gia nhập miền từ Lab 2).
*   **Active Directory:** Các OU `HANOI\IT` và `HANOI\Sale`, cùng các nhóm (`GG_S_IT`, `GG_S_Sale`) và người dùng (`hungnq`, `nghialv`) đã được tạo từ Lab trước.

<h4>Tạo OU HANOI\IT và HANOI\SALE bằng Powershell</h4>

<hr>

**Tạo OU IT và Sale trong OU HANOI**

```powershell
New-ADOrganizationalUnit -Name "IT" -Path "OU=HANOI,DC=bkaptech,DC=vn" -ErrorAction SilentlyContinue
New-ADOrganizationalUnit -Name "Sale" -Path "OU=HANOI,DC=bkaptech,DC=vn" -ErrorAction SilentlyContinue
```
**Tạo 2 Nhóm bảo mật (Security Groups)**
```powershell
New-ADGroup -Name "GG_S_IT" -GroupScope Global -Path "OU=IT,OU=HANOI,DC=bkaptech,DC=vn"
New-ADGroup -Name "GG_S_Sale" -GroupScope Global -Path "OU=Sale,OU=HANOI,DC=bkaptech,DC=vn"
```
**Thêm User vào Nhóm tương ứng**
```powershell
Add-ADGroupMember -Identity "GG_S_IT" -Members "hungnq"
Add-ADGroupMember -Identity "GG_S_Sale" -Members "nghialv"
```

**3. Hướng dẫn chi tiết**

#### **Phần A: Tạo cấu trúc thư mục và chia sẻ trên FILESRV01**

Trên máy chủ `FILESRV01`, chúng ta sẽ tạo một thư mục gốc chứa dữ liệu cho các phòng ban.

**Cách 1: Sử dụng giao diện đồ họa (GUI)**

Trên máy `FILESRV01`, mở **File Explorer** và tạo cấu trúc thư mục sau: `C:\Data\IT` và `C:\Data\Sale`.

**Cách 2: Sử dụng Windows PowerShell**

Mở PowerShell với quyền quản trị và chạy lệnh:
```powershell
New-Item -Path "C:\Data\IT" -ItemType Directory
New-Item -Path "C:\Data\Sale" -ItemType Directory
```

#### **Phần B: Phân quyền cho Thư mục IT**

Chúng ta sẽ kết hợp cả hai loại quyền: **Share Permissions** (quyền truy cập qua mạng) và **NTFS Permissions** (quyền truy cập trực tiếp trên ổ đĩa). Đây là phương pháp bảo mật tốt nhất.

**Bước 1: Cấu hình Share Permissions**

1.  Chuột phải vào thư mục `C:\Data\IT`, chọn **Properties**.
2.  Chuyển sang tab **Sharing** và nhấn **Advanced Sharing...**.
3.  Tích vào ô **Share this folder**.
4.  Nhấn vào nút **Permissions**.
5.  Chọn **Everyone** và nhấn **Remove**.
6.  Nhấn **Add...**, gõ `Authenticated Users` và nhấn **OK**.
7.  Chọn nhóm `Authenticated Users`, và trong khung **Permissions**, tích vào ô **Allow** cho quyền **Change** và **Read**.
8.  Nhấn **OK** hai lần để đóng các cửa sổ.

**Bước 2: Cấu hình NTFS Permissions**

1.  Vẫn trong cửa sổ **Properties** của thư mục `IT`, chuyển sang tab **Security**.
2.  Nhấn nút **Advanced**.
3.  Nhấn nút **Disable inheritance** và chọn **Remove all inherited permissions from this object**.
4.  Nhấn **Add**, sau đó nhấn vào **Select a principal**.
5.  Nhập `GG_S_IT` và nhấn **Check Names**, sau đó **OK**.
6.  Trong **Basic permissions**, cấp quyền **Modify**. Nhấn **OK**.
7.  Nhấn **Add** một lần nữa, thêm nhóm `Administrators` và cấp quyền **Full control**.
8.  Nhấn **Apply** và **OK**.

#### **Phần C: Phân quyền cho Thư mục Sale (Tương tự)**

Lặp lại các bước trong Phần B cho thư mục `C:\Data\Sale`, nhưng thay `GG_S_IT` bằng `GG_S_Sale`.

#### **Tổng hợp các bước bằng PowerShell (Nâng cao)**

Bạn có thể thực hiện tất cả các bước trên cho cả hai thư mục bằng đoạn script PowerShell sau:

```powershell
# Đường dẫn đến các thư mục
$pathIT = "C:\Data\IT"
$pathSale = "C:\Data\Sale"

# ---- Cấu hình cho thư mục IT ----
# Tạo Share với quyền Change cho Authenticated Users
New-SmbShare -Name "IT_Data" -Path $pathIT -ChangeAccess "Authenticated Users"

# Cấu hình quyền NTFS
$acl = Get-Acl $pathIT
$acl.SetAccessRuleProtection($true, $false) # Tắt kế thừa và xóa quyền cũ
$ruleAdmin = New-Object System.Security.AccessControl.FileSystemAccessRule("Administrators", "FullControl", "ContainerInherit,ObjectInherit", "None", "Allow")
$ruleIT = New-Object System.Security.AccessControl.FileSystemAccessRule("bkaptech\GG_S_IT", "Modify", "ContainerInherit,ObjectInherit", "None", "Allow")
$acl.AddAccessRule($ruleAdmin)
$acl.AddAccessRule($ruleIT)
Set-Acl -Path $pathIT -AclObject $acl

# ---- Cấu hình cho thư mục Sale ----
# Tạo Share
New-SmbShare -Name "Sale_Data" -Path $pathSale -ChangeAccess "Authenticated Users"

# Cấu hình quyền NTFS
$acl = Get-Acl $pathSale
$acl.SetAccessRuleProtection($true, $false)
$ruleAdmin = New-Object System.Security.AccessControl.FileSystemAccessRule("Administrators", "FullControl", "ContainerInherit,ObjectInherit", "None", "Allow")
$ruleSale = New-Object System.Security.AccessControl.FileSystemAccessRule("bkaptech\GG_S_Sale", "Modify", "ContainerInherit,ObjectInherit", "None", "Allow")
$acl.AddAccessRule($ruleAdmin)
$acl.AddAccessRule($ruleSale)
Set-Acl -Path $pathSale -AclObject $acl
```

#### **Phần D: Kiểm tra Phân quyền**

1.  Trên máy trạm `W10-CLIENT01`, đăng nhập bằng tài khoản `hungnq` (thuộc phòng IT).
2.  Mở File Explorer, truy cập đường dẫn `\\FILESRV01`.
3.  Mở thư mục `IT_Data` -> Bạn có thể tạo, xóa, sửa file thành công.
4.  Mở thư mục `Sale_Data` -> Bạn sẽ nhận được thông báo từ chối truy cập.
5.  Đăng xuất và đăng nhập lại bằng tài khoản `nghialv` (thuộc phòng Sale) và thực hiện kiểm tra tương tự. Kết quả sẽ ngược lại.

---

### **9.2 Cấu hình Shadow Copies và Windows Server Backup**

**Mục tiêu:** Bảo vệ dữ liệu trong các thư mục chia sẻ khỏi việc bị xóa hoặc sửa đổi nhầm bằng cách tạo các bản sao lưu tức thời (Shadow Copies) và sao lưu định kỳ (Backup).

#### **Phần A: Cấu hình Shadow Copies**

1.  Trên máy `FILESRV01`, mở **This PC**.
2.  Chuột phải vào ổ đĩa chứa dữ liệu (ví dụ: ổ **C:**), chọn **Properties**.
3.  Chuyển sang tab **Shadow Copies**.
4.  Chọn ổ đĩa **C:** và nhấn **Enable**. Nhấn **Yes** để xác nhận.
5.  Để tạo một bản sao lưu ngay lập tức, nhấn **Create Now**.
6.  **Kiểm tra:**
    *   Vào thư mục `C:\Data\IT`, xóa một tệp tin bất kỳ.
    *   Chuột phải vào thư mục `IT`, chọn **Properties**.
    *   Chuyển sang tab **Previous Versions**.
    *   Bạn sẽ thấy một phiên bản của thư mục tại thời điểm bạn tạo Shadow Copy. Bạn có thể nhấn **Open** để xem hoặc **Restore** để phục hồi.

#### **Phần B: Cấu hình Windows Server Backup**

**Bước 1: Cài đặt tính năng**

*   **GUI:** Trên `FILESRV01`, mở **Server Manager** -> **Manage** -> **Add Roles and Features**. Trong mục **Features**, tìm và cài đặt **Windows Server Backup**.
*   **PowerShell:** `Install-WindowsFeature Windows-Server-Backup`

**Bước 2: Thực hiện Sao lưu (Backup)**

1.  Mở **Server Manager** -> **Tools** -> **Windows Server Backup**.
2.  Ở menu bên phải, chọn **Backup Once...**.
3.  Chọn **Different options** và nhấn **Next**.
4.  Chọn **Custom** và nhấn **Next**.
5.  Nhấn **Add Items**, chọn thư mục `C:\Data` và nhấn **OK**.
6.  Chọn nơi lưu trữ bản sao lưu (ví dụ: một ổ đĩa khác) và làm theo các bước hướng dẫn để hoàn tất.

**Bước 3: Thực hiện Phục hồi (Restore)**

1.  Xóa thư mục `C:\Data\IT` để giả lập sự cố.
2.  Trong **Windows Server Backup**, chọn **Recover...**.
3.  Chọn **This server** và chọn bản sao lưu bạn vừa tạo.
4.  Chọn **Files and folders**.
5.  Trong cây thư mục, tìm đến thư mục `IT` đã bị xóa.
6.  Chọn **Original location** và làm theo các bước để phục hồi. Thư mục và dữ liệu sẽ được khôi phục lại.

---

### **9.3 Cấu hình Offline Files**

**Mục tiêu:** Cho phép người dùng phòng IT có thể truy cập và làm việc với dữ liệu trong thư mục `IT_Data` ngay cả khi máy tính của họ không kết nối được với máy chủ `FILESRV01`.

**1. Cấu hình trên Server (`FILESRV01`)**
1.  Chuột phải vào thư mục `C:\Data\IT`, chọn **Properties** -> **Sharing** -> **Advanced Sharing...**.
2.  Nhấn vào nút **Caching**.
3.  Chọn tùy chọn **All files and programs that users open from the shared folder are automatically available offline**.
4.  Nhấn **OK** hai lần.

**2. Cấu hình trên Client (`W10-CLIENT01`)**
1.  Đăng nhập bằng tài khoản `hungnq`.
2.  Mở **Control Panel** -> **Sync Center**.
3.  Nhấn vào **Manage offline files** ở menu bên trái.
4.  Nhấn **Enable offline files** và khởi động lại máy tính.
5.  Sau khi khởi động lại, truy cập vào thư mục chia sẻ `\\FILESRV01\IT_Data`.
6.  Chuột phải vào thư mục `IT_Data`, chọn **Always available offline**. Hệ thống sẽ bắt đầu đồng bộ dữ liệu về máy.

**3. Kiểm tra**
1.  Sau khi đồng bộ xong, trên máy chủ `FILESRV01`, hãy tắt card mạng để giả lập mất kết nối.
2.  Trên máy client `W10-CLIENT01`, người dùng `hungnq` vẫn có thể mở, đọc, và sửa các tệp tin trong thư mục `IT_Data` bình thường.
3.  Tạo một file mới trong thư mục này trên máy client.
4.  Bật lại card mạng trên máy chủ `FILESRV01`.
5.  Dữ liệu sẽ tự động được đồng bộ trở lại. File mới tạo trên client sẽ xuất hiện trên server.
