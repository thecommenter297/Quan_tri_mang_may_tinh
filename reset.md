Nếu bạn đang hết dung lượng ổ đĩa và muốn "reset" lại máy Domain Controller (DC) để làm lại từ đầu mà không cần cài mới Windows, bạn phải thực hiện quy trình **Demote** (hạ cấp) máy chủ từ Domain Controller xuống thành Server thường, sau đó gỡ bỏ các vai trò (Role).

**Lưu ý quan trọng:**
*   Hành động này sẽ **xóa sạch** cấu hình Domain trên máy chủ này.
*   Hãy chắc chắn bạn đã **Snapshot** máy ảo trước khi thực hiện để phòng trường hợp lỗi không mong muốn.

Dưới đây là các bước sử dụng PowerShell để thực hiện:

### Bước 1: Hạ cấp cưỡng bức bằng Giao diện (Force Demote)

1.  Mở **Server Manager** -> **Manage** -> **Remove Roles and Features**.
2.  Nhấn **Next** đến phần **Server Roles**, bỏ tích chọn **Active Directory Domain Services**.
3.  Một cửa sổ hiện ra, nhấn **Remove Features**.
4.  Ngay lập tức, một thông báo lỗi sẽ hiện ra yêu cầu bạn phải hạ cấp DC trước. Hãy nhấn vào dòng chữ xanh: **Demote this domain controller**.
5.  **Cực kỳ quan trọng:** Tại màn hình hiện ra, hãy tích vào ô: **Force the removal of this domain controller**. Nhấn **Next**.
6.  Tích vào ô **Proceed with removal**. Nhấn **Next**.
7.  Thiết lập mật khẩu mới cho tài khoản Administrator cục bộ (ví dụ: `Admin@123`). Nhấn **Next**.
8.  Nhấn **Demote**. Máy chủ sẽ thực hiện gỡ bỏ và tự động khởi động lại.


### Bước 2: Gỡ bỏ các vai trò AD DS và DNS
Sau khi máy khởi động lại, máy chủ không còn là DC nữa, nhưng các dịch vụ vẫn còn cài đặt. Hãy gỡ bỏ chúng để giải phóng tài nguyên:
> Nhớ mở Powershell với quyền Admin.
```powershell
Uninstall-WindowsFeature -Name AD-Domain-Services, DNS -IncludeManagementTools -Restart
```

### Bước 3: Dọn dẹp không gian ổ đĩa (Free up disk space)
Để giải phóng dung lượng cho máy chủ Windows Server 2019, hãy chạy các lệnh sau trong PowerShell (Admin):

**1. Xóa các file tạm:**
```powershell
# Xóa file trong thư mục Temp
Get-ChildItem -Path "C:\Windows\Temp\*" -Recurse | Remove-Item -Force -ErrorAction SilentlyContinue
Get-ChildItem -Path "C:\Users\Administrator\AppData\Local\Temp\*" -Recurse | Remove-Item -Force -ErrorAction SilentlyContinue
```

**2. Dọn dẹp thư mục WinSxS (Thư mục này thường chiếm rất nhiều dung lượng):**
Lệnh này sẽ xóa các bản cập nhật cũ không còn cần thiết:
```powershell
Dism.exe /online /Cleanup-Image /StartComponentCleanup
```

**3. Xóa cache Windows Update:**
```powershell
Stop-Service -Name wuauserv
Remove-Item -Path "C:\Windows\SoftwareDistribution\*" -Recurse -Force
Start-Service -Name wuauserv
```

### Bước 4: Kiểm tra và bắt đầu lại
Sau khi thực hiện các bước trên:
1.  Máy chủ của bạn đã trở về trạng thái như một máy mới cài đặt Windows Server 2019 (nhưng không mất công cài lại OS).
2.  Bạn có thể đổi tên máy (nếu cần) bằng lệnh:
    ```powershell
    Rename-Computer -NewName "Tên_Mới" -Restart
    ```
3.  Sau khi mọi thứ đã "sạch", **hãy tạo một Snapshot mới** trong VMware. Từ đây, nếu làm sai bất cứ bài lab nào, bạn chỉ cần `Revert` về Snapshot này là xong, không cần phải demote hay dọn dẹp lại từ đầu nữa.

Bạn có thể tiếp tục với các bài Lab tiếp theo sau khi đã thực hiện xong các bước này.
