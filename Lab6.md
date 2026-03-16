### **Bài 6: CẤU HÌNH VÀ QUẢN LÝ Ổ ĐĨA**

**Các nội dung chính:**
*   Quản lý đĩa cơ bản (Basic Disk) và các loại phân vùng (MBR/GPT).
*   Cấu hình RAID mềm bằng đĩa động (Dynamic Disk).
*   Sử dụng Storage Spaces, giải pháp lưu trữ hiện đại.

---

### **9.1 Cấu hình và Quản lý Ổ đĩa cứng**

**1. Yêu cầu chuẩn bị**
*   Chuẩn bị một máy chủ Windows Server 2019 (có thể sử dụng máy `FILESRV01` từ Lab 5).
*   Tắt máy ảo và gắn thêm **5 ổ cứng ảo** mới, mỗi ổ có dung lượng ví dụ là 20 GB.

**Hướng dẫn thêm ổ cứng ảo trên VMware:**
1.  Trong VMware Workstation, chuột phải vào máy ảo và chọn **Settings**.
2.  Nhấn nút **Add...**, chọn **Hard Disk**, và nhấn **Next**.
3.  Chọn loại đĩa **SCSI** (Recommended) và nhấn **Next**.
4.  Chọn **Create a new virtual disk**.
5.  Đặt dung lượng là **20 GB** và chọn **Store virtual disk as a single file**. Nhấn **Next**.
6.  Nhấn **Finish**.
7.  Lặp lại các bước trên để thêm đủ 5 ổ cứng mới.
8.  Khởi động máy ảo.

---

### **Phần A: Đĩa Cơ bản (Basic Disks) - So sánh MBR và GPT**

**Mục tiêu:** Hiểu rõ sự khác biệt và giới hạn giữa hai kiểu phân vùng MBR (Master Boot Record) và GPT (GUID Partition Table).

**1. Mở công cụ quản lý đĩa**
*   **GUI:** Mở **Server Manager** -> **Tools** -> **Computer Management**. Sau đó chọn **Disk Management** ở menu bên trái.
*   **PowerShell:**
    ```powershell
    diskmgmt.msc
    ```
Khi Disk Management khởi động lần đầu tiên, một cửa sổ **Initialize Disk** sẽ hiện ra, yêu cầu bạn khởi tạo các ổ đĩa mới. **Hãy nhấn Cancel ở bước này** để chúng ta thực hiện thủ công.

**2. Cấu hình theo kiểu MBR (Master Boot Record)**
MBR là kiểu cũ, chỉ hỗ trợ tối đa 4 phân vùng Primary, hoặc 3 Primary và 1 Extended.

*   **GUI:**
    1.  Trong Disk Management, chuột phải vào **Disk 1** (đang ở trạng thái `Offline`) và chọn **Online**.
    2.  Chuột phải vào **Disk 1** một lần nữa (đang ở trạng thái `Not Initialized`) và chọn **Initialize Disk**.
    3.  Chọn **Disk 1**, chọn kiểu **MBR (Master Boot Record)**, và nhấn **OK**.
    4.  Bây giờ, hãy tạo 3 phân vùng Primary: Chuột phải vào vùng `Unallocated` của Disk 1, chọn **New Simple Volume...**, đặt dung lượng là 5 GB (5120 MB) và hoàn tất các bước. Lặp lại 3 lần.
    5.  Sau khi có 3 phân vùng Primary, chuột phải vào vùng `Unallocated` còn lại, bạn sẽ thấy tùy chọn **New Simple Volume...** sẽ tự động tạo một phân vùng **Extended** màu xanh lá cây.
    6.  Chuột phải vào vùng **Free space** bên trong phân vùng Extended, chọn **New Simple Volume...** để tạo một **Logical Drive**.

*   **PowerShell:**
    ```powershell
    # Đưa đĩa vào trạng thái Online
    Set-Disk -Number 1 -IsOffline $false

    # Khởi tạo đĩa theo kiểu MBR
    Initialize-Disk -Number 1 -PartitionStyle MBR

    # Tạo 3 phân vùng Primary
    New-Partition -DiskNumber 1 -Size 5GB -AssignDriveLetter | Format-Volume -FileSystem NTFS -NewFileSystemLabel "DATA-P1"
    New-Partition -DiskNumber 1 -Size 5GB -AssignDriveLetter | Format-Volume -FileSystem NTFS -NewFileSystemLabel "DATA-P2"
    New-Partition -DiskNumber 1 -Size 5GB -AssignDriveLetter | Format-Volume -FileSystem NTFS -NewFileSystemLabel "DATA-P3"
    
    # Tạo phân vùng cuối cùng (sẽ là logical drive)
    New-Partition -DiskNumber 1 -UseMaximumSize -AssignDriveLetter | Format-Volume -FileSystem NTFS -NewFileSystemLabel "DATA-L1"
    ```

**3. Cấu hình theo kiểu GPT (GUID Partition Table)**
GPT là kiểu mới hơn, hỗ trợ lên đến 128 phân vùng Primary trên Windows.

*   **GUI:**
    1.  Thực hiện Online và Initialize **Disk 2** tương tự như trên, nhưng ở bước Initialize, chọn kiểu **GPT (GUID Partition Table)**.
    2.  Chuột phải vào vùng `Unallocated` và tạo hơn 4 phân vùng Primary (ví dụ, 5 phân vùng, mỗi phân vùng 4 GB). Bạn sẽ thấy không có giới hạn nào.

*   **PowerShell:**
    ```powershell
    Set-Disk -Number 2 -IsOffline $false
    Initialize-Disk -Number 2 -PartitionStyle GPT
    
    # Tạo 5 phân vùng Primary để chứng minh GPT không bị giới hạn
    New-Partition -DiskNumber 2 -Size 4GB -AssignDriveLetter | Format-Volume -FileSystem NTFS
    New-Partition -DiskNumber 2 -Size 4GB -AssignDriveLetter | Format-Volume -FileSystem NTFS
    New-Partition -DiskNumber 2 -Size 4GB -AssignDriveLetter | Format-Volume -FileSystem NTFS
    New-Partition -DiskNumber 2 -Size 4GB -AssignDriveLetter | Format-Volume -FileSystem NTFS
    New-Partition -DiskNumber 2 -Size 4GB -AssignDriveLetter | Format-Volume -FileSystem NTFS
    ```

---

### **Phần B: Đĩa Động (Dynamic Disks) - RAID Mềm**

**Lưu ý:** Dynamic Disks là một công nghệ cũ (legacy). **Storage Spaces** (Phần C) là giải pháp hiện đại và được Microsoft khuyến nghị sử dụng.

**1. Chuyển đổi sang Dynamic Disk**
Để tạo các volume RAID, bạn cần chuyển đổi các đĩa từ Basic sang Dynamic. Chúng ta sẽ sử dụng Disk 3, 4, và 5.
*   **GUI:**
    1.  Trong Disk Management, đưa các Disk 3, 4, 5 về trạng thái **Online** và **Initialize** chúng (chọn MBR hoặc GPT đều được).
    2.  Chuột phải vào **Disk 3**, chọn **Convert to Dynamic Disk...**.
    3.  Trong cửa sổ hiện ra, tích chọn cả **Disk 3**, **Disk 4**, và **Disk 5**. Nhấn **OK**.
*   **PowerShell:**
    ```powershell
    # Online và Initialize các đĩa
    Set-Disk -Number 3,4,5 -IsOffline $false
    Initialize-Disk -Number 3,4,5

    # Chuyển đổi sang Dynamic
    ConvertTo-DynamicDisk -DiskNumber 3,4,5
    ```

**2. Tạo các loại Volume (RAID)**

Bây giờ bạn có thể chuột phải vào vùng `Unallocated` trên bất kỳ đĩa Dynamic nào để tạo:

*   **Spanned Volume:** Gộp dung lượng từ nhiều ổ đĩa thành một volume lớn. Không tăng hiệu suất, không có khả năng chịu lỗi.
*   **Striped Volume (RAID 0):** Ghi dữ liệu xen kẽ trên nhiều đĩa để tăng tốc độ đọc/ghi. Yêu cầu ít nhất 2 đĩa. **Không có khả năng chịu lỗi**, nếu 1 đĩa hỏng, toàn bộ dữ liệu sẽ mất.
*   **Mirrored Volume (RAID 1):** Sao chép dữ liệu y hệt nhau lên 2 ổ đĩa. Cung cấp khả năng chịu lỗi cao (chịu được hỏng 1 đĩa). Yêu cầu 2 đĩa.
*   **RAID-5 Volume:** Ghi dữ liệu và thông tin chẵn lẻ (parity) xen kẽ trên các đĩa. Cung cấp khả năng chịu lỗi (hỏng 1 đĩa) và hiệu quả về dung lượng. Yêu cầu ít nhất 3 đĩa.

---

### **Phần C: Storage Spaces - Giải pháp Quản lý Đĩa Hiện đại**

Storage Spaces cho phép bạn gộp nhiều ổ đĩa vật lý (bất kể kích thước, giao diện) thành một "bể" lưu trữ (Storage Pool), sau đó tạo các ổ đĩa ảo (Virtual Disks) từ bể đó với các mức độ chịu lỗi khác nhau.

**1. Tạo Storage Pool**

*   **GUI:**
    1.  Mở **Server Manager** -> **File and Storage Services** -> **Storage Pools**.
    2.  Trong mục **STORAGE POOLS**, nhấn **TASKS** -> **New Storage Pool...**.
    3.  Đặt tên cho Pool, ví dụ: `CompanyDataPool`.
    4.  Chọn các ổ đĩa vật lý có sẵn (Physical disks) để thêm vào Pool. Nhấn **Next** và **Create**.

*   **PowerShell:**
    ```powershell
    # Lấy danh sách các đĩa có thể gộp thành pool
    $PhysicalDisks = Get-PhysicalDisk -CanPool $true

    # Tạo một pool mới với tất cả các đĩa đó
    New-StoragePool -FriendlyName "CompanyDataPool" -StorageSubSystemFriendlyName "Windows Storage*" -PhysicalDisks $PhysicalDisks
    ```

**2. Tạo Virtual Disk và Volume**

Từ Storage Pool, chúng ta sẽ tạo ra các ổ đĩa ảo với các cấu hình khác nhau.

*   **GUI:**
    1.  Trong mục **VIRTUAL DISKS**, nhấn **TASKS** -> **New Virtual Disk...**.
    2.  Chọn Pool vừa tạo.
    3.  Đặt tên cho đĩa ảo, ví dụ `VDISK-Mirror`.
    4.  **Storage Layout:** Chọn **Mirror**.
    5.  **Provisioning type:** Chọn **Thin** (cấp phát dung lượng khi cần) hoặc **Fixed** (cấp phát toàn bộ ngay lập tức). Thin linh hoạt hơn.
    6.  **Size:** Nhập dung lượng cho đĩa ảo, ví dụ: 50 GB.
    7.  Nhấn **Create**.
    8.  Sau khi tạo xong, một wizard mới sẽ tự động chạy để tạo Volume. Bạn chỉ cần chọn kích thước và gán ký tự ổ đĩa (ví dụ: E:).

*   **PowerShell (Tạo ổ Two-way Mirror 50GB và ổ Parity 80GB):**
    ```powershell
    # Tạo một Virtual Disk dạng Two-way Mirror, cấp phát mỏng
    New-VirtualDisk -StoragePoolFriendlyName "CompanyDataPool" -FriendlyName "VDISK-Mirror" -ResiliencySettingName Mirror -NumberOfDataCopies 2 -Size 50GB -ProvisioningType Thin

    # Tạo một Virtual Disk dạng Parity (tương tự RAID-5), cấp phát mỏng
    New-VirtualDisk -StoragePoolFriendlyName "CompanyDataPool" -FriendlyName "VDISK-Parity" -ResiliencySettingName Parity -Size 80GB -ProvisioningType Thin

    # Lấy thông tin, khởi tạo, tạo phân vùng và format các đĩa ảo vừa tạo
    Get-VirtualDisk -FriendlyName "VDISK-Mirror" | Get-Disk | Initialize-Disk -Passthru | New-Partition -AssignDriveLetter -UseMaximumSize | Format-Volume -FileSystem NTFS -NewFileSystemLabel "MirrorData"
    Get-VirtualDisk -FriendlyName "VDISK-Parity" | Get-Disk | Initialize-Disk -Passthru | New-Partition -AssignDriveLetter -UseMaximumSize | Format-Volume -FileSystem NTFS -NewFileSystemLabel "ParityData"
    ```

**3. Kiểm tra khả năng chịu lỗi (Tùy chọn)**
*   Trong mục **PHYSICAL DISKS** của Storage Pool, bạn có thể chuột phải vào một ổ đĩa và chọn **Set Usage -> Retired** để giả lập ổ đĩa sắp hỏng. Dữ liệu sẽ tự động được di chuyển sang các ổ còn lại.
*   Nếu bạn tắt máy ảo, gỡ bỏ một ổ đĩa vật lý (đã thuộc Pool), các Volume dạng Mirror hoặc Parity vẫn sẽ hoạt động (trạng thái `Warning`), trong khi Volume dạng Simple sẽ bị lỗi.