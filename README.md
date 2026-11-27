# Cinema Booking DB

Đồ án Cơ sở dữ liệu (DT03): hệ thống đặt vé xem phim, trọng tâm là chống đặt trùng ghế.

**Stack:** TypeScript · NestJS · Prisma · PostgreSQL 18 · Docker

## Yêu cầu cài đặt

- Git
- Docker Desktop (đang chạy)
- Node.js LTS (20 hoặc 22), chỉ cần khi chạy ứng dụng ngoài Docker
- DBeaver hoặc pgAdmin để xem dữ liệu

## Chạy dự án trên máy sạch (5 lệnh)

```bash
git clone <địa-chỉ-repo>
cd cinema-booking-db
copy .env.example .env        # macOS/Linux: cp .env.example .env
docker compose up -d --build
# mở trình duyệt: http://localhost:3000/api/docs
```

Khi `docker compose up` chạy:
1. PostgreSQL 18 khởi động, script `database/init/01-roles.sh` tạo 3 role: `role_admin`, `role_nhan_vien`, `role_khach_hang`.
2. Container ứng dụng chạy `prisma migrate deploy` rồi bật NestJS.
3. Swagger có tại `http://localhost:3000/api/docs`.

Sau khi đổi mật khẩu hoặc muốn tạo lại từ đầu: `docker compose down -v` rồi chạy lại lệnh trên.

## Kết nối cơ sở dữ liệu

| Mục | Giá trị |
|---|---|
| Host | `localhost` |
| Port | `5432` |
| Database | `cinema_booking` |
| Tài khoản | xem file `.env` (`cinema_owner` để quản trị, 3 role còn lại cho ứng dụng) |

Ứng dụng **không dùng superuser**. `cinema_owner` chỉ dùng để chạy migration.

## Làm việc hằng ngày (dev)

```bash
docker compose up -d db          # chỉ bật PostgreSQL
cd app
npm install
npm run start:dev                # chạy NestJS, tự reload
```

Một số lệnh khác (chạy trong thư mục `app/`):

```bash
npm run db:migrate               # tạo và áp dụng migration mới
npm run db:deploy                # áp dụng migration có sẵn
npm run db:studio                # xem dữ liệu bằng Prisma Studio
```

## Cấu trúc thư mục

```
cinema-booking-db/
├── app/                    # Code NestJS + Prisma
│   └── prisma/             # schema.prisma và migrations
├── database/
│   ├── init/               # Script chạy khi DB khởi tạo lần đầu (tạo role)
│   ├── seed/               # Script sinh dữ liệu mẫu (10k đến 100k+ dòng)
│   ├── programmability/    # View, procedure, function, trigger
│   ├── queries/            # Truy vấn nghiệp vụ của từng người
│   ├── security/           # GRANT/REVOKE, view bảo mật
│   ├── performance/        # Kết quả EXPLAIN ANALYZE trước và sau
│   ├── backup/             # Script pg_dump và pg_restore
│   └── queries.sql         # Tổng hợp >= 20 truy vấn của cả nhóm
├── tests/                  # Testcontainers và script k6
├── docs/                   # ERD, báo cáo, slide
├── docker-compose.yml
├── .env.example
├── README.md
└── CONTRIBUTING.md         # Phân công và quy ước làm việc nhóm
```

Lưu ý: Prisma lưu migration tại `app/prisma/migrations/`. Những thứ Prisma không tạo được (CHECK, trigger, procedure, partial index, `EXCLUDE`) được viết thành file SQL trong `database/programmability/` và gọi từ một migration tạo bằng `prisma migrate dev --create-only`.

## Lỗi thường gặp

| Lỗi | Cách xử lý |
|---|---|
| `variable is not set` khi chạy compose | File `.env` chưa có hoặc tên sai. Chạy `copy .env.example .env` |
| Container DB cứ `Restarting` | Xem `docker compose logs db`. Script `.sh` phải dùng dấu xuống dòng LF |
| Script tạo role không chạy | Volume cũ còn dữ liệu. Chạy `docker compose down -v` |
| DBeaver báo lỗi múi giờ `Asia/Saigon` | Thêm `-Duser.timezone=Asia/Ho_Chi_Minh` vào `dbeaver.ini` |
| Cổng 5432 bị chiếm | Đổi `"5432:5432"` thành `"5433:5432"` trong `docker-compose.yml` |

## Tài liệu

- Quy ước làm việc và phân công: [CONTRIBUTING.md](CONTRIBUTING.md)
- Thiết kế CSDL (ERD): [docs/ERD.md](docs/ERD.md)
- API: `http://localhost:3000/api/docs`
