# 💻 CODE TEMPLATES - REALESTATE MAP

## 📁 BACKEND TEMPLATES

### 1. Migration - Create Rooms with PostGIS

```ruby
# db/migrate/XXXXXX_create_rooms.rb
class CreateRooms < ActiveRecord::Migration[7.1]
  def change
    # Enable PostGIS extension
    enable_extension 'postgis'

    create_table :rooms do |t|
      t.string :title, null: false
      t.integer :price, null: false
      t.float :area
      t.text :address
      t.float :latitude
      t.float :longitude
      t.string :room_type
      t.string :status, default: 'available'
      t.text :description

      t.timestamps
    end

    # Add PostGIS geography column
    add_column :rooms, :location, :geography, limit: { srid: 4326, type: "point" }

    # Add indexes
    add_index :rooms, :location, using: :gist
    add_index :rooms, :price
    add_index :rooms, :status
    add_index :rooms, :room_type
  end
end
```

---

### 1.5. Migration - Add Phone Field (Optional)

```ruby
# rails g migration AddPhoneToRooms phone:string
class AddPhoneToRooms < ActiveRecord::Migration[7.1]
  def change
    add_column :rooms, :phone, :string
  end
end
```

---

### 2. Room Model with PostGIS & GeoJSON Support

```ruby
# app/models/room.rb
class Room < ApplicationRecord
  # Validations
  validates :title, presence: true
  validates :price, presence: true, numericality: { greater_than: 0 }
  validates :latitude, presence: true
  validates :longitude, presence: true
  validates :status, inclusion: { in: %w[available rented] }

  # Callbacks
  before_save :set_location

  # Scopes for filtering
  scope :available, -> { where(status: 'available') }
  scope :by_room_type, ->(type) { where(room_type: type) if type.present? }
  scope :price_between, ->(min, max) {
    query = all
    query = query.where('price >= ?', min) if min.present?
    query = query.where('price <= ?', max) if max.present?
    query
  }
  scope :area_between, ->(min, max) {
    query = all
    query = query.where('area >= ?', min) if min.present?
    query = query.where('area <= ?', max) if max.present?
    query
  }

  # PostGIS queries
  scope :within_bounds, ->(north, south, east, west) {
    where(
      "location && ST_MakeEnvelope(?, ?, ?, ?, 4326)",
      west, south, east, north
    )
  }

  scope :within_radius, ->(lat, lng, radius) {
    where(
      "ST_DWithin(location, ST_MakePoint(?, ?)::geography, ?)",
      lng, lat, radius
    )
  }

  # 🆕 Convert to GeoJSON Feature (Chuẩn Mapbox)
  def to_geojson_feature
    {
      type: "Feature",
      geometry: {
        type: "Point",
        coordinates: [longitude, latitude] # [lng, lat] - Important!
      },
      properties: {
        id: id,
        title: title,
        price: price,
        area: area,
        address: address,
        roomType: room_type,
        status: status,
        description: description,
        phone: phone,
        phoneFormatted: phone_formatted
      }
    }
  end

  # 🆕 Convert collection to GeoJSON FeatureCollection
  def self.to_geojson_feature_collection(rooms)
    {
      type: "FeatureCollection",
      features: rooms.map(&:to_geojson_feature)
    }
  end

  # Format phone number
  def phone_formatted
    return nil unless phone.present?
    phone.gsub(/(\d{3})(\d{4})(\d{4})/, '(\1) \2-\3')
  end

  private

  def set_location
    if latitude.present? && longitude.present?
      self.location = "POINT(#{longitude} #{latitude})"
    end
  end
end
```

---

### 3. Rooms Controller with Filters

```ruby
# app/controllers/api/v1/rooms_controller.rb
module Api
  module V1
    class RoomsController < ApplicationController
      before_action :set_pagination

      def index
        @rooms = Room.all

        # Apply filters
        @rooms = apply_geo_filters(@rooms)
        @rooms = apply_attribute_filters(@rooms)

        # Limit results
        @rooms = @rooms.limit(100)

        # 🆕 Return GeoJSON FeatureCollection
        render json: Room.to_geojson_feature_collection(@rooms)
      end

      def show
        @room = Room.find(params[:id])
        # 🆕 Return GeoJSON Feature
        render json: @room.to_geojson_feature
      rescue ActiveRecord::RecordNotFound
        render json: { error: 'Room not found' }, status: :not_found
      end

      private

      def set_pagination
        @page = params[:page] || 1
        @per_page = [params[:per_page].to_i, 100].min
        @per_page = 50 if @per_page <= 0
      end

      def apply_geo_filters(scope)
        # Bounding box filter
        if params[:north] && params[:south] && params[:east] && params[:west]
          scope = scope.within_bounds(
            params[:north].to_f,
            params[:south].to_f,
            params[:east].to_f,
            params[:west].to_f
          )
        # Radius filter
        elsif params[:lat] && params[:lng] && params[:radius]
          scope = scope.within_radius(
            params[:lat].to_f,
            params[:lng].to_f,
            params[:radius].to_f
          )
        end

        scope
      end

      def apply_attribute_filters(scope)
        # Price filter
        scope = scope.price_between(params[:min_price], params[:max_price])

        # Area filter
        scope = scope.area_between(params[:min_area], params[:max_area])

        # Room type filter
        scope = scope.by_room_type(params[:room_type])

        # Status filter
        scope = scope.where(status: params[:status]) if params[:status].present?

        scope
      end

    end
  end
end
```

---

### 4. Seeds with Sample Data

```ruby
# db/seeds.rb
puts "Clearing existing data..."
Room.destroy_all

puts "Creating rooms..."

# Hanoi locations
hanoi_rooms = [
  { title: "Studio cozy gần Hồ Tây", lat: 21.0545, lng: 105.8189, price: 3500000, area: 25, type: "studio", phone: "02438345678" },
  { title: "Phòng trọ sinh viên Đống Đa", lat: 21.0245, lng: 105.8412, price: 2000000, area: 18, type: "room", phone: "02438123456" },
  { title: "Căn hộ 1PN Cầu Giấy", lat: 21.0333, lng: 105.7943, price: 5000000, area: 45, type: "apartment", phone: "02437654321" },
  { title: "Phòng đẹp có ban công Hai Bà Trưng", lat: 21.0122, lng: 105.8589, price: 3000000, area: 22, type: "room", phone: "02438567890" },
  { title: "Studio full nội thất Tây Hồ", lat: 21.0652, lng: 105.8231, price: 4500000, area: 30, type: "studio", phone: "02438234567" },
  { title: "Nhà trọ giá rẻ Thanh Xuân", lat: 20.9967, lng: 105.8053, price: 1800000, area: 15, type: "room", phone: "02437890123" },
  { title: "Căn hộ dịch vụ Hoàn Kiếm", lat: 21.0285, lng: 105.8542, price: 8000000, area: 60, type: "apartment", phone: "02438901234" },
  { title: "Phòng trọ có gác Long Biên", lat: 21.0451, lng: 105.8932, price: 2500000, area: 20, type: "room", phone: "02438112233" },
  { title: "Studio view hồ Ba Đình", lat: 21.0351, lng: 105.8190, price: 4000000, area: 28, type: "studio", phone: "02438334455" },
  { title: "Phòng ở ghép Nam Từ Liêm", lat: 21.0411, lng: 105.7564, price: 1500000, area: 12, type: "room", phone: "02437556677" },
]

hanoi_rooms.each_with_index do |room_data, index|
  Room.create!(
    title: room_data[:title],
    price: room_data[:price],
    area: room_data[:area],
    address: "#{100 + index} Đường ABC, Hà Nội",
    latitude: room_data[:lat],
    longitude: room_data[:lng],
    room_type: room_data[:type],
    status: ['available', 'available', 'available', 'rented'].sample, # 75% available
    phone: room_data[:phone],
    description: "Phòng #{room_data[:type]} tại #{room_data[:title]}. Gần trường học, siêu thị, bệnh viện."
  )
end

# Add more random rooms around Hanoi
20.times do |i|
  Room.create!(
    title: "Phòng trọ ##{i + 9}",
    price: rand(1500000..7000000),
    area: rand(15..50),
    address: "#{200 + i} Đường XYZ, Hà Nội",
    latitude: 21.0285 + rand(-0.05..0.05),
    longitude: 105.8542 + rand(-0.05..0.05),
    room_type: ['room', 'studio', 'apartment'].sample,
    status: ['available', 'available', 'available', 'rented'].sample,
    phone: "024#{rand(30000000..39999999)}",
    description: "Phòng trọ tiện nghi, đầy đủ nội thất."
  )
end

puts "Created #{Room.count} rooms!"
puts "Available rooms: #{Room.available.count}"
puts "Rented rooms: #{Room.where(status: 'rented').count}"
puts "\nSample GeoJSON Feature:"
puts Room.first.to_geojson_feature.to_json
```

---

### 5. CORS Configuration

```ruby
# Gemfile - add this line
gem 'rack-cors'

# Run: bundle install
```

```ruby
# config/initializers/cors.rb
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins '*' # In production, specify your domain

    resource '*',
      headers: :any,
      methods: [:get, :post, :put, :patch, :delete, :options, :head],
      expose: ['Authorization']
  end
end
```

---

### 6. Routes

```ruby
# config/routes.rb
Rails.application.routes.draw do
  namespace :api do
    namespace :v1 do
      resources :rooms, only: [:index, :show]
    end
  end

  # Health check
  get '/health', to: proc { [200, {}, ['OK']] }
end
```

---

## 🎨 FRONTEND TEMPLATES

### 1. Complete HTML Structure

```html
<!-- public/index.html -->
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Realestate Map - Tìm phòng trên bản đồ</title>

  <!-- Mapbox GL JS -->
  <link href='https://api.mapbox.com/mapbox-gl-js/v2.15.0/mapbox-gl.css' rel='stylesheet' />
  <script src='https://api.mapbox.com/mapbox-gl-js/v2.15.0/mapbox-gl.js'></script>

  <style>
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }

    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      overflow: hidden;
    }

    #app {
      display: flex;
      height: 100vh;
    }

    #sidebar {
      width: 350px;
      background: white;
      box-shadow: 2px 0 10px rgba(0,0,0,0.1);
      overflow-y: auto;
      z-index: 1;
    }

    #map {
      flex: 1;
      height: 100vh;
    }

    /* Header */
    .header {
      padding: 20px;
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      color: white;
    }

    .header h1 {
      font-size: 24px;
      margin-bottom: 5px;
    }

    .header p {
      font-size: 14px;
      opacity: 0.9;
    }

    /* Filters */
    .filters {
      padding: 20px;
      border-bottom: 1px solid #eee;
    }

    .filter-group {
      margin-bottom: 15px;
    }

    .filter-group label {
      display: block;
      font-size: 13px;
      font-weight: 600;
      margin-bottom: 5px;
      color: #333;
    }

    .filter-group input,
    .filter-group select {
      width: 100%;
      padding: 10px;
      border: 1px solid #ddd;
      border-radius: 6px;
      font-size: 14px;
    }

    .filter-row {
      display: flex;
      gap: 10px;
    }

    .filter-row .filter-group {
      flex: 1;
    }

    button.apply-filters {
      width: 100%;
      padding: 12px;
      background: #667eea;
      color: white;
      border: none;
      border-radius: 6px;
      font-size: 14px;
      font-weight: 600;
      cursor: pointer;
      transition: background 0.3s;
    }

    button.apply-filters:hover {
      background: #5568d3;
    }

    /* Room Detail */
    .room-detail {
      padding: 20px;
      display: none;
    }

    .room-detail.active {
      display: block;
    }

    .room-detail h2 {
      font-size: 20px;
      margin-bottom: 15px;
      color: #333;
    }

    .room-info {
      margin-bottom: 12px;
    }

    .room-info label {
      font-weight: 600;
      color: #666;
      display: block;
      font-size: 13px;
      margin-bottom: 3px;
    }

    .room-info .value {
      font-size: 16px;
      color: #333;
    }

    .price {
      font-size: 24px !important;
      color: #667eea;
      font-weight: bold;
    }

    .back-btn {
      padding: 10px 20px;
      background: #eee;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      margin-top: 20px;
    }

    /* Loading */
    .loading {
      padding: 20px;
      text-align: center;
      color: #667eea;
      font-size: 14px;
    }

    /* Mapbox Popup */
    .mapboxgl-popup-content {
      padding: 15px;
      min-width: 200px;
    }

    .popup-title {
      font-weight: bold;
      margin-bottom: 8px;
      font-size: 16px;
    }

    .popup-price {
      color: #667eea;
      font-weight: bold;
      font-size: 18px;
      margin-bottom: 5px;
    }

    .popup-info {
      font-size: 14px;
      color: #666;
      margin-bottom: 3px;
    }

    /* Responsive */
    @media (max-width: 768px) {
      #app {
        flex-direction: column;
      }

      #sidebar {
        width: 100%;
        height: 40vh;
      }

      #map {
        height: 60vh;
      }
    }
  </style>
</head>
<body>
  <div id="app">
    <!-- Sidebar -->
    <div id="sidebar">
      <div class="header">
        <h1>🏠 Realestate Map</h1>
        <p>Tìm phòng trọ trên bản đồ</p>
      </div>

      <!-- Filters -->
      <div class="filters">
        <div class="filter-row">
          <div class="filter-group">
            <label>Giá tối thiểu</label>
            <input type="number" id="min-price" placeholder="VD: 2000000">
          </div>
          <div class="filter-group">
            <label>Giá tối đa</label>
            <input type="number" id="max-price" placeholder="VD: 5000000">
          </div>
        </div>

        <div class="filter-row">
          <div class="filter-group">
            <label>Diện tích tối thiểu</label>
            <input type="number" id="min-area" placeholder="VD: 20">
          </div>
          <div class="filter-group">
            <label>Loại phòng</label>
            <select id="room-type">
              <option value="">Tất cả</option>
              <option value="room">Phòng trọ</option>
              <option value="studio">Studio</option>
              <option value="apartment">Căn hộ</option>
            </select>
          </div>
        </div>

        <button class="apply-filters" onclick="applyFilters()">Áp dụng bộ lọc</button>
      </div>

      <!-- Room Detail -->
      <div class="room-detail" id="room-detail">
        <button class="back-btn" onclick="hideRoomDetail()">← Quay lại</button>
        <h2 id="detail-title"></h2>
        <div class="room-info">
          <label>Giá thuê</label>
          <div class="value price" id="detail-price"></div>
        </div>
        <div class="room-info">
          <label>Diện tích</label>
          <div class="value" id="detail-area"></div>
        </div>
        <div class="room-info">
          <label>Địa chỉ</label>
          <div class="value" id="detail-address"></div>
        </div>
        <div class="room-info">
          <label>Loại phòng</label>
          <div class="value" id="detail-type"></div>
        </div>
        <div class="room-info">
          <label>Trạng thái</label>
          <div class="value" id="detail-status"></div>
        </div>
        <div class="room-info" id="phone-row" style="display: none;">
          <label>Số điện thoại</label>
          <div class="value" id="detail-phone"></div>
        </div>
      </div>

      <!-- Loading -->
      <div class="loading" id="loading" style="display: none;">
        Đang tải dữ liệu...
      </div>
    </div>

    <!-- Map -->
    <div id="map"></div>
  </div>

  <script src="app.js"></script>
</body>
</html>
```

---

### 2. Complete JavaScript

```javascript
// public/app.js

// Configuration
const CONFIG = {
  MAPBOX_TOKEN: 'YOUR_MAPBOX_TOKEN_HERE', // Replace with your token
  API_BASE_URL: 'http://localhost:3000/api/v1',
  DEFAULT_CENTER: [105.8542, 21.0285], // Hanoi [lng, lat]
  DEFAULT_ZOOM: 12
};

// State
let map;
let markers = [];
let currentFilters = {};

// Initialize map
mapboxgl.accessToken = CONFIG.MAPBOX_TOKEN;

map = new mapboxgl.Map({
  container: 'map',
  style: 'mapbox://styles/mapbox/streets-v12',
  center: CONFIG.DEFAULT_CENTER,
  zoom: CONFIG.DEFAULT_ZOOM
});

// Add controls
map.addControl(new mapboxgl.NavigationControl());
map.addControl(new mapboxgl.FullscreenControl());

// Add geolocate control
map.addControl(
  new mapboxgl.GeolocateControl({
    positionOptions: {
      enableHighAccuracy: true
    },
    trackUserLocation: true
  })
);

// Load rooms when map is ready
map.on('load', () => {
  loadRooms();
});

// Reload rooms when map moves
let moveTimeout;
map.on('moveend', () => {
  clearTimeout(moveTimeout);
  moveTimeout = setTimeout(() => {
    loadRooms();
  }, 300); // Debounce 300ms
});

// 🆕 Fetch rooms from API - returns GeoJSON
async function loadRooms() {
  try {
    showLoading(true);

    const bounds = map.getBounds();
    const url = buildApiUrl(bounds, currentFilters);

    const response = await fetch(url);
    if (!response.ok) throw new Error('API request failed');

    const geojson = await response.json(); // GeoJSON FeatureCollection
    console.log('Loaded GeoJSON:', geojson);

    renderMarkers(geojson.features || []); // Array of features

    showLoading(false);
  } catch (error) {
    console.error('Error loading rooms:', error);
    showLoading(false);
    alert('Không thể tải dữ liệu phòng. Vui lòng thử lại.');
  }
}

// Build API URL with filters
function buildApiUrl(bounds, filters) {
  const params = new URLSearchParams({
    north: bounds.getNorth(),
    south: bounds.getSouth(),
    east: bounds.getEast(),
    west: bounds.getWest()
  });

  if (filters.minPrice) params.append('min_price', filters.minPrice);
  if (filters.maxPrice) params.append('max_price', filters.maxPrice);
  if (filters.minArea) params.append('min_area', filters.minArea);
  if (filters.roomType) params.append('room_type', filters.roomType);

  return `${CONFIG.API_BASE_URL}/rooms?${params.toString()}`;
}

// 🆕 Render markers from GeoJSON features
function renderMarkers(features) {
  // Clear old markers
  markers.forEach(marker => marker.remove());
  markers = [];

  // Create new markers from GeoJSON features
  features.forEach(feature => {
    const { coordinates } = feature.geometry; // [lng, lat]
    const props = feature.properties;

    const el = createMarkerElement(props);

    const popup = new mapboxgl.Popup({ offset: 25 })
      .setHTML(createPopupHTML(props));

    const marker = new mapboxgl.Marker(el)
      .setLngLat(coordinates) // GeoJSON format: [lng, lat]
      .setPopup(popup)
      .addTo(map);

    // Click marker to show detail
    el.addEventListener('click', () => {
      showRoomDetail(props);
    });

    markers.push(marker);
  });

  console.log(`Rendered ${markers.length} markers`);
}

// Create custom marker element
function createMarkerElement(props) {
  const el = document.createElement('div');
  el.className = 'custom-marker';
  el.style.cssText = `
    background-color: ${props.status === 'available' ? '#667eea' : '#999'};
    width: 30px;
    height: 30px;
    border-radius: 50%;
    border: 3px solid white;
    box-shadow: 0 2px 8px rgba(0,0,0,0.3);
    cursor: pointer;
    transition: transform 0.2s;
  `;

  el.addEventListener('mouseenter', () => {
    el.style.transform = 'scale(1.2)';
  });

  el.addEventListener('mouseleave', () => {
    el.style.transform = 'scale(1)';
  });

  return el;
}

// Create popup HTML from GeoJSON properties
function createPopupHTML(props) {
  const roomTypes = {
    room: 'Phòng trọ',
    studio: 'Studio',
    apartment: 'Căn hộ'
  };

  return `
    <div class="popup-title">${props.title}</div>
    <div class="popup-price">${formatPrice(props.price)} VNĐ</div>
    <div class="popup-info">📐 ${props.area} m²</div>
    <div class="popup-info">🏠 ${roomTypes[props.roomType] || props.roomType}</div>
    ${props.phoneFormatted ? `<div class="popup-info">📞 ${props.phoneFormatted}</div>` : ''}
    <div class="popup-info" style="font-size: 12px; margin-top: 8px;">Click để xem chi tiết</div>
  `;
}

// Show room detail in sidebar from GeoJSON properties
function showRoomDetail(props) {
  const roomTypes = {
    room: 'Phòng trọ',
    studio: 'Studio',
    apartment: 'Căn hộ'
  };

  const statuses = {
    available: '✅ Còn trống',
    rented: '❌ Đã cho thuê'
  };

  document.getElementById('detail-title').textContent = props.title;
  document.getElementById('detail-price').textContent = formatPrice(props.price) + ' VNĐ/tháng';
  document.getElementById('detail-area').textContent = props.area + ' m²';
  document.getElementById('detail-address').textContent = props.address;
  document.getElementById('detail-type').textContent = roomTypes[props.roomType] || props.roomType;
  document.getElementById('detail-status').textContent = statuses[props.status] || props.status;

  // Show phone if available
  const phoneRow = document.getElementById('phone-row');
  if (props.phoneFormatted) {
    document.getElementById('detail-phone').textContent = props.phoneFormatted;
    phoneRow.style.display = 'block';
  } else {
    phoneRow.style.display = 'none';
  }

  document.getElementById('room-detail').classList.add('active');
  document.querySelector('.filters').style.display = 'none';
}

// Hide room detail
function hideRoomDetail() {
  document.getElementById('room-detail').classList.remove('active');
  document.querySelector('.filters').style.display = 'block';
}

// Apply filters
function applyFilters() {
  currentFilters = {
    minPrice: document.getElementById('min-price').value,
    maxPrice: document.getElementById('max-price').value,
    minArea: document.getElementById('min-area').value,
    roomType: document.getElementById('room-type').value
  };

  loadRooms();
}

// Show/hide loading
function showLoading(show) {
  document.getElementById('loading').style.display = show ? 'block' : 'none';
}

// Format price
function formatPrice(price) {
  return new Intl.NumberFormat('vi-VN').format(price);
}

// Initial load
console.log('Realestate Map initialized with GeoJSON support!');
console.log('Map center:', CONFIG.DEFAULT_CENTER);
```

---

## 🔧 ENVIRONMENT TEMPLATES

### .env.example

```bash
# Database
DATABASE_HOST=localhost
DATABASE_USERNAME=postgres
DATABASE_PASSWORD=your_password

# Rails
RAILS_ENV=development
SECRET_KEY_BASE=your_secret_key_here

# Mapbox (for reference, used in frontend)
MAPBOX_ACCESS_TOKEN=your_mapbox_token_here
```

---

### Gemfile additions

```ruby
# Add these gems
gem 'rack-cors' # CORS
gem 'kaminari' # Pagination
gem 'rgeo' # Geographic data
gem 'rgeo-geojson' # GeoJSON support
gem 'activerecord-postgis-adapter' # PostGIS for ActiveRecord
```

---

## 🧪 TEST COMMANDS

### Test API Endpoints

```bash
# Get all rooms
curl http://localhost:3000/api/v1/rooms

# Get rooms with bounding box
curl "http://localhost:3000/api/v1/rooms?north=21.04&south=21.02&east=105.86&west=105.84"

# Get rooms with filters
curl "http://localhost:3000/api/v1/rooms?min_price=2000000&max_price=5000000&room_type=studio"

# Get single room
curl http://localhost:3000/api/v1/rooms/1
```

---

**Lưu ý quan trọng:**
1. Thay `YOUR_MAPBOX_TOKEN_HERE` bằng token thật từ mapbox.com
2. Update `API_BASE_URL` nếu deploy backend
3. Tùy chỉnh colors, styles theo ý thích
4. Add more fields vào Room model nếu cần (images, amenities, etc.)

