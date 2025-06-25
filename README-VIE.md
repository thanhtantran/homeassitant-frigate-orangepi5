
[🇺🇸 English version here](README.md)

# Home Assistant và Frigate chạy trên Orange Pi 5 bằng Docker

Frigate sử dụng FFMPEG, do đó bạn cần cài RKMMP trên Orange Pi 5 (hoặc bất kỳ phiên bản nào dùng RK3588).

Bạn có thể cài hệ điều hành Ubuntu 22.04 từ Joshua Riek (https://github.com/Joshua-Riek/ubuntu-rockchip), hệ điều hành này đã có sẵn RKMMP, kiểm tra bằng cách chạy lệnh sau:

```bash
for dev in dri dma_heap mali0 rga mpp_service \
   iep mpp-service vpu_service vpu-service \
   hevc_service hevc-service rkvdec rkvenc vepu h265e ; do \
  [ -e "/dev/$dev" ] && echo " --device /dev/$dev"; \
 done
```

Nó sẽ trả về các thiết bị như sau:
```
--device /dev/dri
--device /dev/dma_heap
--device /dev/mali0
--device /dev/rga
--device /dev/mpp_service
```

Bạn cũng nên kiểm tra `/dev/rknpu` đã sẵn sàng hay chưa bằng cách chạy `ls -l /dev/rknpu`.

Nếu chưa có, chạy các lệnh sau để tạo thủ công:
```
sudo mknod /dev/rknpu c 226 1
sudo chmod 666 /dev/rknpu
```
Các số `226` và `1` lấy từ:
```bash
ubuntu@ubuntu:~/frigate$ cat /sys/class/drm/card1/dev
226:1
```

# Hướng dẫn cài đặt

## 1. Cài Docker
Cài đặt Docker bằng một dòng lệnh:
```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

## 2. Thêm người dùng vào nhóm docker
Chạy lệnh sau:
```
sudo usermod -aG docker $USER
```
Sau đó đăng xuất và đăng nhập lại. Kiểm tra Docker hoạt động với `docker ps`.

## 3. Kéo các image cần thiết
Kéo các image trước:
```
docker pull eclipse-mosquitto
docker pull ghcr.io/home-assistant/home-assistant:stable
docker pull ghcr.io/blakeblackshear/frigate:stable-rk
```

## 4. Clone GitHub repo
```
git clone https://github.com/thanhtantran/homeassitant-frigate-orangepi5
cd homeassitant-frigate-orangepi5
```

## 5. Tạo mật khẩu cho mosquito (MQTT)
```
cd homeassistant
sudo touch ./mosquitto/config/passwd
docker run --rm -it \
  -v "$(pwd)/mosquitto/config:/mosquitto/config" \
  eclipse-mosquitto \
  mosquitto_passwd -c /mosquitto/config/passwd mqttuser
```
Lệnh trên sẽ tạo người dùng MQTT là `mqttuser`, bạn sẽ chọn mật khẩu trong bước này.

## 6. Triển khai Home Assistant
```
docker compose up -d
```

## 8. Cấu hình Frigate
Vào thư mục `frigate` và sửa file `config.yml`. Thêm camera bằng cách sửa tên camera `cam1` và địa chỉ RTSP. Độ phân giải phụ thuộc vào model bạn chọn.
```
cameras:
  cam1:
    ffmpeg:
      inputs:
        - path: rtsp://admin:password@192.168.10.28:554/12
          roles:
            - detect
            - rtmp
    detect:
      width: 320
      height: 320
      fps: 5
```

## 10. Triển khai Frigate
```
docker compose up -d
```

## 11. Cấu hình trong Home Assistant
Vào Home Assistant và cấu hình MQTT, lắng nghe topic `frigate` là có thể kết nối Frigate vào Home Assistant. Xem video hướng dẫn đầy đủ ở đây:

[![🎬 Hướng dẫn cài đặt Home Assistant + MQTT + Frigate trên Orange Pi 5](http://img.youtube.com/vi/3ERqOcUjRTs/0.jpg)](http://www.youtube.com/watch?v=3ERqOcUjRTs)

Nếu gặp vấn đề, hãy hỏi tại [Diễn đàn Orange Pi Việt Nam](https://forum.orangepi.vn/)
