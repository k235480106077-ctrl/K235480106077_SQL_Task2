# BT2-QTCSDL

## Phần mở đầu
Thông tin cá nhân
- Tên :  Nguyễn Văn Tuyến
- Lớp : K59KMT.K01
- MSSV : K235480106077
- Chủ đề : Quản lý khách sạn

## Phần 1 : Thiết kế và Khởi tạo Cấu trúc Dữ liệu
- Tên db : QuanLyKhachSan_K235480106077
<img width="1908" height="1079" alt="Screenshot 2026-05-03 144801" src="https://github.com/user-attachments/assets/8ddf3cdf-fab9-4819-8b96-5ba6e6b5e1cd" />

*Ảnh này cho thấy em đã tạo thành công db với tên QuanLyKhachSan_K235480106077
Hệ thống được thiết kế để quản lý quy trình vận hành khách sạn từ khâu phân loại phòng đến việc đặt phòng của khách hàng.

Loại phòng: Định nghĩa giá tiền và đặc điểm của từng loại (Vip, Thường, Đơn, Đôi).

Phòng: Quản lý chi tiết từng phòng cụ thể và trạng thái hiện tại (Trống/Có khách).

Phiếu đặt phòng: Ghi nhận lịch sử giao dịch, số ngày ở và tính toán doanh thu.

1.2. Cấu trúc bảng và Các ràng buộc (Constraints)
Em đã áp dụng các kỹ thuật quản trị dữ liệu chuyên sâu:

Primary Key: MaLoai, MaPhong, MaPhieu để đảm bảo định danh không trùng lặp.

Foreign Key: Ràng buộc MaLoai trong bảng Phong và MaPhong trong PhieuDatPhong để đảm bảo tính nhất quán dữ liệu.

Check Constraint: * DonGia >= 0: Ngăn chặn lỗi nhập giá tiền âm.

SoNgayO > 0: Đảm bảo phiếu đặt phòng luôn có thời gian lưu trú hợp lý.

Default Value: TrangThai mặc định là N'Trống' khi mới thêm phòng vào hệ thống.

Hình ảnh minh họa cấu trúc Database:
<img width="1919" height="1079" alt="Screenshot 2026-05-03 145037" src="https://github.com/user-attachments/assets/ee5dd7f7-136e-4be5-9a56-8b3a6f806390" />
```sql
Tạo Database
CREATE DATABASE [QuanLyKhachSan_K235480106077];
GO
USE [QuanLyKhachSan_K235480106077];
GO

Tạo bảng Loại Phòng
CREATE TABLE [LoaiPhong] (
    [MaLoai] INT PRIMARY KEY IDENTITY(1,1),
    [TenLoai] NVARCHAR(50) NOT NULL,
    [DonGia] MONEY CHECK ([DonGia] >= 0)
);

Tạo bảng Phòng
CREATE TABLE [Phong] (
    [MaPhong] VARCHAR(10) PRIMARY KEY,
    [TenPhong] NVARCHAR(50),
    [MaLoai] INT FOREIGN KEY REFERENCES [LoaiPhong]([MaLoai]),
    [TrangThai] NVARCHAR(20) DEFAULT N'Trống'
);

-- Tạo bảng Phiếu Đặt Phòng
CREATE TABLE [PhieuDatPhong] (
    [MaPhieu] INT PRIMARY KEY IDENTITY(1,1),
    [MaPhong] VARCHAR(10) FOREIGN KEY REFERENCES [Phong]([MaPhong]),
    [NgayDat] DATE DEFAULT GETDATE(),
    [SoNgayO] INT CHECK ([SoNgayO] > 0),
    [TongTien] MONEY
);
```
PHẦN 2: XÂY DỰNG FUNCTION (HÀM)
1. Các loại hàm Built-in phổ biến
SQL Server chia hàm có sẵn thành các nhóm chính:

Hàm chuỗi (String Functions): LEN, LEFT, RIGHT, REPLACE, UPPER, LOWER... xử lý văn bản.

Hàm ngày tháng (Date and Time): GETDATE(), DATEPART, DATEDIFF, EOMONTH... quản lý thời gian.

Hàm toán học (Mathematical): ABS, ROUND, CEILING, FLOOR... tính toán số học.

Hàm hệ thống (System Functions): ISNULL, CAST, CONVERT, COALESCE... tương tác với cấu trúc hệ thống và xử lý logic dữ liệu.

2. Các System Function "đặc sắc" & Cách khai thác
Thay vì các hàm cơ bản, đây là 3 hàm cực kỳ mạnh mẽ giúp code của bạn chuyên nghiệp hơn:

A. Hàm COALESCE – "Kẻ cứu rỗi" dữ liệu NULL
Thay vì dùng ISNULL chỉ kiểm tra được 1 giá trị, COALESCE cho phép bạn truyền vào một danh sách. nó sẽ trả về giá trị không NULL đầu tiên mà nó tìm thấy.

Khai thác: Rất hữu ích khi bạn có nhiều cột thông tin liên hệ (Email, Số điện thoại, Địa chỉ) và muốn lấy bất cứ thông tin nào có sẵn để gửi thông báo.

SQL
-- Lấy thông tin liên lạc ưu tiên: Email -> Điện thoại -> 'Chưa có thông tin'
SELECT [MaHoiVien], 
       COALESCE([Email], [SoDienThoai], N'Chưa có liên hệ') AS [LienHe]
FROM [HoiVien];
B. Hàm EOMONTH – Tìm ngày cuối tháng cực nhanh
Việc tính ngày cuối cùng của một tháng (28, 29, 30 hay 31) thường rất phiền phức. EOMONTH (End of Month) giải quyết việc này chỉ trong 1 dòng code.

Khai thác: Dùng để tính ngày hết hạn gói tập hoặc ngày chốt lương cuối tháng.

SQL
-- Tính ngày cuối cùng của tháng hiện tại
SELECT EOMONTH(GETDATE()) AS [NgayChotSo];

-- Tính ngày cuối cùng của tháng sau (số 1 là cộng thêm 1 tháng)
SELECT EOMONTH(GETDATE(), 1) AS [HetHanGoiTapThangSau];
C. Hàm FORMAT – "Phù thủy" hiển thị dữ liệu
Mặc dù CONVERT có thể định dạng ngày tháng, nhưng FORMAT mang lại sự linh hoạt tuyệt đối như trong ngôn ngữ C# hoặc Java.

Khai thác: Định dạng tiền tệ hoặc ngày tháng theo chuẩn Việt Nam một cách đẹp mắt ngay trong kết quả truy vấn.

SQL
-- Định dạng tiền tệ theo chuẩn VNĐ (đ) và ngày tháng kiểu VN (dd/MM/yyyy)
SELECT [MaPhieu], 
       FORMAT([TongTien], 'C', 'vi-VN') AS [SoTienDep],
       FORMAT([NgayDat], 'dd/MM/yyyy') AS [NgayVietNam]
FROM [PhieuDatPhong];
3. Trình bày vào báo cáo GitHub (Vùng cốt riêng)
Để bài làm của bạn "xịn" như yêu cầu, hãy đưa phần này vào Phần 2 (Function) trong file README.md bằng cách bọc chúng trong khối mã:

Markdown
### 2.4. Khai thác các System Functions đặc sắc
Em đã tìm hiểu và khai thác các hàm hệ thống có sẵn để tối ưu hóa truy vấn:

**Sử dụng hàm FORMAT để trình bày báo cáo chuyên nghiệp:**
```sql
SELECT [MaPhong], 
       FORMAT([NgayDat], 'D', 'vi-VN') AS [NgayTiengViet] 
FROM [PhieuDatPhong];
Sử dụng hàm COALESCE để xử lý dữ liệu trống:

SQL
SELECT [MaPhong], 
       COALESCE([GhiChu], N'Phòng tiêu chuẩn') AS [TrangThaiThucTe] 
FROM [Phong];
```
<img width="1919" height="1079" alt="Screenshot 2026-05-03 145919" src="https://github.com/user-attachments/assets/604d677f-9f31-40c0-9bd8-d9bce8d75777" />

2.1. Scalar Function (Hàm vô hướng - Trả về một giá trị duy nhất)
Mô tả: Hàm fn_TinhTongTien dùng để tính toán số tiền thực tế khách hàng phải trả. Hàm này sẽ lấy đơn giá nhân với số ngày lưu trú.

```SQL
CREATE FUNCTION [fn_TinhTongTien] (
    @DonGia MONEY, 
    @SoNgay INT
)
RETURNS MONEY
AS
BEGIN
    -- Nếu số ngày ở < 1 (ví dụ khách ở trong ngày), tính tròn 1 ngày
    IF (@SoNgay <= 0) SET @SoNgay = 1;
    
    RETURN @DonGia * @SoNgay;
END;
GO
```
Cách khai thác:

```SQL
SELECT [MaPhieu], 
       dbo.fn_TinhTongTien(500000, [SoNgayO]) AS [ThanhTien] 
FROM [PhieuDatPhong];
```
2.2. Inline Table-Valued Function (Hàm bảng đơn giản - Trả về một tập dữ liệu)
Mô tả: Hàm fn_TimPhongTrong giúp lễ tân lọc nhanh các phòng đang có trạng thái 'Trống' theo từng loại phòng cụ thể.

```SQL
CREATE FUNCTION [fn_TimPhongTrong] (
    @MaLoai INT
)
RETURNS TABLE
AS
RETURN (
    SELECT [MaPhong], [TenPhong], [TrangThai]
    FROM [Phong]
    WHERE [MaLoai] = @MaLoai AND [TrangThai] = N'Trống'
);
GO
```
Cách khai thác:

```SQL
SELECT * FROM dbo.fn_TimPhongTrong(1); -- Tìm phòng trống thuộc loại 1 (VIP)
```
2.3. Multi-statement Table-Valued Function (Hàm bảng phức tạp)
Mô tả: Hàm fn_BaoCaoChiTietPhieu thực hiện logic phức tạp: kết hợp dữ liệu từ nhiều bảng và phân loại mức độ chi tiêu của khách hàng (VIP/Thường).

```SQL
CREATE FUNCTION [fn_BaoCaoChiTietPhieu]()
RETURNS @BaoCao TABLE (
    MaPhieu INT,
    TenPhong NVARCHAR(50),
    TongTien MONEY,
    PhanLoaiKhach NVARCHAR(20)
)
AS
BEGIN
    INSERT INTO @BaoCao
    SELECT p.MaPhieu, ph.TenPhong, p.TongTien,
           CASE 
               WHEN p.TongTien > 5000000 THEN N'Khách VIP'
               ELSE N'Khách Thường'
           END
    FROM PhieuDatPhong p
    JOIN Phong ph ON p.MaPhong = ph.MaPhong;
    
    RETURN;
END;
GO
```
2.4. Khai thác các Built-in Functions (Hàm có sẵn đặc sắc)
Trong SQL Server, em đã tìm hiểu và ứng dụng các hàm hệ thống để làm báo cáo trở nên chuyên nghiệp hơn:

Hàm FORMAT: Dùng để định dạng tiền tệ theo chuẩn Việt Nam (đ) và ngày tháng.

Hàm COALESCE: Dùng để xử lý các giá trị NULL trong cột ghi chú, đảm bảo báo cáo không bị trống thông tin.

Hàm DATEDIFF: Tính toán số ngày giữa ngày đặt và ngày hiện tại.

Code khai thác hàm có sẵn:

```SQL
SELECT [MaPhieu], 
       -- Sử dụng FORMAT để hiện thị tiền tệ đẹp mắt
       FORMAT([TongTien], 'C', 'vi-VN') AS [SoTienDinhDang],
       -- Sử dụng COALESCE để thay thế giá trị NULL
       COALESCE([MaPhong], N'Đã hủy') AS [KiemTraPhong],
       -- Sử dụng GETDATE và DATEDIFF
       DATEDIFF(day, [NgayDat], GETDATE()) AS [SoNgayTuLucDat]
FROM [PhieuDatPhong];
```
Minh chứng thực thi Function và Built-in Function:
<img width="1919" height="1079" alt="Screenshot 2026-05-03 150511" src="https://github.com/user-attachments/assets/0c36244e-6925-4f5c-ab6d-6b1763b4d5cc" />

Đây là nội dung chi tiết cho Phần 3: Stored Procedure (Thủ tục lưu trữ). Phần này mình sẽ viết rất sâu vào logic bẫy lỗi và các lệnh hệ thống để bài làm của bạn nổi bật hơn hẳn bài của các bạn khác.

PHẦN 3: STORED PROCEDURE - THỦ TỤC LƯU TRỮ (KIẾN THỨC 10)
Stored Procedure (SP) là một tập hợp các câu lệnh T-SQL được biên soạn sẵn và lưu trữ trong Database. Việc sử dụng SP giúp tăng tốc độ thực thi, bảo mật dữ liệu và giảm thiểu việc gửi quá nhiều câu lệnh từ Client lên Server.

3.1. System Stored Procedures (Thủ tục hệ thống)
Đây là các hàm có sẵn của SQL Server giúp người quản trị kiểm tra thông tin nhanh mà không cần viết lệnh phức tạp. Em đã nghiên cứu và áp dụng các lệnh sau:

sp_help: Dùng để xem chi tiết định dạng của một bảng (kiểu dữ liệu, các ràng buộc).

sp_helptext: Dùng để xem lại mã nguồn của một Function hoặc Procedure đã viết trước đó.

sp_who: Kiểm tra danh sách người dùng và các tiến trình đang kết nối vào Database khách sạn.

Mã nguồn khai thác:

```SQL
-- Kiểm tra cấu trúc bảng Phòng
EXEC sp_help 'Phong';
```

-- Xem lại mã nguồn của hàm tính tiền đã viết ở Phần 2
EXEC sp_helptext 'fn_TinhTongTien';
3.2. User-defined Stored Procedure (Thủ tục người dùng tự định nghĩa)
Em xây dựng các thủ tục để quản lý nghiệp vụ khách sạn một cách an toàn và logic.

A. Thủ tục Cập nhật trạng thái phòng (Có bẫy lỗi logic)
Mô tả: Khi lễ tân muốn đổi trạng thái phòng (ví dụ từ 'Trống' sang 'Đang sửa chữa'), thủ tục này sẽ kiểm tra xem mã phòng đó có tồn tại hay không trước khi thực hiện lệnh UPDATE.

```SQL
CREATE PROCEDURE [sp_CapNhatTrangThaiPhong]
    @MaP VARCHAR(10), 
    @TrangThaiMoi NVARCHAR(20)
AS
BEGIN
    -- Kiểm tra sự tồn tại của phòng
    IF NOT EXISTS (SELECT 1 FROM [Phong] WHERE [MaPhong] = @MaP)
    BEGIN
        PRINT N'********** CẢNH BÁO **********';
        PRINT N'Lỗi: Mã phòng ' + @MaP + N' không tồn tại trên hệ thống!';
    END
    ELSE
    BEGIN
        UPDATE [Phong] 
        SET [TrangThai] = @TrangThaiMoi 
        WHERE [MaPhong] = @MaP;
        
        PRINT N'Thông báo: Cập nhật trạng thái cho phòng ' + @MaP + N' thành công.';
    END
END;
GO
```
B. Thủ tục Thêm mới Phiếu đặt phòng (Tự động tính tiền)
Mô tả: Thủ tục này giúp lễ tân đặt phòng nhanh, tự động lấy giá tiền từ bảng LoaiPhong để điền vào phiếu mà không cần nhập thủ công.

```SQL
CREATE PROCEDURE [sp_TaoPhieuDatNhanh]
    @MaP VARCHAR(10),
    @SoNgay INT
AS
BEGIN
    DECLARE @Gia MONEY;
    
    -- Lấy đơn giá dựa trên mã phòng truyền vào
    SELECT @Gia = lp.DonGia 
    FROM Phong p JOIN LoaiPhong lp ON p.MaLoai = lp.MaLoai
    WHERE p.MaPhong = @MaP;

    -- Thêm phiếu mới vào bảng
    INSERT INTO [PhieuDatPhong] ([MaPhong], [NgayDat], [SoNgayO], [TongTien])
    VALUES (@MaP, GETDATE(), @SoNgay, @Gia * @SoNgay);

    PRINT N'Đã tạo phiếu đặt phòng thành công cho phòng: ' + @MaP;
END;
GO
```
3.3. Thực thi và Kiểm tra kết quả
Để chứng minh Procedure hoạt động chính xác, em thực hiện các trường hợp sau:

Trường hợp 1: Cập nhật thành công

```SQL
EXEC [sp_CapNhatTrangThaiPhong] 'P101', N'Sửa chữa';
Trường hợp 2: Thử nghiệm bẫy lỗi (Nhập sai mã phòng)
```
```SQL
EXEC [sp_CapNhatTrangThaiPhong] 'P999', N'Đang có khách';
```
Hình ảnh minh họa kết quả thực thi Procedure:
<img width="1919" height="1079" alt="Screenshot 2026-05-03 150655" src="https://github.com/user-attachments/assets/57aa8400-8fae-4aad-9592-835c628ebe25" />

Tất nhiên rồi! Phần 4: Trigger là phần thể hiện sự "thông minh" và tự động hóa của Database. Để phần này thật dài và chi tiết, mình sẽ chia làm 2 loại: Trigger xử lý nghiệp vụ thực tế và Trigger giả lập lỗi đệ quy để nghiên cứu sâu.

PHẦN 4: TRIGGER - RÀNG BUỘC TOÀN VẸN ĐỘNG (KIẾN THỨC 10)
Trigger là một dạng đặc biệt của Stored Procedure, nó không được gọi trực tiếp mà sẽ tự động kích hoạt khi có các sự kiện INSERT, UPDATE hoặc DELETE xảy ra trên bảng.

4.1. Trigger cập nhật trạng thái phòng tự động (AFTER INSERT)
Mô tả logic: Trong nghiệp vụ khách sạn, ngay khi lễ tân tạo một phiếu đặt phòng mới, phòng đó phải lập tức chuyển từ trạng thái 'Trống' sang 'Đang có khách' để tránh việc đặt trùng. Em sử dụng Trigger trên bảng PhieuDatPhong để tự động hóa việc này.

```SQL
CREATE TRIGGER [trg_TuDongCapNhatTrangThai]
ON [PhieuDatPhong]
AFTER INSERT
AS
BEGIN
    -- Sử dụng bảng giả 'inserted' để lấy mã phòng vừa được đặt
    UPDATE [Phong]
    SET [TrangThai] = N'Đang có khách'
    FROM [Phong] P
    JOIN inserted i ON P.[MaPhong] = i.[MaPhong];

    PRINT N'>>> Hệ thống: Đã tự động chuyển trạng thái phòng sang Đang có khách.';
END;
GO
```
Khai thác và kiểm tra:

```SQL
-- Thêm một phiếu đặt phòng mới
INSERT INTO [PhieuDatPhong] ([MaPhong], [SoNgayO], [TongTien])
VALUES ('P102', 3, 1500000);


-- Kiểm tra lại bảng Phong xem trạng thái P102 đã đổi chưa
SELECT * FROM [Phong] WHERE [MaPhong] = 'P102';
```
<img width="1919" height="1079" alt="Screenshot 2026-05-03 150805" src="https://github.com/user-attachments/assets/f4b5cea2-1f42-45ab-aba9-cceea4feb703" />

4.2. Trigger kiểm tra logic dữ liệu (INSTEAD OF DELETE)
Mô tả logic: Ngăn chặn việc xóa những phòng đang có khách ở. Nếu người dùng cố tình xóa, Trigger sẽ hủy lệnh xóa và in ra cảnh báo.

```SQL
CREATE TRIGGER [trg_ChanXoaPhongDangDung]
ON [Phong]
INSTEAD OF DELETE
AS
BEGIN
    IF EXISTS (SELECT 1 FROM deleted WHERE [TrangThai] = N'Đang có khách')
    BEGIN
        PRINT N'********** SAI LẦM NGHIỆP VỤ **********';
        PRINT N'Lỗi: Không thể xóa phòng khi khách đang ở bên trong!';
    END
    ELSE
    BEGIN
        DELETE FROM [Phong] WHERE [MaPhong] IN (SELECT [MaPhong] FROM deleted);
        PRINT N'Xóa phòng thành công.';
    END
END;
GO
```
<img width="1919" height="1079" alt="Screenshot 2026-05-03 150840" src="https://github.com/user-attachments/assets/88bb945b-c164-4aaf-a377-82b42b1be139" />

4.3. Thử nghiệm lỗi Đệ quy (Recursive Trigger)
Đây là phần đặc sắc nhất để chứng minh kiến thức quản trị hệ thống. Em đã giả lập tình huống Trigger vòng lặp vô tận:

Trigger A (trên bảng Phong): Khi cập nhật trạng thái phòng -> sẽ tự cập nhật ngày ở bảng PhieuDatPhong.

Trigger B (trên bảng PhieuDatPhong): Khi cập nhật ngày -> sẽ quay lại cập nhật bảng Phong.

Mã nguồn giả lập lỗi:

``SQL
-- Trigger 1
CREATE TRIGGER [trg_VongLap_A] ON [Phong] AFTER UPDATE AS
BEGIN
    UPDATE [PhieuDatPhong] SET [NgayDat] = GETDATE();
END;
GO
```

-- Trigger 2 (Gây đệ quy)
```sql
CREATE TRIGGER [trg_VongLap_B] ON [PhieuDatPhong] AFTER UPDATE AS
BEGIN
    UPDATE [Phong] SET [TrangThai] = N'Bận';
END;
GO
```
Kết quả quan sát: Khi thực hiện lệnh UPDATE trên một trong hai bảng, SQL Server sẽ báo lỗi:

"Maximum stored procedure, function, trigger, or view nesting level exceeded (limit 32)."

Bài học rút ra: Em hiểu rằng SQL Server chỉ cho phép lặp tối đa 32 lần. Trong thực tế, cần thiết kế Trigger cẩn thận hoặc sử dụng IF UPDATE(ColumnName) để ngắt vòng lặp này, tránh làm treo hệ thống.
Hình ảnh minh họa thực thi Trigger:
<img width="1919" height="1079" alt="Screenshot 2026-05-03 150924" src="https://github.com/user-attachments/assets/d64ceb5d-0837-4dbf-8cb6-dae050efa98b" />

PHẦN 5: CURSOR VÀ TỐI ƯU HÓA DUYỆT DỮ LIỆU (KIẾN THỨC 11)
5.1. Triển khai Cursor duyệt danh sách phòng
Mô tả bài toán: Em sử dụng Cursor để duyệt qua từng phòng trong hệ thống nhằm thực hiện một tác vụ quản trị: In ra thông báo nhắc nhở kiểm tra thiết bị định kỳ cho nhân viên kỹ thuật.

``SQL
-- Khai báo biến để lưu trữ dữ liệu từ Cursor
DECLARE @MaP VARCHAR(10);
DECLARE @TenP NVARCHAR(50);

-- 1. Khai báo Cursor
DECLARE cur_KiemTraThietBi CURSOR FOR 
SELECT [MaPhong], [TenPhong] FROM [Phong];

-- 2. Mở Cursor
OPEN cur_KiemTraThietBi;

-- 3. Đọc dòng đầu tiên
FETCH NEXT FROM cur_KiemTraThietBi INTO @MaP, @TenP;

-- 4. Vòng lặp duyệt qua toàn bộ danh sách
WHILE @@FETCH_STATUS = 0
BEGIN
    PRINT N'Lệnh công tác: Kiểm tra hệ thống điện và điều hòa tại ' + @TenP + N' (' + @MaP + N')';
    
    -- Đọc dòng tiếp theo
    FETCH NEXT FROM cur_KiemTraThietBi INTO @MaP, @TenP;
END;
```

-- 5. Đóng và giải phóng tài nguyên
```sql
CLOSE cur_KiemTraThietBi;
DEALLOCATE cur_KiemTraThietBi;
GO
```
<img width="1919" height="1079" alt="Screenshot 2026-05-03 150949" src="https://github.com/user-attachments/assets/6b34cb32-4370-4e7a-8893-ce6841568d73" />

5.2. So sánh hiệu năng: Set-based vs Procedural
Trong quá trình thực hiện, em đã thực hiện so sánh hai phương pháp:

Sử dụng Cursor (Procedural - Tuần tự): * Ưu điểm: Kiểm soát cực kỳ chi tiết từng dòng dữ liệu, phù hợp với các logic xử lý phức tạp không thể viết bằng lệnh SQL đơn lẻ.

Nhược điểm: Tốc độ chậm, tốn tài nguyên CPU và bộ nhớ (RAM) do phải mở kết nối và xử lý từng dòng một. Khi dữ liệu lên đến hàng triệu dòng, Cursor sẽ gây "nghẽn cổ chai".

Sử dụng SQL thuần (Set-based - Tập hợp):

Ưu điểm: Tận dụng tối đa sức mạnh của công cụ truy vấn SQL Server, xử lý hàng loạt dữ liệu cùng lúc. Tốc độ nhanh hơn Cursor gấp nhiều lần.

Nhược điểm: Khó thực hiện các bài toán có logic phụ thuộc dòng trước dòng sau.
<img width="1919" height="1079" alt="Screenshot 2026-05-03 151101" src="https://github.com/user-attachments/assets/14d01fcf-8134-4b86-b419-5ba165a664a1" />


5.3. Bài toán đặc thù chỉ Cursor mới giải quyết tốt (Yêu cầu 3)
Bài toán: Tính số dư lũy kế (Running Total) hoặc Tính lãi suất phạt cộng dồn theo ngày.

Lý do:
Trong quản lý khách sạn, có những trường hợp khách hàng nợ tiền và tiền lãi của ngày hôm nay sẽ được tính dựa trên (Tổng nợ cũ + Tiền phát sinh) của ngày hôm trước.

Các câu lệnh SQL thuần (như UPDATE hay SELECT) làm việc theo cơ chế tập hợp, tức là nó xử lý các dòng dữ liệu song song hoặc độc lập. Nó không thể biết được kết quả của dòng vừa tính xong ngay phía trên là bao nhiêu để làm đầu vào cho dòng hiện tại.

Cursor xử lý tuần tự: Nó đi từ dòng 1 sang dòng 2, dòng 3... Vì vậy, nó có thể lưu giá trị của dòng trước vào một biến tạm, sau đó lấy biến đó cộng dồn cho dòng sau. Đây là logic "lũy kế" mà chỉ có cách duyệt tuần tự mới đảm bảo độ chính xác tuyệt đối.

5.4. Kết luận bài làm
Qua bài tập số 2, em đã nắm vững cách vận dụng:

Khởi tạo: Thiết lập nền móng dữ liệu vững chắc.

Function: Tối ưu hóa tính toán.

Procedure: Đóng gói nghiệp vụ an toàn.

Trigger: Tự động hóa hệ thống.

Cursor: Xử lý các logic tuần tự phức tạp.

Hình ảnh minh họa kết quả chạy Cursor:
<img width="1919" height="1079" alt="Screenshot 2026-05-03 151156" src="https://github.com/user-attachments/assets/d05c36f7-98e0-4351-9f54-e54da74ab52a" />



