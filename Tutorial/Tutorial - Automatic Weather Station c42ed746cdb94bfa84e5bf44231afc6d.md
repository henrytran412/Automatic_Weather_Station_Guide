# Tutorial - Automatic Weather Station

## Requirements:

- [ ]  ESP32
- [ ]  Laptop
- [ ]  USB wire
- [ ]  DF Robot Environment Sensor
- [ ]  Adapter
- [ ]  Strong wifi
- [ ]  Basic Python and HTML (optional) language
- [ ]  Thonny Desktop
- [ ]  Thingspeak account

# Step by step

---

# Dowload app

- Download Thonny
    
    [https://thonny.org/](https://thonny.org/)
    
- Create account Thingspeak
    
    [Single Sign On - ThingSpeak IoT](https://thingspeak.com/login)
    

# 1. Hardware setup

- Thonny setup
    - Connect your ESP32 to your laptop through USB or Type C wire
    - Open Thonny and choose run and choose “Configure interpreter”
    
    ![Config interpreter](https://github.com/henrytran412/Automatic_Weather_Station_Guide/raw/main/Pictures/Config_interpreter.png))
    
    - Choose microPython and port that your laptop connect to ESP32
    - Click “Install or update Micropython” to download firmware to ESP32 (so that ESP32 can run Thonny code (Python))
    
    ![Screenshot 2024-08-09 124751.png](Tutorial%20-%20Automatic%20Weather%20Station%20c42ed746cdb94bfa84e5bf44231afc6d/Screenshot_2024-08-09_124751.png)
    
- Connect DF Robot Environmental sensor to I2C port in shell of ESP32

# 2. Software setup

- Install Thonny IDE
- Create Thingspeak account and create channel
    
    ![Screenshot 2024-08-09 130003.png](Tutorial%20-%20Automatic%20Weather%20Station%20c42ed746cdb94bfa84e5bf44231afc6d/Screenshot_2024-08-09_130003.png)
    

# 3. Thingspeak setup

- Create a channel
- Name a channel and choose field for your project (max: 8 fields). Save channel
    
    ![Screenshot 2024-08-12 100513.png](Tutorial%20-%20Automatic%20Weather%20Station%20c42ed746cdb94bfa84e5bf44231afc6d/Screenshot_2024-08-12_100513.png)
    
- Remember the channel id. Note: The id is different in every channel so check the channel id carefully
    
    ![Screenshot 2024-08-12 100910.png](Tutorial%20-%20Automatic%20Weather%20Station%20c42ed746cdb94bfa84e5bf44231afc6d/Screenshot_2024-08-12_100910.png)
    
- Sharing the project to public view so everyone can see it.
    
    ![Screenshot 2024-08-12 101307.png](Tutorial%20-%20Automatic%20Weather%20Station%20c42ed746cdb94bfa84e5bf44231afc6d/Screenshot_2024-08-12_101307.png)
    
- Check and remember the API KEYS for read and write
    
    ![Screenshot 2024-08-12 101353.png](Tutorial%20-%20Automatic%20Weather%20Station%20c42ed746cdb94bfa84e5bf44231afc6d/Screenshot_2024-08-12_101353.png)
    

# 4. Python code

- Code
    
    ```python
    import machine
    import urequests
    import ujson as json
    import time
    import network
    
    # Define the I2C address and communication modes
    I2C_MODE = 0x01
    UART_MODE = 0x02
    DEV_ADDRESS = 0x22
    
    HPA = 0x01
    KPA = 0x02
    TEMP_C = 0x03
    TEMP_F = 0x04
    
    #Thông tin tài khoản Thingspeak 
    HTTP_HEADERS = {'Content-Type': 'application/json'} 
    THINGSPEAK_WRITE_API_KEY = 'CECWK4EDQD9CRCJW'
    THINGSPEAK_CHANNEL_ID= '2604448'
    THINGSPEAK_READ_API_KEY = 'UQNBZGODZAF499TP'
    USER_API_KEY = 'MQHY981MRKM29ZDE'
    
    ssid='MakerLab.vn' #tên mạng wifi
    password='' #password wifi
    
    # Cài đặt kết nối wifi
    sta_if=network.WLAN(network.STA_IF)
    sta_if.active(True)
    
    if not sta_if.isconnected():
        print('connecting to network...')
        sta_if.connect(ssid, password)
        while not sta_if.isconnected():
         pass
    print('network config:', sta_if.ifconfig())
    #xoa du lieu cu
    '''url = f'https://api.thingspeak.com/channels/2604448/feeds.json?api_key=MQHY981MRKM29ZDE'
    response = urequests.delete(url, headers={'Content-Type': 'application/json', 'api_key': USER_API_KEY})
    
    # Check the response
    print(f'Status Code: {response.status_code}')
    print(f'Response Text: {response.text}')
    
    if response.status_code == 200:
        print('ThingSpeak data cleared successfully')
    else:
        print('Failed to clear ThingSpeak data')
    
    response.close()'''
    
    field1_request = 'https://api.thingspeak.com/channels/2604448/fields/1/last.json'
    
    class DFRobot_Environmental_Sensor():
        def __init__(self, i2cbus=None, baud=9600):
            self.i2cbus = i2cbus
            self._baud = baud
            self._uart_i2c = I2C_MODE if i2cbus else UART_MODE
            self._addr = None
    
        def _detect_device_address(self):
            rbuf = self._read_reg(0x04, 2)
            if self._uart_i2c == I2C_MODE:
                data = rbuf[0] << 8 | rbuf[1]
            elif self._uart_i2c == UART_MODE:
                data = rbuf[0]
            return data
    
        def begin(self):
            if self._detect_device_address() != DEV_ADDRESS:
                return False
            return True
    
        def _read_reg(self, reg_addr, length):
            try:
                if self._uart_i2c == I2C_MODE:
                    rslt = self.i2cbus.readfrom_mem(self._addr, reg_addr, length)
                elif self._uart_i2c == UART_MODE:
                    # Implement UART read here if needed
                    rslt = [-1] * length
            except Exception as e:
                print("Error reading register:", e)
                rslt = [-1] * length
            return rslt
    
        def get_temperature(self, units):
            rbuf = self._read_reg(0x14, 2)
            if self._uart_i2c == I2C_MODE:
                data = rbuf[0] << 8 | rbuf[1]
            elif self._uart_i2c == UART_MODE:
                data = rbuf[0]
            temp = (-45) + ((data * 175.00) / 1024.00 / 64.00)
            if units == TEMP_F:
                temp = temp * 1.8 + 32
            return round(temp, 2)
    
        def get_humidity(self):
            rbuf = self._read_reg(0x16, 2)
            if self._uart_i2c == I2C_MODE:
                humidity = rbuf[0] << 8 | rbuf[1]
            elif self._uart_i2c == UART_MODE:
                humidity = rbuf[0]
            humidity = (humidity / 1024) * 100 / 64
            return humidity
    
        def get_ultraviolet_intensity(self):
            version = self._read_reg(0x05, 2)
            if (version[0] << 8 | version[1]) == 0x1001:
                rbuf = self._read_reg(0x10, 2)
                data = rbuf[0] << 8 | rbuf[1]
                ultraviolet = data / 1800
            else:
                rbuf = self._read_reg(0x10, 2)
                if self._uart_i2c == I2C_MODE:
                    data = rbuf[0] << 8 | rbuf[1]
                elif self._uart_i2c == UART_MODE:
                    data = rbuf[0]
                outputVoltage = 3.0 * data / 1024
                ultraviolet = abs((outputVoltage - 0.99) * (15.0 - 0.0) / (2.9 - 0.99) + 0.0)/25
            return round(ultraviolet, 2)
    
        def get_luminousintensity(self):
            rbuf = self._read_reg(0x12, 2)
            if self._uart_i2c == I2C_MODE:
                data = rbuf[0] << 8 | rbuf[1]
            elif self._uart_i2c == UART_MODE:
                data = rbuf[0]
            luminous = data * (1.0023 + data * (8.1488e-5 + data * (-9.3924e-9 + data * 6.0135e-13)))
            return round(luminous, 2)
    
        def get_atmosphere_pressure(self, units):
            rbuf = self._read_reg(0x18, 2)
            if self._uart_i2c == I2C_MODE:
                atmosphere = rbuf[0] << 8 | rbuf[1]
            elif self._uart_i2c == UART_MODE:
                atmosphere = rbuf[0]
            if units == KPA:
                atmosphere /= 10
            return atmosphere
    
        def get_elevation(self):
            rbuf = self._read_reg(0x18, 2)
            if self._uart_i2c == I2C_MODE:
                elevation = rbuf[0] << 8 | rbuf[1]
            elif self._uart_i2c == UART_MODE:
                elevation = rbuf[0]
            elevation = 44330 * (1.0 - pow(elevation / 1015.0, 0.1903))
            return round(elevation, 2)
    
    class DFRobot_Environmental_Sensor_I2C(DFRobot_Environmental_Sensor):
        def __init__(self, i2cbus, addr):
            super().__init__(i2cbus)
            self._addr = addr
    
        def _read_reg(self, reg_addr, length):
            return self.i2cbus.readfrom_mem(self._addr, reg_addr, length)
    
    class DFRobot_Environmental_Sensor_UART(DFRobot_Environmental_Sensor):
        def __init__(self, baud, addr):
            super().__init__(None, baud)
            self._addr = addr
            # Initialize UART here if needed
    
        def _read_reg(self, reg_addr, length):
            # Implement UART read here if needed
            return [-1] * length
    
    def setup():
        i2c = machine.I2C(0, scl=machine.Pin(22), sda=machine.Pin(21))
        sensor = DFRobot_Environmental_Sensor_I2C(i2c, DEV_ADDRESS)
        
        if not sensor.begin():
            print("Sensor initialization failed!!")
            return
    
        print("Sensor initialization success!!")
    
        while True:
            temperature_c = sensor.get_temperature(TEMP_C)
            temperature_f = sensor.get_temperature(TEMP_F)
            humidity = sensor.get_humidity()
            ultraviolet_intensity = sensor.get_ultraviolet_intensity()
            luminous_intensity = sensor.get_luminousintensity()
            atmospheric_pressure = sensor.get_atmosphere_pressure(HPA)
            elevation = sensor.get_elevation()
            
            print("-----------------------")
            print("Temp: " + str(temperature_c) + " °C")
            print("Temp: " + str(temperature_f) + " °F")
            print("Humidity: " + str(humidity) + " %")
            print("Ultraviolet intensity: " + str(ultraviolet_intensity) + " mw/cm2")
            print("Luminous Intensity: " + str(luminous_intensity) + " lx")
            print("Atmospheric pressure: " + str(atmospheric_pressure) + " hPa")
            print("Elevation: " + str(elevation) + " m")
            print("-----------------------")
            
            sensor_reading = {'field1':temperature_c, 'field2':humidity, 'field3':ultraviolet_intensity, 'field4':luminous_intensity, 'field5':atmospheric_pressure, 'field6': elevation}
            print(sensor_reading) #in 2 giá trị gửi
            request = urequests.post( 'http://api.thingspeak.com/update?api_key=' + THINGSPEAK_WRITE_API_KEY,json = sensor_reading, headers = HTTP_HEADERS )  
            request.close()
            time.sleep(1)
    
    if __name__ == "__main__":
        setup()
     
    
    ```
    
- Explain
    - **Imports and Configuration**
        
        ```python
        import machine
        import urequests
        import ujson as json
        import time
        import network
        
        ```
        
        - **`machine`**: A module for interacting with hardware components like pins and I2C buses.
        - **`urequests`**: A library for making HTTP requests, used here to send data to ThingSpeak.
        - **`ujson`**: A module for working with JSON data, used to format the data being sent.
        - **`time`**: Provides time-related functions, used to control timing between sensor readings.
        - **`network`**: Handles network-related tasks, such as connecting to Wi-Fi.
    - **Constants and Configuration**
        
        ```python
        I2C_MODE = 0x01
        UART_MODE = 0x02
        DEV_ADDRESS = 0x22
        
        HPA = 0x01
        KPA = 0x02
        TEMP_C = 0x03
        TEMP_F = 0x04
        
        ```
        
    - Wi-Fi Connection Setup
        
        ```python
        ssid='MakerLab.vn' # Wi-Fi network name
        password='' # Wi-Fi password
        
        ```
        
        ```python
        sta_if=network.WLAN(network.STA_IF)
        sta_if.active(True)
        if not sta_if.isconnected():
            print('connecting to network...')
            sta_if.connect(ssid, password)
            while not sta_if.isconnected():
                pass
        print('network config:', sta_if.ifconfig())
        
        ```
        
        - This block connects the ESP32 to the specified Wi-Fi network. If not connected, it attempts to connect and waits until successful.
        
    - Setup function
        
        ```python
        def setup():
            i2c = machine.I2C(0, scl=machine.Pin(22), sda=machine.Pin(21))
            sensor = DFRobot_Environmental_Sensor_I2C(i2c, DEV_ADDRESS)
            
            if not sensor.begin():
                print("Sensor initialization failed!!")
                return
        
            print("Sensor initialization success!!")
        
        ```
        
    - Data Collection and Uploading
        
        ```python
        while True:
            temperature_c = sensor.get_temperature(TEMP_C)
            temperature_f = sensor.get_temperature(TEMP_F)
            humidity = sensor.get_humidity()
            ultraviolet_intensity = sensor.get_ultraviolet_intensity()
            luminous_intensity = sensor.get_luminousintensity()
            atmospheric_pressure = sensor.get_atmosphere_pressure(HPA)
            elevation = sensor.get_elevation()
        
        ```
        
        ```python
        sensor_reading = {'field1':temperature_c, 'field2':humidity, 'field3':ultraviolet_intensity, 'field4':luminous_intensity, 'field5':atmospheric_pressure, 'field6': elevation}
        request = urequests.post( 'http://api.thingspeak.com/update?api_key=' + THINGSPEAK_WRITE_API_KEY,json = sensor_reading, headers = HTTP_HEADERS )  
        request.close()
        time.sleep(1)
        
        ```
        
    

# 5. Web Platform

![Untitled](Tutorial%20-%20Automatic%20Weather%20Station%20c42ed746cdb94bfa84e5bf44231afc6d/Untitled.png)

## Structure of the HTML Document

### `<body>`

The main content of the webpage, divided into several sections:

### 1. **Loading Screen**

- A class named `.loading` is used to control the visibility of the content during the initial loading of the webpage.

### 2. **Location and Weather Information**

- Displays current location, date, temperature, humidity, UV index, and other weather data.

### 3. **Data Display Containers**

- Several sections are created for displaying weather metrics like temperature, humidity, UV intensity, etc., using embedded iframes to show dynamic charts from ThingSpeak.

### 4. **Footer**

- Contains credits and links to social media and open-source repositories.

### 5. **Scripts**

- JavaScript code is embedded to handle the fetching and display of weather data from external APIs.

## JavaScript Functionality

### External API Calls

The application fetches data from multiple sources:

1. **ThingSpeak API**: Fetches real-time data for temperature, humidity, and UV intensity.
    - **getData1()**: Fetches the latest temperature data.
    - **getData2()**: Fetches the latest humidity data.
    - **getData3()**: Fetches the latest UV intensity data.
2. **OpenWeatherMap API**: Fetches data for additional weather parameters like wind speed and weather condition, as well as air quality information.
    - **checkWeather(city)**: Fetches weather data for the specified city.
    - **updateAirQuality()**: Fetches air quality data using latitude and longitude coordinates.

### DOM Manipulation

JavaScript is used to dynamically update the HTML content based on the fetched data:

- **Temperature, Humidity, and UV Display**: Updates the respective elements with the latest data.
- **Error Handling**: Displays error messages if the API fails to return valid data.
- **Dynamic Content Update**: Data such as weather conditions, AQI, and pollutant levels are updated in real-time using periodic API calls.

### Event Listeners

Several event listeners are added to improve user interaction:

- **Window Load Event**: Removes the loading screen after a short delay.
- **Search Box Keydown Event**: Triggers a weather search when the user presses the "Enter" key.
- **DOMContentLoaded Event**: Automatically fetches and displays weather data for a default location ("Ho Chi Minh City") when the page loads.

## CSS Styling

### Custom Styles

The application uses custom styles to:

- Control visibility (`.hidden`, `.loading`).
- Apply transitions (`.flow-up`, `.flow-down`) for smooth visual effects.
- Format elements like weather icons, temperature displays, and social media buttons.

### Responsive Design

The CSS includes settings for making the application responsive across different devices using the viewport meta tag.
