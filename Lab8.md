### **Bài 8: CÀI ĐẶT VÀ CẤU HÌNH DỊCH VỤ DNS**

**Các nội dung chính:**
*   Cài đặt và cấu hình máy chủ phân giải tên miền (Primary DNS Server).
*   Tạo các phân vùng (Zone) và các bản ghi (Record) cơ bản.
*   Cấu hình máy chủ DNS dự phòng (Secondary DNS Server / Zone Transfer).

**1. Yêu cầu chuẩn bị**
*   **Máy chủ Primary DNS:** Hệ điều hành Windows Server 2019, tên máy `SRV01-2019`, IP tĩnh `192.168.1.5/24`.
*   **Máy chủ Backup DNS:** Hệ điều hành Windows Server 2019, tên máy `SRV02-2019`, IP tĩnh `192.168.1.3/24`.
*   **Máy trạm Client:** Hệ điều hành Windows 10 x64, tên máy `W10-CLIENT01`, IP tĩnh `192.168.1.10/24`.
*   Tắt Firewall trên tất cả các máy hoặc mở port 53 (TCP/UDP) để đảm bảo dịch vụ phân giải và đồng bộ hoạt động.

---

### **8.1 Cài đặt và cấu hình Primary DNS Server**

Thực hiện các bước sau trên máy chủ `SRV01-2019`.

#### **Bước 1: Cài đặt dịch vụ DNS Server**

**Cách 1: Dùng giao diện (GUI)**
1. Mở **Server Manager** -> **Manage** -> **Add Roles and Features**.
2. Nhấn Next đến phần **Server Roles**, tích chọn **DNS Server**.
3. Nhấn **Add Features** ở bảng hiện ra, tiếp tục nhấn Next và chọn **Install**.

**Cách 2: Dùng Windows PowerShell**
Mở PowerShell (Admin) và thực thi lệnh:
```powershell
Install-WindowsFeature -Name DNS -IncludeManagementTools
```

#### **Bước 2: Cấu hình Forward Lookup Zone (Phân giải tên miền sang IP)**

**Cách 1: Dùng giao diện**
1. Mở **Server Manager** -> **Tools** -> **DNS**.
2. Tại cây thư mục bên trái, chuột phải vào **Forward Lookup Zones** chọn **New Zone...**.
3. Chọn **Primary zone** -> Nhấn Next.
4. **Zone Name:** Nhập `bkaptech.vn` -> Nhấn Next.
5. Giữ nguyên tên file tạo tự động -> Nhấn Next.
6. Tại mục Dynamic Update, chọn **Allow both nonsecure and secure dynamic updates** -> Nhấn Next và **Finish**.

**Cách 2: Dùng Windows PowerShell**
```powershell
Add-DnsServerPrimaryZone -Name "bkaptech.vn" -ZoneFile "bkaptech.vn.dns" -DynamicUpdate NonSecureAndSecure
```

#### **Bước 3: Cấu hình Reverse Lookup Zone (Phân giải IP sang tên miền)**

**Cách 1: Dùng giao diện**
1. Chuột phải vào **Reverse Lookup Zones** chọn **New Zone...**.
2. Chọn **Primary zone** -> Chọn **IPv4 Reverse Lookup Zone** -> Nhấn Next.
3. **Network ID:** Nhập `192.168.1` -> Nhấn Next.
4. Giữ nguyên tên file tạo tự động -> Nhấn Next.
5. Chọn **Allow both nonsecure and secure dynamic updates** -> Nhấn Next và **Finish**.

**Cách 2: Dùng Windows PowerShell**
```powershell
Add-DnsServerPrimaryZone -NetworkId "192.168.1.0/24" -ZoneFile "1.168.192.in-addr.arpa.dns" -DynamicUpdate NonSecureAndSecure
```

#### **Bước 4: Tạo các bản ghi (Records) cơ bản**

**Cách 1: Dùng giao diện**
Mở thư mục `bkaptech.vn` trong **Forward Lookup Zones**, chuột phải vào khoảng trắng và tạo các bản ghi sau:
1. **Bản ghi Host (A) và PTR:** 
   * Chọn **New Host (A or AAAA)...**.
   * Name: `srv01-2019`. IP address: `192.168.1.5`. 
   * Tích chọn ô **Create associated pointer (PTR) record** để máy tự động tạo bản ghi nghịch. Nhấn **Add Host**.
   * Làm tương tự tạo bản ghi cho máy `srv02-2019` với IP `192.168.1.3`.
2. **Bản ghi CNAME (Bí danh):**
   * Chọn **New Alias (CNAME)...**.
   * Alias name: `www`.
   * Fully qualified domain name (FQDN) for target host: Điền `srv01-2019.bkaptech.vn`. Nhấn **OK**.
3. **Bản ghi MX (Mail Server):**
   * Chọn **New Mail Exchanger (MX)...**.
   * Bỏ trống ô Host or child domain.
   * FQDN of mail server: Điền `srv01-2019.bkaptech.vn`.
   * Mail server priority: `10`. Nhấn **OK**.

**Cách 2: Dùng Windows PowerShell**
```powershell
# Tạo bản ghi A và PTR
Add-DnsServerResourceRecordA -Name "srv01-2019" -ZoneName "bkaptech.vn" -IPv4Address "192.168.1.5" -CreatePtr
Add-DnsServerResourceRecordA -Name "srv02-2019" -ZoneName "bkaptech.vn" -IPv4Address "192.168.1.3" -CreatePtr

# Tạo bản ghi CNAME
Add-DnsServerResourceRecordCName -Name "www" -HostNameAlias "srv01-2019.bkaptech.vn" -ZoneName "bkaptech.vn"

# Tạo bản ghi MX
Add-DnsServerResourceRecordMX -Name "." -MailExchange "srv01-2019.bkaptech.vn" -Preference 10 -ZoneName "bkaptech.vn"
```

---

### **8.2 Khai báo DNS Client và Kiểm tra**

Thực hiện trên máy trạm `W10-CLIENT01`.

1. **Đổi DNS Server:** Mở cấu hình card mạng (ncpa.cpl), chuột phải vào card mạng chọn Properties -> IPv4. Đặt **Preferred DNS server** là `192.168.1.5`.
2. Mở Command Prompt (cmd) và thực hiện các lệnh kiểm tra:
   * `nslookup bkaptech.vn`
   * `nslookup www.bkaptech.vn`
   * `nslookup 192.168.1.5`
   Kết quả trả về hiển thị đúng IP và tên máy chủ quản lý chứng tỏ DNS hoạt động bình thường.

---

### **8.3 Cấu hình dịch vụ Backup DNS (Secondary Zone)**

Để dự phòng khi máy chủ Primary lỗi, ta sẽ dựng Secondary Zone trên máy chủ `SRV02-2019`.

#### **Bước 1: Cho phép Zone Transfer trên Primary Server (`SRV01-2019`)**

Thao tác này bắt buộc thực hiện trước để máy chủ Primary cấp quyền đồng bộ dữ liệu cho máy chủ Secondary.

**Cách 1: Dùng giao diện**
1. Trên máy `SRV01-2019`, mở công cụ DNS.
2. Chuột phải vào Zone `bkaptech.vn`, chọn **Properties**.
3. Chuyển sang tab **Zone Transfers**.
4. Tích vào ô **Allow zone transfers**.
5. Chọn **Only to the following servers**. Nhấn nút **Edit**, nhập IP `192.168.1.3` (Máy SRV02). Nhấn OK.
6. Làm tương tự các bước 2-5 cho Zone `1.168.192.in-addr.arpa` trong Reverse Lookup Zones.

**Cách 2: Dùng Windows PowerShell**
```powershell
Set-DnsServerPrimaryZone -Name "bkaptech.vn" -SecureSecondaries TransferToSecureServers -SecondaryServers "192.168.1.3"
Set-DnsServerPrimaryZone -Name "1.168.192.in-addr.arpa" -SecureSecondaries TransferToSecureServers -SecondaryServers "192.168.1.3"
```

#### **Bước 2: Tạo Secondary Zone trên Backup Server (`SRV02-2019`)**

1. Tiến hành cài đặt dịch vụ DNS Server trên máy `SRV02-2019` (Tham khảo Bước 1 - Phần 8.1).
2. Mở công cụ DNS trên `SRV02-2019`.

**Cách 1: Dùng giao diện**
1. Chuột phải vào **Forward Lookup Zones**, chọn **New Zone...** -> Nhấn Next.
2. Chọn **Secondary zone** -> Nhấn Next.
3. Zone Name: Nhập `bkaptech.vn` -> Nhấn Next.
4. **Master DNS Servers:** Nhập IP của máy chủ Primary là `192.168.1.5`. Đợi hệ thống xác thực (hiện dấu tích xanh) -> Nhấn Next -> **Finish**.
5. Làm tương tự cho **Reverse Lookup Zones**: Chọn Secondary Zone -> IPv4 -> Network ID `192.168.1` -> Nhập Master IP `192.168.1.5` -> Finish.

**Cách 2: Dùng Windows PowerShell**
```powershell
Add-DnsServerSecondaryZone -Name "bkaptech.vn" -ZoneFile "bkaptech.vn.dns" -MasterServers "192.168.1.5"
Add-DnsServerSecondaryZone -Name "1.168.192.in-addr.arpa" -ZoneFile "1.168.192.in-addr.arpa.dns" -MasterServers "192.168.1.5"
```

#### **Bước 3: Đồng bộ dữ liệu và Kiểm tra**

1. Trên máy `SRV02-2019`, trong công cụ DNS, click vào Zone `bkaptech.vn`. Nếu dữ liệu chưa xuất hiện hoặc báo lỗi "Zone Not Loaded by DNS Server".
2. Chuột phải vào Zone `bkaptech.vn`, chọn **Transfer from Master**.
3. Bấm `F5` (Refresh) để làm mới. Toàn bộ các bản ghi từ máy chủ `SRV01-2019` sẽ được tải về thành công.
4. Lặp lại thao tác **Transfer from Master** cho phần Reverse Lookup Zone. Lập cấu hình máy trạm trỏ DNS phụ về `192.168.1.3` để hoàn tất mô hình dự phòng.