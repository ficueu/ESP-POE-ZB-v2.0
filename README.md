# ESP-POE-ZB-v1.2
ESP32 POE: ZigBee + BLE gateway

Features:
* ESPHome compatible,
* ESP32-S (with U.FL connector) used as BLE receiver, main gateway controller,
* Ebyte E72-2G4M20S1E (CC2652p with U.FL connector) as ZigBee coordinator,
* POE 802.3af/802.3at (36-57 VDC) with LAN8720,
* passive POE 12-57V (experimental, need to solder jumper on PCB),
* USBC for flashing, based on CH340C (can be used for powering),
* power connector: 10-57 VDC (abs max 70V - needs to replace capacitor),
* external connectors: 5xGPIO (3 of them internal pulled up, for SCL, SDA, 1Wire), 3.3V, 5V, GND,
* internal connectors: SCL, SDA, 3.3V, GND (SCL and SDA shared with external connector; more gpio available without zigbee module),
* compatible with Kradex Z102 enclosure for din rail,
* support for RS485/MODBUS (need to add ISL83485 or similar - note: RS485 disable I2C and 1Wire),
* support for CAN (canbus) (need to add 3V3 CAN transceiver),
* BUS (input/POE) voltage monitoring,
* LEDs: power (green) and status (amber).


### DO NOT POWER ON MODULE WITHOUT ANTENNAS

Example ESPHome yaml file: https://github.com/ficueu/ESP-POE-ZB-v2.0/blob/main/esp-poe-test1.yaml

Enclosure STL files: https://github.com/ficueu/ESP-POE-ZB-v2.0/tree/main/enclosure

Pinout (top screw terminal connector):
```
1: RS485 B or GPIO13
2: RS485 A or GPIO14
3: CAN HIGH or GPIO15
4: CAN LOW or GPIO16
5: GPIO32 or GPIO33
6: +5V
7: +3.3V
8: GND
```

Pinout (bottom screw terminal connector):
```
1: VCC (INPUT: 10-57 VDC)
2: GND
```

Pinout (ESP32 side):
```
i2c:
  sda: 14
  scl: 13
  id: i2cbus

dallas:
  - pin: 16

uart:
  id: modbus
  tx_pin: 14
  rx_pin: 13
  baud_rate: 115200
  stop_bits: 1

modbus:
  flow_control_pin: 16  //it can be changed to GPIO33 by jumper on PCB)
  id: modbus1
  
canbus:
  - platform: esp32_can
    tx_pin: GPIO15
    rx_pin: GPIO16
    can_id: 1
    bit_rate: 50kbps
	
switch:
  - platform: gpio
    pin: 32
    id: zRST_gpio
    inverted: yes
    restore_mode: ALWAYS_OFF
	
  - platform: gpio
    pin: 33
    name: "Zigbee BSL"

status_led:
  pin:
    number: 2
    inverted: true

ethernet:
  type: LAN8720
  mdc_pin: GPIO23
  mdio_pin: GPIO18
  clk_mode: GPIO17_OUT
  phy_addr: 1
  power_pin: GPIO12
  
uart: //for ZigBee
  - id: uart_bus
    rx_pin: GPIO4
    tx_pin: GPIO5
    baud_rate: 115200

sensor:
  - platform: adc
    attenuation: 11dB
    filters:
      - multiply: 20.6
    pin: GPIO35
    name: "Bus Voltage"
    update_interval: 60s
```

ZigBee2MQTT config:

```
serial:
  port: tcp://YOUR-IP:1234
```

Updating ZigBee firmware:
1. Read https://www.zigbee2mqtt.io/guide/adapters/flashing/flashing_via_cc2538-bsl.html
2. Prepare cc2538-bsl tool.
3. Turn on "Zigbee BSL" on your EespHome device integration.
4. Wait 10s.
5. Run cc2538-bsl: "python cc2538-bsl.py -p socket://IP-OF-DEVICE:1234 -evw FILENAME.HEX" eg: "python cc2538-bsl.py -p socket://192.168.0.126:1234 -evw CC1352P2_CC2652P_other_coordinator_20220219.hex"
6. Wait until firmware was verified successfully.

![alt text](https://github.com/ficueu/ESP-POE-ZB-v1.2/blob/main/images/board-2.0.png)