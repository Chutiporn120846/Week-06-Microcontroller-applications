# Lab 6-1: สร้าง Basic ESP32 Project Structure

## วัตถุประสงค์
เรียนรู้การสร้างโครงสร้าง project ESP32 แบบ standard และเข้าใจการทำงานของ CMake build system

## ขั้นตอนการทำ Lab

### 1. เตรียม Docker Environment (ต่อเนื่องจากสัปดาห์ก่อน)

```bash
# สร้าง directory สำหรับ Lab 6
mkdir Lab6
cd Lab6

# สร้างไฟล์ docker-compose.yml (เหมือนสัปดาห์ก่อน)
```

สร้างไฟล์ `docker-compose.yml`:

```yaml
version: '3.8'

services:
  esp32-dev:
    image: espressif/idf:latest
    container_name: esp32-lab5
    volumes:
      - .:/project
    working_dir: /project
    tty: true
    stdin_open: true
    environment:
      - IDF_PATH=/opt/esp/idf
    command: /bin/bash
    networks:
      - esp32-network

networks:
  esp32-network:
    driver: bridge
```

### 2. รัน Docker และสร้าง Project

```bash
# รัน Docker container (คำสั่งเดิมที่คุ้นเคย)
docker-compose up -d


# ถ้าขึ้นข้อความลักษณะนี้ แสดงว่ายังไม่ได้รัน Docker Desktop ให้ไปรันให้เรียบร้อยก่อน 
# error during connect: Get "http://%2F%2F.%2Fpipe%2FdockerDesktopLinuxEngine/v1.51/images/espressif/idf:latest/json": open //./pipe/dockerDesktopLinuxEngine: The system cannot find the file specified.


# เข้า container
docker-compose exec esp32-dev bash

# ใน container - setup ESP-IDF environment
source $IDF_PATH/export.sh

# สร้าง project ใหม่
idf.py create-project lab6_1_basic_build
cd lab6_1_basic_build
```

### 3. สร้างไฟล์ .gitignore สำหรับ ESP32

สร้างไฟล์ `.gitignore` ในโฟลเดอร์ project ให้มีเนื้อหาต่อไปนี้

```bash
# ESP-IDF Build Output
build/
sdkconfig.old
dependencies.lock

# ESP-IDF Flash Encryption/Secure Boot Keys
*.key
*.pem

# ESP-IDF Development Tools
.vscode/
.devcontainer/

# Python
__pycache__/
*.py[cod]
*$py.class
*.pyc

# OS Generated Files
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
ehthumbs.db
Thumbs.db

# IDE Files
*.swp
*.swo
*~
.idea/

# Build artifacts
*.bin
*.elf
*.map
*.hex

# Temporary files
*.tmp
*.temp
*.log

# ESP32 specific
managed_components/
EOF
```

```bash
# แสดงไฟล์ที่สร้างขึ้น
cat .gitignore
```

### 4. ศึกษาโครงสร้าง Project ที่สร้างขึ้น

```
lab1_basic_build/
├── CMakeLists.txt          # Top-level CMake configuration
├── main/                   # Main application directory
│   ├── CMakeLists.txt     # Main component CMake file
│   └── lab1_basic_build.c # Main source file (auto-generated)
├── .gitignore             # Git ignore file (เพิ่มใหม่)
└── README.md              # Project documentation
```


### 5. แก้ไข main/lab1_basic_build.c

```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_log.h"

static const char *TAG = "LAB6_1";

void app_main(void)
{
    ESP_LOGI(TAG, "Lab 6.1: Basic ESP32 Project Structure");
    ESP_LOGI(TAG, "ESP-IDF Version: %s", esp_get_idf_version());
    ESP_LOGI(TAG, "Free heap size: %d bytes", esp_get_free_heap_size());
    
    int counter = 0;
    
    while (1) {
        ESP_LOGI(TAG, "Build system test - Counter: %d", counter++);
        vTaskDelay(pdMS_TO_TICKS(2000)); // 2 seconds delay
    }
}
```

### 6. ศึกษาไฟล์ CMakeLists.txt

#### Top-level CMakeLists.txt
```cmake
# The following lines of boilerplate have to be in your project's
# CMakeLists.txt, at minimum.
cmake_minimum_required(VERSION 3.16)

include($ENV{IDF_PATH}/tools/cmake/project.cmake)
project(lab1_basic_build)
```

#### main/CMakeLists.txt
```cmake
idf_component_register(SRCS "lab1_basic_build.c"
                       INCLUDE_DIRS ".")
```

### 7. การ Build Project (ใน Docker Container)

```bash
# ตรวจสอบว่าอยู่ใน container และ directory ที่ถูกต้อง
pwd  # ควรแสดง /project/lab6_1_basic_build

# Configure project (first time only)
idf.py set-target esp32

# Build project
idf.py build

# Check build output
ls build/

# ตรวจสอบรายการ ควรเห็นไฟล์  lab6_1_basic_build.elf

```

### 8. การวิเคราะห์ Build Output (ใน Docker Container)

```bash
# ดูขนาด binary
idf.py size

# บันทึกภาพตาราง ใส่ในไฟล์ส่งงาน

# ดูรายละเอียดขนาดตาม component
idf.py size-components

# ถ้ามีปัญหาในการดูรายละเอียดขนาดตาม component บนหน้าจอ ให้ใช้คำสั่ง
idf.py size-components > size-components.txt

# แล้วแนบไฟล์ size-components.txt ในโฟลเดอร์ส่งงาน

# ดูรายละเอียดขนาดตาม source file
idf.py size-files

# ถ้ามีปัญหาในการดูรายละเอียดขนาดตาม source file บนหน้าจอ ให้ใช้คำสั่ง
idf.py size-files > size-files.txt

# แล้วแนบไฟล์ size-files.txt ในโฟลเดอร์ส่งงาน

```




### 9. การ Flash และทดสอบ (ต้องออกจาก Docker หรือใช้ QEMU)

#### วิธีที่ 1: ใช้ QEMU Emulator (แนะนำสำหรับ Lab)
```bash
# ใน Docker container
idf.py qemu 

# กด Ctrl+z เพื่อออกจาก monitor
```

## 🐳 Docker Environment Overview

### โครงสร้างไฟล์หลังจากทำ Lab เสร็จ:

```
Lab1-ESP32-Basic-Project/         # โฟลเดอร์หลัก
├── docker-compose.yml            # Docker configuration (เหมือนสัปดาห์ก่อน)
└── lab1_basic_build/            # ESP32 project ที่สร้างใน container
    ├── CMakeLists.txt           # Top-level CMake configuration
    ├── main/                    # Main application directory
    │   ├── CMakeLists.txt      # Main component CMake file
    │   └── lab1_basic_build.c  # Main source file
    ├── build/                   # Build output (สร้างหลัง build)
    └── README.md               # Project documentation
```

### ข้อดีของการใช้ Docker:
- **🔧 Environment Consistency**: ESP-IDF environment เหมือนกันทุกเครื่อง
- **📦 No Installation Hassle**: ไม่ต้องติดตั้ง ESP-IDF บน host system
- **🔄 Easy Cleanup**: ลบ container ได้ง่าย ไม่ทิ้งไฟล์ขยะ
- **👥 Team Collaboration**: ทุกคนใช้ environment เดียวกัน

### คำสั่ง Docker ที่ใช้บ่อย:
```bash
# เริ่ม container
docker-compose up -d

# เข้า container
docker-compose exec esp32-dev bash

# ดู container ที่กำลังทำงาน
docker-compose ps

# หยุด container
docker-compose down

# ดู logs
docker-compose logs esp32-dev
```

## การทดลองเพิ่มเติม

### 1. เพิ่ม Build Information (ใน Docker Container)

แก้ไข main/lab6_1_basic_build.c:

```bash
# เข้า container (ถ้ายังไม่ได้เข้า)
docker-compose exec esp32-dev bash
source $IDF_PATH/export.sh
cd lab6_1_basic_build

# แก้ไขไฟล์
```

```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_log.h"

static const char *TAG = "LAB1";

void print_build_info(void)
{
    ESP_LOGI(TAG, "=== Build Information ===");
    ESP_LOGI(TAG, "Project Name: lab6_1_basic_build");
    ESP_LOGI(TAG, "ESP-IDF Version: %s", esp_get_idf_version());
    ESP_LOGI(TAG, "Compile Date: %s", __DATE__);
    ESP_LOGI(TAG, "Compile Time: %s", __TIME__);
    ESP_LOGI(TAG, "Chip Model: %s", CONFIG_IDF_TARGET);
    ESP_LOGI(TAG, "Free Heap: %d bytes", esp_get_free_heap_size());
}

void app_main(void)
{
    print_build_info();
    
    int counter = 0;
    
    while (1) {
        ESP_LOGI(TAG, "Running... Counter: %d", counter++);
        
        // แสดงสถานะ memory ทุกๆ 10 ครั้ง
        if (counter % 10 == 0) {
            ESP_LOGI(TAG, "Current free heap: %d bytes", esp_get_free_heap_size());
        }
        
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}
```


บันทึกผลการ simulate ในโฟลเดอร์ส่งงาน


## 🔍 คำถามทบทวน

1. **Docker vs Native Setup**: อธิบายข้อดีของการใช้ Docker เปรียบเทียบกับการติดตั้ง ESP-IDF บน host system
การใช้ Docker ทำให้ได้สภาพแวดล้อม ESP-IDF ที่พร้อมใช้งานทันที เสถียร เหมือนกันทุกเครื่อง ลดปัญหา dependency และสลับเวอร์ชันได้ง่าย ส่วน Native Setup เข้าถึงฮาร์ดแวร์สะดวกและรันเร็วกว่า แต่ติดตั้งและดูแลระบบยากกว่า และเสี่ยงเจอปัญหา dependency conflict
2. **Build Process**: อธิบายขั้นตอนการ build ของ ESP-IDF ใน Docker container ตั้งแต่ source code จนได้ binary
การ build ของ ESP-IDF ใน container มีขั้นตอนหลัก ๆ ดังนี้
Source code: โค้ดโครงการและไฟล์ CMakeLists.txt อยู่ในโฟลเดอร์โปรเจ็กต์ (mounted เข้า container)
idf.py build: คำสั่งนี้จะเรียก CMake เพื่อ generate build system (เช่น Ninja หรือ Make)
โหลด component ทั้งหมด (ทั้งของโครงการและของ ESP-IDF core)
Compile: ninja หรือ make จะ compile source .c/.cpp → object files .o
Linking: รวม object files และ library ต่าง ๆ เป็น binary firmware .elf
Convert: ใช้ esptool.py แปลง .elf → .bin สำหรับ flash ลง ESP32
Output: ไฟล์ .bin จะอยู่ในโฟลเดอร์ build/ ใน host และ container (เพราะแชร์ volume กัน)
3. **CMake Files**: บทบาทของไฟล์ CMakeLists.txt แต่ละไฟล์คืออะไร และทำงานอย่างไรใน Docker environment?
โฟลเดอร์โปรเจ็กต์หลัก:
ระบุชื่อโปรเจ็กต์ (project(name))
รวม component หลักที่จะ build
โฟลเดอร์ component (เช่น main/):
ใช้ idf_component_register(SRCS ... INCLUDE_DIRS ...)
บอกว่า component นี้มีไฟล์อะไรและ include path อะไรบ้าง
4. **Git Ignore**: ไฟล์ .gitignore มีความสำคัญอย่างไรสำหรับ ESP32 project development?
กันไม่ให้ไฟล์ที่ไม่จำเป็น (เช่น binary, build cache, config ส่วนตัว) เข้า git repo
ไฟล์ที่ควร ignore เช่น
/build/ → ไฟล์ binary, object, temporary จากการ build
sdkconfig (ถ้าใช้ส่วนตัว) หรือใช้ sdkconfig.defaults แทน
ไฟล์ log เช่น *.log
ข้อดีคือทำให้ repo สะอาด ขนาดเล็ก และ clone ได้เร็ว
5. **Container Persistence**: ข้อมูลใดบ้างที่จะหายไปเมื่อ restart container และข้อมูลใดที่จะอยู่ต่อ?
ไฟล์โค้ดและ build ที่อยู่ใน volume mapping (เช่น /project mapping ไป host) เพราะจริง ๆ แล้วอยู่บน hostจะอยู่ต่อ
ส่วนไฟล์/โฟลเดอร์ที่อยู่ใน container เท่านั้น (ไม่ได้อยู่ใน volume) เช่น ไฟล์ที่สร้างใน /tmp หรือ home ของ container
การติดตั้ง package ใน container ที่ไม่ได้ commit เป็น image ใหม่จะหายไป
6. **Development Workflow**: เปรียบเทียบ workflow การพัฒนาระหว่างการใช้ Docker กับการทำงานบน native system
Docker เหมาะสำหรับทีมและ environment ที่ต้องการความเหมือนกัน
Native เร็วกว่าเล็กน้อยและเข้าถึง hardware ตรง ๆ ง่ายกว่า แต่ต้องระวัง dependency conflict
## 📋 ผลลัพธ์ที่คาดหวัง

เมื่อทำ lab สำเร็จ นักเรียนจะ:
- **🐳 เข้าใจการใช้ Docker**: สามารถใช้ Docker สำหรับ ESP32 development ได้
- **🏗️ เข้าใจโครงสร้าง project**: สามารถสร้างและจัดการ ESP32 project structure
- **📂 Git Management**: เข้าใจการใช้ .gitignore สำหรับ ESP32 development
- **⚙️ ใช้ idf.py ได้**: สามารถใช้คำสั่ง idf.py เบื้องต้นใน Docker environment
- **📊 วิเคราะห์ build output**: เข้าใจ build process และสามารถวิเคราะห์ผลลัพธ์ได้
- **🔧 ปรับแต่ง build**: สามารถแก้ไข CMakeLists.txt เพื่อปรับแต่ง build configuration

## 🛠️ การแก้ไขปัญหาที่พบบ่อย

### ปัญหา: Docker Container ไม่ start
```bash
docker-compose up -d
# Error: Cannot connect to Docker daemon
```
**วิธีแก้**: 
- ตรวจสอบว่า Docker Desktop เปิดอยู่
- รัน `docker --version` เพื่อตรวจสอบการติดตั้ง

### ปัญหา: Cannot access /project directory
```bash
bash: cd: /project: No such file or directory
```
**วิธีแก้**: 
- ตรวจสอบ volume mapping ใน docker-compose.yml
- ตรวจสอบว่าอยู่ใน directory ที่ถูกต้องบน host

### ปัญหา: ESP-IDF not found
```bash
idf.py: command not found
```
**วิธีแก้**: 
- รัน `source $IDF_PATH/export.sh` ใน container
- ตรวจสอบ environment variables

### ปัญหา: Permission denied when building
```bash
Permission denied: cannot create directory 'build'
```
**วิธีแก้**: 
- ตรวจสอบ file permissions บน host system
- ใช้ `chmod 755` ให้กับ project directory

### ปัญหา: Container stops immediately
```bash
docker-compose ps
# Shows container as "Exited"
```
**วิธีแก้**: 
- ตรวจสอบ docker-compose.yml configuration
- ใช้ `docker-compose logs esp32-dev` เพื่อดู error logs

Docker environment ที่ setup ในสัปดาห์นี้จะใช้ต่อเนื่องตลอดเทอม
