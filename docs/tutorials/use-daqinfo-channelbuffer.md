# Using `DaqInfo` with `ChannelBuffer`

In this tutorial, we'll demonstrate how to use the `DaqInfo` object (created in a previous tutorial) alongside the `ChannelBuffer` module. This will allow us to scale and store acquired data for easy access and further processing.

## Overview
The key steps to follow in this tutorial are:

- Load the `daqinfo.toml` file and create a `DaqInfo` object.
- Create an `AcqBufferPool` object, which will hold an `AcqBuffer` for each channel.
- Fill the buffer with data from the acquisition process.
- Read back and display the stored data.
- Extra: how to perform [cyclical calculation](#cyclical-calculation)

**Prerequisites:** Ensure you’ve completed the previous tutorial and saved your `DaqInfo` configuration as `daqinfo.toml`.

## Getting started
This example demonstrates how to acquire data, store it in a buffer, and retrieve it for visualization.

1. **Connect the Function Generator:**

    Set up the wiring as shown in the diagram:

    ![due-breadboard-fgen-1](resources/due-breadboard-fgen-1.png)

    Connect the "inner" part of the BNC connector (signal) to the A1 pin, and the GND to the A0 pin. To ensure a common ground, connect the Arduino's GND pin to the signal generator's GND through a 10kΩ resistor.

1. **Configure the Function Generator:**

    Adjust the function generator to output a sine wave signal with the following settings:

    - **Waveform:** Sine
    - **Frequency:** 50 Hz
    - **Amplitude:** 2 Vpp
    - **Offset:** 1 V

    !!! note
        The Arduino Due inputs cannot handle negative voltages, so an offset must be applied to the sine wave.

2. **Enable the Function Generator Output**

    Make sure the function generator is actively outputting the signal.

3. **Prepare the script for reading the data**

    ```python
    from daqopen.channelbuffer import AcqBufferPool
    from daqopen.daqinfo import DaqInfo
    from daqopen.duedaq import DueDaq
    import matplotlib.pyplot as plt
    import tomllib
    import time
    
    # Load DaqInfo from daqinfo.toml
    with open("daqinfo.toml", "rb") as f:
        daq_info_dict = tomllib.load(f)
    
    # Create DaqInfo Object from dict
    my_daq_info = DaqInfo.from_dict(daq_info_dict)
    
    # Create the Buffer with daqinfo
    acq_buffer = AcqBufferPool(daq_info=my_daq_info)
    
    # Create an instance of DueDaq and search for the device
    myDaq = DueDaq()
    myDaq.start_acquisition()
    
    # Read some data packages and insert the data in the buffer
    for i in range(20):
        data = myDaq.read_data()
        acq_buffer.put_data_with_samplerate(data, my_daq_info.samplerate) # put data to buffer
    
    # Stop acquisition
    myDaq.stop_acquisition()
    
    # Read back 0.2s of data
    scaled_data_A1 = acq_buffer.channel["A1"].read_data_by_index(0, int(my_daq_info.samplerate*0.2))
    ts = acq_buffer.time.read_data_by_index(0, int(my_daq_info.samplerate*0.2))
    
    # Plot data
    plt.plot(ts/1e6, scaled_data_A1)
    plt.show()
    ```

    Explanation of the Script:

    - **Loading `daqinfo.toml`:** The configuration file `daqinfo.toml` is loaded using `tomllib.load(f)`.
    - **Creating `DaqInfo`:** The `DaqInfo` object is instantiated using the dictionary from the TOML file with `DaqInfo.from_dict()`.
    - **Setting Up the Buffer:** The `AcqBufferPool` is initialized with the `DaqInfo` object. This buffer will hold the acquired data, with the correct scaling and buffer width based on the configuration.
    - **Starting Data Acquisition:** The `DueDaq` object manages the hardware and begins the acquisition process.
    - **Inserting Data into the Buffer:** Data is read in packets from the hardware and stored in the buffer. The `put_data_with_samplerate` method ensures that timestamps are aligned with the acquisition rate.
    - **Reading Back Data:** After acquisition, the first 200ms of data is retrieved from channel A1 using the buffer’s indexing system.
    - **Visualizing the Data:** The retrieved data is plotted, showing the sine wave signal over a 200ms period.

4. **Run the script**

    Once the script is executed, you should see a sine wave displayed, representing the signal from 0 to 0.2 seconds:

    ![daqinfo-channelbuffer-simple](resources/daqinfo-channelbuffer-simple.png)

    Notice that there are exactly 10 cycles of the 50 Hz signal within the 0.2s window, confirming the correct sampling rate.

This tutorial provides a clear example of how to acquire, store, and visualize signal data using the `DaqInfo` and `ChannelBuffer` modules. Make sure to adapt the code and settings as needed for your specific use case.

## Cyclical calculation
In this section, we will use the same setup as before to perform a cyclical calculation every 0.2 seconds. The challenge here is to acquire the signal in an infinite loop, wait for the next 200 ms of data to be gathered, and then perform the necessary calculation. One advantage of this approach is that it’s easy to change the interval, as we can directly read the required data from the buffer without needing to track individual data packets.

1. **Script Adaption**

    ```python
    from daqopen.helper import GracefulKiller
    ...
    
    # Create an instance of DueDaq and search for the connected device
    myDaq = DueDaq()
    myDaq.start_acquisition()
    
    # Create an exit handler to gracefully terminate the script
    app_killer = GracefulKiller()
    
    # Define the calculation interval
    calc_interval_samples = int(0.2 * my_daq_info.samplerate)  # number of samples for each 200ms interval
    previous_calc_sidx = 0  # stores the previous calculation's sample index
    
    # Infinite loop for data acquisition and calculations
    while not app_killer.kill_now:
       # Read one block of data from the DAQ device
       data = myDaq.read_data()
    
       # Insert the data into the acquisition buffer
       acq_buffer.put_data_with_samplerate(data, my_daq_info.samplerate)
    
       # Check if enough samples are available for the next calculation
       while (acq_buffer.time.sample_count - calc_interval_samples) > previous_calc_sidx:
           # Read a chunk of data corresponding to the interval size
           data = acq_buffer.channel["A1"].read_data_by_index(previous_calc_sidx, previous_calc_sidx + calc_interval_samples)
           print(data.mean())  # Perform the calculation
           previous_calc_sidx += calc_interval_samples # Update the index for the next cycle
    
    # Stop acquisition when the loop ends
    myDaq.stop_acquisition()
    ```

    Explanation of the Script:

    - **Graceful Exit Handling**:
     The `GracefulKiller` class enables the script to exit cleanly. It allows you to stop the infinite loop and shut down data acquisition in an orderly manner (e.g., when pressing `Ctrl+C`).

    - **Defining the Calculation Interval**:
     We define the calculation interval by converting 0.2 seconds into a number of samples using `calc_interval_samples`. The `previous_calc_sidx` variable helps track where the last calculation ended so that the next calculation starts from the right place.

    - **Main Acquisition and Calculation Loop**:
     The script runs an infinite loop where:

     1. Data is read from the DAQ device.
     2. The read samples are inserted into the acquisition buffer.
     3. The buffer is checked to see if enough data is available for the next calculation. If so, data is retrieved in chunks of `calc_interval_samples`.
     4. The mean value of the retrieved data is calculated and printed.
     5. The index is updated so that the next calculation uses the correct data.

    !!! note "Timing Considerations"
        Since the calculation interval is determined by an integer number of samples, the interval may not be perfectly accurate. We will address this in a future tutorial, explaining how to manage such discrepancies.



    !!! note "CPU Time Usage"
        Be mindful of operations that consume a lot of CPU time. The DAQ device readout must be completed within the buffer size’s time limit to avoid acquisition errors. A potential solution is to decouple the acquisition process from other tasks using separate processes or queues (e.g., ZMQ).

2. **Run the script**

To run the script, simply execute it from your Python environment. Ensure the DAQ device is connected and properly configured before starting the acquisition.



