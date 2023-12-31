# 설명

* Adafruit tmp117(temperature) sensor is connected to Arduino Uno (R3)
* Arduino Uno is connected to Raspi via USB cable
* Arduino IDE is installed in Raspi
* python file is created in Raspi to receive data from Arduino
  
<br>
참조: <a href="https://learn.adafruit.com/adafruit-tmp117-high-accuracy-i2c-temperature-monitor/arduino"> Arduino tmp117 사용 예제</a>

<br><br>

### Raspi Presettings

![image](https://github.com/Zarina-dev/Sensors/assets/61898376/41f1ba4a-e599-4c49-8eae-b90d382a9408)


raspi icon - 기본 설정 - Raspberry Pi Configuration은 이런 상태에서 실행했음 (SPI: On, I2C: On, Serial Console: On)

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

<br><br>


### Arduino가 연결된 serial 포트 확인

1. ```$lsusb``` 하면  'Arduino SA UNO R3(CDC ACM)'과 같은 아두이노 인식됐다는 내용을 볼 수 있음 <br>

2. ```$ls -l /dev/tty*``` 하면 새로 연결된 포트에 대한 새로운 device file을 확인할 수 있음.<br>
(/dev/tty* 들이 많은데 어떻게 구분을 하지?? - /dev/tty*의 created time을 보고 제일 최근에 생성된 파일이 그거임을 알 수 있음.<br>
usb serial port는 보통 /dev/ttyACMx or /dev/ttyUSBx로 표시됨.) <br>

4. Arduino IDE - tool - 포트에서 확인이 가능하다.


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
* Arduino IDE 앱에서 
    - tool - 보드: Arduino Uno를 선택
    - tool - 포트: **/dev/ttyACM0**(Arduino Uno) 와 같은 새로 생긴 포트를 선택
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
        readdata = ser.readline()                # reads data by one line(\n나올때까지)
                                                 # can be: ser.readline().decode('utf-8').strip()
        print("Received data from Arduino: "+str(readdata))



ser.close()
```

실행:
```
python3 SerialArduino.py        
```
하여 코드 실행.
<br><br>
**Error 경우**:
대부분 에러를 아두이노도 파이썬파일도 다 닫은 후 처음부터 실행하여 해결할 수 있음. 
<br>

* **Device or resource busy: /dev/ttyACM0** 에러가 뜨면 <br>
동작중인 Serial Monitor 화면을 닫은 다음에 실행하면 됨. <br>
*(python file에서 ser.readline()과 Serial Monitor를 동시에 실행하면 안될 거 같아)*
<br>

* Arduino IDE에서 얼보드시 **avrdude: stk500_disable(): protocol error, expect=0x14,resp=0x72**에러가 뜨면<br>
이미 background에서 sensor값을 읽어와서 파이썬파일로 보내고 있기 때문에 뜬 에러임.<br>
먼저 python file을 닫은 후 아두이노 스케츠를 업로드하면 됨.<br>
<br>





<br><br><br>


### 실행 결과

Arduino- Serial Monitor:

![Alt text](image-8.png)

<br><br>

python file에서:

![Alt text](image-9.png)


### How it works

Arduino and Raspberri Pi is doing Serial Communicatication. <br>
For this, it is using USB cable as a communication link(channel), this cable carries both power and data.<br>
When connect Arduino and Raspi via USB cable, this is called USB-to-Serial connection.
<br><br>

**Serial Communication** is a method of sending data by one bit at a time sequentially over communication channel. Devices can communicate with this method. If one device is transmitter, second one will be receiver. <br><br>

**Arduino** has a UART that allows to serial communicate. <br>
when Arduino does ```Serial.print(data)```, it writes data to communication channel(which is USB cable) and data gets sent over that channel.<br><br>

on **Raspberry Pi**, when we connect Arduino via USB cable, Raspberry Pi recognizes Arduino as a USB device. That USB device is then asssigned a device file, such as /dev/ttyACM0. <br><br>

on Raspberry Pi, we can use **Python** language(can be other programming language also) to open that /dev/ttyACM0 device file, than read data sent by Arduino through USB cable. 
<br><br><br>

**abstract visualization:**

Connecting: <br>

![Alt text](image-11.png)


<br>

Data transmitting & Receiving: <br>

![Alt text](image-12.png)

<br><br><br><br>


