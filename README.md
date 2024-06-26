# micropython-camera-driver

此倉庫由 [lemariva/micropython-camera-driver](https://github.com/lemariva/micropython-camera-driver) fork 而來，對文件做了翻譯並添加 ESP32-WROVER-DEV 對相機（OV2640）的支持。

## 安裝教學

[下載驅動](https://github.com/lemariva/micropython-camera-driver/blob/1392e59751a5140c7915dd8965705301e8640160/firmware/micropython_v1.21.0_camera_no_ble.bin)並透過 Thonny 安裝

<img src="https://github.com/yesaouo/esp32-wrover-dev-micropython-camera-driver/assets/88719692/e036f580-cfcb-4570-9468-479e96db9b6d" alt="drawing" width="400"/>
<img src="https://github.com/yesaouo/esp32-wrover-dev-micropython-camera-driver/assets/88719692/c2cbed82-1861-44a8-a6cb-da4f9c5d6920" alt="drawing" width="400"/>

## 使用方法

```python
import camera

# ESP32-CAM (預設配置)
camera.init(0, format=camera.JPEG, fb_location=camera.PSRAM)

# ESP32-WROVER-DEV (修改引腳)
camera.init(0, d0=4, d1=5, d2=18, d3=19, d4=36, d5=39, d6=34, d7=35,
            format=camera.JPEG, framesize=camera.FRAME_VGA, xclk_freq=camera.XCLK_10MHz,
            href=23, vsync=25, reset=-1, sioc=27, siod=26, xclk=21, pclk=22, fb_location=camera.PSRAM)

# 其他設定：
# 上下翻轉
camera.flip(1)
# 左右翻轉
camera.mirror(1)

# 設定幀大小
camera.framesize(camera.FRAME_240x240)
# 可選擇的幀大小選項有：
# FRAME_96X96 FRAME_QQVGA FRAME_QCIF FRAME_HQVGA FRAME_240X240
# FRAME_QVGA FRAME_CIF FRAME_HVGA FRAME_VGA FRAME_SVGA
# FRAME_XGA FRAME_HD FRAME_SXGA FRAME_UXGA FRAME_FHD
# FRAME_P_HD FRAME_P_3MP FRAME_QXGA FRAME_QHD FRAME_WQXGA
# FRAME_P_FHD FRAME_QSXGA
# 更多信息請參考此鏈接：https://bit.ly/2YOzizz

# 特效
camera.speffect(camera.EFFECT_NONE)
# 可選擇的特效選項有：
# EFFECT_NONE（預設）EFFECT_NEG EFFECT_BW EFFECT_RED EFFECT_GREEN EFFECT_BLUE EFFECT_RETRO

# 白平衡
camera.whitebalance(camera.WB_NONE)
# 可選擇的白平衡選項有：
# WB_NONE（預設）WB_SUNNY WB_CLOUDY WB_OFFICE WB_HOME

# 飽和度
camera.saturation(0)
# -2到2（預設0）。-2 為灰度

# 亮度
camera.brightness(0)
# -2到2（預設0）。2 為最亮

# 對比度
camera.contrast(0)
# -2到2（預設0）。2 為高對比

# 質量
camera.quality(10)
# 10-63（預設12）。數字越低質量越高

buf = camera.capture()
```

## 我的範例

```python
import time, camera
from machine import reset

# 初始化攝像頭
def init_camera():
    camera.init(0, d0=4, d1=5, d2=18, d3=19, d4=36, d5=39, d6=34, d7=35,
        format=camera.JPEG, framesize=camera.FRAME_VGA, xclk_freq=camera.XCLK_10MHz,
        href=23, vsync=25, reset=-1, sioc=27, siod=26, xclk=21, pclk=22, fb_location=camera.PSRAM)
    
# 拍攝照片並保存
def capture_image():
    init_camera()
    time.sleep(2)  # 等待攝像頭穩定
    buf = camera.capture()
    if buf:
        with open("/temp.png", "wb") as f:
            f.write(buf)
        print("Image saved as temp.png")
    else:
        print("Failed to capture image")

# 釋放攝像頭資源
def deinit_camera():
    camera.deinit()

# 主程序
def main():
    try:
        capture_image()
        deinit_camera()
    except Exception as e:
        print("An error occurred: ", e)
        reset()

main()
```

## 注意事項
* 除非使用 CIF 或更低解析度的 JPEG，否則驅動程序需要安裝並啟用 PSRAM。這已經啟用，但由於 MicroPython 需要 RAM，使用受限。
* 使用 YUV 或 RGB 對芯片造成很大壓力，因為寫入 PSRAM 速度不特別快。結果是圖像數據可能會丟失。如果啟用了 WiFi，這尤其真實。如果需要 RGB 數據，建議先捕獲 JPEG，然後使用 `fmt2rgb888` 或 `fmt2bmp/frame2bmp` 轉換為 RGB。轉換不受支持。格式已包括，但幾乎每次捕獲除 JPEG 之外的圖像格式時都會內存不足。
* 固件在編譯時未啟用 BLE 支持。否則會出現 `region 'iram0_0_seg' overflowed by xxx bytes` 的錯誤。
