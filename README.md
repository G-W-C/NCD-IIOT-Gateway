
# NCD-IIOT-Gateway
This tutorial is for the Robustel EG5100 and EG5120 Industrial Edge Computing Gateway, sold by Nation Control Devices (NCD) as 
[Enterprise IIoT Gateway](https://store.ncd.io/product/enterprise-iiot-gateway/) and [Enterprise IIoT Gateway Lite](https://store.ncd.io/product/enterprise-iiot-gateway-lite/).
 ###### All code was tested on an Enterprise IIoT Gateway Lite (EG5100)
## About the Device
![image](https://github.com/user-attachments/assets/1e855e03-1780-464b-abd6-cdee66e96c69)

- Industrialized ARM SBC
- 792 MHz Cortex A7 CPU  
- Linux OS based on Debian 11 (bullseye) Referred to as 'RobustOS Pro'
- Linux 5.4.24-imx6ull arm LE
### I/O Hardware 
* 1 x RS232/RS485 (on NCD models serial port 1 is dedicated to the wireless communication module)
* 1 x RS232/RS485 (software configurable) 
* 2 x DI & 2 x DO for simple monitoring and control

### Software
- Node-Red v3.1.9(Installed on NCD devices runing as under pm2)
- Node.js v16.19.1

## Purpose
Utilize Direct Inputs (DI), Direct Outputs (DO) and RS485 with Node-red
### Background
All I/O is bound to RobustOS with the intent to make configuration and functionality 'easier'.\
This inherently limits direct integrations with any other software or custom applications, like Node-RED, as GPIO is exclusive, meaning it can only be accessed by one application at a time. This restriction prevents multiple services from controlling the same GPIO pins simultaneously.\
Unfortunately, but not surprisingly, Robustel's documentation and support are exceedingly poor. This lack of support trickles down to the reseller, NCD, as their primary use case is as a cloud IIoT gateway for their wireless sensor products and not as local I/O.\
The Robustel docs reference a CLI command set but fail to provide a complete picture of how to utilize it.\
Much time was spent debugging this to make Command

#### Options
We have two options to complete our goal of using the I/O on this device:
- Unbind the gpio from the host OS\
    or
- Utilize the host OS through and API\
We will use both options to gain access to the I/O with Node-Red.

## Enabled SSH
To Expose the DI and DO SSH access must be enabled.\
Follow the NCD tutorial on SSH access here https://ncd.io/blog/ssh-into-your-iot-enterprise-gateway/#
## Wiring Diagram for DI and DO
Proper wiring is essential to ensure safety and performance. Max voltage of DI/DO is 30VDC.\
All testing was preformed at 24VDC.
###### Excerpt from the Robustel Hardware Manual.
![Robustel DIDO Diagram](https://github.com/user-attachments/assets/da208216-00de-4b92-8e3d-3a9cdaf302aa)

# DO
### Method
To utilize the DO with Node-RED, we will use the API via a CLI command. This requires the user to elevate privileges using sudo and enter a password. Since Node-RED runs under the user 'ncdio', we need to modify privileges to allow Node-RED to execute this specific command without a password.
### Elevate privileges for the dido cmd set only
1. Open the sudoers.tmp file with the following cmd:
   ```cmd
   sudo visudo
   ```
2. Add the following line to the file under the ```# Allow members of group sudo to execute any command```
. This will restrict all privileges to only this command set:
   ```
      ncdio ALL=(ALL) NOPASSWD: /usr/bin/rmsg dido cmd *
   ```
  The file should look like this: [sudoers.tmp](sudoers.tmp)
  
 4. Save the changes ```ctrl- x``` to exit ```shift - Y ``` to save.
### Control with Node-Red
Outputs are referred to internally as:
* DO1 = index 3
* DO2 = index 4
##### Example for DO1
1. Use a EXEC Node with the following command
    ```
      sudo rmsg dido cmd DO_CTRL index 3 event inversion
    ```
2. Use a inject and debug node to confirm the the command should have a return code of ``` { code: 0 } ```.
    ![DO_node-red](https://github.com/user-attachments/assets/6f76ce61-b25a-4400-8fdc-98109881880a)
### Reading dido state
RobustOS is contantly checking the state of the DI/DO and updating its status to files in ``` /var/status/dido ``` \

# DI

# Modbus RS485
