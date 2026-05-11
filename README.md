# QUẢN LÝ CẦM ĐỒ 

Họ và tên: Nguyễn Hữu Doan

MSSV: K235480106008

## 1. Mô tả bài toán

Hệ thống quản lý cầm đồ cần hỗ trợ các chức năng nghiệp vụ chính sau:

- Quản lý thông tin **khách hàng**
- Quản lý **hợp đồng cầm đồ**
- Quản lý **tài sản thế chấp**
- Tính **lãi đơn** trước `deadline1`
- Tính **lãi kép** sau `deadline1`
- Ghi nhận việc **trả nợ từng phần**
- Xử lý **chuộc lại tài sản**
- Truy vấn **danh sách nợ xấu**
- Hỗ trợ **thanh lý tài sản**
- Lưu **lịch sử thanh toán** phục vụ đối soát và kiểm tra

Bài toán này không chỉ dừng ở mức lưu thông tin đơn giản mà còn phải xử lý logic nghiệp vụ như:
- tính tiền phải trả theo thời gian
- theo dõi trạng thái hợp đồng
- theo dõi trạng thái tài sản
- đảm bảo tài sản còn lại luôn đủ giá trị bảo đảm cho khoản nợ

---

## 2. Phân tích logic nghiệp vụ

### 2.1. Khách hàng và hợp đồng
- Một `khachHang` có thể có nhiều `hopDong`
- Một `hopDong` chỉ thuộc về một `khachHang`

### 2.2. Hợp đồng và tài sản
- Một `hopDong` có thể bao gồm nhiều `taiSan`
- Một `taiSan` trong một lần cầm cố gắn với một `hopDong`

### 2.3. Cơ chế tính lãi
- Trước `deadline1`: tính **lãi đơn**
- Sau `deadline1`: tính **lãi kép**
- Mức lãi theo đề bài:
  - `5.000đ / 1.000.000đ / ngày`
- Tương đương:
  - `0.5% / ngày`
  - tức `0.005` nếu biểu diễn dưới dạng số thập phân

### 2.4. Trả nợ từng phần
- Khách có thể trả tiền làm nhiều lần
- Mỗi lần trả phải ghi thành 1 dòng trong bảng lịch sử thanh toán
- Không được chỉ cập nhật tổng nợ còn lại vì sẽ mất lịch sử giao dịch

### 2.5. Điều kiện trả lại tài sản
Khách chỉ được chuộc bớt tài sản nếu:

- **tổng giá trị tài sản còn giữ lại >= dư nợ còn lại**

Điều này giúp đảm bảo tiệm cầm đồ vẫn còn đủ tài sản bảo đảm cho phần nợ chưa thanh toán.

### 2.6. Quá hạn và thanh lý
- Sau `deadline1`: hợp đồng chuyển sang `QuaHan`
- Sau `deadline2`: tài sản chuyển sang `SanSangThanhLy`
- Nếu hợp đồng chuyển sang `DaThanhLy` thì tài sản chuyển sang `DaBanThanhLy`

---

## 3. Quy ước đặt tên

Toàn bộ bảng, cột, procedure và function được đặt tên theo quy ước:

- tiếng Việt không dấu
- dạng **bướu lạc đà** (camelCase)

Ví dụ:
- `khachHang`
- `hopDong`
- `taiSan`
- `chiTietHopDongTaiSan`
- `lichSuThanhToan`
- `spDangKyHopDongMoi`
- `fnTinhTienHopDong`

---

<img width="1467" height="827" alt="Untitled (2)" src="https://github.com/user-attachments/assets/fb18e038-067e-47fb-9108-f42e75bbf9b1" />

Sơ đồ ERD 


## 4. Tạo cơ sở dữ liệu

```sql
-- Tạo cơ sở dữ liệu quản lý cầm đồ
CREATE DATABASE quanLyCamDo;
GO

-- Chọn cơ sở dữ liệu vừa tạo để thao tác
USE quanLyCamDo;
GO
```

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/50393f01-6bc1-47b6-ba10-30d63130459f" />
Tạo cơ sở dữ liệu quản lý cầm đồ



5. Tạo bảng khachHang

```sql
CREATE TABLE khachHang (
    khachHangId INT IDENTITY(1,1) PRIMARY KEY,         -- Khóa chính khách hàng
    hoTen NVARCHAR(100) NOT NULL,                      -- Họ tên khách hàng
    soDienThoai VARCHAR(20) NOT NULL UNIQUE,           -- Số điện thoại duy nhất
    canCuocCongDan VARCHAR(20) NOT NULL UNIQUE,        -- CCCD duy nhất
    diaChi NVARCHAR(255) NULL,                         -- Địa chỉ khách hàng
    ngayTao DATETIME NOT NULL DEFAULT GETDATE()        -- Thời điểm tạo hồ sơ
);
GO
```*

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/74286224-9054-44cb-9ffb-e0e9f212fa83" />

Giải thích
Bảng này dùng để lưu toàn bộ thông tin khách hàng.

soDienThoai và canCuocCongDan được đặt UNIQUE để tránh nhập trùng dữ liệu.


6. Tạo bảng nhanVien
```sql
CREATE TABLE nhanVien (
    nhanVienId INT IDENTITY(1,1) PRIMARY KEY,          -- Khóa chính nhân viên
    hoTen NVARCHAR(100) NOT NULL,                      -- Họ tên nhân viên
    soDienThoai VARCHAR(20) NULL,                      -- Số điện thoại
    chucVu NVARCHAR(50) NULL,                          -- Chức vụ
    ngayTao DATETIME NOT NULL DEFAULT GETDATE()        -- Ngày tạo
);
GO
```

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/1630b826-0d14-4939-8fcf-8977121f9903" />


Giải thích
Bảng này dùng để lưu nhân viên thu tiền hoặc quản lý giao dịch.

7. Tạo bảng hopDong

```sql
CREATE TABLE hopDong (
    hopDongId INT IDENTITY(1,1) PRIMARY KEY,            -- Khóa chính hợp đồng
    khachHangId INT NOT NULL,                           -- Khóa ngoại tham chiếu khách hàng
    ngayLap DATE NOT NULL DEFAULT GETDATE(),            -- Ngày lập hợp đồng
    soTienGoc DECIMAL(18,2) NOT NULL,                   -- Số tiền vay gốc
    deadline1 DATE NOT NULL,                            -- Mốc bắt đầu quá hạn / lãi kép
    deadline2 DATE NOT NULL,                            -- Mốc bắt đầu xử lý thanh lý
    trangThai NVARCHAR(50) NOT NULL DEFAULT N'DangVay', -- Trạng thái hợp đồng
    ghiChu NVARCHAR(255) NULL,                          -- Ghi chú thêm
    ngayCapNhat DATETIME NOT NULL DEFAULT GETDATE(),    -- Ngày cập nhật gần nhất

    CONSTRAINT fk_hopDong_khachHang
        FOREIGN KEY (khachHangId) REFERENCES khachHang(khachHangId)
);
GO
```

<img width="1917" height="1078" alt="image" src="https://github.com/user-attachments/assets/f637ef0a-4002-43a8-97fb-562292b7ce6c" />

```sql
8. Tạo bảng taiSan

CREATE TABLE taiSan (
    taiSanId INT IDENTITY(1,1) PRIMARY KEY,                 -- Khóa chính tài sản
    tenTaiSan NVARCHAR(100) NOT NULL,                       -- Tên tài sản
    loaiTaiSan NVARCHAR(50) NULL,                           -- Loại tài sản
    giaTriDinhGia DECIMAL(18,2) NOT NULL,                   -- Giá trị định giá
    moTa NVARCHAR(255) NULL,                                -- Mô tả chi tiết
    trangThai NVARCHAR(50) NOT NULL DEFAULT N'DangCamCo',   -- Trạng thái tài sản
    ngayTao DATETIME NOT NULL DEFAULT GETDATE()             -- Ngày tạo tài sản
);
GO
9. Tạo bảng chiTietHopDongTaiSan

CREATE TABLE chiTietHopDongTaiSan (
    chiTietId INT IDENTITY(1,1) PRIMARY KEY,            -- Khóa chính dòng chi tiết
    hopDongId INT NOT NULL,                             -- Hợp đồng chứa tài sản
    taiSanId INT NOT NULL,                              -- Tài sản được cầm cố
    giaTriCamCo DECIMAL(18,2) NOT NULL,                 -- Giá trị cầm cố dùng để đảm bảo khoản vay
    daTraKhach BIT NOT NULL DEFAULT 0,                  -- Đã trả lại tài sản cho khách hay chưa
    ngayTraKhach DATE NULL,                             -- Ngày trả tài sản

    CONSTRAINT fk_chiTietHopDongTaiSan_hopDong
        FOREIGN KEY (hopDongId) REFERENCES hopDong(hopDongId),

    CONSTRAINT fk_chiTietHopDongTaiSan_taiSan
        FOREIGN KEY (taiSanId) REFERENCES taiSan(taiSanId)
);
GO
10. Tạo bảng lichSuThanhToan

CREATE TABLE lichSuThanhToan (
    thanhToanId INT IDENTITY(1,1) PRIMARY KEY,          -- Khóa chính thanh toán
    hopDongId INT NOT NULL,                             -- Hợp đồng được thanh toán
    ngayThanhToan DATETIME NOT NULL DEFAULT GETDATE(),  -- Ngày giờ thanh toán
    soTienTra DECIMAL(18,2) NOT NULL,                   -- Số tiền khách trả
    nhanVienId INT NOT NULL,                            -- Nhân viên thu tiền
    ghiChu NVARCHAR(255) NULL,                          -- Ghi chú

    CONSTRAINT fk_lichSuThanhToan_hopDong
        FOREIGN KEY (hopDongId) REFERENCES hopDong(hopDongId),

    CONSTRAINT fk_lichSuThanhToan_nhanVien
        FOREIGN KEY (nhanVienId) REFERENCES nhanVien(nhanVienId)
);
GO
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/586ce946-97b8-473a-8a8e-6867fc8c3b9e" />

Tạo các bảng thanh toán

11. Chèn dữ liệu mẫu vào bảng khachHang

```sql
INSERT INTO khachHang (hoTen, soDienThoai, canCuocCongDan, diaChi)
VALUES
(N'Nguyen Van A', '0901000001', '001001000001', N'Ha Noi'),
(N'Tran Thi B',  '0901000002', '001001000002', N'Hai Phong'),
(N'Le Van C',    '0901000003', '001001000003', N'Da Nang');
GO

INSERT INTO nhanVien (hoTen, soDienThoai, chucVu)
VALUES
(N'Pham Thu Ngan', '0912000001', N'ThuNgan'),
(N'Hoang Minh Duc', '0912000002', N'QuanLy');
GO

INSERT INTO hopDong (khachHangId, ngayLap, soTienGoc, deadline1, deadline2, trangThai, ghiChu)
VALUES
(1, '2026-05-01', 10000000, '2026-05-10', '2026-05-20', N'DangVay', N'Cam xe may'),
(2, '2026-05-02', 15000000, '2026-05-12', '2026-05-22', N'DangVay', N'Cam laptop'),
(3, '2026-05-03', 8000000,  '2026-05-13', '2026-05-23', N'DangVay', N'Cam dien thoai');
GO

INSERT INTO taiSan (tenTaiSan, loaiTaiSan, giaTriDinhGia, moTa, trangThai)
VALUES
(N'Xe may Honda Vision', N'XeMay', 18000000, N'Mau do, bien so 29A1-12345', N'DangCamCo'),
(N'Laptop Dell XPS 13', N'Laptop', 20000000, N'Core i7, RAM 16GB', N'DangCamCo'),
(N'iPhone 14 Pro Max', N'DienThoai', 22000000, N'Ban 256GB', N'DangCamCo'),
(N'Day chuyen vang 18K', N'TrangSuc', 12000000, N'Trong luong 2 chi', N'DangCamCo');
GO

INSERT INTO chiTietHopDongTaiSan (hopDongId, taiSanId, giaTriCamCo, daTraKhach, ngayTraKhach)
VALUES
(1, 1, 18000000, 0, NULL),
(2, 2, 20000000, 0, NULL),
(3, 3, 22000000, 0, NULL),
(3, 4, 12000000, 0, NULL);
GO


INSERT INTO lichSuThanhToan (hopDongId, ngayThanhToan, soTienTra, nhanVienId, ghiChu)
VALUES
(1, '2026-05-05 09:30:00', 1000000, 1, N'Khach tra lan 1'),
(1, '2026-05-08 15:00:00', 500000, 1, N'Khach tra lan 2'),
(2, '2026-05-09 10:00:00', 2000000, 2, N'Khach tra mot phan');
GO

```

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/c67184e6-78c4-4430-a38f-ee5546fa4d1b" />

Chèn dữ liệu mẫu

17. Stored Procedure đăng ký hợp đồng mới
18. 
17.1. Phân tích logic
    
Khi tạo hợp đồng mới, hệ thống cần:

Kiểm tra khách hàng đã tồn tại chưa theo canCuocCongDan

Nếu chưa có thì thêm mới khách hàng

Tạo hợp đồng mới

Tạo tài sản mới

Gắn tài sản vào hợp đồng

Phiên bản dưới đây xử lý:

1 khách hàng

1 hợp đồng

1 tài sản

Có thể mở rộng sau này để hỗ trợ nhiều tài sản trong 1 lần gọi.

17.2. Procedure

```sql 
CREATE OR ALTER PROCEDURE spDangKyHopDongMoi
    @hoTen NVARCHAR(100),
    @soDienThoai VARCHAR(20),
    @canCuocCongDan VARCHAR(20),
    @diaChi NVARCHAR(255),
    @soTienGoc DECIMAL(18,2),
    @deadline1 DATE,
    @deadline2 DATE,
    @ghiChuHopDong NVARCHAR(255),
    @tenTaiSan NVARCHAR(100),
    @loaiTaiSan NVARCHAR(50),
    @giaTriDinhGia DECIMAL(18,2),
    @moTaTaiSan NVARCHAR(255)
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @khachHangId INT;
    DECLARE @hopDongId INT;
    DECLARE @taiSanId INT;

    -- Kiểm tra khách hàng đã tồn tại theo CCCD chưa
    SELECT @khachHangId = khachHangId
    FROM khachHang
    WHERE canCuocCongDan = @canCuocCongDan;

    -- Nếu chưa tồn tại thì thêm khách hàng mới
    IF @khachHangId IS NULL
    BEGIN
        INSERT INTO khachHang (hoTen, soDienThoai, canCuocCongDan, diaChi)
        VALUES (@hoTen, @soDienThoai, @canCuocCongDan, @diaChi);

        SET @khachHangId = SCOPE_IDENTITY();
    END

    -- Tạo hợp đồng mới
    INSERT INTO hopDong (khachHangId, ngayLap, soTienGoc, deadline1, deadline2, trangThai, ghiChu)
    VALUES (@khachHangId, GETDATE(), @soTienGoc, @deadline1, @deadline2, N'DangVay', @ghiChuHopDong);

    SET @hopDongId = SCOPE_IDENTITY();

    -- Tạo tài sản mới
    INSERT INTO taiSan (tenTaiSan, loaiTaiSan, giaTriDinhGia, moTa, trangThai)
    VALUES (@tenTaiSan, @loaiTaiSan, @giaTriDinhGia, @moTaTaiSan, N'DangCamCo');

    SET @taiSanId = SCOPE_IDENTITY();

    -- Gắn tài sản vào hợp đồng
    INSERT INTO chiTietHopDongTaiSan (hopDongId, taiSanId, giaTriCamCo, daTraKhach, ngayTraKhach)
    VALUES (@hopDongId, @taiSanId, @giaTriDinhGia, 0, NULL);

    -- Trả kết quả để dễ kiểm tra
    SELECT 
        @khachHangId AS khachHangId,
        @hopDongId AS hopDongId,
        @taiSanId AS taiSanId;
END;
GO
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/2e671a25-e87f-4bcb-8005-f6af98d0863b" />

17.3. Ví dụ gọi procedure
```sql
EXEC spDangKyHopDongMoi
    @hoTen = N'Vo Thi D',
    @soDienThoai = '0901000004',
    @canCuocCongDan = '001001000004',
    @diaChi = N'Can Tho',
    @soTienGoc = 12000000,
    @deadline1 = '2026-05-18',
    @deadline2 = '2026-05-28',
    @ghiChuHopDong = N'Cam vang',
    @tenTaiSan = N'Nhan vang 24K',
    @loaiTaiSan = N'TrangSuc',
    @giaTriDinhGia = 15000000,
    @moTaTaiSan = N'Nhan 1 chi';
GO
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/4ac400e0-21b2-41cb-9d85-03e80c7c67d0" />

18. Function tính tiền phải trả đến ngày bất kỳ

18.1. Phân tích logic
    
Ta cần tính số tiền phải trả của hợp đồng tại một ngày cụ thể:

Nếu ngayTinh <= deadline1:

chỉ tính lãi đơn

Nếu ngayTinh > deadline1:

tính lãi đơn từ ngayLap đến deadline1

sau đó lấy (gốc + lãi đơn) làm cơ sở tính lãi kép

Sau cùng trừ đi:

tổng số tiền khách đã thanh toán

Công thức

lãi suất ngày = 0.005

lãi đơn:

soTienGoc * 0.005 * soNgay

lãi kép:

(coSoLaiKep * POWER(1 + 0.005, soNgaySauDeadline1))

18.2. Function

```sql
CREATE OR ALTER FUNCTION fnTinhTienHopDong
(
    @hopDongId INT,
    @ngayTinh DATE
)
RETURNS DECIMAL(18,2)
AS
BEGIN
    DECLARE @soTienGoc DECIMAL(18,2);
    DECLARE @ngayLap DATE;
    DECLARE @deadline1 DATE;
    DECLARE @tongDaTra DECIMAL(18,2) = 0;
    DECLARE @soNgayTruocDeadline1 INT = 0;
    DECLARE @soNgaySauDeadline1 INT = 0;
    DECLARE @laiSuatNgay DECIMAL(18,6) = 0.005; -- 0.5% mỗi ngày
    DECLARE @laiDon DECIMAL(18,2) = 0;
    DECLARE @tongTien DECIMAL(18,2) = 0;
    DECLARE @coSoLaiKep DECIMAL(18,2) = 0;

    -- Lấy thông tin hợp đồng
    SELECT 
        @soTienGoc = soTienGoc,
        @ngayLap = ngayLap,
        @deadline1 = deadline1
    FROM hopDong
    WHERE hopDongId = @hopDongId;

    -- Nếu không có hợp đồng thì trả về 0
    IF @soTienGoc IS NULL
        RETURN 0;

    -- Tính tổng số tiền đã thanh toán
    SELECT @tongDaTra = ISNULL(SUM(soTienTra), 0)
    FROM lichSuThanhToan
    WHERE hopDongId = @hopDongId;

    -- Nếu ngày tính trước ngày lập hợp đồng thì chưa phát sinh nợ
    IF @ngayTinh < @ngayLap
        RETURN 0;

    -- Nếu ngày tính chưa vượt deadline1 thì chỉ tính lãi đơn
    IF @ngayTinh <= @deadline1
    BEGIN
        SET @soNgayTruocDeadline1 = DATEDIFF(DAY, @ngayLap, @ngayTinh);
        SET @laiDon = @soTienGoc * @laiSuatNgay * @soNgayTruocDeadline1;
        SET @tongTien = @soTienGoc + @laiDon - @tongDaTra;

        IF @tongTien < 0
            SET @tongTien = 0;

        RETURN @tongTien;
    END

    -- Nếu ngày tính vượt deadline1 thì tính lãi đơn trước
    SET @soNgayTruocDeadline1 = DATEDIFF(DAY, @ngayLap, @deadline1);
    SET @laiDon = @soTienGoc * @laiSuatNgay * @soNgayTruocDeadline1;

    -- Cơ sở để tính lãi kép là gốc + lãi đơn đã tích lũy
    SET @coSoLaiKep = @soTienGoc + @laiDon;

    -- Tính số ngày sau deadline1
    SET @soNgaySauDeadline1 = DATEDIFF(DAY, @deadline1, @ngayTinh);

    -- Áp dụng công thức lãi kép
    SET @tongTien = (@coSoLaiKep * POWER(1 + @laiSuatNgay, @soNgaySauDeadline1)) - @tongDaTra;

    IF @tongTien < 0
        SET @tongTien = 0;

    RETURN @tongTien;
END;
GO
```
<img width="1917" height="1078" alt="image" src="https://github.com/user-attachments/assets/95ac3197-cd51-4f3c-861a-8fd38d78b918" />

18.3. Ví dụ sử dụng function

```sql
SELECT dbo.fnTinhTienHopDong(1, '2026-05-09') AS tongTienPhaiTra;
GO

SELECT dbo.fnTinhTienHopDong(1, GETDATE()) AS tongTienHienTai;
GO
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/16eb48ac-384b-4da8-8843-93f30097e4b4" />



19. Procedure xử lý trả nợ từng phần
    
19.1. Phân tích logic

Khi khách mang tiền đến trả, hệ thống xử lý theo các bước:

Kiểm tra hợp đồng có tồn tại không

Nếu hợp đồng đã DaThanhLy thì từ chối thu tiền

Tính tổng nợ hiện tại

Ghi nhận khoản thanh toán mới

Tính lại dư nợ

Nếu đã trả hết:

cập nhật hợp đồng thành DaThanhToan

trả toàn bộ tài sản

chuyển trạng thái tài sản thành DaTraKhach

Nếu chưa trả hết:

cập nhật hợp đồng thành DangTraGop

trả về danh sách tài sản có thể hoàn trả cho khách

19.2. Procedure

```sql
CREATE OR ALTER PROCEDURE spXuLyTraNoTungPhan
    @hopDongId INT,
    @soTienTra DECIMAL(18,2),
    @nhanVienId INT,
    @ghiChu NVARCHAR(255) = NULL
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @trangThaiHopDong NVARCHAR(50);
    DECLARE @tongNoHienTai DECIMAL(18,2);
    DECLARE @duNoConLai DECIMAL(18,2);

    -- Lấy trạng thái hợp đồng
    SELECT @trangThaiHopDong = trangThai
    FROM hopDong
    WHERE hopDongId = @hopDongId;

    IF @trangThaiHopDong IS NULL
    BEGIN
        RAISERROR(N'Hop dong khong ton tai.', 16, 1);
        RETURN;
    END

    -- Nếu hợp đồng đã thanh lý thì không thu tiền
    IF @trangThaiHopDong = N'DaThanhLy'
    BEGIN
        RAISERROR(N'Hop dong da thanh ly, khong thu tien va khong tra do.', 16, 1);
        RETURN;
    END

    -- Tính tổng nợ hiện tại trước khi thu tiền
    SET @tongNoHienTai = dbo.fnTinhTienHopDong(@hopDongId, CAST(GETDATE() AS DATE));

    -- Ghi log thanh toán
    INSERT INTO lichSuThanhToan (hopDongId, ngayThanhToan, soTienTra, nhanVienId, ghiChu)
    VALUES (@hopDongId, GETDATE(), @soTienTra, @nhanVienId, @ghiChu);

    -- Tính lại dư nợ còn lại sau khi vừa trả
    SET @duNoConLai = dbo.fnTinhTienHopDong(@hopDongId, CAST(GETDATE() AS DATE));

    -- Nếu đã trả hết thì cập nhật trạng thái hoàn tất
    IF @duNoConLai <= 0
    BEGIN
        UPDATE hopDong
        SET 
            trangThai = N'DaThanhToan',
            ngayCapNhat = GETDATE()
        WHERE hopDongId = @hopDongId;

        -- Đánh dấu toàn bộ tài sản đã trả khách
        UPDATE chiTietHopDongTaiSan
        SET 
            daTraKhach = 1,
            ngayTraKhach = CAST(GETDATE() AS DATE)
        WHERE hopDongId = @hopDongId
          AND daTraKhach = 0;

        -- Cập nhật trạng thái tài sản
        UPDATE ts
        SET ts.trangThai = N'DaTraKhach'
        FROM taiSan ts
        INNER JOIN chiTietHopDongTaiSan ct ON ts.taiSanId = ct.taiSanId
        WHERE ct.hopDongId = @hopDongId;

        SELECT 
            N'Khach da thanh toan het no. Da tra toan bo tai san.' AS thongBao,
            @tongNoHienTai AS tongNoTruocKhiTra,
            0 AS duNoConLai;

        RETURN;
    END

    -- Nếu chưa trả hết thì chuyển trạng thái sang đang trả góp
    UPDATE hopDong
    SET 
        trangThai = N'DangTraGop',
        ngayCapNhat = GETDATE()
    WHERE hopDongId = @hopDongId;

    -- Trả thông tin sau thanh toán
    SELECT 
        N'Khach da tra mot phan. Hop dong chuyen sang DangTraGop.' AS thongBao,
        @tongNoHienTai AS tongNoTruocKhiTra,
        @duNoConLai AS duNoConLai;

    -- Danh sách gợi ý tài sản có thể trả cho khách
    ;WITH giaTriTaiSanChuaTra AS
    (
        SELECT 
            ct.chiTietId,
            ct.hopDongId,
            ct.taiSanId,
            ts.tenTaiSan,
            ct.giaTriCamCo
        FROM chiTietHopDongTaiSan ct
        INNER JOIN taiSan ts ON ct.taiSanId = ts.taiSanId
        WHERE ct.hopDongId = @hopDongId
          AND ct.daTraKhach = 0
    ),
    tongGiaTri AS
    (
        SELECT SUM(giaTriCamCo) AS tongGiaTriConGiu
        FROM giaTriTaiSanChuaTra
    )
    SELECT 
        g.chiTietId,
        g.taiSanId,
        g.tenTaiSan,
        g.giaTriCamCo,
        (t.tongGiaTriConGiu - g.giaTriCamCo) AS giaTriConLaiNeuTraTaiSanNay,
        @duNoConLai AS duNoConLai
    FROM giaTriTaiSanChuaTra g
    CROSS JOIN tongGiaTri t
    WHERE (t.tongGiaTriConGiu - g.giaTriCamCo) >= @duNoConLai;
END;
GO
```

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/c382c93f-f5a0-4985-9c85-4beb7bbcc9c7" />

19.3. Ví dụ gọi procedure

EXEC spXuLyTraNoTungPhan
    @hopDongId = 1,
    @soTienTra = 2000000,
    @nhanVienId = 1,
    @ghiChu = N'Khach tra them dot tiep theo';
GO

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/7a4c4d6b-423e-48e6-b26a-627c3bea2e16" />
Gọi spXuLyTraNoTungPhan

20. Query danh sách nợ xấu
    
20.1. Phân tích logic

Nợ xấu là các hợp đồng:

đã quá deadline1

vẫn còn tiền phải trả

chưa ở trạng thái DaThanhToan

chưa ở trạng thái DaThanhLy

20.2. Query
```sql
SELECT
    kh.hoTen AS tenKhachHang,
    kh.soDienThoai,
    hd.soTienGoc,
    DATEDIFF(DAY, hd.deadline1, CAST(GETDATE() AS DATE)) AS soNgayQuaHan,
    dbo.fnTinhTienHopDong(hd.hopDongId, CAST(GETDATE() AS DATE)) AS tongTienPhaiTraHienTai,
    dbo.fnTinhTienHopDong(hd.hopDongId, DATEADD(MONTH, 1, CAST(GETDATE() AS DATE))) AS tongTienPhaiTraSau1Thang
FROM hopDong hd
INNER JOIN khachHang kh ON hd.khachHangId = kh.khachHangId
WHERE CAST(GETDATE() AS DATE) > hd.deadline1
  AND dbo.fnTinhTienHopDong(hd.hopDongId, CAST(GETDATE() AS DATE)) > 0
  AND hd.trangThai NOT IN (N'DaThanhToan', N'DaThanhLy');
GO
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/16f6a1c1-3804-4d52-a123-eaffc8e5bc97" />
Danh sách nợ xấu
21. Trigger cập nhật trạng thái quá hạn và thanh lý

21.1. Lưu ý

Trong SQL Server, trigger không tự chạy chỉ vì thời gian trôi qua.

Trigger chỉ chạy khi có thao tác:

INSERT
UPDATE
DELETE
Vì vậy trong bài này, trigger được thiết kế để:

mỗi khi có thao tác thêm/sửa trên bảng hopDong

hệ thống sẽ kiểm tra và cập nhật trạng thái phù hợp

Nếu muốn hoàn toàn tự động theo giờ/ngày trong thực tế, nên dùng thêm SQL Server Agent Job.

21.2. Trigger cập nhật hợp đồng sang QuaHan

```sql
CREATE OR ALTER TRIGGER trgCapNhatHopDongQuaHan
ON hopDong
AFTER INSERT, UPDATE
AS
BEGIN
    SET NOCOUNT ON;

    -- Nếu đã vượt deadline1 và hợp đồng còn đang vay hoặc đang trả góp
    UPDATE hd
    SET 
        hd.trangThai = N'QuaHan',
        hd.ngayCapNhat = GETDATE()
    FROM hopDong hd
    INNER JOIN inserted i ON hd.hopDongId = i.hopDongId
    WHERE CAST(GETDATE() AS DATE) > hd.deadline1
      AND hd.trangThai IN (N'DangVay', N'DangTraGop');
END;
GO
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/22b63ece-9c69-42ce-95ce-43cdc8ffb092" />
Trigger cập nhật hợp đồng sang QuaHan

21.3. Trigger cập nhật tài sản sang SanSangThanhLy

```sql
CREATE OR ALTER TRIGGER trgCapNhatTaiSanSanSangThanhLy
ON hopDong
AFTER INSERT, UPDATE
AS
BEGIN
    SET NOCOUNT ON;

    -- Nếu hợp đồng đã quá hạn và vượt deadline2 thì tài sản sẵn sàng thanh lý
    UPDATE ts
    SET ts.trangThai = N'SanSangThanhLy'
    FROM taiSan ts
    INNER JOIN chiTietHopDongTaiSan ct ON ts.taiSanId = ct.taiSanId
    INNER JOIN hopDong hd ON ct.hopDongId = hd.hopDongId
    INNER JOIN inserted i ON hd.hopDongId = i.hopDongId
    WHERE hd.trangThai = N'QuaHan'
      AND CAST(GETDATE() AS DATE) > hd.deadline2
      AND ts.trangThai = N'DangCamCo';
END;
GO
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/4c72f9aa-40e6-49b3-9800-4e100eeee9e8" />
Trigger cập nhật tài sản sang SanSangThanhLy

21.4. Trigger cập nhật tài sản sang DaBanThanhLy

```sql
CREATE OR ALTER TRIGGER trgCapNhatTaiSanDaBanThanhLy
ON hopDong
AFTER UPDATE
AS
BEGIN
    SET NOCOUNT ON;

    -- Khi hợp đồng vừa chuyển sang DaThanhLy thì toàn bộ tài sản thành DaBanThanhLy
    UPDATE ts
    SET ts.trangThai = N'DaBanThanhLy'
    FROM taiSan ts
    INNER JOIN chiTietHopDongTaiSan ct ON ts.taiSanId = ct.taiSanId
    INNER JOIN inserted i ON ct.hopDongId = i.hopDongId
    INNER JOIN deleted d ON i.hopDongId = d.hopDongId
    WHERE i.trangThai = N'DaThanhLy'
      AND d.trangThai <> N'DaThanhLy';
END;
GO
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/e854119a-7c62-4011-86c4-80c57d7d7507" />

Trigger cập nhật tài sản sang DaBanThanhLy

### Xử lý gia hạn hợp đồng

#### Ý tưởng xử lý

Gia hạn hợp đồng là tình huống khách hàng không trả gốc, mà chỉ thanh toán **toàn bộ tiền lãi đang phát sinh đến thời điểm hiện tại** để dời kỳ hạn mới.

Cách xử lý gồm các bước:

- lấy thông tin hợp đồng
- tính tổng nợ hiện tại
- xác định phần lãi phải trả = tổng nợ hiện tại - số tiền gốc còn lại
- nếu khách thanh toán đủ phần lãi:
  - ghi nhận thanh toán vào `lichSuThanhToan`
  - cập nhật `deadline1` mới
  - cập nhật `deadline2` mới
  - chuyển trạng thái hợp đồng về `DangVay` nếu trước đó chưa bị thanh lý
- nếu không đủ:
  - từ chối gia hạn

> Ghi chú: Trong phiên bản hiện tại, phần gốc còn lại được xác định bằng `soTienGoc - tongSoTienDaTraVaoGoc` theo hướng đơn giản hóa. Vì hệ thống đang ưu tiên trừ toàn bộ thanh toán vào tổng nợ, nên khi áp dụng thực tế có thể cần bóc tách rõ phần tiền nào trả vào lãi, phần nào trả vào gốc.

#### Procedure gia hạn hợp đồng

```sql
CREATE OR ALTER PROCEDURE spGiaHanHopDong
    @hopDongId INT,
    @soNgayGiaHanDeadline1 INT,
    @soNgayGiaHanDeadline2 INT,
    @soTienKhachTra DECIMAL(18,2),
    @nhanVienId INT,
    @ghiChu NVARCHAR(255) = NULL
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @soTienGoc DECIMAL(18,2);
    DECLARE @tongNoHienTai DECIMAL(18,2);
    DECLARE @tongDaTra DECIMAL(18,2);
    DECLARE @laiPhaiTra DECIMAL(18,2);
    DECLARE @deadline1Moi DATE;
    DECLARE @deadline2Moi DATE;
    DECLARE @trangThaiHopDong NVARCHAR(50);

    -- Lấy thông tin hợp đồng
    SELECT
        @soTienGoc = soTienGoc,
        @trangThaiHopDong = trangThai
    FROM hopDong
    WHERE hopDongId = @hopDongId;

    IF @soTienGoc IS NULL
    BEGIN
        RAISERROR(N'Hop dong khong ton tai.', 16, 1);
        RETURN;
    END

    IF @trangThaiHopDong = N'DaThanhLy'
    BEGIN
        RAISERROR(N'Hop dong da thanh ly, khong the gia han.', 16, 1);
        RETURN;
    END

    -- Tính tổng nợ hiện tại
    SET @tongNoHienTai = dbo.fnTinhTienHopDong(@hopDongId, CAST(GETDATE() AS DATE));

    -- Tính tổng tiền đã trả
    SELECT @tongDaTra = ISNULL(SUM(soTienTra), 0)
    FROM lichSuThanhToan
    WHERE hopDongId = @hopDongId;

    -- Tạm xác định phần lãi phải trả để được gia hạn
    -- Ở mức bài tập, xem lãi = tổng nợ hiện tại - gốc
    SET @laiPhaiTra = @tongNoHienTai - @soTienGoc;

    IF @laiPhaiTra < 0
        SET @laiPhaiTra = 0;

    -- Nếu khách trả chưa đủ lãi thì không cho gia hạn
    IF @soTienKhachTra < @laiPhaiTra
    BEGIN
        RAISERROR(N'Khach chua thanh toan du tien lai de gia han hop dong.', 16, 1);
        RETURN;
    END

    -- Ghi nhận thanh toán phục vụ audit log
    INSERT INTO lichSuThanhToan (hopDongId, ngayThanhToan, soTienTra, nhanVienId, ghiChu)
    VALUES (@hopDongId, GETDATE(), @soTienKhachTra, @nhanVienId, ISNULL(@ghiChu, N'Gia han hop dong'));

    -- Tính deadline mới kể từ ngày hiện tại
    SET @deadline1Moi = DATEADD(DAY, @soNgayGiaHanDeadline1, CAST(GETDATE() AS DATE));
    SET @deadline2Moi = DATEADD(DAY, @soNgayGiaHanDeadline2, CAST(GETDATE() AS DATE));

    -- Cập nhật hợp đồng sau khi gia hạn
    UPDATE hopDong
    SET
        deadline1 = @deadline1Moi,
        deadline2 = @deadline2Moi,
        trangThai = N'DangVay',
        ngayCapNhat = GETDATE(),
        ghiChu = ISNULL(ghiChu, N'') + N' | Gia han hop dong'
    WHERE hopDongId = @hopDongId;

    SELECT
        N'Gia han hop dong thanh cong.' AS thongBao,
        @laiPhaiTra AS soTienLaiBatBuoc,
        @soTienKhachTra AS soTienKhachDaTra,
        @deadline1Moi AS deadline1Moi,
        @deadline2Moi AS deadline2Moi;
END;
GO
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/3d31b6e9-5909-4b73-90fd-7bfc748c30a1" />
Tạo sp Gia han hop dong

Ví dụ chạy thử

```sql
EXEC spGiaHanHopDong
    @hopDongId = 1,
    @soNgayGiaHanDeadline1 = 10,
    @soNgayGiaHanDeadline2 = 20,
    @soTienKhachTra = 450000,
    @nhanVienId = 1,
    @ghiChu = N'Khach xin gia han them ky moi';
GO
```

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/21ef371b-17a0-4467-86f2-a7c61c3ccda4" />

###  Lịch sử hợp đồng (Audit Log) 

#### Vấn đề nghiệp vụ

Trong một số thiết kế CSDL đơn giản, người ta có xu hướng:
- thêm một cột `soNoConLai` vào bảng `hopDong`
- mỗi lần khách trả tiền thì UPDATE trực tiếp cột này

Cách làm này **rất nguy hiểm** vì:
- mất toàn bộ lịch sử giao dịch
- không biết khách đã trả bao nhiêu lần, khi nào, ai thu
- không thể đối soát nếu có tranh chấp
- không thể tính lại công nợ tại bất kỳ thời điểm nào trong quá khứ

#### Giải pháp: Audit Log

Hệ thống này sử dụng **bảng `lichSuThanhToan` như một bảng audit log đầy đủ**.  
Mỗi lần khách trả tiền, hệ thống **không cập nhật cột nào cả**, mà chỉ **thêm một dòng mới** vào bảng này.

Bảng `lichSuThanhToan` lưu:
- `hopDongId` - tham chiếu đến hợp đồng
- `ngayThanhToan` - thời điểm thu tiền (có giờ phút)
- `soTienTra` - số tiền khách trả trong lần này
- `nhanVienId` - người trực tiếp thu tiền (có thể kiểm tra trách nhiệm)
- `ghiChu` - ghi chú nghiệp vụ (ví dụ: "Trả lãi tháng 5", "Trả góp lần 3")

#### Tại sao Audit Log lại quan trọng?

**1. Truy xuất lịch sử đầy đủ**

```sql
SELECT 
    lt.thanhToanId,
    lt.ngayThanhToan,
    lt.soTienTra,
    nv.hoTen AS nguoiThu,
    lt.ghiChu
FROM lichSuThanhToan lt
INNER JOIN nhanVien nv ON lt.nhanVienId = nv.nhanVienId
WHERE lt.hopDongId = 1
ORDER BY lt.ngayThanhToan DESC;
```

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/4e1c6143-793e-4460-8dd3-e153bc100591" />

Kết quả: thấy rõ khách đã trả mấy lần, khi nào, ai thu, bao nhiêu tiền.

2. Tính tổng tiền đã trả bất kỳ lúc nào


SELECT 
    SUM(soTienTra) AS tongDaTra
FROM lichSuThanhToan
WHERE hopDongId = 1;

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/887beefc-64b9-433a-8d69-7b14919a592e" />

