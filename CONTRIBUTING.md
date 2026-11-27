# Hướng dẫn làm việc nhóm

## 1. Phân công

| Người | Mảng nghiệp vụ | Bảng phụ trách |
|---|---|---|
| A - Phong Ngọc Tiến | Đặt vé và ghế, xử lý đồng thời | `dat_ve`, `ve`, `giu_ghe` |
| B - Trần Đăng Long | Phim, rạp, phòng, suất chiếu, ERD, seed | `phim`, `the_loai`, `phim_the_loai`, `rap`, `phong_chieu`, `ghe`, `suat_chieu` |
| C - Bùi Xuân Sang | Người dùng, bảo mật, kiểm toán, backup | `nguoi_dung`, `vai_tro`, `nhat_ky_kiem_toan` |
| D - Đoàn Nguyên Đức | Đơn hàng, thanh toán, doanh thu, kiểm thử | `don_hang`, `thanh_toan`, `combo`, `don_hang_combo` |
| E - Hoàng Đình Quang | Khuyến mãi, thành viên, đánh giá, tài liệu | `khuyen_mai`, `the_thanh_vien`, `danh_gia` |

Chữ cái A-E cũng là **tiền tố tên file** của từng người (ví dụ `A_sp_dat_ve.sql`).

Danh sách việc chi tiết nằm trên bảng Kanban của GitHub Projects (và trong `PHAN_CONG.md`).

Nguyên tắc: **mỗi người tự viết truy vấn, procedure, trigger, test và API của mảng mình.** Ai cũng phải giải thích được code mình viết và luồng đặt vé từ đầu đến cuối khi vấn đáp.

File dùng chung có người làm chủ:
- `app/prisma/schema.prisma`, ERD: **B (Trần Đăng Long)**. Cần thêm hoặc sửa bảng thì nhắn B hoặc mở PR cho B duyệt.
- Phân quyền, role: **C (Bùi Xuân Sang)**.
- Hạ tầng test (Testcontainers): **D (Đoàn Nguyên Đức)**.
- `database/queries.sql`, từ điển dữ liệu, báo cáo: **E (Hoàng Đình Quang)** tổng hợp.

## 2. Làm việc với Git

Nhánh:
- `main`: bản ổn định, nộp bài.
- `develop`: nhánh tích hợp.
- `feature/<chữ-cái>-<việc>`: mỗi việc một nhánh, ví dụ `feature/A-sp-dat-ve`.

Quy trình:

```bash
git checkout develop
git pull origin develop
git checkout -b feature/A-sp-dat-ve
# ... viết code ...
git add database/programmability/A_sp_dat_ve.sql
git commit -m "feat: thêm sp_dat_ve có khóa hàng"
git push -u origin feature/A-sp-dat-ve
```

Sau đó mở **Pull Request vào `develop`**, ghi `Closes #số-issue` nếu việc có issue, nhờ **1 bạn khác duyệt** rồi merge. Không push thẳng vào `main` và `develop`. Khi `develop` ổn định, mở PR từ `develop` vào `main`.

Mỗi người cấu hình Git bằng **tên và email thật của mình** để lịch sử commit chứng minh ai làm phần nào:

```bash
git config --global user.name "Họ Tên"
git config --global user.email "email-github@example.com"
```

Quy ước commit:

| Tiền tố | Dùng khi |
|---|---|
| `feat:` | Thêm chức năng mới |
| `fix:` | Sửa lỗi |
| `docs:` | Tài liệu |
| `test:` | Kiểm thử |
| `refactor:` | Sắp xếp lại code, không đổi chức năng |
| `chore:` | Cấu hình, cài đặt |

Commit nhỏ, mỗi commit một việc. Không commit `.env`, mật khẩu thật hay `node_modules`.

## 3. Đặt file ở đâu

Thêm **tiền tố chữ cái của người viết** vào tên file để tránh đụng nhau và dễ truy vết.

| Loại | Thư mục | Ví dụ |
|---|---|---|
| Procedure, function, trigger, view | `database/programmability/` | `A_sp_dat_ve.sql`, `C_trg_kiem_toan.sql`, `D_vw_doanh_thu.sql` |
| Truy vấn nghiệp vụ | `database/queries/` | `A_queries.sql`, `B_queries.sql` |
| Dữ liệu mẫu | `database/seed/` | `B_seed_suat_chieu.ts` |
| Role, GRANT/REVOKE, view bảo mật | `database/security/` | `C_roles_grants.sql` |
| Kết quả đo hiệu năng | `database/performance/` | `B_explain_suat_chieu.md` |
| Backup và restore | `database/backup/` | `C_backup.ps1`, `C_restore.ps1` |
| Test | `tests/` | `D_booking.spec.ts`, `A_dat_cung_ghe.k6.js` |
| API NestJS | `app/src/<module>/` | `app/src/dat-ve/dat-ve.service.ts` |
| ERD, báo cáo, slide | `docs/` | `ERD.md`, `ERD.drawio` |

Mỗi người chỉ sửa file có tiền tố của mình. `database/queries.sql` do E gộp từ các file trong `database/queries/`, không ai sửa tay trực tiếp.

## 4. Quy ước code

**SQL**
- Tên bảng, cột: `snake_case`, tiếng Việt không dấu (`dat_ve`, `suat_chieu_id`).
- Từ khóa SQL viết HOA (`SELECT`, `JOIN`, `WHERE`).
- Procedure, function, view viết dạng chạy lại được: `CREATE OR REPLACE ...`. Trigger dùng `DROP TRIGGER IF EXISTS` trước `CREATE TRIGGER`.
- Mỗi file đầu có comment: người viết, mục đích, ví dụ gọi.
- Mỗi truy vấn nghiệp vụ có một dòng comment nêu **câu hỏi nghiệp vụ** nó trả lời.
- Mọi thao tác đặt vé nằm trong giao dịch, bắt lỗi và ném lỗi rõ ràng.

**TypeScript / NestJS**
- Bật `strict`, không dùng `any`.
- Biến và hàm `camelCase`, class `PascalCase`, file `kebab-case` (`dat-ve.service.ts`).
- Mỗi chức năng là một module: controller, service, dto, module. Controller mỏng, logic ở service.
- DTO kiểm tra đầu vào bằng `class-validator`; mô tả API bằng decorator Swagger.
- Truy vấn **luôn tham số hóa**, không nối chuỗi SQL.
- Lỗi trả đúng mã HTTP (400, 401, 403, 404, 409 khi ghế đã bị đặt).

**Prisma**
- Mọi thay đổi schema đi qua `prisma migrate`, không sửa DB bằng tay trên DBeaver rồi quên ghi lại.
- Commit cả thư mục migration.

## 5. Checklist trước khi mở Pull Request

- [ ] Code chạy được trên máy mình, file SQL chạy lại nhiều lần không lỗi.
- [ ] Đã `git pull origin develop` và xử lý xung đột (nếu có).
- [ ] Không có `.env`, mật khẩu hay file rác trong commit.
- [ ] Tên file có tiền tố người viết, đặt đúng thư mục.
- [ ] Có comment ở đầu file và câu hỏi nghiệp vụ cho truy vấn.
- [ ] Đã thêm test hoặc ví dụ chạy thử khi viết procedure, trigger.
- [ ] PR có mô tả ngắn và `Closes #số-issue` (nếu có).

## 6. Bảng Kanban

Cột: **Tồn đọng**, **Cần làm (Tuần này)**, **Đang làm**, **Chờ duyệt (PR)**, **Hoàn thành**.

- Đầu tuần họp ngắn, chọn việc kéo sang **Cần làm (Tuần này)**.
- Mỗi người giữ tối đa 2 việc ở **Đang làm**.
- Mở PR thì kéo thẻ sang **Chờ duyệt (PR)**; merge xong sang **Hoàn thành**.
- Việc lớn thì tách nhỏ, mỗi việc làm xong trong 1 đến 3 ngày.
