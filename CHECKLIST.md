# ✅ CHECKLIST PHÁT TRIỂN 2 NGÀY

## 📅 NGÀY 1 - BACKEND

### Buổi Sáng (8:00-12:00)

#### 1️⃣ Environment Setup (8:00-9:00)
- [ ] Cài Ruby 3.2+
- [ ] Cài Rails 7.1+
- [ ] Cài PostgreSQL 14+
- [ ] Tạo Rails project: `rails new realestate_map_api --api --database=postgresql`
- [ ] Test server chạy: `rails s`

#### 2️⃣ Database & Model (9:00-10:30)
- [ ] Enable PostGIS extension
- [ ] Generate Room model
- [ ] Add PostGIS location column
- [ ] Add indexes (GIST, BTREE)
- [ ] Add validations trong model
- [ ] Run migrations: `rails db:create db:migrate`
- [ ] Test model trong console: `rails c`

#### 3️⃣ Seed Data (10:30-11:00)
- [ ] Viết seeds.rb với 20-30 phòng
- [ ] Run seed: `rails db:seed`
- [ ] Verify data: `Room.count` trong rails console

#### 4️⃣ API Controllers - Part 1 (11:00-12:00)
- [ ] Generate controller: `rails g controller Api::V1::Rooms`
- [ ] Setup routes cho index & show
- [ ] Implement basic index action
- [ ] Test endpoint: `curl http://localhost:3000/api/v1/rooms`

---

### Buổi Chiều (13:00-17:00)

#### 5️⃣ API Controllers - Part 2 (13:00-15:00)
- [ ] Implement bounding box query
- [ ] Implement radius search
- [ ] Implement price filter
- [ ] Implement area filter
- [ ] Implement room_type filter
- [ ] Implement status filter
- [ ] Setup JSON serializer/jbuilder
- [ ] Test tất cả filters

#### 6️⃣ Testing API (15:00-16:00)
- [ ] Test GET all rooms
- [ ] Test bounding box query
- [ ] Test radius search
- [ ] Test price filter
- [ ] Test combined filters
- [ ] Fix bugs nếu có
- [ ] Document API trong README

#### 7️⃣ CORS & Deploy (16:00-17:00)
- [ ] Add rack-cors gem
- [ ] Configure CORS
- [ ] Add pagination (kaminari/pagy)
- [ ] Add error handling
- [ ] Test từ browser console
- [ ] (Optional) Deploy lên Railway/Render

---

## 📅 NGÀY 2 - FRONTEND

### Buổi Sáng (8:00-12:00)

#### 1️⃣ Frontend Setup (8:00-9:00)
- [ ] Tạo folder `public/` hoặc `frontend/`
- [ ] Tạo `index.html`
- [ ] Đăng ký Mapbox account
- [ ] Get Mapbox Access Token
- [ ] Include Mapbox GL JS CDN
- [ ] Basic HTML structure
- [ ] Test HTML mở được trong browser

#### 2️⃣ Initialize Map (9:00-10:30)
- [ ] Tạo file `app.js`
- [ ] Initialize Mapbox map
- [ ] Set center (Hanoi hoặc HCMC)
- [ ] Add navigation controls
- [ ] Add zoom controls
- [ ] Test map hiển thị đúng

#### 3️⃣ Fetch & Display Markers (10:30-12:00)
- [ ] Viết function `fetchRooms()`
- [ ] Viết function `renderMarkers()`
- [ ] Fetch rooms từ API
- [ ] Create markers từ data
- [ ] Add popup cho markers
- [ ] Test markers hiển thị

---

### Buổi Chiều (13:00-17:00)

#### 4️⃣ Dynamic Loading (13:00-14:30)
- [ ] Listen to 'moveend' event
- [ ] Get map bounds
- [ ] Fetch rooms trong bounds
- [ ] Clear old markers
- [ ] Render new markers
- [ ] Add debounce (300ms)
- [ ] Test kéo map
- [ ] Test zoom map

#### 5️⃣ Room Detail View (14:30-16:00)
- [ ] Tạo sidebar/modal HTML
- [ ] Style sidebar/modal CSS
- [ ] Viết function `showRoomDetail(room)`
- [ ] Click marker → show detail
- [ ] Show: title, price, area, address, room_type
- [ ] Add close button
- [ ] Test click markers

#### 6️⃣ Search/Filter UI (16:00-17:00)
- [ ] Tạo filter form HTML
- [ ] Style filter form CSS
- [ ] Add event listeners
- [ ] Build filter URL với params
- [ ] Apply filters khi fetch
- [ ] Test filter by price
- [ ] Test filter by room_type
- [ ] Test combined filters

---

### Tối (Optional - 19:00-21:00)

#### 7️⃣ Polish & Testing
- [ ] Responsive CSS cho mobile
- [ ] Add loading spinner
- [ ] Add error messages
- [ ] Test trên Chrome
- [ ] Test trên Safari
- [ ] Test trên mobile
- [ ] Fix UI bugs
- [ ] Add favicon
- [ ] Update README

---

## 🎯 FINAL CHECKS

- [ ] Backend API hoạt động
- [ ] Frontend map hiển thị
- [ ] Markers render đúng
- [ ] Click marker xem detail
- [ ] Kéo map update markers
- [ ] Filters hoạt động
- [ ] Không có console errors
- [ ] Code có comments
- [ ] README đầy đủ
- [ ] Git commit & push

---

## 📊 Progress Tracker

**Ngày 1:**
- Sáng: ⬜⬜⬜⬜ (0/4 tasks)
- Chiều: ⬜⬜⬜ (0/3 tasks)

**Ngày 2:**
- Sáng: ⬜⬜⬜ (0/3 tasks)
- Chiều: ⬜⬜⬜ (0/3 tasks)

**Overall:** 0/13 tasks completed

---

## 🚨 Nếu Bị Stuck

### Backend Issues:
- PostGIS không cài được → Google "install postgis [your OS]"
- API không trả data → Check rails console, check database
- CORS errors → Check rack-cors config

### Frontend Issues:
- Map không hiển thị → Check console, verify Mapbox token
- Markers không render → Check API response, check console.log
- Filters không work → Check URL params, check API

### Time Management:
- Mỗi task có time estimate → Nếu quá thời gian, skip details, move on
- Ưu tiên functionality > styling
- Commit code thường xuyên

---

**Pro tip:** In checklist này ra giấy và tick bằng bút cho đã! 🖊️

