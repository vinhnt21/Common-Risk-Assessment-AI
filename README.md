# Kế hoạch triển khai AI phát hiện rủi ro tàu cá

## Xác định nghiệp vụ
- Định nghĩa rủi ro
  - Thế nào là `Bình thường`, `Theo dõi`, `Rủi ro`, `Rủi ro cao`
- Xác định input, output của mô hình
- Xác định tiêu chí đánh giá:
  - Giảm thời gian kiểm tra
  - Tính chính xác của mô hình so với thực tế
  - ... **tùy nghiệp vụ**

## Pipeline xử lý data
### Các nguồn dữ liệu tiềm năng
- Dữ liệu nội bộ
- API, Web Service của các bộ ngành liên quan 
- Các dữ liệu bên ngoài: lịch sử thời tiết, danh sách đen,...
### Flow chung
- Thu thập dữ liệu từ các nguồn
- Làm sạch, đồng bộ thời gian và các trường dữ liệu, chuẩn hóa schema
- Trích chọn các trường dữ liệu cần thiết 
  - Đặc tính tĩnh của tàu: Tuổi tàu, công suất máy, loại ngư cụ đăng ký, lịch sử công ty sở hữu.
  - Đặc tính động của hành trình: Tần suất tắt/bật AIS, thời gian trung bình của chuyến đi, khu vực hoạt động thường xuyên, số ngày trên biển so với lượng cá báo cáo.
  - Đặc tính về con người & pháp lý: Lịch sử vi phạm của chủ tàu/thuyền trưởng, số lượng lao động so với đăng ký, tần suất thay đổi thuyền viên.
  - ... **tùy nghiệp vụ**
- Tạo dữ liệu huấn luyện
```mermaid
graph LR
    %% --- Định nghĩa Style ---
    classDef dataNode fill:#e6f3ff,stroke:#007bff,stroke-width:2px,color:#000;
    classDef processNode fill:#d4edda,stroke:#28a745,stroke-width:2px,color:#000;
    classDef finalData fill:#fff3cd,stroke:#ffc107,stroke-width:2px,color:#000;

    %% --- Các bước trong Pipeline ---
    A["fa:fa-database Thu thập dữ liệu"]:::dataNode
    B["fa:fa-broom Làm sạch & Đồng bộ"]:::processNode
    C["fa:fa-cogs Trích chọn đặc trưng"]:::processNode
    D["fa:fa-table Dữ liệu huấn luyện"]:::finalData

    A --> B --> C --> D
```

## Xây dựng mô hình
### Mục tiêu
#### 1. Mô hình phân loại theo nhãn  (Bình thường / Theo dõi / Rủi ro / Rủi ro cao)
#### 2. Mô hình phát hiện bất thường

### Flow tổng thể
![](./flow.svg)

#### Giai đoạn 1: Xây dựng Model Nền tảng
- Tận dụng bộ dữ liệu nhỏ đã được gán nhãn ban đầu để huấn luyện một mô hình phân loại có giám sát (Supervised Model v1). Mục tiêu của giai đoạn này là nhanh chóng có được một mô hình cơ bản (baseline) để làm nền tảng và đánh giá ban đầu.
#### Giai đoạn 2: Vòng lặp Làm giàu Dữ liệu (Human-in-the-loop)
- Áp dụng một mô hình không giám sát (Unsupervised Model) trên toàn bộ dữ liệu chưa có nhãn để tự động phát hiện các trường hợp bất thường, tiềm ẩn rủi ro.
- Những "tàu đáng ngờ" này sau đó được chuyển cho chuyên gia đánh giá và gán nhãn chính xác. Quy trình này giúp tạo ra một nguồn dữ liệu có nhãn mới, chất lượng cao một cách hiệu quả.


#### Giai đoạn 3: Cải thiện Model
- Kết hợp dữ liệu có nhãn ban đầu và dữ liệu mới được gán nhãn từ Giai đoạn 2 để tạo ra một tập huấn luyện (Tập dữ liệu huấn luyện v2) lớn và phong phú hơn.
- Dữ liệu này được dùng để huấn luyện lại mô hình phân loại, tạo ra phiên bản nâng cao (Model v2) với độ chính xác và khả năng nhận diện rủi ro tốt hơn đáng kể.

#### Giai đoạn 4: Cải tiến liên tục với dữ liệu mới
- Triển khai Model v2 vào hệ thống để đưa ra các dự đoán rủi ro trong thời gian thực, hỗ trợ chuyên gia ra quyết định.
- Hệ thống liên tục nhận dữ liệu mới, vừa để dự đoán, vừa để đưa vào vòng lặp phát hiện bất thường ở Giai đoạn 2 và giám sát hiệu quả của mô hình. Mô hình sẽ được huấn luyện lại định kỳ để đảm bảo luôn cập nhật và chính xác theo thời gian.