# 🏨 Hotel & Travel Management System (Microservices Architecture)

## 📘 Giới thiệu

**Hotel & Travel Management System** là nền tảng web cho phép người dùng:
- Đặt phòng khách sạn, xem thông tin chi tiết phòng, giá, đánh giá.
- Đặt tour du lịch (kết hợp phòng + tour).
- Thanh toán, quản lý booking, đánh giá dịch vụ và tích điểm khách hàng thân thiết.

Hệ thống được phát triển theo **kiến trúc microservices**, nhằm đảm bảo:
- **Khả năng mở rộng cao (Scalability)**
- **Tách biệt logic nghiệp vụ (Separation of Concerns)**
- **Dễ dàng bảo trì và triển khai (DevOps-ready)**

---

## 🧭 Mục tiêu 8 tháng phát triển

| Giai đoạn | Thời gian | Nội dung chính |
|------------|------------|----------------|
| **Phase 1 (Tháng 1–4)** | Phát triển core system | Xây dựng các microservice cơ bản (User, Hotel, Room, Search) + Config Server, Discovery Server, API Gateway |
| **Phase 2 (Tháng 5–8)** | Mở rộng & triển khai thực tế | Thêm các service nâng cao (Booking, Payment, Tour, Review, Loyalty, Notification) + Kafka + Kubernetes + CI/CD |

---

## 🧩 Kiến trúc tổng thể

```
                 ┌────────────────────┐
                 │     API Gateway    │
                 └─────────┬──────────┘
                           │
     ┌───────────┬────────┼──────────┬───────────┐
     │           │        │          │           │
┌────────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
│UserService │ │HotelSvc│ │RoomSvc │ │Search │ │Booking │
└──────┬─────┘ └────────┘ └────────┘ └────────┘ └────┬───┘
       │                                              │
       │          ┌──────────┐                        │
       │          │PaymentSvc│                        │
       │          └────┬─────┘                        │
       │               │                              │
       │          ┌────▼──────┐                       │
       │          │Kafka Bus  │◄──────────────────────┘
       │          └────┬──────┘
       │               │
       │          ┌────▼──────────────────────────────┐
       │          │ Notification, Review, Loyalty     │
       │          └───────────────────────────────────┘
       │
┌──────▼────────┐ ┌──────────────┐
│Config Server  │ │Discovery/Eureka│
└───────────────┘ └──────────────┘
```

**Databases:**
- **PostgreSQL**: transactional data (users, hotels, bookings, payments)
- **MongoDB**: non-relational data (reviews, search cache)

---

## 🧱 Các Microservice

| Service | Mô tả | Database |
|----------|-------|-----------|
| **User Service** | Quản lý thông tin người dùng, đăng ký, đăng nhập, JWT auth | PostgreSQL |
| **Hotel Service** | Quản lý khách sạn, tiện nghi, hình ảnh, mô tả | PostgreSQL |
| **Room Service** | Quản lý phòng, giá, tình trạng | PostgreSQL |
| **Search Service** | Cung cấp tìm kiếm nâng cao (phòng, khách sạn, địa điểm) | MongoDB |
| **Booking Service** | Xử lý đặt phòng, tour; gửi event qua Kafka | PostgreSQL |
| **Payment Service** | Thanh toán (giả lập gateway), xử lý event từ Kafka | PostgreSQL |
| **Tour Service** | Quản lý tour du lịch và kết hợp đặt tour + phòng | PostgreSQL |
| **Review Service** | Lưu và hiển thị đánh giá của khách hàng | MongoDB |
| **Notification Service** | Gửi email / thông báo Kafka consumer | - |
| **Loyalty Service** | Tích điểm, quản lý hạng thành viên | PostgreSQL |
| **Config Server** | Cấu hình tập trung cho toàn hệ thống | Git Config Repo |
| **Discovery Server (Eureka)** | Service registry cho microservices | - |
| **API Gateway (Spring Cloud Gateway)** | Điểm truy cập duy nhất, định tuyến API | - |

---

## 🐳 Giai đoạn 1: Docker Compose (Tháng 1–4)

### 📂 Cấu trúc thư mục:

```
/hotel-travel-system/
├── config-server/
├── discovery-server/
├── api-gateway/
├── user-service/
├── hotel-service/
├── room-service/
├── search-service/
├── docker-compose.yml
└── README.md
```

### ▶️ Cách chạy:

```bash
# Build & start all services
docker-compose up -d --build
```

---

## ☸️ Giai đoạn 2: Kubernetes + Kafka (Tháng 5–8)

### 📂 Thư mục Kubernetes:

```
/k8s/
 ├── config-server.yaml
 ├── discovery-server.yaml
 ├── api-gateway.yaml
 ├── user-service.yaml
 ├── hotel-service.yaml
 ├── booking-service.yaml
 ├── kafka-deployment.yaml
 ├── ingress.yaml
```

### 🚀 Triển khai trên Minikube:

```bash
minikube start
kubectl apply -f k8s/
kubectl get pods
kubectl get services
```

### 📨 Tích hợp Kafka

**Topics chính:**
- `booking-events`
- `payment-events`
- `notification-events`

**Luồng dữ liệu:**
```
BookingService → Kafka → PaymentService
PaymentService → Kafka → NotificationService, LoyaltyService
```
---

## 🧰 CI/CD Pipeline

Sử dụng **GitHub Actions** hoặc **Jenkins** để:
1. Tự động build và push Docker image lên Docker Hub.
2. Tự động deploy lên Kubernetes cluster.

**Ví dụ pipeline cơ bản:**

```yaml
on:
  push:
    branches: [ main ]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build Docker Image
        run: docker build -t yourname/user-service:latest ./user-service
      - name: Push Docker Image
        run: docker push yourname/user-service:latest
```

---

## 🧠 Công nghệ sử dụng

| Loại | Công nghệ |
|------|-----------|
| **Backend** | Spring Boot, Spring Cloud (Config, Eureka, Gateway, Feign, Sleuth) |
| **Database** | PostgreSQL, MongoDB |
| **Message Broker** | Apache Kafka |
| **Containerization** | Docker, Docker Compose |
| **Deployment** | Kubernetes, Helm |
| **Authentication** | JWT, Spring Security |
| **CI/CD** | GitHub Actions, Jenkins |
| **Frontend** (gợi ý mở rộng) | Next.js / Vue.js |
| **Monitoring** | Prometheus, Grafana (phase 2) |
