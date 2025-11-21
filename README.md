# Tổng quan HX771 Loadcell 
- HX711 là IC ADC 24-bit chuyên dụng cho loadcell. Nó thường đi kèm board HX711 có 6 pin ra (GND, DT, SCK, VCC, E+, E-, A+, A-, B+, B-)
- Cảm biến này không cần lập trình tương tác với thanh ghi như I2C/SPI mà chỉ cần điều khiển thông qua GPIO pin theo giao thức nối tiếp (Serial Interface)
- Đầu vào là bộ mupltiplexer có thể chọn 1 trong 2 kênh vi sai A/B để đưa vào bộ khuếch đại có độ nhiễu thấp và hệ số khuếch đại có thể lập trình (Programmable Gain Amplifier - PGA):
	- Kênh A có thể được thiết lập gain 128 hoặc 64, tương đương với mức điện áp đầu vào vi sai ±20 mV hoặc ±40 mV khi cấp nguồn 5V cho chân AVDD của bộ nguồn analog.
	- Kênh B có giá trị gain cố định là 32
- Chân clock rất linh hoạt - có thể dùng xung clock ngoài, tinh thể, hoặc sử dụng bộ dao động nội không cần linh kiện ngoài được tích hợp sẵn bên trong chip
- Bộ mạch Reset nguồn tích hợp giúp việc khởi tạo giao diện kỹ thuật số trở nên đơn giản hơn
- Tốc độ lấy mẫu (Sample rate) có thể chọn 10 SPS hoạc 80 SPS
- Khả năng loại bỏ nhiễu nguồn 50Hz và 60Hz đồng thời
	- Dòng tiêu thụ khi ở trạng thái Normal Operation < 1.5mA
	- Khi Power Down < 1uA

## Output Data Rate và Format

## Reset and Power-Down

## 1. Pinouts nối với vi điều khiển 
- Thường có 4 chân chính:

|Pin| Tên khác|Chức năng|
|---|---------|---------|
|VCC| 2.6V - 5.5V | Cấp nguồn cho HX711 (thường 3.3V - 5V)|
|GND| -- | Mass chung|
|DT| D-OUT|Chân xuất dữ liệu (Data-out). MCU đọc dữ liệu từ chân này. `Data out` của cảm biến sẽ là `data in` của MCU|
|SCK|PD_SCK|Clock để HX711 dịch bit và cũng để chọn gain/channel. Xung này được cấp từ MCU nên sẽ được coi là output từ MCU|

### Serial Interface (Giao thức nối tiếp)
- Chân PD_SCK và DOUT được dùng để truy xuất dữ liệu, lựa chọn đầu vào, độ lợi (Gain) và điều chỉnh nguồn tiêu thụ

![Images/TimingDiagram.png](Alt_text)

- Giải thích thêm về luồng hoạt động: 
	- Khi dữ liệu chưa sẵn sàng để truy xuất, tín hiệu đầu ra DOUT sẽ ở mức HIGH. Lúc này tín hiệu xung nhịp PD_SCK phải duy trì ở mức LOW.
	- Khi DOUT chuyển xuống mức LOW, điều đó cho biết dữ liệu đã sẵn sàng để đọc (có dữ liệu mới) -> MCU nhìn thấy LOW -> Bắt đầu đọc 24-bit dữ liệu
	- MCU sau đó sẽ phát ra 24 xung clock vào chân PD_SCK để dịch 24-bit dữ liệu đã có từ DOUT. Mỗi xung PD_SCK dịch ra 1 bit, bẳt đầu từ bit MSB cho đến khi 24-bit hoàn toàn được dịch ra
	- Sau khi 24-bit dữ liệu đã được dịch ra hết, MCU tiếp tục phát ra 1-2 hoặc 3 xung clock dương (xung thứ 25, 26, 27). Tổng số xung (25, 26 hoặc 27) sẽ xác định cấu hình cho lần chuyển đổi (đo lường) tiếp theo
	- Xung thứ 25 trở đi tại chân PD_SCK quyết định gain cho lần đo sau và làm chân DOUT trở lại mức HIGH
	- Số lượng xung PD_SCK không được ít hơn 25 hoặc nhiều hơn 27 trong 1 chu kỳ chuyển đổi, để tránh lỗi giao tiếp nối tiếp
- Việc chọn đầu vào (Channel A/B) và chọn gain được điều khiển bằng số lượng xung PD_SCK ở bảng dưới 

|PD_SCK Pulses|Input channel| Gain|
|-------------|-------------|-----|
|25|A|128|
|26|B|32|
|27|A|64|


## 2. Pinouts nối với loadcell
|Pin|Chức năng|
|---|---------|
|E+|Excitation+: Cấp nguồn dương cho loadccell|
|E-|Excitation-: Cấp nguồn âm cho loadcell|
|A+ (dây trắng)| Tín hiệu + tại Channel A|
|A- (dây xanh)| Tín hiệu - tại Channel A|
|B+ | Tín hiệu + tại Channel B (ít độ phân giải hơn)|
|B- | Tín hiệu - tại Channel B|

- Lưu ý: 
	- Kênh A: full 24-bit, có gain 64-128
	- Kênh B: chỉ gian 32, dùng ít hơn, độ nhạy thấp hơn
