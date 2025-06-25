# Homeassistant and Frigate run on Orange Pi 5 via docker

Frigate run on FFMPEG, so you need RKMMP installed on your Orange Pi 5 (or any version of 5 using RK3588)

You can install the Ubuntu 22.04 OS from Joshua Riek (https://github.com/Joshua-Riek/ubuntu-rockchip), it has RKMMP prebuilt, check it by running this command

```bash
for dev in dri dma_heap mali0 rga mpp_service \
   iep mpp-service vpu_service vpu-service \
   hevc_service hevc-service rkvdec rkvenc vepu h265e ; do \
  [ -e "/dev/$dev" ] && echo " --device /dev/$dev"; \
 done
```
it will return these
```
--device /dev/dri
--device /dev/dma_heap
--device /dev/mali0
--device /dev/rga
--device /dev/mpp_service
```

You will also check the `/dev/rknpu` is ready or not by checking `ls -l /dev/rknpu`

If not, run these commands to create it manually
```
sudo mknod /dev/rknpu c 226 1
sudo chmod 666 /dev/rknpu
```
the number 226 and 1 get from 
```bash
ubuntu@ubuntu:~/frigate$ cat /sys/class/drm/card1/dev
226:1
```

# Installation instruction

## 1. Install docker
   Using one line command to install docker
 ```
 curl -fsSL https://get.docker.com -o get-docker.sh
 sudo sh get-docker.sh
 Executing docker install script, commit: 7cae5f8b0decc17d6571f9f52eb840fbc13b2737
 <...>
```
It will install automatically docker, add your user to docker group `sudo usermod -aG docker $USER`. Logout and login again

## 3. Pull the images
Pre pull the images
```
docker pull eclipse-mosquitto
docker pull ghcr.io/home-assistant/home-assistant:stable
```

### 4. Clone the github
```
git clone https://github.com/thanhtantran/homeassitant-frigate-orangepi5
cd homeassitant-frigate-orangepi5
```

5. Create password for mosquito
```
cd homeassistant
sudo touch ./mosquitto/config/passwd
docker run --rm -it \
  -v "$(pwd)/mosquitto/config:/mosquitto/config" \
  eclipse-mosquitto \
  mosquitto_passwd -c /mosquitto/config/passwd mqttuser
```
it will create user for MQTT as mqttuser, and your password as your choice

6. Deploy Home Assistant container
```
docker compose up -d
```
8. Config the Frigate
go to frigate folder and edit the config.yml. Add more camera to this by editing camera name `cam1` and the URL of the RTSP. The width / height of detect based on the model you choose
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
10. Deploy the Frigate
```
docker compose up -d
```
11. Go to Home Assistant and config the MQTT, listen to frigate topic and you will have Frigate connected to Home Assistant. More installation with Frigate camera, Frigate card you can see my video here

https://www.youtube.com/watch?v=3ERqOcUjRTs

