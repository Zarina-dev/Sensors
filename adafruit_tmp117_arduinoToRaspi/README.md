# 설명

* Adafruit tmp117(temperature) sensor is connected to Arduino Uno (R3)
* Arduino Uno is connected to Raspi via USB cable
* Arduino IDE is installed in Raspi
* python file is created in Raspi to receive data from Arduino



### Raspi Presettings

![image](https://github.com/Zarina-dev/Sensors/assets/61898376/88acbaf3-8249-47bd-9175-3e80345428b9)

raspi는 이런 상태에서 실행했음.

<br><br>

### Arduino IDE 설치 in Raspi

* check raspi bit
```
$ getconfig LONG_BIT        # 32Bit
```
* <a href="https://www.raspberrypi-spy.co.uk/2020/12/install-arduino-ide-on-raspberry-pi/">install Arduino 절차</a>

<br><br>

### Arduino를 raspi와 Hw 연결

* arduino완 raspi를 USB(blue) 케이블로 연결

![Alt text](image.png) <br>

*(이렇게 하면 아두이노 power on 기능도 함)*

* ```$lsusb``` 하면  'Arduino SA UNO R3(CDC ACM)'과 같은 아두이노 인식됐다는 내용을 볼 수 있음

<br><br>


### Arduino가 연결된 serial 포트 확인
* Arduino IDE 앱에서 
    - tool - 보드: Arduino Uno를 선택
    - tool - 포트: **/dev/ttyACM0**(Arduino Uno) 와 같은 새로운 포트가 생김. **이는 아두이노를 라베파에 연결했기 때문에 그거에 대한 포트임**. 그걸 선택.
*(Serial.print할때 그포트인 /dev/ttyACM0에 데이터가 써지게 됨(??))*


<br><br>

### Arduino에 temp sensor 연결
Arduino pinout이 다음과 같기 때문에: 

![Alt text](image-2.png)

<br>
tmp117 센서를 해당한 핀에 다음과 같이 연결함:

![Alt text](image-1.png)

<br><br>


### Arduino에 temp sensor 소스 준비 및 실행
 <a href="https://learn.adafruit.com/adafruit-tmp117-high-accuracy-i2c-temperature-monitor/arduino">이 link와</a> 똑같이 함:

* Arduino IDE 열고
* Sketch - Manage Libraries에서 
    1. adafruit_tmp117
    2. adafruit_busio
    3. adafruit Unified sensor library들을 설치함.

* Sample 예제 실행:
    - File-Examples-Adafruit TMP117-basic_test 선택한 후

    ![Alt text](image-5.png)
    ![Alt text](image-6.png)

    클릭하여 실행한 후

    <br><br>

    tool-Serial Monitor 클릭하면 Serial.print 내용들이 출력됨.

    ![Alt text](image-10.png)

    Note: 보트레이트를 115200로 설정!

<br><br>


### Raspi에 python file 생성 및 소스 작성

* communicate_with_arduino라는 폴더 내에 SerialArduino.py, venv 생성
* venv 실행 후 serial packet 설치
```
pip install pyserial
```
* SerialArduino.py에 다음 코드 작성:
```
import serial 
import string
import time

# open serial port
ser = serial.Serial('/dev/ttyACM0', 115200)     # /dev/ttyACM0 Serial 포트를 Arduino uno 
                                                  -> 포트와 똑같이 해줘야 Arduino에서 
                                                  Serial.print한 내용을 여기서 ser.readline()할 수 있음.
                                                # board rate를 115200에 맞춰줘야 함!

print('Connected to Arduino')
print('Waiting for data...')

while True:
        readdata = ser.readline()
        print("Received data from Arduino: "+str(readdata))



ser.close()
```

실행:
```
python3 SerialArduino.py        
```
하여 코드 실행.

if 여기서 **Device or resource busy: /dev/ttyACM0** 에러가 뜨면<br>
동작중인 Serial Monitor 화면을 닫은 다음에 실행하면 됨. <br>

*(python file에서 ser.readline()과 Serial Monitor를 동시에 실행하면 안될 거 같아)*

<br><br><br>


### 실행 결과

Arduino- Serial Monitor:

![Alt text](image-8.png)

<br><br>

python file에서:

![Alt text](image-9.png)
