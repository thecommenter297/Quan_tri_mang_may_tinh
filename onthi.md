## Câu 1:

### 1. PHẦN TRÍCH XUẤT LỆNH DS (Chạy trong Command Prompt/CMD)

```cmd
# --- TẠO OU ---
dsadd ou "OU=OU_KTMT,DC=khoacntt,DC=edu,DC=vn"
dsadd ou "OU=OU_KHMT,DC=khoacntt,DC=edu,DC=vn"
dsadd ou "OU=OU_CNPM,OU=OU_KHMT,DC=khoacntt,DC=edu,DC=vn"

# --- TẠO NHÓM (GROUP) ---
dsadd group "CN=NhomKT,OU=OU_KTMT,DC=khoacntt,DC=edu,DC=vn" -secgrp yes -scope g
dsadd group "CN=NhomKH,OU=OU_KHMT,DC=khoacntt,DC=edu,DC=vn" -secgrp yes -scope g
dsadd group "CN=NhomCN,OU=OU_CNPM,OU=OU_KHMT,DC=khoacntt,DC=edu,DC=vn" -secgrp yes -scope g

# --- TẠO NGƯỜI DÙNG (USER) ---
dsadd user "CN=TP_KT,OU=OU_KTMT,DC=khoacntt,DC=edu,DC=vn" -samid TP_KT -pwd Abc@123 -mustchpwd no -disabled no
dsadd user "CN=KTMT1,OU=OU_KTMT,DC=khoacntt,DC=edu,DC=vn" -samid KTMT1 -pwd Abc@123 -mustchpwd no -disabled no
dsadd user "CN=KTMT2,OU=OU_KTMT,DC=khoacntt,DC=edu,DC=vn" -samid KTMT2 -pwd Abc@123 -mustchpwd no -disabled no

dsadd user "CN=TP_KH,OU=OU_KHMT,DC=khoacntt,DC=edu,DC=vn" -samid TP_KH -pwd Abc@123 -mustchpwd no -disabled no
dsadd user "CN=KHMT1,OU=OU_KHMT,DC=khoacntt,DC=edu,DC=vn" -samid KHMT1 -pwd Abc@123 -mustchpwd no -disabled no
dsadd user "CN=KHMT2,OU=OU_KHMT,DC=khoacntt,DC=edu,DC=vn" -samid KHMT2 -pwd Abc@123 -mustchpwd no -disabled no

# --- THÊM USER VÀO NHÓM (ADD MEMBERS) ---
dsmod group "CN=NhomKT,OU=OU_KTMT,DC=khoacntt,DC=edu,DC=vn" -addmbr "CN=TP_KT,OU=OU_KTMT,DC=khoacntt,DC=edu,DC=vn"
dsmod group "CN=NhomKT,OU=OU_KTMT,DC=khoacntt,DC=edu,DC=vn" -addmbr "CN=KTMT1,OU=OU_KTMT,DC=khoacntt,DC=edu,DC=vn"
dsmod group "CN=NhomKT,OU=OU_KTMT,DC=khoacntt,DC=edu,DC=vn" -addmbr "CN=KTMT2,OU=OU_KTMT,DC=khoacntt,DC=edu,DC=vn"

dsmod group "CN=NhomKH,OU=OU_KHMT,DC=khoacntt,DC=edu,DC=vn" -addmbr "CN=TP_KH,OU=OU_KHMT,DC=khoacntt,DC=edu,DC=vn"
dsmod group "CN=NhomKH,OU=OU_KHMT,DC=khoacntt,DC=edu,DC=vn" -addmbr "CN=KHMT1,OU=OU_KHMT,DC=khoacntt,DC=edu,DC=vn"
dsmod group "CN=NhomKH,OU=OU_KHMT,DC=khoacntt,DC=edu,DC=vn" -addmbr "CN=KHMT2,OU=OU_KHMT,DC=khoacntt,DC=edu,DC=vn"

# --- TẠO MÁY TÍNH (COMPUTER) ---
dsadd computer "CN=PC_TP_KT,OU=OU_KTMT,DC=khoacntt,DC=edu,DC=vn"
dsadd computer "CN=PC_KTMT1,OU=OU_KTMT,DC=khoacntt,DC=edu,DC=vn"
dsadd computer "CN=PC_KTMT2,OU=OU_KTMT,DC=khoacntt,DC=edu,DC=vn"

dsadd computer "CN=PC_TP_KH,OU=OU_KHMT,DC=khoacntt,DC=edu,DC=vn"
dsadd computer "CN=PC_KHMT1,OU=OU_KHMT,DC=khoacntt,DC=edu,DC=vn"
dsadd computer "CN=PC_KHMT2,OU=OU_KHMT,DC=khoacntt,DC=edu,DC=vn"

# --- KIỂM TRA ---
dsquery ou
dsquery group
dsquery user
dsquery computer
```

---

### 2. PHẦN BỔ SUNG LỆNH POWERSHELL (Nhanh và hiện đại hơn)

Nếu đề bài cho phép hoặc yêu cầu PowerShell, dùng bộ lệnh này sẽ ngắn gọn và dễ quản lý biến hơn nhiều:

```powershell
# 1. Khai báo biến chung
$domain = "DC=khoacntt,DC=edu,DC=vn"
$pwd = ConvertTo-SecureString "Abc@123" -AsPlainText -Force

# 2. Tạo OU
New-ADOrganizationalUnit -Name "OU_KTMT" -Path $domain
New-ADOrganizationalUnit -Name "OU_KHMT" -Path $domain
New-ADOrganizationalUnit -Name "OU_CNPM" -Path "OU=OU_KHMT,$domain"

# 3. Tạo Nhóm
New-ADGroup -Name "NhomKT" -GroupScope Global -Path "OU=OU_KTMT,$domain"
New-ADGroup -Name "NhomKH" -GroupScope Global -Path "OU=OU_KHMT,$domain"
New-ADGroup -Name "NhomCN" -GroupScope Global -Path "OU=OU_CNPM,OU=OU_KHMT,$domain"

# 4. Tạo User và đưa vào nhóm ngay lập tức
# Nhóm KTMT
$usersKT = "TP_KT", "KTMT1", "KTMT2"
foreach ($u in $usersKT) {
    New-ADUser -Name $u -SamAccountName $u -AccountPassword $pwd -Path "OU=OU_KTMT,$domain" -Enabled $true
    Add-ADGroupMember -Identity "NhomKT" -Members $u
    New-ADComputer -Name "PC_$u" -Path "OU=OU_KTMT,$domain"
}

# Nhóm KHMT
$usersKH = "TP_KH", "KHMT1", "KHMT2"
foreach ($u in $usersKH) {
    New-ADUser -Name $u -SamAccountName $u -AccountPassword $pwd -Path "OU=OU_KHMT,$domain" -Enabled $true
    Add-ADGroupMember -Identity "NhomKH" -Members $u
    New-ADComputer -Name "PC_$u" -Path "OU=OU_KHMT,$domain"
}
```

**Mẹo:** Trong phòng thi, nếu gõ `dsadd` bị báo lỗi *"command not found"*, hãy chắc chắn bạn đã cài đặt role **Active Directory Domain Services** và đang chạy CMD với quyền **Administrator**.

## Câu 2:
* Dùng Active Directory Users and Computers (ADUC) để cấu hình theo yêu cầu

## Câu 3:
* Dùng Active Directory Users and Computers (ADUC), chuột phải vào OU yêu cầu để phân quyền, nhấn **Delegate Control**, sau đó cài đặt quyền cụ thể cho cá nhân được yêu cầu

## Câu 4, 5, 6:

---

### 1. CHÍNH SÁCH NHÓM (GPO) & PHÂN QUYỀN GPO

**A. Quy trình cấu hình GPO nhanh:**
1.  **Mở công cụ:** `Group Policy Management`.
2.  **Tạo & Link:** Chuột phải vào OU -> **Create a GPO in this domain, and Link it here...**.
3.  **Tìm thiết lập:** Chuột phải GPO -> **Edit**.
4.  **Đường dẫn tắt:** `User Configuration` -> `Administrative Templates` -> **All Settings**.
    *   *Mẹo:* Nhấn phím ký tự đầu (ví dụ: `P` cho *Prevent access...*) để tìm nhanh tên chính sách.
5.  **Cập nhật:** Trên Client gõ `gpupdate /force`.

**B. Phân quyền quản trị GPO (Câu 5):**
1.  **Chọn đối tượng áp dụng:** Tại tab **Scope** -> mục **Security Filtering** -> Add nhóm muốn thực thi chính sách (ví dụ: `Nhom_NhanVien`).
2.  **Cấp quyền chỉnh sửa:** Tại tab **Delegation** -> Add người muốn quản lý (ví dụ: `TP_KT`) -> Chọn quyền **Edit settings, delete, and modify security**.

---

### 2. QUẢN LÝ TÀI NGUYÊN MẠNG

**A. Chia sẻ File và Phân quyền NTFS:**
1.  **Sharing:** Chuột phải Folder -> `Properties` -> `Sharing` -> `Advanced Sharing` -> Cấp quyền **Change** cho `Everyone`.
2.  **NTFS (Security):** Tab `Security` -> `Advanced` -> **Disable Inheritance** -> **Remove all...** -> Add nhóm cụ thể (ví dụ: `GG_S_KT`) với quyền **Modify**.
3.  **PowerShell tạo Share nhanh:** `New-SmbShare -Name "Ten_Share" -Path "C:\Data" -FullAccess "Everyone"`

**B. Hạn mức dung lượng (Quota):**
1.  **Công cụ:** `File Server Resource Manager` (FSRM).
2.  **Thực hiện:** `Quota Management` -> `Quotas` -> **Create Quota**.
3.  **Thông số:** Chọn đường dẫn thư mục -> Chọn **Hard Quota** -> Nhập dung lượng (100MB/500MB).

**C. Quản lý Ổ đĩa (RAID):**
1.  **Công cụ:** `diskmgmt.msc`.
2.  **Chuẩn bị:** Chuột phải Disk -> **Online** -> **Initialize** (chọn GPT).
3.  **Gộp đĩa:** Chuột phải Disk -> **Convert to Dynamic Disk**.
4.  **Tạo RAID:** Chuột phải vào vùng trống -> Chọn **New Mirrored Volume** (RAID 1) hoặc **New RAID-5 Volume**.

**D. Sao lưu (Backup):**
1.  **Công cụ:** `Windows Server Backup`.
2.  **Thực hiện:** Chọn **Backup Once** -> **Custom** -> **Add Items** (chọn thư mục cần sao lưu) -> Chọn nơi lưu (ổ đĩa khác hoặc folder mạng).

---

### 3. DỊCH VỤ DNS & DHCP

**A. DHCP Server:**
1.  **Ủy quyền (Bắt buộc):** Chuột phải tên Server trong giao diện DHCP -> **Authorize**.
2.  **Tạo Scope:** Chuột phải `IPv4` -> `New Scope`. Nhập dải IP, Subnet Mask.
3.  **Dải loại trừ:** Mục `Add Exclusions` (nhập dải IP dành cho Server/Router).
4.  **Cấu hình Option:** Tích chọn cấu hình DNS (006) và Router (003) ngay trong Wizard.
5.  **Giữ chỗ (Reservation):** Chuột phải mục `Reservations` -> Nhập IP muốn cố định và địa chỉ MAC của máy đó.

**B. DNS Server:**
1.  **Forward Zone:** Chuột phải `Forward Lookup Zones` -> `New Zone` -> Nhập tên miền (ví dụ: `bkaptech.vn`).
2.  **Reverse Zone:** Chuột phải `Reverse Lookup Zones` -> `New Zone` -> Nhập Network ID (3 lớp đầu của IP, ví dụ `192.168.1`).
3.  **Tạo bản ghi:**
    *   **Host (A):** Trỏ tên máy về IP.
    *   **Alias (CNAME):** Trỏ tên `www` về tên máy đã có bản ghi A.
    *   **Pointer (PTR):** Chuột phải bản ghi A -> Tích chọn **Update associated pointer record**.

---

### 4. GIÁM SÁT HỆ THỐNG (MONITORING)

**A. Giám sát xóa file (Auditing) - Full Giao diện:**
1.  **Bật công tắc:** Mở `Local Security Policy` (`secpol.msc`) -> `Advanced Audit Policy` -> `Object Access` -> **Audit File System** -> Chọn **Success**.
2.  **Chọn Folder:** Chuột phải Folder -> `Properties` -> `Security` -> `Advanced` -> Tab **Auditing** -> Add `Everyone` -> Tích chọn quyền **Delete**.
3.  **Xem log:** Mở `Event Viewer` -> `Security` -> **Filter Current Log** -> Nhập ID **4663**.

**B. Performance Monitor (Theo dõi hiệu năng):**
1.  **Công cụ:** `perfmon.msc`.
2.  **Thêm thông số:** Nhấn dấu **+** (xanh).
3.  **Counter hay dùng:** 
    *   `Processor` -> `% Processor Time` (CPU).
    *   `Memory` -> `Available MBytes` (RAM trống).

**C. Task Manager:**
1.  **Mở nhanh:** `Ctrl + Shift + Esc`.
2.  **Ứng dụng:** Xem nhanh ứng dụng nào đang treo hoặc ngốn RAM/CPU để End Task.

---

### MẸO JOIN DOMAIN BẰNG POWERSHELL (Cho máy Client Win 10)
Nếu bạn lười click chuột qua nhiều bước trên Win 10:
```powershell
# 1. Đặt DNS về máy Server
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses "192.168.1.100"

# 2. Join Domain (Máy sẽ hỏi mật khẩu Admin Domain)
Add-Computer -DomainName "bkaptech.vn" -Restart
```

**Lời khuyên:** Hãy bám sát các từ khóa tiếng Anh trong bảng này, bạn sẽ tìm thấy mọi thứ trong mục **All Settings** của GPO.
