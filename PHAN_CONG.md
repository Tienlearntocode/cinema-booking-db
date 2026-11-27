# Phân công nhóm: DT03 Đặt vé xem phim

Stack: TypeScript · NestJS · Prisma · PostgreSQL 18

## Tổng quan

| Người | Mảng nghiệp vụ | Bảng phụ trách |
|---|---|---|
| A - Phong Ngọc Tiến | Đặt vé và ghế, xử lý đồng thời | `dat_ve`, `ve`, `giu_ghe` |
| B - Trần Đăng Long | Phim, rạp, phòng, suất chiếu, ERD, seed | `phim`, `the_loai`, `phim_the_loai`, `rap`, `phong_chieu`, `ghe`, `suat_chieu` |
| C - Bùi Xuân Sang | Người dùng, bảo mật, kiểm toán, backup | `nguoi_dung`, `vai_tro`, `nhat_ky_kiem_toan` |
| D - Đoàn Nguyên Đức | Đơn hàng, thanh toán, doanh thu, kiểm thử | `don_hang`, `thanh_toan`, `combo`, `don_hang_combo` |
| E - Hoàng Đình Quang | Khuyến mãi, thành viên, đánh giá, tài liệu | `khuyen_mai`, `the_thanh_vien`, `danh_gia` |

Quy ước: mỗi người tự viết truy vấn, procedure, trigger và API cho mảng của mình.

---

## Việc của từng người (mỗi dòng là một issue)

### A (Phong Ngọc Tiến): Đặt vé và ghế
- [ ] [A] Cài k6, dựng môi trường test tải
- [ ] [A] Ràng buộc `UNIQUE(suat_chieu_id, ghe_id)` cho `ve` và `giu_ghe`
- [ ] [A] Viết `sp_dat_ve` (có tham số, `BEGIN/COMMIT/ROLLBACK`, `SELECT ... FOR UPDATE`)
- [ ] [A] Trigger chặn đặt vé khi suất đã bắt đầu hoặc ghế đang bảo trì
- [ ] [A] 4-5 truy vấn (JOIN >= 3 bảng, `NOT EXISTS` tìm ghế trống)
- [ ] [A] Script k6: 100 request đồng thời vào cùng 1 ghế, chứng minh lỗi trước khi khóa và đúng 1 người thành công sau khi sửa

### B (Trần Đăng Long): Phim, rạp, suất chiếu
- [ ] [B] Cài công cụ vẽ ERD (draw.io hoặc dbdiagram.io) và `@faker-js/faker`
- [ ] [B] Ghép ERD tổng thể (>= 10 thực thể; có 1-1, 1-n, n-n, thực thể yếu `ghe`, quan hệ đệ quy `the_loai.parent_id`)
- [ ] [B] Chứng minh chuẩn hóa 3NF cho >= 3 bảng
- [ ] [B] CHECK constraint (`thoi_luong > 0`, `gio_ket_thuc > gio_bat_dau`) và quy tắc `ON DELETE/UPDATE`
- [ ] [B] Script seed dữ liệu tiếng Việt (>= 100.000 dòng cho `suat_chieu` hoặc `ve`)
- [ ] [B] CTE đệ quy cây thể loại; truy vấn tỷ lệ lấp đầy phòng (`GROUP BY + HAVING`)
- [ ] [B] Composite index `suat_chieu(phim_id, thoi_gian_chieu)` và `EXPLAIN ANALYZE` trước/sau

### C (Bùi Xuân Sang): Người dùng, bảo mật, kiểm toán
- [ ] [C] Cài thư viện bcrypt hoặc argon2 trong NestJS
- [ ] [C] Tạo >= 3 role (`role_admin`, `role_nhan_vien`, `role_khach_hang`), `GRANT/REVOKE` quyền tối thiểu; app kết nối bằng user thường
- [ ] [C] View bảo mật ẩn cột nhạy cảm (`mat_khau_hash`, `so_dien_thoai`)
- [ ] [C] Trigger ghi `nhat_ky_kiem_toan` (JSONB cũ/mới) khi UPDATE/DELETE trên `ve`, `suat_chieu`
- [ ] [C] Script backup và restore (`pg_dump`, `pg_restore`) trong `database/backup/`, kèm demo xóa nhầm rồi phục hồi
- [ ] [C] API đăng ký, đăng nhập (hash mật khẩu, truy vấn tham số hóa)

### D (Đoàn Nguyên Đức): Đơn hàng, thanh toán, kiểm thử
- [ ] [D] Cấu hình Testcontainers (Jest hoặc Vitest) chạy trên PostgreSQL thật
- [ ] [D] Function tính tổng tiền đơn (vé + combo)
- [ ] [D] Stored procedure thanh toán (cập nhật `don_hang`, sinh `thanh_toan`, chuyển trạng thái `ve`)
- [ ] [D] View doanh thu theo rạp, theo phim, theo tháng
- [ ] [D] Truy vấn Window Function (`RANK()`, doanh thu lũy kế) và OUTER JOIN (combo chưa từng bán)
- [ ] [D] >= 15 test case trong `tests/`, có ca cố tình vi phạm CHECK, UNIQUE, FK

### E (Hoàng Đình Quang): Khuyến mãi, thành viên, đánh giá, tài liệu
- [ ] [E] Ràng buộc `khuyen_mai` (`so_luong_con_lai >= 0`, `ngay_ket_thuc >= ngay_bat_dau`) và `danh_gia` (`diem_so BETWEEN 1 AND 5`, `UNIQUE(nguoi_dung_id, phim_id)`)
- [ ] [E] Function kiểm tra mã khuyến mãi và tính số tiền giảm
- [ ] [E] Stored procedure cộng điểm tích lũy và tự nâng hạng thẻ sau khi đơn hoàn tất
- [ ] [E] Truy vấn con tương quan (phim có điểm cao hơn trung bình thể loại) và phép chia (khách đã xem mọi phim thể loại "Hành động")
- [ ] [E] Tổng hợp `database/queries.sql` (>= 20 câu của cả nhóm, có chú thích câu hỏi nghiệp vụ)
- [ ] [E] Tổng hợp từ điển dữ liệu (mọi cột của mọi bảng)
- [ ] [E] Biên tập báo cáo PDF (30-50 trang) và slide bảo vệ

---

## Cách dùng với GitHub Projects

1. Mỗi dòng trên là một **Issue**. Tên issue giữ nguyên tiền tố `[A]`, `[B]`...
2. Gán **Assignee** là người phụ trách, thêm **Label** theo mảng (`mảng-A`...).
3. Thêm issue vào bảng Kanban, mặc định ở cột **Tồn đọng**.
4. Đầu tuần kéo các việc đã chọn sang **Cần làm (Tuần này)**.
5. Tạo nhánh `feature/<chữ-cái>-<việc>` khi bắt đầu, mở PR kèm `Closes #số-issue` khi xong.
