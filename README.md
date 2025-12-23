# HỆ THỐNG API ĐỊNH DANH BỆNH NHÂN BẰNG NHẬN DIỆN KHUÔN MẶT  
# VÀ ĐIỀU PHỐI HÀNG ĐỢI THỜI GIAN THỰC

---

## TÓM TẮT
Repository này trình bày quá trình nghiên cứu và xây dựng một hệ thống API phục vụ **định danh bệnh nhân bằng nhận diện khuôn mặt** kết hợp **bốc số và điều phối hàng đợi khám bệnh theo thời gian thực**. Hệ thống được thiết kế theo hướng kiến trúc API-centric, cho phép tích hợp linh hoạt với kiosk tiếp nhận, bảng hiển thị và các hệ thống quản lý y tế hiện có.

Giải pháp sử dụng **mô hình học sâu ArcFace (InsightFace)** để trích xuất đặc trưng khuôn mặt, kết hợp **cơ sở dữ liệu vector FAISS** nhằm tối ưu tốc độ so khớp. Đồng thời, cơ chế **WebSocket** được áp dụng để đảm bảo cập nhật trạng thái hàng đợi theo thời gian thực với độ trễ thấp. Kết quả thử nghiệm cho thấy hệ thống đáp ứng tốt các yêu cầu về độ chính xác, tính ổn định và khả năng mở rộng trong bối cảnh y tế.

---

## 1. ĐẶT VẤN ĐỀ
Trong các cơ sở y tế, quy trình tiếp nhận bệnh nhân và quản lý hàng đợi vẫn còn phụ thuộc nhiều vào thao tác thủ công, dẫn đến tình trạng ùn tắc, sai sót định danh và trải nghiệm người bệnh chưa tối ưu. Việc ứng dụng công nghệ nhận diện khuôn mặt kết hợp hệ thống điều phối hàng đợi theo thời gian thực là một hướng tiếp cận phù hợp với xu thế chuyển đổi số y tế hiện nay.

Đề tài tập trung giải quyết hai bài toán chính:
- Định danh bệnh nhân nhanh chóng và chính xác mà không phụ thuộc vào thẻ giấy hay mã thủ công
- Quản lý và điều phối hàng đợi khám bệnh một cách linh hoạt, minh bạch và hiệu quả

---

## 2. CƠ SỞ LÝ THUYẾT: AI – HỌC MÁY – HỌC SÂU

**Trí tuệ nhân tạo (Artificial Intelligence – AI)** là lĩnh vực nghiên cứu trong khoa học máy tính nhằm xây dựng các hệ thống có khả năng thực hiện những nhiệm vụ vốn đòi hỏi trí tuệ con người, như nhận thức, suy luận, ra quyết định và tương tác với môi trường. AI là khái niệm bao trùm, không chỉ giới hạn ở các phương pháp học từ dữ liệu mà còn bao gồm các hệ thống dựa trên luật, logic và tri thức chuyên gia.

**Học máy (Machine Learning – ML)** là một nhánh của AI, tập trung vào việc phát triển các thuật toán cho phép hệ thống tự động học ra quy luật từ dữ liệu thông qua trải nghiệm, thay vì được lập trình tường minh bằng các tập luật cố định.

**Học sâu (Deep Learning – DL)** là một nhánh con của học máy, sử dụng mạng nơ-ron nhân tạo nhiều tầng để học các biểu diễn dữ liệu ở nhiều mức trừu tượng khác nhau. Điểm khác biệt cốt lõi của học sâu là khả năng tự động trích xuất đặc trưng từ dữ liệu thô, giúp đạt hiệu quả cao trong các bài toán thị giác máy tính và nhận diện khuôn mặt.

Trong phạm vi đề tài này, hệ thống được xây dựng dựa trên **mô hình học sâu ArcFace**, thuộc lĩnh vực **học máy**, và là một ứng dụng cụ thể của **trí tuệ nhân tạo trong y tế**.

---

## 3. KIẾN TRÚC TỔNG THỂ HỆ THỐNG

![Kiến trúc hệ thống](figures/system_architecture.png)

**Hình 1.** Kiến trúc tổng thể hệ thống.

Hệ thống được thiết kế theo mô hình phân lớp, bao gồm:
- **Lớp thiết bị đầu cuối:** kiosk tiếp nhận bệnh nhân, camera, bảng hiển thị
- **Lớp server ứng dụng:** các API xử lý nhận diện khuôn mặt và hàng đợi
- **Lớp xử lý nhận diện:** mô-đun Face ID hoạt động độc lập
- **Lớp dữ liệu:** cơ sở dữ liệu quan hệ và cơ sở dữ liệu vector FAISS

Thiết kế này giúp hệ thống dễ mở rộng, dễ bảo trì và thuận tiện tích hợp với các hệ thống HIS/EHR trong tương lai.

---

## 4. XÂY DỰNG VÀ TRIỂN KHAI HỆ THỐNG API

### 4.1. Định hướng thiết kế
Hệ thống được xây dựng theo hướng **API-first**, trong đó mọi chức năng đều được truy cập thông qua các API chuẩn hóa. Cách tiếp cận này cho phép tách biệt giao diện người dùng và logic nghiệp vụ, đồng thời hỗ trợ nhiều loại client khác nhau.

Backend được triển khai bằng **FastAPI**, hỗ trợ xử lý bất đồng bộ và đáp ứng tốt các yêu cầu thời gian thực.

---

### 4.2. Mô-đun nhận diện khuôn mặt

![Pipeline Face ID](figures/face_pipeline.png)

**Hình 2.** Quy trình nhận diện khuôn mặt.

Pipeline nhận diện gồm ba bước:
1. Phát hiện và chuẩn hóa khuôn mặt
2. Trích xuất vector đặc trưng bằng ArcFace (512 chiều)
3. So khớp định danh bằng FAISS

Hệ thống chỉ lưu trữ vector đặc trưng thay vì ảnh gốc nhằm đảm bảo quyền riêng tư cho bệnh nhân.

---

### 4.3. Hệ thống bốc số và quản lý hàng đợi
Mô-đun hàng đợi được xây dựng như một dịch vụ độc lập, đảm nhiệm:
- Cấp số khám bệnh
- Gán bệnh nhân vào hàng đợi phù hợp
- Theo dõi và cập nhật trạng thái khám

Các xung đột khi nhiều kiosk hoạt động đồng thời được xử lý bằng khóa logic ở tầng ứng dụng.

---

### 4.4. Điều phối hàng đợi hai lớp theo thời gian thực

![Mô hình hàng đợi](figures/queue_model.png)

**Hình 3.** Mô hình điều phối hàng đợi hai lớp.

Hệ thống áp dụng mô hình:
- **Lớp dịch vụ:** phân loại theo loại hình khám
- **Lớp phòng/bác sĩ:** điều phối theo nguồn lực thực tế

Cơ chế này giúp giảm quá tải cục bộ và tối ưu hóa luồng bệnh nhân.

---

### 4.5. Cơ chế thời gian thực
WebSocket được sử dụng để cập nhật trạng thái hàng đợi theo thời gian thực, giúp giảm độ trễ, giảm tải cho server và cải thiện trải nghiệm người dùng so với cơ chế polling truyền thống.

---

## 5. THỬ NGHIỆM VÀ ĐÁNH GIÁ
Hệ thống được thử nghiệm trong môi trường mô phỏng phòng khám quy mô vừa. Kết quả cho thấy:
- Thời gian nhận diện khuôn mặt trung bình dưới 100 ms
- Độ trễ cập nhật trạng thái dưới 200 ms
- Hệ thống hoạt động ổn định với nhiều yêu cầu đồng thời

---

## 6. ĐÓNG GÓP CỦA ĐỀ TÀI
- Đề xuất kiến trúc API cho bài toán định danh bệnh nhân và quản lý hàng đợi
- Triển khai mô-đun Face ID có khả năng mở rộng
- Xây dựng mô hình điều phối hàng đợi hai lớp theo thời gian thực
- Đánh giá hiệu năng và khả năng triển khai trong bối cảnh y tế

---

## 7. HẠN CHẾ VÀ HƯỚNG PHÁT TRIỂN
**Hạn chế:**
- Hệ thống ở mức prototype
- Chưa triển khai trên môi trường bệnh viện thực tế
- Chưa tích hợp chuẩn dữ liệu y tế HIS/EHR

**Hướng phát triển:**
- Tích hợp HL7/FHIR
- Triển khai trên nền tảng cloud/edge
- Kết hợp xác thực đa yếu tố (Face ID + QR/NFC)

---

## THÔNG TIN TÁC GIẢ
**Tác giả:** Nguyễn Hoàng Khánh  
**MSSV:** B2113312  
**Ngành:** Khoa học Máy tính  
**Trường:** Đại học Cần Thơ  
**GVHD:** TS. Lưu Tiến Đạo  
**Năm:** 2025  

---

## GIẤY PHÉP
Dự án phục vụ **mục đích học tập và nghiên cứu**.  
Không sử dụng cho mục đích thương mại khi chưa có sự cho phép của tác giả.
# Hospital-API
# Hospital-API
