# XV11 LIDAR test

## Preamble 1

This repository is phased out from ev3dev-mapping. Some information here may be not up-to-date for a while.
Precisely: http://www.ev3dev.org/docs/tutorials/using-xv11-lidar/ was updated to always load dc-motor driver manually.
This change is not reflected here yet.
If you want to use this code with 2-wire soldered motor change the output port mode to dc-motor and use dc-motor instead of tacho-motor.

## Preamble 2

Using this repository for anything apart from rough testing is not recommended. Especially redirecting continous binary output (you will have no way to tell when the communication fails and it fails sometimes). This implementation is also not resitant to losing sync with laser.

## Overview

This is code for XV11 LIDAR test. The test outputs semicolon separated distance data for angles 0-359.
The output can be redirected to file, transferred to PC and plotted.

Alternatively, xv11test can be called with -raw argument to output synchronized binary data.
You can pipe the data to your application written in any language (effectively getting LIDAR data on you standard input).
See `examples` directory for C, C# and Java code.

If you just want simple C/C++ library to communicate with the LIDAR see [xv11lidar](https://github.com/bmegli/xv11lidar) repository.

## LIDAR test

### Instructions Assumptions 
- hardware is connected as in http://www.ev3dev.org/docs/tutorials/using-xv11-lidar/
- lidar data connector is connected to port 1
- lidar motor interface is avaliable at `/sys/class/tacho-motor/motor0`

### Building the xv11test

Instructions are for compiling the code directly on EV3 (get the files to EV3 and use ssh)

- Get the build system (or just get the gcc and make package instead)
```bash
sudo apt-get update
sudo apt-get install build-essential
```
- Compile the code - enter directory and type `make`

### Running the test

- Put the port in `other-uart` mode
```bash
 echo other-uart > /sys/class/lego-port/port0/mode
```
- Spin the motor around 200-300 RPM CCW
```bash
echo 40 > /sys/class/tacho-motor/motor0/duty_cycle_sp
echo run-direct > /sys/class/tacho-motor/motor0/command
```
- Run xv11test
```bash
./xv11test /dev/tty_in1
```
- Run xv11test and redirect output to file
```bash
./xv11test /dev/tty_in1 > distances.csv
```
- Stop the motor
```bash 
echo stop > /sys/class/tacho-motor/motor0/command
```

### Results plot

The file `xv11plot.ods` is Open Document Spreadsheet with the formulas for converting xv11test angle/distance output to 2D points graph.
You can open `xv11plot.ods` in OpenOffice Calc, Microsoft Excel or any other software supporting Open Document Spreadsheets.

To make a plot:
- Get the xv11test output `distances.csv` file to your PC
- Open `xv11plot.ods` in software of your choice (e.g. OpenOffice Calc, Microsoft Excel, ...) 
- Copy your result from `distances.csv` to lines in `xv11plot.ods` where it says so

See your own plot and note how to convert LIDAR angle/distance output and apply geometric correction in the spreadsheet

#### Example Plot

![Alt text](img/xv11plot.png "XV11 scan sample image")

### Working with binary data

If you are working in C/C++ you can use [xv11lidar](https://github.com/bmegli/xv11lidar) library with functions `InitLaser`, `ReadLaser` and `CloseLaser`.

Everything below is *depracated*.

If you don't feel comfortable with C/C++ code or don't want to write UART communication code, you can run xv11test with -raw argument and pipe its output to your application in any language.

See also `examples` subdirectory for C, C# and Java examples.

The xv11test syntax to get synchronized binary output is following:
```bash
xv11test terminal_device -raw [frames_per_read] [frames_limit]
```

Where:
- `terminal_device` is path to tty device (e.g. `/dev/tty_in1`)
- `frames_per_read` is optional number of LIDAR frames per single read/write, 15 by default (60 degrees)
- `frames_limit` is optional limit for number of frames, 90 by default (360 degrees), use 0 to get continuous binary data

By piping xv11test with -raw argument to your application you can read binary LIDAR data from your standard input. 

Assumptions for those use cases are the same as before (port in `other-uart` mode, motor spinning, etc.)

#### Example 1 - 360 degree scan piped to other_program

```bash
./xv11test /dev/tty_in1 -raw | ./other_program
```

Or the same explicitly - read/write in chunks of 15 frames, limit to 90 frames (each 4 degrees):

```bash
./xv11test /dev/tty_in1 -raw 15 90 | ./other_program
```

#### Example 2 - 360 degree scan redirected to binary file

 ```bash
./xv11test /dev/tty_in1 -raw > scan360.bin
```

#### Example 3 - 360 degree scan redirected to binary file and used in other_program

```bash
./xv11test /dev/tty_in1 -raw > scan360.bin
./other_program < scan360.bin
```

#### Example 4 - continuous binary data piped to other_program

```bash
./xv11test /dev/tty_in1 -raw 15 0 | ./other_program
```

