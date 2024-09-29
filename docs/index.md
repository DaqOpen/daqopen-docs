# DaqOpen Library

*Efficient real-time data acquisition and streaming with Arduino Due*

---

## What is DaqOpen?

DaqOpen is a Python library designed for developers working with data acquisition (DAQ) systems. It allows you to collect, process, and stream real-time data from Arduino Due (and maybe other controllers in future)

---

## Why Use DaqOpen?

- **Easy Integration** into your own application
- **Real-time Data Acquisition** and processing
- **Simulated Data Acquisition** for testing without hardware
- **Flexible Data Buffering** with support for delays and offsets
- **ZeroMQ Streaming** to transfer data over networks
- **System Synchronization** to ensure time accuracy

---

## Getting Started

Install the library via pip:

```bash
pip install daqopen-lib
```

Example of basic usage:

```python
from daqopen.duedaq import DueDaq

# Initialize DAQ in simulation mode
my_daq = DueDaq(serial_port_name="SIM")
my_daq.start_acquisition()
data = my_daq.read_data()
print(data)
my_daq.stop_acquisition()
```

[Read the full Getting Started Guide](tutorials/getting-started-with-due.md)

---

## Core Features

- **ADC driver:** Driver for communicating with Arduino Due (included firmware) and packing the data to numpy arrays.
- **Circular Channel Buffer:** A class representing a circular buffer for holding needed amount of data for viewing, calculating and storing.
- **DAQ-Info Class:** Can be used to exchange  informations regarding the interpretation of the data packages. It holds adjustment values and info about the acquisition rate.
- **ZMQ-Support:** Transfer the acquired data in realtime via zmq to other applications or hosts

---

## Tutorials

- [Getting Started with Data Acquisition](tutorials/getting-started-with-due.md)
- [Data Scaling with DaqInfo](tutorials/create-daqinfo.md)
- [Using Channelbuffer](tutorials/use-daqinfo-channelbuffer.md)

## How-to guides

- [Prepare Arduino Due](how-to/prepare-arduino-due.md)
- [Setup Python Environment](how-to/setup-python-environment.md)

---

## Library Reference

- [API Documentation](reference/index.md)

---

## Contributing and Support

- [GitHub Repository](https://github.com/DaqOpen/daqopen-lib)
- [Report an Issue](https://github.com/DaqOpen/daqopen-lib/issues)

---

## License

This project is licensed under the MIT License.

*Version 0.1.0*