# Prepare Arduino Due

To use the Arduino Due for Data Acquisition, we have to flash a specific firmware first. This firmware configures the Arduino Due to do the following:

- Setup the ADC in DMA mode and enable AD0, AD2, AD4, AD6, AD10, AD12 in differential mode

  | SAMD-Notation | Arduino Due Pin |
  | ------------- | --------------- |
  | AD0           | A7-A6           |
  | AD2           | A5-A4           |
  | AD4           | A3-A2           |
  | AD6           | A1-A0           |
  | AD10          | A8-A9           |
  | AD12          | A10-A11         |

- Run the ADC-Scan with about 48 kHz per Channel (~300k Samples/s in total)

- Transfer the data in packages of 2048 samples per channel



## 0. Install Arduino IDE

If you don't have the Arduino IDE installed yet, please do so by following the instructions written here:

https://www.arduino.cc/en/software

In some cases, you must add your user to the dialup-group to flash the device successfully and further use the USB connection for gathering the measured data.

```bash
sudo usermod -a -G dialup username
```

After executing this command, you have once to logout and login to your PC.



## 1. Download and Install Board Library

By default, the Arduino IDE only comes with the AVR support. The Arduino DUE has an ARM controller (ATSAMD), and so you must install an additional library to compile and upload the code.

![image-20240919121103265](resources/screenshot-arduino-ide-boards-manager.png)

1. Open the Boards Manager on the left panel
2. Type "due" in the search field on the top
3. Press "INNSTALL"
4. Wait until installation is finished



## 2. Compile and Flash

Now, you are ready to compile and flash the due-daq firmware.

1. Download the due-daq.ino file here https://github.com/DaqOpen/daqopen-lib/blob/main/firmware/due-daq/due-daq.ino or clone the whole repository with `git clone https://github.com/DaqOpen/daqopen-lib.git`
2. Open the due-daq.ino in Arduino IDE
3. Connect the Arduino Due to your PC: Use the Programming Port (this is the micro-USB port near to the power socket)
4. Select the Programming Port on the top
5. Press the "Upload" button



## 3. Finished

Now you are ready to use the Arduino Due as DAQ-device together with the daqopen-lib!