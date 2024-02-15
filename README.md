# Mathematical library for working with ECG data from the Callibri sensor.
 
The main functionality is the calculation of cardio-interval lengths, heart rate and Stress Index (SI).

During the first 6 seconds the algorithm is learning, if no 5 RR-intervals are found in the signal 5 RR-intervals are not found, the training is repeated. Further work with the library is iterative (adding new data, calculating indicators).

# Getting started

## Install

### Windows (cpp)

Download from [GitHub](https://github.com/BrainbitLLC/callibri-ecg-cpp) add .dll from folder `windows` to your project by your preferred way.

### Linux (cpp)

Download from [GitHub](https://github.com/BrainbitLLC/callibri-ecg-cpp) and add .so from folder `linux_x86_64` to your project by your preferred way.

Library buit on Astra Linux CE 2.12.46, kernel 5.15.0-70-generic. Arch: x86_64 GNU/Linux

```
user@astra:~$ ldd --version
ldd (Debian GLIBC 2.28-10+deb10u1) 2.28
Copyright (C) 2018 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
Written by Roland McGrath and Ulrich Drepper.
```

That library depends from others library, for instance:
```
linux-vdso.so.1
libblkid.so.1
libc++.so.1
libc++abi.so.1 
libc.so.6
libdl.so.2
libffi.so.6
libgcc_s.so.1
libgio-2.0.so.0
libglib-2.0.so.0
libgmodule-2.0.so.0
libgobject-2.0.so.0
libm.so.6
libmount.so.1
libpcre.so.3
libpthread.so.0
libresolv.so.2 
librt.so.1
libselinux.so.1
libstdc++.so.6
libudev.so.1
libuuid.so.1
libz.so.1
ld-linux-x86-64.so.2
```

If you are using a OS other than ours, these dependencies will be required for the library. You can download dependencies from [here](https://github.com/BrainbitLLC/linux_neurosdk2/tree/main/dependencies).

## Initialization
### Determine the basic parameters

1. Raw signal sampling frequency. Integer type. The allowed values are 250 or 1000.
2. Data processing window size. Integer type. Valid values of sampling_rate / 4 or sampling_rate / 2.
3. Number of windows to calculate SI. Integer type. Allowable values [20...50].
4. The averaging parameter of the IN calculation. Default value is 6.

### Creating a library instance
Firstly you need to determine lybrary parameters and then put them to library. Tne next step is initialize the filters. In the current version the filters are built-in and clearly defined: Butterworth 2nd order BandPass 5_15 Hz.

You can initialize averaging for SI calculation. It is optional value.

```cpp
// 1. Raw signal sampling frequency
int sampling_rate = 250;
// 2. Data processing window size
int data_window = sampling_rate / 2;
// 3. Number of windows to calculate SI
int nwins_for_pressure_index = 30;

CallibriMathLib* tCallibriMathPtr = createCallibriMathLib(sampling_rate, data_window, nwins_for_pressure_index);
CallibriMathLibInitFilter(tCallibriMathPtr);

// optional
// 4. The averaging parameter of the IN calculation. Default value is 6.
int pressure_index_average = 6;
CallibriMathLibSetPressureAverage(tCallibriMathPtr, pressure_index_average);
```

## Initializing a data array for transfer to the library:
The size of the transmitted array has to be of a certain length:
- 25 values for a signal frequency of 250 Hz 
- 100 values for a signal frequency of 1000 Hz

```cpp
double* raw_data = new double[25];
// or
double* raw_data = new double[100];
```

## Optional functions (not necessary for the library to work)
Check for initial signal corruption. This method should be used if you want to detect and notify of a distorted signal explicitly. 

```cpp
if(CallibriMathLibInitialSignalCorrupted(tCallibriMathPtr)){
    // Signal corrupted!!!
}
```
### Work with the library
1. Adding and process data:

```cpp
CallibriMathLibPushData(tCallibriMathPtr, raw_data, 25);
// or
CallibriMathLibPushData(tCallibriMathPtr, raw_data, 100);

MathLibProcessDataArr(tCallibriMathPtr); 
```
2. Getting the results:
```cpp
if (CallibriMathLibRRdetected(tCallibriMathPtr))
// check for a new peak in the signal
{               
    // RR-interval length
    double RR_int = CallibriMathLibGetRR(tCallibriMathPtr);
    // HR     
    double HeartRate = CallibriMathLibGetHR(tCallibriMathPtr);
    // SI
    double PressureIndex = CallibriMathLibGetPressureIndex(tCallibriMathPtr);
    // Moda
    double Moda = CallibriMathLibGetModa(tCallibriMathPtr);
    // Amplitude of mode
    double AmplModa = CallibriMathLibGetAmplModa(tCallibriMathPtr);
    // Variation range
    double VariationDist = CallibriMathLibGetVariationDist(tCallibriMathPtr);
    CallibriMathLibSetRRchecked(tCallibriMathPtr);		
}
```
## Finishing work with the library:
```cpp
CallibriMathLibClearData(tCallibriMathPtr);
```