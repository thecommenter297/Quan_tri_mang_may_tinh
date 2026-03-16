### **Bài 4: TRIỂN KHAI CHÍNH SÁCH NHÓM (GROUP POLICY)**

**Các nội dung chính:**
*   Triển khai các chính sách GPO cơ bản để giới hạn người dùng.
*   Cấu hình môi trường làm việc chung (màn hình nền).
*   Sao lưu và phục hồi GPO.
*   Sử dụng AppLocker để giới hạn phần mềm.
*   Giám sát truy cập và xóa tập tin.

**1. Yêu cầu chuẩn bị**
*   Máy chủ `DC01-2019` đã được nâng cấp lên Domain Controller.
*   Máy trạm `W10-CLIENT01` đã gia nhập vào miền `bkaptech.vn`.
*   Cấu trúc OU `HANOI\IT` đã được tạo từ Lab 3, cùng với các tài khoản người dùng trong OU này (ví dụ: `hungnq`).

---

### **Phần A: Triển khai các Chính sách Giới hạn Người dùng**

**Mục tiêu:** Tạo một Group Policy Object (GPO) để áp đặt các giới hạn chung cho người dùng trong phòng ban IT, bao gồm: khóa Registry, khóa Task Manager, cấm Command Prompt, và loại bỏ mục RUN khỏi Start Menu.

**1. Tạo Group Policy Object (GPO)**
1.  Trên máy chủ `DC01-2019`, mở **Server Manager** -> **Tools** -> **Group Policy Management**.
2.  Trong cây thư mục bên trái, điều hướng đến **Forest: bkaptech.vn** -> **Domains** -> **bkaptech.vn** -> **HANOI**.
3.  Chuột phải vào OU **IT**, chọn **Create a GPO in this domain, and Link it here...**.
4.  Đặt tên cho GPO là `IT_User_Restrictions` và nhấn **OK**.

**2. Cấu hình các chính sách**
1.  Chuột phải vào GPO `IT_User_Restrictions` vừa tạo và chọn **Edit...**.
2.  Cửa sổ **Group Policy Management Editor** sẽ mở ra. Tất cả các chính sách dưới đây đều nằm trong mục: `User Configuration` -> `Policies` -> `Administrative Templates`.

    *   **Khóa Registry Editor:**
        *   Điều hướng tới: `System`.
        *   Tìm chính sách `Prevent access to registry editing tools`.
        *   Mở chính sách, chọn **Enabled**, sau đó nhấn **Apply** và **OK**.

    *   **Khóa Task Manager:**
        *   Điều hướng tới: `System` -> `Ctrl+Alt+Del Options`.
        *   Tìm chính sách `Remove Task Manager`.
        *   Mở chính sách, chọn **Enabled**, nhấn **Apply** và **OK**.

    *   **Cấm Command Prompt (cmd):**
        *   Điều hướng tới: `System`.
        *   Tìm chính sách `Prevent access to the command prompt`.
        *   Mở chính sách, chọn **Enabled**, nhấn **Apply** và **OK**.

    *   **Loại bỏ mục RUN:**
        *   Điều hướng tới: `Start Menu and Taskbar`.
        *   Tìm chính sách `Remove Run menu from Start Menu`.
        *   Mở chính sách, chọn **Enabled**, nhấn **Apply** và **OK**.

**3. Cập nhật và Kiểm tra**
1.  Để chính sách có hiệu lực ngay lập tức, mở **Command Prompt** hoặc **PowerShell** trên cả máy chủ và máy client, sau đó chạy lệnh:
    ```
    gpupdate /force
    ```
2.  Trên máy trạm `W10-CLIENT01`, đăng nhập bằng tài khoản thuộc OU IT (ví dụ: `hungnq`).
3.  Kiểm tra:
    *   Nhấn `Ctrl + Shift + Esc` -> Task Manager sẽ bị chặn.
    *   Thử mở `regedit.exe` -> Sẽ bị chặn.
    *   Thử mở `cmd.exe` -> Sẽ bị chặn.
    *   Chuột phải vào nút Start -> Menu Run sẽ không còn.

---

### **Phần B: Cấu hình Màn hình nền Desktop chung**

**Mục tiêu:** Thiết lập một màn hình nền chung cho tất cả người dùng trong phòng IT để đảm bảo tính đồng bộ.

**1. Chuẩn bị Thư mục chia sẻ (Share Folder)**
1.  Trên máy chủ `DC01-2019`, tạo một thư mục trong ổ `C:\`, ví dụ `C:\Wallpaper`.
2.  Sao chép một file ảnh (ví dụ: `background.jpg`) vào thư mục này.
3.  Chuột phải vào thư mục `Wallpaper`, chọn **Properties** -> tab **Sharing** -> **Advanced Sharing...**.
4.  Tích vào **Share this folder**. Nhấn vào **Permissions**, cấp cho nhóm **Everyone** quyền **Read**. Nhấn **OK** hai lần.

**2. Tạo và cấu hình GPO**
1.  Trong **Group Policy Management**, chuột phải vào OU **IT** và tạo một GPO mới tên là `IT_Desktop_Wallpaper`.
2.  Chuột phải vào GPO vừa tạo, chọn **Edit...**.
3.  Điều hướng đến: `User Configuration` -> `Policies` -> `Administrative Templates` -> `Desktop` -> `Desktop`.
4.  Mở chính sách **Desktop Wallpaper**.
5.  Chọn **Enabled**.
6.  Trong ô **Wallpaper Name**, nhập đường dẫn UNC đến file ảnh: `\\DC01-2019\Wallpaper\background.jpg`.
7.  Chọn **Wallpaper Style** là **Fill** hoặc **Stretch**.
8.  Nhấn **Apply** và **OK**.

**3. Cập nhật và Kiểm tra**
1.  Chạy `gpupdate /force` trên máy client.
2.  Đăng xuất và đăng nhập lại bằng tài khoản `hungnq`. Màn hình nền sẽ được tự động áp dụng.

---

### **Phần C: Sao lưu và Phục hồi GPO**

**Mục tiêu:** Học cách sao lưu các GPO đã cấu hình để có thể phục hồi khi cần thiết.

**Cách 1: Sử dụng giao diện đồ họa (GUI)**

1.  **Sao lưu (Backup):**
    *   Trong **Group Policy Management**, chuột phải vào container **Group Policy Objects** và chọn **Back Up All...**.
    *   Chọn một vị trí để lưu bản sao lưu và nhập mô tả. Nhấn **Back Up**.
2.  **Phục hồi (Restore):**
    *   Để giả lập sự cố, bạn có thể xóa một GPO (ví dụ: `IT_User_Restrictions`).
    *   Chuột phải vào **Group Policy Objects** và chọn **Manage Backups...**.
    *   Chọn bản sao lưu từ danh sách và chọn GPO bạn muốn khôi phục. Nhấn **Restore**.

**Cách 2: Sử dụng Windows PowerShell**

Mở PowerShell với quyền quản trị trên `DC01-2019`.

1.  **Sao lưu một GPO cụ thể:**
    ```powershell
    Backup-GPO -Name "IT_User_Restrictions" -Path "C:\GPO_Backups"
    ```
2.  **Sao lưu tất cả GPO:**
    ```powershell
    Backup-GPO -All -Path "C:\GPO_Backups"
    ```
3.  **Phục hồi một GPO:**
    ```powershell
    Restore-GPO -Name "IT_User_Restrictions" -Path "C:\GPO_Backups"
    ```

---

### **Phần D: Giới hạn Phần mềm với AppLocker**

**Mục tiêu:** Ngăn chặn người dùng trong phòng IT chạy phần mềm không được phép (ví dụ: Firefox).

**1. Bật dịch vụ Application Identity**
AppLocker yêu cầu dịch vụ này phải chạy để hoạt động. Chúng ta sẽ bật nó thông qua GPO.
1.  Tạo một GPO mới liên kết tới OU **IT**, đặt tên là `IT_AppLocker_Service`.
2.  Edit GPO, điều hướng đến: `Computer Configuration` -> `Policies` -> `Windows Settings` -> `Security Settings` -> `System Services`.
3.  Tìm dịch vụ **Application Identity**.
4.  Mở nó ra, chọn **Define this policy setting** và đặt chế độ khởi động là **Automatic**. Nhấn **OK**.

**2. Tạo chính sách AppLocker**
1.  Tạo một GPO mới liên kết tới OU **IT**, đặt tên là `IT_AppLocker_Block_Firefox`.
2.  Edit GPO, điều hướng đến: `Computer Configuration` -> `Policies` -> `Windows Settings` -> `Security Settings` -> `Application Control Policies` -> `AppLocker`.
3.  **Quan trọng:** Chuột phải vào **Executable Rules** và chọn **Create Default Rules**. Thao tác này đảm bảo các file hệ thống của Windows vẫn có thể chạy.
4.  Chuột phải vào **Executable Rules** một lần nữa và chọn **Create New Rule...**.
5.  Nhấn **Next**.
6.  Tại màn hình **Permissions**, chọn Action là **Deny**.
7.  Tại màn hình **Conditions**, chọn **Path**.
8.  Nhấn **Browse Files...** và trỏ đến file thực thi của Firefox (ví dụ: `C:\Program Files\Mozilla Firefox\firefox.exe`).
9.  Nhấn **Next** qua các bước còn lại và **Create**.

**3. Cập nhật và Kiểm tra**
1.  Chạy `gpupdate /force` trên máy client và khởi động lại máy để chính sách máy tính có hiệu lực.
2.  Đăng nhập bằng tài khoản `hungnq` và thử chạy Firefox. Bạn sẽ nhận được thông báo chương trình đã bị chặn bởi quản trị viên.