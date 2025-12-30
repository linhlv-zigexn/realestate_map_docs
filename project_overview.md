Project name: Realestate Map (hệ thống Bản đồ tìm phòng)
Project description: Dự án cho phép client tìm kiếm thông tin bất động sản tại các khu vực mà họ muốn sinh sống.
Dựa trên bản đồ Mapbox và thông tin bất động sản được đính kèm tương ứng.
⸻

1. 🎯 Mục tiêu hệ thống

Người dùng truy cập website có thể:
	•	Xem bản đồ Mapbox
	•	Thấy các điểm phòng trọ đang cho thuê trên bản đồ
	•	Click vào điểm để xem chi tiết phòng
  •	Tìm kiếm & lọc phòng theo nhiều tiêu chí
	•	Đăng kí xem phòng và đặt cọc.
  •	Cho phép đăng phòng và ghim lên map.

  2. 🏗 Kiến trúc tổng thể (High-level Architecture)

Browser (Web)
   │
   │  HTTP / JSON
   ▼
Backend API (Rails)
   │
   │  SQL / Geo Query
   ▼
Database (PostgreSQL + PostGIS)

Thành phần chính
	•	Frontend: Hiển thị bản đồ, marker, UI
	•	Backend API: Cung cấp dữ liệu phòng trọ
	•	Database: Lưu thông tin phòng + tọa độ
	•	Mapbox API: Bản đồ, marker, geocode

⸻

3. 🧰 Công nghệ đề xuất

Frontend
	•	HTML / CSS / JavaScript
	•	Framework (tuỳ chọn): React
	•	Mapbox JavaScript API

Backend
	•	Ruby on Rails (API mode)
	•	RESTful API

Database
	•	PostgreSQL
	•	PostGIS extension (xử lý dữ liệu không gian)

⸻

4. 🗄 Database Design

Bảng rooms

Field	Type	Mô tả
id	bigint	ID phòng
title	string	Tên phòng
price	integer	Giá thuê
area	float	Diện tích
address	string	Địa chỉ
latitude	float	Vĩ độ
longitude	float	Kinh độ
location	geography(Point)	Toạ độ (PostGIS)
room_type	string	Loại phòng
status	string	available / rented
created_at	datetime

👉 Index bắt buộc:
	•	GIST index cho location

⸻

5. 🔌 Backend API Design

5.1 Load phòng theo vùng bản đồ (Bounding Box)

Request

GET /api/rooms?north=...&south=...&east=...&west=...

Ý nghĩa
	•	Chỉ load phòng nằm trong vùng map đang hiển thị
	•	Tránh load toàn bộ database

⸻

5.2 Load phòng theo bán kính

GET /api/rooms?lat=21.02&lng=105.83&radius=2000

PostGIS query:

ST_DWithin(location, ST_MakePoint(lng, lat)::geography, radius)


⸻

5.3 Filter nâng cao

GET /api/rooms?
  min_price=2000000
  &max_price=5000000
  &min_area=20
  &room_type=studio

⸻

6. 🖥 Frontend Flow

Khi load trang
	1.	Load Google Map
	2.	Xác định vị trí mặc định (hoặc vị trí user)
	3.	Gọi API load phòng trong vùng bản đồ
	4.	Render marker

⸻

Khi user kéo / zoom bản đồ
	•	Lắng nghe sự kiện:

map.addListener("idle", () => {
  // map bounds changed
})

	•	Gọi lại API
	•	Clear marker cũ → vẽ marker mới

⸻

Khi click marker
	•	Hiển thị:
	•	InfoWindow
	•	Hoặc panel chi tiết bên cạnh bản đồ

⸻

7. 🚀 Tối ưu Marker & Hiệu năng

Marker Cluster
	•	Gộp marker khi zoom out
	•	Giảm lag

Lazy Loading
	•	Chỉ load phòng trong viewport
	•	Không load toàn bộ dữ liệu

⸻

8. 🗺 Google Maps API sử dụng

API	Mục đích
Maps JavaScript API	Hiển thị bản đồ
Geocoding API	Địa chỉ → lat/lng
Places API (tuỳ chọn)	Tìm kiếm địa điểm
MarkerClusterer	Gộp marker

11. 🛣 Roadmap triển khai

Phase 1 – MVP
	•	Hiển thị map
	•	Marker phòng
	•	Click xem thông tin
	•	Đăng kí xem phòng

Phase 2 – Search & Filter
	•	Giá, diện tích
	•	Reload theo map bounds

Phase 3 – UX nâng cao
	•	Marker cluster
	•	Danh sách phòng bên cạnh map

Phase 4 – Scale
	•	Cache
	•	PostGIS index tối ưu
	•	Async loading

⸻

12. 🔄 Luồng dữ liệu tổng quát

User → Load Map
     → Map bounds
     → API /rooms
     → Database (Geo Query)
     → JSON
     → Render markers


⸻
14. 🧩 ERD Diagram (Database)

+----------------+
|     rooms      |
+----------------+
| id (PK)        |
| title          |
| price          |
| area           |
| address        |
| latitude       |
| longitude      |
| location (GIS) |
| room_type      |
| status         |
| created_at     |
| updated_at     |
+----------------+

Index đề xuất
	•	GIST (location) – bắt buộc cho geo query
	•	BTREE (price)
	•	BTREE (status)

⸻


15. Tài nguyên tham khảo.
Hướng dẫn làm.
https://docs.mapbox.com/help/tutorials/building-a-store-locator-react/?step=0

Source demo: https://github.com/mapbox/public-tools-and-demos/tree/main/projects/demo-realestate
