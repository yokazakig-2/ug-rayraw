# Software

Here, the author describes the hul-common-lib and the cirasame-soft.

## hul-common-lib

[Github repository](https://github.com/spadi-alliance/hul-common-lib)

Since the basic structure of The CIRASAME firmware is the same that of AMANEQ's firmware, the CIRASAME board can be controlled by hul-common-lib.
Before reading this section, see the [AMANEQ user guide](https://spadi-alliance.github.io/ug-amaneq/software/software/).
In this user guide, the author additionally explains about programs for CIRASAME.

### Executable files

#### set_max1932

It can set 8-bit register value to the internal DAC of MAX1932. Usage is as follows.
```shell
            [IP]          [DAC value]
set_max1932 192.168.10.16 100
```
Please provides the DAC value between 1-255 in decimal notation.
If you set 0, the MAX1932 is turned down.

## cirasame-soft

[Github repository](https://github.com/spadi-alliance/cirasame-soft)

The cirasame-soft provides programs dedicated for the CIRASAME firmware.
It requires hul-common-lib to be complied.

### Executable files

#### ad9220

This program accesses the AD9220 block in the FPGA and reads out the ADC data for the specified number of events.
The data that has just been read is written to the file specified by the argument.
As described in the firmware section, no header or trailer words are added to indicate the end of an event.
Process every 132 words.
**This program should not be terminated by Ctrl-C.**
If users stop it by Ctrl-C, data in the next run will be broken.

Example to read 100 events and store them to hoge.dat.
```shell
       [IP]          [file]   [Events]
ad9220 192.168.10.16 hoge.dat 100
```

#### hgddelay

Please read the Hold Generator sub-section in CIRASAME skeleton firmware section.
In this program, the register value given at 2nd argument is transformed to the mask pattern.
The value of 1 provides the smallest delay amount, and the delay amount increases as the given value increases.
**Please set the register value before calling ad9220 program**