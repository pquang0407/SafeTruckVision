<div align="center">

# SafeTruckVision: Hệ Thống Thị Giác Máy Tính Giám Sát và Cảnh Báo Điểm Mù Xe Tải Thời Gian Thực

**Hệ thống Nhận diện Điểm Mù Xe tải ứng dụng Mô hình Học Sâu & Phân tích Đa Vùng ROI Hướng Không Gian 3D**

[![Python](https://img.shields.io/badge/Python-3.9%2B-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![PyTorch](https://img.shields.io/badge/PyTorch-2.0%2B-EE4C2C?style=flat-square&logo=pytorch&logoColor=white)](https://pytorch.org/)
[![Ultralytics YOLO](https://img.shields.io/badge/Ultralytics-YOLOv8%20%7C%20YOLOv11-00FFFF?style=flat-square)](https://github.com/ultralytics/ultralytics)
[![OpenCV](https://img.shields.io/badge/OpenCV-Computer%20Vision-5C3EE8?style=flat-square&logo=opencv&logoColor=white)](https://opencv.org/)
[![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)

**Đề tài Nghiên cứu Khoa học Sinh viên – Trường Đại học Thủy Lợi (Phân hiệu TP. Hồ Chí Minh)**  
**Bộ môn:** Công nghệ Thông tin | **Lớp:** S27-66TTNT  

[Báo cáo Nghiên cứu (DOCX)](./docs/Bao_Cao_NCKH_SafeTruckVision.docx) • [Cấu hình Vùng Nguy hiểm (ROI)](./src/configs/roi_definitions.json) • [Hướng dẫn Cài đặt](#-hướng-dẫn-cài-đặt-và-chạy-thử)

</div>

---

## 📌 1. TỔNG QUAN ĐỀ TÀI

Tai nạn giao thông liên quan đến xe tải và xe siêu trường, siêu trọng phần lớn bắt nguồn từ **hiện tượng điểm mù (Blind Spots)** — những vùng không gian xung quanh xe mà tài xế hoàn toàn không thể quan sát trực tiếp qua gương chiếu hậu hoặc kính chắn gió.

**SafeTruckVision** là giải pháp thị giác máy tính thông minh kết hợp mạng học sâu hiện đại (**YOLO**) với kỹ thuật **hình chiếu phối cảnh đa chiều (Homography/Perspective Transformation)** nhằm:
1. Phát hiện và phân loại tức thời các đối tượng giao thông dễ bị tổn thương (Người đi bộ, xe máy, xe đạp, ô tô con) xung quanh xe tải.
2. Thiết lập động các vùng cảnh báo nguy hiểm (**ROI - Region of Interest**) theo từng góc nhìn: Phía trước (Front), Bên hông (Side - Điểm mù nguy hiểm nhất) và Phía sau (Rear).
3. Đưa ra tín hiệu cảnh báo va chạm theo thời gian thực (Real-time Blind Spot Collision Warning) với độ trễ cực thấp.

---

## 🏗️ 2. KIẾN TRÚC HỆ THỐNG VÀ LUỒNG XỬ LÝ (PIPELINE)

```text
  [ Camera góc rộng / Video Stream ]
                 │
                 ▼
     ┌────────────────────────┐
     │   Pre-processing Unit  │ (Resize, Chuẩn hóa màu, Trích xuất Frame)
     └───────────┬────────────┘
                 │
                 ▼
     ┌────────────────────────┐
     │   Object Detection     │ (YOLOv8 / YOLOv11 Engine)
     │  (Bbox, Class, Conf)   │ Bounding box định vị Người đi bộ, Xe máy, Xe con
     └───────────┬────────────┘
                 │
                 ▼
     ┌────────────────────────┐
     │ Bottom-Center Mapping  │ Tính toán tọa độ tiếp đất của bánh xe/chân đối tượng
     └───────────┬────────────┘
                 │
                 ▼
     ┌────────────────────────┐
     │  Polygon ROI Checking  │ Kiểm tra điểm tiếp đất thuộc Vùng Nguy Hiểm (Danger Zone)
     │ (Ray-Casting Algorithm)│ hay Vùng Chú Ý (Caution Zone)
     └───────────┬────────────┘
                 │
                 ▼
     ┌────────────────────────┐
     │ Collision Alarm Engine │ ⚠️ Bật còi/đèn cảnh báo tài xế & Hiển thị HUD Dashboard
     └────────────────────────┘
```

---

## 📊 3. ĐIỂM NỔI BẬT & KẾT QUẢ THỰC NGHIỆM

- **Mô hình cốt lõi:** Huấn luyện và tinh chỉnh trên tập dữ liệu đặc thù giao thông hỗn hợp tại Việt Nam (xe máy di chuyển áp sát thân xe tải).
- **Phân định đa vùng ROI đa cấp độ:**
  - `CRITICAL_ZONE` (Vùng đỏ - Nguy cấp): Khoảng cách $< 1.5\text{m}$ cạnh sườn xe $\rightarrow$ Kích hoạt cảnh báo ngay lập tức.
  - `WARNING_ZONE` (Vùng vàng - Cảnh giác): Khoảng cách $1.5\text{m} - 3.5\text{m}$.
- **Tốc độ xử lý:** Đạt **$\ge 35\text{ FPS}$** trên GPU NVIDIA RTX và tối ưu hóa chạy ổn định trên các thiết bị nhúng phục vụ triển khai thực tế trên cabin xe.

---

## 📂 4. CẤU TRÚC THƯ MỤC DỰ ÁN

```text
SafeTruckVision/
├── docs/                                      # Tài liệu & Báo cáo nghiên cứu khoa học hoàn chỉnh
│   └── Bao_Cao_NCKH_SafeTruckVision.docx      # Bản báo cáo học thuật đầy đủ nộp Hội đồng NCKH
├── src/                                       # Mã nguồn lõi của hệ thống
│   ├── configs/                               
│   │   └── roi_definitions.json               # Cấu hình đa giác các vùng điểm mù nguy hiểm (ROI)
│   ├── data_conversion/                       # Tiền xử lý dữ liệu và biến đổi hình học
│   │   ├── main_converter.py                  # Chuẩn hóa nhãn và bộ dữ liệu huấn luyện
│   │   └── utils_projection.py                # Thuật toán chiếu phối cảnh và xác định tọa độ
│   ├── inference/                             # Module chạy kiểm thử và cảnh báo trực tiếp
│   │   ├── roi_analysis.py                    # Thuật toán kiểm tra đối tượng xâm nhập ROI
│   │   └── run_demo.py                        # Kịch bản chạy demo phát hiện điểm mù qua video/webcam
│   └── training/                              # Module huấn luyện mô hình YOLO
│       └── train.py                           # Kịch bản train mô hình với Ultralytics
├── notebooks/                                 # Jupyter Notebooks phục vụ EDA và thử nghiệm
├── requirements.txt                           # Danh sách các thư viện phụ thuộc
└── README.md                                  # Tài liệu giới thiệu và hướng dẫn tổng quan
```

---

## 🚀 5. HƯỚNG DẪN CÀI ĐẶT VÀ CHẠY THỬ

### 5.1. Yêu cầu môi trường
- Python `3.9+`
- NVIDIA GPU + CUDA Toolkit (khuyên dùng để đạt FPS cao nhất)

### 5.2. Cài đặt các thư viện
```bash
# Clone kho lưu trữ
git clone https://github.com/pquang0407/SafeTruckVision.git
cd SafeTruckVision

# Tạo môi trường ảo (khuyên dùng)
python -m venv venv
source venv/bin/activate  # Trên Linux/macOS
# hoặc: .\venv\Scripts\activate trên Windows

# Cài đặt dependencies
pip install -r requirements.txt
```

### 5.3. Chạy demo phát hiện điểm mù trên video/webcam
```bash
# Chạy suy luận trực tiếp với trọng số mô hình và video đầu vào
python src/inference/run_demo.py --source data/test_video.mp4 --weights weights/best.pt
```

### 5.4. Huấn luyện lại mô hình (Re-training)
```bash
python src/training/train.py --data configs/dataset.yaml --epochs 100 --imgsz 640 --batch 16
```

---

## 👥 6. NHÓM TÁC GIẢ & VAI TRÒ THỰC HIỆN

Đề tài Nghiên cứu Khoa học Sinh viên được bảo vệ thành công trước Hội đồng Khoa học Khoa Công nghệ Thông tin – Trường Đại học Thủy Lợi (Phân hiệu TP. Hồ Chí Minh):

| STT | Thành viên thực hiện | Lớp / Khóa | Vai trò trong đề tài |
| :-: | :--- | :---: | :--- |
| **01** | **Nguyễn Văn Phú Quang** | **S27-66TTNT** | **Trưởng nhóm Đề tài (Project Lead & Technical Supervisor)**<br>• Định hướng nghiên cứu, thiết kế kiến trúc hệ thống và giải pháp thị giác máy tính.<br>• Hướng dẫn xây dựng pipeline huấn luyện, kiểm thử và tích hợp thuật toán ROI.<br>• Biên soạn và chuẩn hóa toàn bộ báo cáo NCKH học thuật. |
| **02** | **Trần Anh Hải** | S27-66TTNT | Thành viên nghiên cứu (Thu thập, gán nhãn dữ liệu và thực thi mã nguồn). |
| **03** | **Nguyễn Lê Nhất** | S27-66TTNT | Thành viên nghiên cứu (Thử nghiệm mô hình và hỗ trợ tài liệu). |

**Giảng viên Hướng dẫn:**
- **ThS. Đặng Tuấn Phong** – Giảng viên Bộ môn Công nghệ Thông tin.
- **TS. Đặng Đồng Nguyên** – Giảng viên Bộ môn Công nghệ Thông tin.

---

<div align="center">
  <sub>Kho Lưu Trữ Đề Tài Nghiên Cứu Khoa Học Sinh Viên – Trường Đại học Thủy Lợi (2026)</sub>
</div>
