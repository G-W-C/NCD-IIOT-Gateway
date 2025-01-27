
# NCD-IIOT-Gateway
This tutorial is for the Robustel EG5100 and EG5120 Industrial Edge Computing Gateway, sold by Nation Control Devices (NCD) as 
[Enterprise IIoT Gateway](https://store.ncd.io/product/enterprise-iiot-gateway/) and [Enterprise IIoT Gateway Lite](https://store.ncd.io/product/enterprise-iiot-gateway-lite/).
>[!NOTE]
>All code was tested on an Enterprise IIoT Gateway Lite (EG5100).\
>I/O testing was performed at 24VDC.
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
- Node-RED v3.1.9(Installed on NCD devices running as under pm2)
- Node.js v16.19.1

## Purpose
Utilize Direct Inputs (DI), Direct Outputs (DO) and RS485 with Node-RED
### Background
All I/O is bound to RobustOS with the intent to make configuration and functionality 'easier'.\
This inherently limits direct integrations with any other software or custom applications, like Node-RED, as GPIO is exclusive, meaning it can only be accessed by one application. This restriction prevents multiple services from controlling the same GPIO pins simultaneously.\
Unfortunately, but not surprisingly, Robustel's documentation and support are exceedingly poor. This lack of support trickles down to the reseller, NCD, as their primary use case is as a cloud IIoT gateway for their wireless sensor products and not as local I/O.\
The Robustel docs reference a CLI command set but fail to provide a complete picture of how to utilize it.\
Much time was spent debugging to make sense of the command set in the documentation.......

#### Methods
We have two options to complete our goal of using the I/O on this device:
- ~~Unbind the gpio from the host OS~~ (While this does work quite well and gives you the most direct control, I have left it out of this tutorial unless someone asks for it.)\

- Utilize the host OS through and API\
We will use both options to gain access to the I/O with Node-RED.

## Enabled SSH
To Expose the DI and DO SSH access must be enabled.\
Follow the NCD tutorial on SSH access here https://ncd.io/blog/ssh-into-your-iot-enterprise-gateway/#
## Wiring Diagram for DI and DO
>[!CAUTION]
>Proper wiring is essential to ensure safety and performance.
>Max voltage of DI/DO is 30VDC.

###### Excerpt from the Robustel Hardware Manual.
![Robustel DIDO Diagram](https://github.com/user-attachments/assets/da208216-00de-4b92-8e3d-3a9cdaf302aa)

# DO
The DO is by far the most straightforward of all the I/O. No additional nodes, packages, or programs are required to make this work.\
We can toggle DO1 from the shell with this command:
    ```
      sudo rmsg dido cmd DO_CTRL index 3 event inversion
    ```
### Configure DO in RobustOS
>[!WARNING]
>If DO is unconfigured in RobustOS, DO pins will pull low on settings updates or gateway restarts, potentially energizing equipment downstream!\
>DI pins are configured to trigger DO by default and must be disabled for independent control.
>It is therefore imperative that we enable and configure the DO in RobustOS first!

1. Login to the web configuration UI of the IIoT Gateway [See the NCD quick start guide for details](https://ncd.io/blog/quick-start-guide-for-the-ncd-enterprise-iiot-gateway/)
2. In the left hand menu select ***Interface*** >>>> ***DIDO***.
3. Within the DIDO Settings select ![image](https://github.com/user-attachments/assets/70f03cfe-80ef-4e7c-ab0b-6aac82ddfdbd) to enter the settings for each DO.
4. In General Settings
    * Toggle ***Enable*** to ![image](https://github.com/user-attachments/assets/8ef15f18-dd51-4eda-a23a-bbf1f5e0abd8)

    * ***Initial State*** ![image](https://github.com/user-attachments/assets/6e33c2b8-c6fe-47d2-8467-8c815b3c2c81) (de-energized)

    * Toggle ***Triggered by DI*** ![image](https://github.com/user-attachments/assets/9a05e521-bd80-4945-a85c-bcd7f423ba18)

    * Click ![image](https://github.com/user-attachments/assets/832c7a94-1c16-4f06-9209-c92785f47cf5)
      
5.Click ***Save & Apply*** ![image](https://github.com/user-attachments/assets/ebb3047a-099c-41f6-8597-0fd8d3be9fed) in the top right of the screen.

![RobustOS DO Config](https://github.com/user-attachments/assets/57fe74d1-850a-4ad3-9b8c-5baef3b812ca)

### Controlling the DO with Node-RED
To utilize the DO with Node-RED, we will run this CLI command in a EXEC node. This command requires the user to elevate privileges using sudo and enter a password. Since Node-RED runs under the user 'ncdio', we need to modify privileges to allow Node-RED to execute this specific command without a password.
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
### Control with Node-RED
Outputs are referred to internally as:
* DO1 = index 3
* DO2 = index 4
##### Example for DO1
1. Use an EXEC Node with the following command
    ```
      sudo rmsg dido cmd DO_CTRL index 3 event inversion
    ```
2. Use an inject and debug node to confirm the the command should have a return code of ``` { code: 0 } ```.
    ![DO_node-red](https://github.com/user-attachments/assets/6f76ce61-b25a-4400-8fdc-98109881880a)
### Reading DO state
RobustOS is constantly checking the state of the DO and updating its status to files in ``` /var/status/dido ``` \
We can use the watch node to monitor this directory and retrieve the DO status updates. 

![dido_read_node-red](https://github.com/user-attachments/assets/6f8ed3ba-2a51-4dde-9b6b-2da3481089d4)
##### Simple UI control and Monitoring
Combining both DO write and read into a simple UI control (looks great on a cell phone, and you can connect directly to the EG5100 via WiFi with the default AP mode).

![UI controlDO_node-red](https://github.com/user-attachments/assets/85355754-b73c-457c-a272-31aae0be12db)

Here is a complete flow for Controlling and reading the DO from the node-red UI\
[DO_Control.json](DO_Control.json)


# DI
As mentioned in the previous section, [**Reading DO State**](https://github.com/G-W-C/NCD-IIOT-Gateway/blob/main/README.md#reading-do-state), RobustOS writes status updates to ``` /var/status/dido ``` for the DO by default.
> [!NOTE]
> Enable the DI in RobustOS to begin updating the dido file, then we can use watch node to monitor this directory and retrieve the DI status updates.
### Enabling DI in RobustOS
1. Login to the web configuration UI of the IIoT Gateway [See the NCD quick start guide for details](https://ncd.io/blog/quick-start-guide-for-the-ncd-enterprise-iiot-gateway/)
2. In the left hand menu select ***Interface*** >>>> ***DIDO***.
3. Within the DIDO Settings select ![image](https://github.com/user-attachments/assets/70f03cfe-80ef-4e7c-ab0b-6aac82ddfdbd) to enter the settings for each DI.
4. In General Settings
    * Toggle ***Enable*** to ![image](https://github.com/user-attachments/assets/8ef15f18-dd51-4eda-a23a-bbf1f5e0abd8)

    * Click ![image](https://github.com/user-attachments/assets/832c7a94-1c16-4f06-9209-c92785f47cf5)
      
5.Click ***Save & Apply*** ![image](https://github.com/user-attachments/assets/ebb3047a-099c-41f6-8597-0fd8d3be9fed) in the top right of the screen.
![RobustOS DI Config](https://github.com/user-attachments/assets/5ced6d5a-b0ce-4b17-9f5e-493d9ee74552)

### Node-RED
This flow reads the dido file and formats the DI and DO Status into on nice JSON object.

![dido_obj_read_node-red](https://github.com/user-attachments/assets/952b72d0-a157-4817-b091-219b10493b69)

[DIDO_Read.json](DIDO_Read.json)
# Modbus RS485
