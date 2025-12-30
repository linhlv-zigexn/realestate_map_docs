# 📋 KẾ HOẠCH PHÁT TRIỂN 2 NGÀY - REALESTATE MAP MVP

**Mục tiêu:** Hoàn thành MVP với các tính năng cốt lõi để demo được sản phẩm

**Phạm vi:** Phase 1 - MVP (Minimum Viable Product)

---

## 🗓 NGÀY 1 - BACKEND & DATABASE FOUNDATION

### ⏰ Buổi Sáng (4h - 8:00-12:00)

#### 1. Environment Setup (1h - 8:00-9:00)
- [ ] Cài đặt Ruby on Rails (API mode)
- [ ] Setup PostgreSQL + PostGIS extension
- [ ] Tạo Rails project mới
  ```bash
  rails new realestate_map_api --api --database=postgresql
  ```
- [ ] Configure database.yml
- [ ] Verify PostGIS extension
  ```sql
  CREATE EXTENSION postgis;
  ```

**Output:** Rails project hoạt động + PostgreSQL + PostGIS ready

---

#### 2. Database Schema & Model (1.5h - 9:00-10:30)
- [ ] Generate Room model
  ```bash
  rails g model Room title:string price:integer area:float address:string latitude:float longitude:float room_type:string status:string
  ```
- [ ] Add PostGIS location column in migration
  ```ruby
  # migration file
  add_column :rooms, :location, :geography, limit: { srid: 4326, type: "point" }
  add_index :rooms, :location, using: :gist
  add_index :rooms, :price
  add_index :rooms, :status
  ```
- [ ] Setup RGeo gem for PostGIS support
- [ ] Add validations và callbacks trong Room model
- [ ] Run migrations
  ```bash
  rails db:create db:migrate
  ```

**Output:** Database schema hoàn chỉnh với PostGIS support

---

#### 3. Seed Data (0.5h - 10:30-11:00)
- [ ] Tạo seeds.rb với ~20-30 phòng mẫu
- [ ] Data tập trung tại Hà Nội/TP.HCM (chọn 1 thành phố)
- [ ] Đa dạng price, area, room_type
  ```ruby
  # seeds.rb example
  Room.create!(
    title: "Studio cozy near center",
    price: 3500000,
    area: 25,
    address: "123 Nguyen Trai, Hanoi",
    latitude: 21.0285,
    longitude: 105.8542,
    room_type: "studio",
    status: "available"
  )
  ```
- [ ] Run seed
  ```bash
  rails db:seed
  ```

**Output:** Database có dữ liệu test thật

---

#### 4. API Controllers - Part 1 (1h - 11:00-12:00)
- [ ] Generate Rooms controller
  ```bash
  rails g controller Api::V1::Rooms
  ```
- [ ] Setup routes
  ```ruby
  namespace :api do
    namespace :v1 do
      resources :rooms, only: [:index, :show]
    end
  end
  ```
- [ ] Implement `index` action với bounding box query
  ```ruby
  # Filter by map bounds
  if params[:north] && params[:south] && params[:east] && params[:west]
    # PostGIS query here
  end
  ```

**Output:** Basic API endpoint `/api/v1/rooms` ready

---

### ⏰ Buổi Chiều (4h - 13:00-17:00)

#### 5. API Controllers - Part 2 (2h - 13:00-15:00)
- [ ] Implement bounding box query với PostGIS
  ```ruby
  scope :within_bounds, ->(north, south, east, west) {
    where("location && ST_MakeEnvelope(?, ?, ?, ?, 4326)", west, south, east, north)
  }
  ```
- [ ] Implement radius search
  ```ruby
  scope :within_radius, ->(lat, lng, radius) {
    where("ST_DWithin(location, ST_MakePoint(?, ?)::geography, ?)", lng, lat, radius)
  }
  ```
- [ ] Implement filters: price range, area, room_type, status
- [ ] Add serializer/jbuilder cho JSON response
  ```ruby
  # rooms/_room.json.jbuilder
  json.extract! room, :id, :title, :price, :area, :address, :latitude, :longitude, :room_type, :status
  ```

**Output:** Hoàn thiện API với filters

---

#### 6. Testing API (1h - 15:00-16:00)
- [ ] Test với Postman/curl
  - GET `/api/v1/rooms` - all rooms
  - GET `/api/v1/rooms?north=21.04&south=21.02&east=105.86&west=105.84` - bounding box
  - GET `/api/v1/rooms?lat=21.0285&lng=105.8542&radius=5000` - radius
  - GET `/api/v1/rooms?min_price=2000000&max_price=5000000` - price filter
- [ ] Fix bugs nếu có
- [ ] Document API endpoints

**Output:** API tested và ready cho frontend

---

#### 7. CORS & Additional Setup (1h - 16:00-17:00)
- [ ] Enable CORS cho frontend
  ```ruby
  # Gemfile
  gem 'rack-cors'

  # config/initializers/cors.rb
  Rails.application.config.middleware.insert_before 0, Rack::Cors do
    allow do
      origins '*'
      resource '*', headers: :any, methods: [:get, :post, :options]
    end
  end
  ```
- [ ] Add pagination (gem kaminari hoặc pagy)
- [ ] Add error handling
- [ ] Deploy backend lên Heroku/Railway (optional nhưng recommended)

**Output:** Backend hoàn chỉnh, có thể deploy được

---

## 🗓 NGÀY 2 - FRONTEND & INTEGRATION

### ⏰ Buổi Sáng (4h - 8:00-12:00)

#### 1. Frontend Setup (1h - 8:00-9:00)
- [ ] Tạo folder `frontend/` hoặc dùng Rails views
- [ ] Setup HTML boilerplate
- [ ] Get Mapbox Access Token
  - Đăng ký tại https://mapbox.com
  - Copy Access Token
- [ ] Include Mapbox GL JS
  ```html
  <link href='https://api.mapbox.com/mapbox-gl-js/v2.15.0/mapbox-gl.css' rel='stylesheet' />
  <script src='https://api.mapbox.com/mapbox-gl-js/v2.15.0/mapbox-gl.js'></script>
  ```
- [ ] Basic HTML structure
  ```html
  <div id="map"></div>
  <div id="room-list"></div>
  ```

**Output:** Frontend skeleton ready

---

#### 2. Initialize Mapbox (1.5h - 9:00-10:30)
- [ ] Initialize map với Mapbox GL JS
  ```javascript
  mapboxgl.accessToken = 'YOUR_TOKEN';
  const map = new mapboxgl.Map({
    container: 'map',
    style: 'mapbox://styles/mapbox/streets-v11',
    center: [105.8542, 21.0285], // Hanoi
    zoom: 12
  });
  ```
- [ ] Add navigation controls
- [ ] Add geolocation control (optional)
- [ ] Test map rendering

**Output:** Bản đồ hiển thị đúng

---

#### 3. Fetch & Display Markers (1.5h - 10:30-12:00)
- [ ] Create function fetchRooms()
  ```javascript
  async function fetchRooms(bounds) {
    const { north, south, east, west } = bounds;
    const response = await fetch(
      `http://localhost:3000/api/v1/rooms?north=${north}&south=${south}&east=${east}&west=${west}`
    );
    return await response.json();
  }
  ```
- [ ] Create function renderMarkers()
  ```javascript
  function renderMarkers(rooms) {
    rooms.forEach(room => {
      new mapboxgl.Marker()
        .setLngLat([room.longitude, room.latitude])
        .setPopup(new mapboxgl.Popup().setHTML(`
          <h3>${room.title}</h3>
          <p>Price: ${room.price.toLocaleString()} VND</p>
          <p>Area: ${room.area} m²</p>
        `))
        .addTo(map);
    });
  }
  ```
- [ ] Load rooms on map load
- [ ] Test markers rendering

**Output:** Markers hiển thị trên bản đồ

---

### ⏰ Buổi Chiều (4h - 13:00-17:00)

#### 4. Dynamic Loading on Map Move (1.5h - 13:00-14:30)
- [ ] Listen to map 'moveend' event
  ```javascript
  map.on('moveend', () => {
    const bounds = map.getBounds();
    loadRoomsInBounds(bounds);
  });
  ```
- [ ] Clear old markers before adding new
  ```javascript
  let markers = [];
  function clearMarkers() {
    markers.forEach(m => m.remove());
    markers = [];
  }
  ```
- [ ] Implement debounce để tránh quá nhiều requests
- [ ] Test kéo/zoom map

**Output:** Map tự động reload rooms khi di chuyển

---

#### 5. Room Detail View (1.5h - 14:30-16:00)
- [ ] Tạo sidebar hoặc modal hiển thị chi tiết phòng
- [ ] Click marker → show detail
  ```javascript
  marker.getElement().addEventListener('click', () => {
    showRoomDetail(room);
  });
  ```
- [ ] Room detail UI
  ```html
  <div id="room-detail">
    <h2 id="room-title"></h2>
    <p id="room-price"></p>
    <p id="room-area"></p>
    <p id="room-address"></p>
    <button id="btn-register">Đăng ký xem phòng</button>
  </div>
  ```
- [ ] Style cho đẹp với CSS

**Output:** Click marker hiển thị chi tiết phòng

---

#### 6. Basic Search/Filter UI (1h - 16:00-17:00)
- [ ] Tạo filter form
  ```html
  <div id="filters">
    <input type="number" id="min-price" placeholder="Giá tối thiểu">
    <input type="number" id="max-price" placeholder="Giá tối đa">
    <select id="room-type">
      <option value="">Tất cả loại phòng</option>
      <option value="studio">Studio</option>
      <option value="apartment">Apartment</option>
    </select>
    <button id="btn-filter">Lọc</button>
  </div>
  ```
- [ ] Apply filters khi fetch rooms
  ```javascript
  function buildFilterUrl(bounds, filters) {
    let url = `${API_URL}?north=${bounds.north}&south=${bounds.south}&east=${bounds.east}&west=${bounds.west}`;
    if (filters.minPrice) url += `&min_price=${filters.minPrice}`;
    if (filters.maxPrice) url += `&max_price=${filters.maxPrice}`;
    if (filters.roomType) url += `&room_type=${filters.roomType}`;
    return url;
  }
  ```
- [ ] Test filters hoạt động

**Output:** Có thể lọc phòng theo tiêu chí

---

### ⏰ Tối (Optional - 2h - 19:00-21:00)

#### 7. Polish & Testing
- [ ] Responsive CSS cho mobile
- [ ] Loading states
- [ ] Error handling
- [ ] Cross-browser testing
- [ ] Fix UI bugs
- [ ] Add favicon
- [ ] Update README với hướng dẫn chạy project

**Output:** MVP hoàn chỉnh có thể demo

---

## 📊 DELIVERABLES SAU 2 NGÀY

### ✅ Backend
- Rails API hoạt động với PostgreSQL + PostGIS
- Endpoints: GET /api/v1/rooms với filters
- Seed data: 20-30 phòng mẫu
- Bounding box & radius query
- Price/area/room_type filters

### ✅ Frontend
- Mapbox hiển thị bản đồ
- Markers hiển thị phòng trọ
- Click marker xem chi tiết
- Dynamic loading khi di chuyển map
- Basic filters (price, room_type)

### ✅ Integration
- Frontend ↔ Backend API hoạt động
- CORS configured
- Error handling cơ bản

---

## 🚫 OUT OF SCOPE (để Phase 2+)

- ❌ User authentication
- ❌ Đăng ký xem phòng (form submission)
- ❌ Đặt cọc payment
- ❌ Đăng phòng mới (create room)
- ❌ Marker clustering
- ❌ Advanced caching
- ❌ Image uploads
- ❌ User favorites
- ❌ Reviews/ratings

---

## ⚠️ RISKS & MITIGATION

| Risk | Mitigation |
|------|------------|
| PostGIS setup phức tạp | Follow official PostGIS + Rails tutorial, có sẵn nhiều guide |
| Mapbox API quota limit | Use free tier (50k requests/month) là đủ cho dev |
| CORS issues | Config ngay từ đầu, test với curl |
| Performance với nhiều markers | Giới hạn kết quả API (max 100 rooms), sẽ optimize ở Phase 3 |
| Time overrun | Giảm bớt styling, focus vào functionality first |

---

## 📝 NOTES

### Tips để đúng timeline:
1. **Không over-engineer**: Dùng vanilla JS thay vì React nếu chưa quen
2. **Copy-paste smartly**: Tận dụng Mapbox examples
3. **Test ngay**: Đừng code xong hết mới test
4. **Commit thường xuyên**: Để rollback dễ dàng
5. **Deploy sớm**: Deploy backend lên cloud từ ngày 1 buổi chiều

### Recommended Tools:
- **Code Editor**: VS Code
- **API Testing**: Postman hoặc Thunder Client (VS Code extension)
- **Database GUI**: pgAdmin hoặc TablePlus
- **Deployment**: Railway.app hoặc Render.com (free tier)

### Sample Tech Stack Versions:
- Ruby: 3.2+
- Rails: 7.1+
- PostgreSQL: 14+
- PostGIS: 3.3+
- Mapbox GL JS: 2.15+

---

## 🎯 SUCCESS CRITERIA

MVP được coi là thành công khi:
- ✅ User có thể xem bản đồ
- ✅ Có ít nhất 20 markers hiển thị
- ✅ Click marker → thấy thông tin phòng
- ✅ Kéo map → markers update theo vùng
- ✅ Filter theo giá → markers update
- ✅ Không có major bugs
- ✅ Code clean, có comments
- ✅ README hướng dẫn đầy đủ

---

**Prepared by:** AI Assistant
**Last updated:** Dec 30, 2025
**Estimated total hours:** 16h (2 days × 8h)

