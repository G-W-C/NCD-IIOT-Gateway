
# NCD-IIOT-Gateway
This tutorial is for the Robustel EG5100 and EG5120 Industrial Edge Computing Gateway, sold by Nation Control Devices (NCD) as 
[Enterprise IIoT Gateway](https://store.ncd.io/product/enterprise-iiot-gateway/) and [Enterprise IIoT Gateway Lite](https://store.ncd.io/product/enterprise-iiot-gateway-lite/).
 ###### All code was tested on an Enterprise IIoT Gateway Lite (EG5100)
## About the Device
Industrialized ARM SBC, 792 MHz Cortex A7 CPU , Linux OS based on Debian 11 bullseye, Referred to as 'RobustOS Pro'
### I/O Hardware 
* 1 x RS232/RS485 (on NCD models serial port 1 is dedicated to the wireless communication module)
* 1 x RS232/RS485 (software configurable) 
* 2 x DI & 2 x DO for simple monitoring and control

### Software
- Node-Red (Installed on NCD devices runing as under pm2)

## Purpose
Utilize Direct Inputs (DI), Direct Outputs (DO) and RS485 with Node-red\
### Background
All I/O is bound to RobustOS with the intent make configuration and functionallity 'easier'.\
This inherantly limits integrations with any other software or custom applications, like Node-Red, as gpio is limited to 1


## Wiring Diagram for DI and DO

![Robustel DIDO Diagram](https://github.com/user-attachments/assets/da208216-00de-4b92-8e3d-3a9cdaf302aa)

# DO

# DI

# Modbus RS485
