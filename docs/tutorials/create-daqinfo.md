# Create DaqInfo

In this tutorial, we will re-use the setup from before to scale (aka calibrate) the input in the manner, that the output graph shows  physical values in x (value) and y (time) axis.

The following steps must therefore be performed:

- Apply a known signal to the input
- Calculate scaling and offset factors
- Adjust the time axis
- Create the DaqInfo object and perform a second measurement

![daqinfo-sketch-1](resources/daqinfo-sketch-1.jpg)

DaqInfo consists of the following:

InputInfo

- gain: The gain applied to the input channel.
- offset: The offset applied to the input channel.
- delay: The delay in sample periods for this channel.
- unit: The unit of the measurement.
- ad_index: The index of the analog-to-digital converter channel.

## Value axis scaling (InputInfo)

In this step, we will do a scaling of the value axis (the physical measurement value)

1. **Connect the Function Generator:**

    Use the following wiring setup:

    ![due-breadboard-fgen-1](resources/due-breadboard-fgen-1.png)

    Connect the "inner" part of the BNC connector (signal) to the A1 pin, and the GND to the A0 pin. To ensure a common ground, connect the Arduino's GND pin to the signal generator's GND through a 10kΩ resistor.

2. **Configure the Function Generator:**

    Set the function generator to output a signal with the following properties:

    - **Waveform:** DC
    - **Amplitude:** 0.5 V

3. **Enable the Output of the Function Generator.**

4. **Prepare a script to read the average value**

    ```python
    from daqopen.duedaq import DueDaq
    import numpy as np
    import matplotlib.pyplot as plt
    
    def read_one_daq_block(daq_device: DueDaq):
        daq_device.start_acquisition() # Start the acquisition device
        data = daq_device.read_data() # Read one block of data
        daq_device.stop_acquisition() # Stop the acquisition
        return data
    
    # Create an instance of DueDaq and search for the device
    myDaq = DueDaq()
    
    # First Setpoint
    input("First Setpoint: apply 0.5V and press enter")
    data = read_one_daq_block(myDaq)
    low_value = data.mean(axis=0)[3] # Calculate average to remove noise
    # Print average
    print(f"Value of AD6 (A1-A0): {low_value:.3f}")
    
    # Second Setpoint
    input("Second Setpoint: apply 3.0V and press enter")
    data = read_one_daq_block(myDaq)
    high_value = data.mean(axis=0)[3] # Calculate average to remove noise
    # Print average
    print(f"Value of AD6 (A1-A0): {high_value:.3f}")
    
    # Calculate Gain and Offset
    gain = (3.0 - 0.5) / (high_value - low_value)
    offset = offset = gain * high_value - 3.0
    print(f"Gain: {gain:.5E}   Offset: {offset:.5E}")
    
    # Scale data
    A1_A0 = data[:,3]*gain - offset
    
    # Plot
    plt.plot(A1_A0)
    plt.grid()
    plt.show()
    ```

    In this script, the following steps are performed:

    - **Acquire** the data for the **first** set-point (low-value) and calculate the average value
    - **Acquire** the data for the **second** set-point (high-value) and calculate the average value
    - **Calculate the gain** = (set_point_high - set_point_low) / (high_value - low_value)
    - **Calculate the offset** = gain * high_value - set_point_high
    - **Apply gain and offset** to the last reading and display the data

    You will see an example output like this:

    ```
    Device found on Port: /dev/ttyACM0
    DueDaq Init Done
    First Setpoint: apply 0.5V and press enter
    DueDaq Wait for Frame Start
    DueDaq Search Start
    DueDaq ACQ Started
    Value of AD6 (A1-A0): 5021.273
    Second Setpoint: apply 3.0V and press enter
    DueDaq Wait for Frame Start
    DueDaq Search Start
    DueDaq ACQ Started
    Value of AD6 (A1-A0): 30299.477
    Gain: 9.88994E-05   Offset: -3.39890E-03
    ```

    

5. **Visualize the adjusted data**
    ![image-20240927082645604](resources/daqinfo-data-after-adjustment.jpg)

    As you can see, the average value is now exactly at 3.0 V as expected!

## Time axis scaling

The shown method is for demonstration purpose only. For more accurate scaling, refer to the frequency-measurement tutorial.

1. **Prepare script for reading some data packages and measuring the time**

   ```python
   from daqopen.duedaq import DueDaq
   import numpy as np
   import time
   
   # Create an instance of DueDaq and search for the device
   myDaq = DueDaq(serial_port_name="SIM")
   
   # Start the acquisition device
   myDaq.start_acquisition()
   
   # Remember start timestamp
   start_ts = time.time()
   number_of_samples = 0
   
   # Read 100 block of data
   for i in range(100):
       data = myDaq.read_data()
       number_of_samples += data.shape[0]
   
   # Remember stop timestamp
   stop_ts = time.time()
   
   # Stop the acquisition
   myDaq.stop_acquisition()
   
   # Calculate Samplerate
   samplerate = number_of_samples/(stop_ts - start_ts)
   print(f"Samplerate: {samplerate:0.3f} samples/second")
   ```

   In this script, the following steps are performed:

   - Start the Acquisition and **remember the start timestamp**
   - **Perform an acquisition of 100 Blocks** and count the samples
   - **Remember the stop timestamp**
   - **Calculate the samplerate:** number of samples acquired divided by time which was needed to acquire the samples

   Example output:

   ```
   Device found on Port: /dev/ttyACM0
   DueDaq Init Done
   DueDaq Wait for Frame Start
   DueDaq Search Start
   DueDaq ACQ Started
   Samplerate: 47615.293 samples/second
   ```

   ![image-20240927093803012](resources/daqinfo-data-samplerate.jpg)



## Create DaqInfo

Now, let's collect all the data in a toml file to load it the next time (including placeholder for the other channels of the board):

```toml
# This is a Arduino Due DAQ Configuration File
type = "due-daq"
samplerate = 47615.293

[channel.A1] # This is the channel we inspected now
ad_index = 3
gain= 9.88994E-05
offset = -3.39890E-03
delay = 0
unit = "V"

[channel.A2]
ad_index = 1
gain= 1.0
offset = 0.0
delay = 0
unit = "V"

[channel.A3]
ad_index = 2
gain= 1.0
offset = 0.0
delay = 0
unit = "V"

[channel.A4]
ad_index = 3
gain= 1.0
offset = 0.0
delay = 0
unit = "V"

[channel.A5]
ad_index = 4
gain= 1.0
offset = 0.0
delay = 0
unit = "V"

[channel.A6]
ad_index = 5
gain= 1.0
offset = 0.0
delay = 0
unit = "V"
```

This file will be used in the following tutorials together with the Channelbuffer to demonstrate more advanced use cases.

