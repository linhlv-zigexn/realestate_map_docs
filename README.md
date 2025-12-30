# 🏠 Realestate Map - Hệ thống Bản đồ tìm phòng

Dự án cho phép người dùng tìm kiếm thông tin bất động sản tại các khu vực mà họ muốn sinh sống dựa trên bản đồ Mapbox và thông tin bất động sản được đính kèm tương ứng.

## 📚 Tài liệu dự án

Project này bao gồm các tài liệu sau:

### 1. 📋 [2-DAY-DEVELOPMENT-PLAN.md](./2-DAY-DEVELOPMENT-PLAN.md)
**Kế hoạch phát triển chi tiết cho 2 ngày**
- Timeline từng giờ
- Breakdown tasks cụ thể
- Deliverables sau mỗi giai đoạn
- Risks & mitigation strategies

### 2. ✅ [CHECKLIST.md](./CHECKLIST.md)
**Checklist đơn giản để tick off**
- Format dễ follow
- Chia theo buổi sáng/chiều
- Progress tracker
- Quick reference cho stuck situations

### 3. 💻 [CODE_TEMPLATES.md](./CODE_TEMPLATES.md)
**Code templates sẵn để copy-paste**
- Backend: Models, Controllers, Routes, Seeds (with GeoJSON support)
- Frontend: Complete HTML, CSS, JavaScript (GeoJSON compatible)
- Configuration files
- Test commands

### 4. 📖 [project_overview.md](./project_overview.md)
**Tổng quan dự án**
- Mục tiêu hệ thống
- Kiến trúc tổng thể
- Database design
- API design
- Roadmap

---

## 🚀 Quick Start

### Option 1: Automated Setup (Recommended)

```bash
# Clone or navigate to project directory
cd realestate_map

# Run automated setup script
./setup.sh

# Start Rails server
cd realestate_map_api
rails s

# In another terminal, test API
curl http://localhost:3000/api/v1/rooms
```

### Option 2: Manual Setup

Follow the detailed instructions in [2-DAY-DEVELOPMENT-PLAN.md](./2-DAY-DEVELOPMENT-PLAN.md) and use [CODE_TEMPLATES.md](./CODE_TEMPLATES.md) for code snippets.

---

## 📂 Project Structure

```
realestate_map/
├── README.md                    # This file
├── project_overview.md          # Project overview & architecture
├── 2-DAY-DEVELOPMENT-PLAN.md    # Detailed 2-day development plan
├── CHECKLIST.md                 # Simple checklist to track progress
├── CODE_TEMPLATES.md            # Ready-to-use code templates (GeoJSON)
├── GEOJSON-UPDATE.md            # GeoJSON format documentation
├── QUICK-START-GEOJSON.md       # Quick start with GeoJSON
├── setup.sh                     # Automated setup script
├── data.json                    # Sample GeoJSON data structure
│
└── realestate_map_api/          # Backend API (created by setup.sh)
    ├── app/
    │   ├── models/
    │   │   └── room.rb
    │   └── controllers/
    │       └── api/
    │           └── v1/
    │               └── rooms_controller.rb
    ├── db/
    │   ├── migrate/
    │   └── seeds.rb
    ├── config/
    │   ├── routes.rb
    │   └── database.yml
    └── public/                  # Frontend files
        ├── index.html
        └── app.js
```

---

## 🛠 Tech Stack

### Backend
- **Ruby on Rails** 7.1+ (API mode)
- **PostgreSQL** 14+ with **PostGIS** extension
- **RGeo** for geographic data handling

### Frontend
- **HTML5 / CSS3 / JavaScript** (Vanilla)
- **Mapbox GL JS** 2.15+ for map rendering
- Optional: React for future enhancements

### Tools
- **Kaminari** for pagination
- **Rack-CORS** for cross-origin requests

---

## 📋 Development Workflow

### Day 1 - Backend (8 hours)
- ✅ Setup Rails + PostgreSQL + PostGIS
- ✅ Create Room model with geographic data
- ✅ Implement API endpoints with filters
- ✅ Seed sample data
- ✅ Test & configure CORS

### Day 2 - Frontend (8 hours)
- ✅ Setup Mapbox integration
- ✅ Display markers on map
- ✅ Implement dynamic loading
- ✅ Room detail view
- ✅ Search & filter UI

---

## 🎯 Features

### MVP (Phase 1) - Completed in 2 days
- [x] Display map with Mapbox
- [x] Show room markers on map
- [x] Click marker to view details
- [x] Dynamic loading when moving map
- [x] Filter by price, area, room type
- [x] Bounding box & radius queries
- [x] **GeoJSON format support** (chuẩn Mapbox)
- [x] Phone number display

### Phase 2 - Future enhancements
- [ ] User authentication
- [ ] Room registration form
- [ ] Payment integration
- [ ] Create new room listings
- [ ] Marker clustering
- [ ] Image uploads
- [ ] Reviews & ratings

---

## 🧪 Testing

### Backend API (GeoJSON Format)

```bash
# Get all rooms (GeoJSON FeatureCollection)
curl http://localhost:3000/api/v1/rooms | jq

# Verify GeoJSON structure
curl -s http://localhost:3000/api/v1/rooms | jq '.type'
# Should return: "FeatureCollection"

# Get rooms in bounding box
curl "http://localhost:3000/api/v1/rooms?north=21.04&south=21.02&east=105.86&west=105.84"

# Get rooms with price filter
curl "http://localhost:3000/api/v1/rooms?min_price=2000000&max_price=5000000"

# Get rooms by type
curl "http://localhost:3000/api/v1/rooms?room_type=studio"

# Get single room
curl http://localhost:3000/api/v1/rooms/1
```

### Frontend

1. Open `http://localhost:3000/index.html` in browser
2. Map should display with markers
3. Click markers to see details
4. Move/zoom map to load new rooms
5. Use filters to narrow results

---

## 🔧 Configuration

### Backend

1. **Database**: Edit `config/database.yml`
2. **CORS**: Edit `config/initializers/cors.rb`
3. **Seeds**: Edit `db/seeds.rb` to add more sample data

### Frontend

1. **Mapbox Token**: Get from https://mapbox.com
2. Update `CONFIG.MAPBOX_TOKEN` in `app.js`
3. **API URL**: Update `CONFIG.API_BASE_URL` if backend is deployed

---

## 🚀 Deployment

### Backend (Railway.app example)

```bash
# Install Railway CLI
npm install -g @railway/cli

# Login
railway login

# Initialize project
railway init

# Deploy
railway up

# Add database
railway add postgresql

# Set environment variables
railway variables set DATABASE_PASSWORD=your_password
```

### Frontend

- Option 1: Serve from Rails `public/` folder
- Option 2: Deploy to Vercel/Netlify
- Option 3: Use GitHub Pages

---

## 📊 Database Schema

### Table: rooms

| Field       | Type                | Description                  |
|-------------|---------------------|------------------------------|
| id          | bigint              | Primary key                  |
| title       | string              | Room title                   |
| price       | integer             | Monthly rent (VND)           |
| area        | float               | Area in m²                   |
| address     | text                | Full address                 |
| latitude    | float               | Latitude                     |
| longitude   | float               | Longitude                    |
| location    | geography(Point)    | PostGIS point                |
| room_type   | string              | room/studio/apartment        |
| status      | string              | available/rented             |
| description | text                | Description                  |
| created_at  | datetime            | Creation timestamp           |
| updated_at  | datetime            | Update timestamp             |

**Indexes:**
- GIST index on `location` (required for geo queries)
- BTREE index on `price`
- BTREE index on `status`
- BTREE index on `room_type`

---

## 🆘 Troubleshooting

### PostGIS not found
```bash
# macOS
brew install postgis

# Ubuntu
sudo apt install postgis postgresql-14-postgis-3
```

### CORS errors
- Check `config/initializers/cors.rb` is properly configured
- Restart Rails server after changing CORS config

### Mapbox map not showing
- Verify your Mapbox token is valid
- Check browser console for errors
- Ensure token has correct scopes enabled

### No markers appearing
- Check API is returning data: `curl http://localhost:3000/api/v1/rooms`
- Verify seed data exists: `rails runner 'puts Room.count'`
- Check browser console for JavaScript errors

---

## 👥 Contributors

- Development Plan: AI Assistant
- Project Overview: Product Team
- Implementation: Your Team

---

## 📝 License

This is a learning/demo project. Feel free to use and modify.

---

## 🔗 Resources

- [Mapbox GL JS Documentation](https://docs.mapbox.com/mapbox-gl-js/api/)
- [PostGIS Documentation](https://postgis.net/documentation/)
- [Rails Guides](https://guides.rubyonrails.org/)
- [RGeo GitHub](https://github.com/rgeo/rgeo)

---

## 📞 Support

Nếu gặp vấn đề trong quá trình phát triển:

1. Xem [CHECKLIST.md](./CHECKLIST.md) phần "Nếu Bị Stuck"
2. Check [CODE_TEMPLATES.md](./CODE_TEMPLATES.md) để verify code
3. Review [2-DAY-DEVELOPMENT-PLAN.md](./2-DAY-DEVELOPMENT-PLAN.md) timeline

---

**Good luck with your development! 🚀**

Last updated: Dec 30, 2025

# realestate_map_docs
