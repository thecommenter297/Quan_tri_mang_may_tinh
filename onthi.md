## Câu 1:

### 1. PHẦN TRÍCH XUẤT LỆNH DS (Chạy trong Command Prompt/CMD)

Đây là các lệnh y hệt trong ảnh của bạn:

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
