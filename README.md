HỆ THỐNG THÔNG TIN QUẢN LÝ HOẠT ĐỘNG CẦM ĐỒ

Học phần	Hệ Quản trị Cơ sở Dữ liệu
Lớp	59KMT
Giảng viên:	Đỗ Duy Cốp
Họ và tên: Nguyễn Hữu Doan
MSSV:	K235480106022
1. Mở đầu:
   
Quản lý tiệm cầm đồ là bài toán điển hình về sự đan xen giữa tài sản và dòng tiền biến thiên theo thời gian. Điều khiến bài toán này không tầm thường chính là cơ chế lãi suất lai ghép – lãi đơn trước một cột mốc, lãi kép sau cột mốc đó – và sự tồn tại song song của nhiều vòng đời: vòng đời hợp đồng, vòng đời từng tài sản, và dòng chảy tài chính được ghi nhận theo từng lần khách đến trả.

### Trước khi bắt tay vào lập trình, chúng tôi đã dành thời gian phân tích kỹ bốn rủi ro nghiệp vụ phổ biến nhất trong loại hình này:

- Thất thoát tài sản thế chấp – trả đồ khi chưa đủ điều kiện.

- Sai lệch công nợ – tính thiếu ngày lãi kép hoặc bỏ sót lần trả.

- Lạm dụng quyền nhân viên – sửa số liệu để che giấu thất thoát.

- Tranh chấp khi thanh lý – không có bằng chứng về lịch sử xử lý.

Toàn bộ thiết kế dưới đây được định hình để chống lại bốn rủi ro ấy. Hệ thống không chỉ lưu trữ dữ liệu, mà còn tự động áp đặt các ràng buộc, tự động chuyển trạng thái, và ghi nhận mọi thay đổi tài chính vào một sổ cái chỉ được phép nối thêm (append-only), không bao giờ ghi đè.

2. Kiến trúc dữ liệu: từ mô hình thực thể đến các bảng đã chuẩn hóa
2.1. Sơ đồ quan hệ thực thể (ERD)
Sơ đồ bên dưới mô tả 5 thực thể nền tảng và các liên kết giữa chúng. Điểm mấu chốt là HopDong và TaiSan liên kết với nhau qua một bảng cầu HopDong_TaiSan – nhờ đó một hợp đồng có thể bao gồm nhiều món đồ, và một món đồ có thể xuất hiện trong các hợp đồng khác nhau ở những thời điểm khác nhau.

https://github.com/user-attachments/assets/96ccc4c8-c6bd-4a44-8dc3-652789189823

2.2. Danh sách các bảng và trách nhiệm

- KhachHang – định danh người vay. Mỗi khách hàng được gán một mã duy nhất; số CCCD/CMND có ràng buộc UNIQUE để tránh trùng lặp hồ sơ.

- NhanVien – nhân viên tiệm, sẽ được gắn vào từng giao dịch để truy vết trách nhiệm.

- TaiSan – kho tài sản. Mỗi tài sản mang một trạng thái trong bốn giá trị: đang cầm, đã trả, sẵn sàng thanh lý, hoặc đã bán.

- HopDong – trái tim của hệ thống. Nó chứa số tiền vay gốc, tổng tiền đã thu hồi, hai mốc deadline, và trạng thái hợp đồng. Các cột trạng thái bị giới hạn bởi CHECK constraint để dữ liệu không thể nhận giá trị ngoài tập hợp đã định.

- HopDong_TaiSan – bảng trung gian phân rã quan hệ nhiều-nhiều, với khóa chính kép ngăn chặn một tài sản bị gán hai lần cho cùng một hợp đồng.

- LichSuGiaoDich – nhật ký tài chính, ghi lại số tiền trả, số dư nợ trước và sau giao dịch, cùng với nhân viên thực hiện. Việc tách riêng bảng này thay vì chỉ dùng một cột TongTienDaTra trong HopDong chính là hàng rào bảo vệ quan trọng nhất: mọi đồng tiền ra vào đều có dấu vết, không thể chỉnh sửa mà không để lại chứng cứ.

3. Tạo bảng và dữ liệu mẫu

```sql
-- ==========================================================
-- Khởi tạo cơ sở dữ liệu
-- ==========================================================
IF DB_ID('K235480106022_QuanLyCamDo') IS NULL
    CREATE DATABASE K235480106022_QuanLyCamDo;
GO
USE K235480106022_QuanLyCamDo;
GO

```

```sql
-- ==========================================================
-- Bảng KhachHang
-- ==========================================================
CREATE TABLE KhachHang (
    KhachHangID     INT             NOT NULL IDENTITY(1,1),
    HoTen           NVARCHAR(100)   NOT NULL,
    CMND_CCCD       NVARCHAR(20)    NOT NULL,
    SoDienThoai     NVARCHAR(15)    NOT NULL,
    DiaChi          NVARCHAR(255)   NULL,
    NgayTao         DATETIME        NOT NULL DEFAULT GETDATE(),
    CONSTRAINT PK_KhachHang PRIMARY KEY (KhachHangID),
    CONSTRAINT UQ_CMND UNIQUE (CMND_CCCD)
);

-- ==========================================================
-- Bảng NhanVien
-- ==========================================================
CREATE TABLE NhanVien (
    NhanVienID      INT             NOT NULL IDENTITY(1,1),
    HoTen           NVARCHAR(100)   NOT NULL,
    SoDienThoai     NVARCHAR(15)    NULL,
    ChucVu          NVARCHAR(50)    NULL,
    CONSTRAINT PK_NhanVien PRIMARY KEY (NhanVienID)
);

-- ==========================================================
-- Bảng TaiSan
-- ==========================================================
CREATE TABLE TaiSan (
    TaiSanID        INT             NOT NULL IDENTITY(1,1),
    TenTaiSan       NVARCHAR(200)   NOT NULL,
    LoaiTaiSan      NVARCHAR(100)   NULL,
    GiaTriDinhGia   DECIMAL(18,0)   NOT NULL,
    TrangThai       NVARCHAR(25)    NOT NULL DEFAULT N'Dang cam co',
    CONSTRAINT PK_TaiSan PRIMARY KEY (TaiSanID),
    CONSTRAINT CK_TaiSan_Status CHECK (TrangThai IN (
        N'Dang cam co', N'Da tra khach',
        N'San sang thanh ly', N'Da ban thanh ly'
    ))
);

-- ==========================================================
-- Bảng HopDong
-- ==========================================================
CREATE TABLE HopDong (
    HopDongID       INT             NOT NULL IDENTITY(1,1),
    KhachHangID     INT             NOT NULL,
    NhanVienID      INT             NOT NULL,
    NgayVay         DATE            NOT NULL,
    SoTienVayGoc    DECIMAL(18,0)   NOT NULL,
    SoTienDaTra     DECIMAL(18,0)   NOT NULL DEFAULT 0,
    Deadline1       DATE            NOT NULL,
    Deadline2       DATE            NOT NULL,
    TrangThai       NVARCHAR(20)    NOT NULL DEFAULT N'Dang vay',
    CONSTRAINT PK_HopDong PRIMARY KEY (HopDongID),
    CONSTRAINT FK_HD_Khach FOREIGN KEY (KhachHangID)
        REFERENCES KhachHang(KhachHangID),
    CONSTRAINT FK_HD_NV FOREIGN KEY (NhanVienID)
        REFERENCES NhanVien(NhanVienID),
    CONSTRAINT CK_HD_Status CHECK (TrangThai IN (
        N'Dang vay', N'Qua han', N'Dang tra gop',
        N'Da thanh toan', N'Da thanh ly'
    ))
);

-- ==========================================================
-- Bảng trung gian HopDong_TaiSan
-- ==========================================================
CREATE TABLE HopDong_TaiSan (
    HopDongID       INT NOT NULL,
    TaiSanID        INT NOT NULL,
    CONSTRAINT PK_HD_TS PRIMARY KEY (HopDongID, TaiSanID),
    CONSTRAINT FK_HDTS_HD FOREIGN KEY (HopDongID)
        REFERENCES HopDong(HopDongID),
    CONSTRAINT FK_HDTS_TS FOREIGN KEY (TaiSanID)
        REFERENCES TaiSan(TaiSanID)
);

-- ==========================================================
-- Bảng nhật ký giao dịch LichSuGiaoDich
-- ==========================================================
CREATE TABLE LichSuGiaoDich (
    GiaoDichID          INT             NOT NULL IDENTITY(1,1),
    HopDongID           INT             NOT NULL,
    NhanVienID          INT             NOT NULL,
    NgayGiaoDich        DATETIME        NOT NULL DEFAULT GETDATE(),
    SoTienTra           DECIMAL(18,0)   NOT NULL,
    DuNoTruocKhiTra     DECIMAL(18,0)   NOT NULL,
    DuNoSauKhiTra       DECIMAL(18,0)   NOT NULL,
    LoaiGiaoDich        NVARCHAR(50)    NULL,
    GhiChu              NVARCHAR(500)   NULL,
    CONSTRAINT PK_LSGD PRIMARY KEY (GiaoDichID),
    CONSTRAINT FK_LSGD_HD FOREIGN KEY (HopDongID)
        REFERENCES HopDong(HopDongID),
    CONSTRAINT FK_LSGD_NV FOREIGN KEY (NhanVienID)
        REFERENCES NhanVien(NhanVienID)
);
GO
```

Dữ liệu mẫu được nạp sẵn để phục vụ kiểm thử:

```sql
-- Khách hàng
INSERT INTO KhachHang (HoTen, CMND_CCCD, SoDienThoai, DiaChi) VALUES
(N'Nguyễn Văn An', N'001203456789', N'0912345678', N'Hà Nội'),
(N'Trần Thị Bình', N'001203456790', N'0988123456', N'Hải Phòng'),
(N'Lê Minh Cường', N'001203456791', N'0977666555', N'Đà Nẵng'),
(N'Phạm Thu Dung', N'001203456792', N'0966888999', N'TP Hồ Chí Minh'),
(N'Hoàng Quốc Em', N'001203456793', N'0933555777', N'Cần Thơ');

-- Nhân viên
INSERT INTO NhanVien (HoTen, SoDienThoai, ChucVu) VALUES
(N'Nguyễn Thành Công', N'0909000001', N'Quản lý'),
(N'Trần Mỹ Linh',      N'0909000002', N'Thu ngân'),
(N'Phạm Quốc Bảo',     N'0909000003', N'Nhân viên');

-- Tài sản
INSERT INTO TaiSan (TenTaiSan, LoaiTaiSan, GiaTriDinhGia) VALUES
(N'iPhone 15 Pro Max', N'Điện thoại', 30000000),
(N'Xe máy Honda SH',   N'Xe máy',     85000000),
(N'Laptop Dell XPS',   N'Laptop',     25000000),
(N'Nhẫn vàng 24K',     N'Vàng',       15000000),
(N'MacBook Pro M3',    N'Laptop',     45000000);

-- Hợp đồng (để ở nhiều trạng thái khác nhau phục vụ test)
INSERT INTO HopDong (KhachHangID, NhanVienID, NgayVay, SoTienVayGoc, SoTienDaTra, Deadline1, Deadline2, TrangThai)
VALUES
(1, 1, '2026-05-01', 20000000, 5000000,  '2026-06-01', '2026-07-01', N'Dang tra gop'),
(2, 2, '2026-05-03', 50000000, 0,        '2026-06-03', '2026-07-03', N'Dang vay'),
(3, 2, '2026-04-15', 15000000, 15000000, '2026-05-15', '2026-06-15', N'Da thanh toan'),
(4, 3, '2026-03-10', 10000000, 2000000,  '2026-04-10', '2026-05-10', N'Qua han'),
(5, 1, '2026-05-08', 30000000, 0,        '2026-06-08', '2026-07-08', N'Dang vay');

-- Gán tài sản vào hợp đồng
INSERT INTO HopDong_TaiSan (HopDongID, TaiSanID) VALUES
(1,1), (2,2), (3,4), (4,3), (5,5);

-- Nhật ký giao dịch ban đầu
INSERT INTO LichSuGiaoDich (HopDongID, NhanVienID, NgayGiaoDich, SoTienTra, DuNoTruocKhiTra, DuNoSauKhiTra, LoaiGiaoDich, GhiChu) VALUES
(1, 2, GETDATE(), 3000000, 20000000, 17000000, N'Tra no', N'Khách trả lần 1'),
(1, 2, GETDATE(), 2000000, 17000000, 15000000, N'Tra no', N'Khách trả lần 2'),
(3, 1, GETDATE(), 15000000,15000000, 0,        N'Tra no', N'Tất toán hợp đồng'),
(4, 3, GETDATE(), 2000000, 10000000, 8000000,  N'Tra no', N'Trả một phần'),
(2, 2, GETDATE(), 0,        50000000,50000000, N'Gia han', N'Gia hạn thêm 30 ngày');
```


4. Các khối xử lý nghiệp vụ chính

4.1. Tiếp nhận hợp đồng cầm mới
   
Thủ tục sp_TiepNhanHopDong chịu trách nhiệm tạo mới một hợp đồng, liên kết các tài sản và thiết lập hai cột mốc deadline. Thiết kế của thủ tục này ưu tiên tính an toàn: trước khi ghi bất kỳ dòng nào vào HopDong, hệ thống kiểm tra tổng giá trị tài sản thế chấp có đủ bảo đảm cho khoản vay hay không. Nếu không đủ, toàn bộ giao dịch bị hủy ngay, tránh làm phình số ID tự tăng một cách vô ích.

```sql
CREATE PROCEDURE sp_TiepNhanHopDong
    @customerId         INT,
    @staffId            INT,
    @loanAmount         DECIMAL(18,0),
    @loanDate           DATE,
    @daysToDeadline1    INT,
    @daysFromD1ToD2     INT,
    @assetIdList        NVARCHAR(MAX)       -- chuỗi "1,2,3"
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @newContractId   INT;
    DECLARE @dl1             DATE = DATEADD(DAY, @daysToDeadline1, @loanDate);
    DECLARE @dl2             DATE = DATEADD(DAY, @daysFromD1ToD2, @dl1);
    DECLARE @totalAssetValue DECIMAL(18,0);

    IF @loanAmount <= 0
    BEGIN
        THROW 50001, N'Số tiền vay phải lớn hơn 0.', 1;
        RETURN;
    END

    BEGIN TRY
        BEGIN TRANSACTION;

        -- Tính tổng giá trị tài sản trước khi tạo hợp đồng
        SELECT @totalAssetValue = ISNULL(SUM(ts.GiaTriDinhGia), 0)
        FROM TaiSan ts
        WHERE ts.TaiSanID IN (
            SELECT CAST(TRIM(val) AS INT)
            FROM STRING_SPLIT(@assetIdList, ',')
            WHERE TRIM(val) <> ''
        );

        IF @totalAssetValue < @loanAmount
        BEGIN
            THROW 50002,
                N'Tổng giá trị tài sản không đủ đảm bảo khoản vay.',
                1;
        END

        -- Tạo bản ghi hợp đồng
        INSERT INTO HopDong (
            KhachHangID, NhanVienID, NgayVay,
            SoTienVayGoc, SoTienDaTra,
            Deadline1, Deadline2, TrangThai
        ) VALUES (
            @customerId, @staffId, @loanDate,
            @loanAmount, 0,
            @dl1, @dl2, N'Dang vay'
        );

        SET @newContractId = SCOPE_IDENTITY();

        -- Liên kết tài sản vào hợp đồng
        INSERT INTO HopDong_TaiSan (HopDongID, TaiSanID)
        SELECT @newContractId, CAST(TRIM(val) AS INT)
        FROM STRING_SPLIT(@assetIdList, ',')
        WHERE TRIM(val) <> '';

        -- Đánh dấu tài sản đang được cầm
        UPDATE TaiSan
        SET TrangThai = N'Dang cam co'
        WHERE TaiSanID IN (
            SELECT CAST(TRIM(val) AS INT)
            FROM STRING_SPLIT(@assetIdList, ',')
            WHERE TRIM(val) <> ''
        );

        COMMIT TRANSACTION;

        SELECT
            @newContractId AS HopDongID,
            @loanAmount    AS SoTienVay,
            @dl1           AS Deadline1,
            @dl2           AS Deadline2,
            @totalAssetValue AS TongGiaTriTaiSan,
            N'Tạo hợp đồng thành công' AS ThongBao;
    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0 ROLLBACK TRANSACTION;
        THROW;
    END CATCH
END;
GO
```
