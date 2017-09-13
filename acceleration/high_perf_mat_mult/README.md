High Performance Matrix Multiplication
======================

This README file contains the following sections:

1. OVERVIEW
2. HOW TO DOWLOAD THE REPOSITORY
3. SOFTWARE TOOLS AND SYSTEM REQUIREMENTS
4. DESIGN FILE HIERARCHY
5. COMPILATION AND EXECUTION
6. EXECUTION IN CLOUD ENVIRONMENTS
7. SUPPORT
8. LICENSE AND CONTRIBUTING TO THE REPOSITORY
9. ACKNOWLEDGEMENTS
10. REVISION HISTORY


## 1. OVERVIEW
This example implements a high performance matrix multiplication of two input matrices (A*B=C). The matrix multiplication kernel operates on matrices of type int16 and produces int16 results. Internally, the kernel has a systolic array of 2048 DSP units and is attached to two DDR banks. The DSP array runs at 400 MHz whereas the logic around the array runs at 300 MHz.
 
The design is targeting execution on an SDAccel supported FPGA acceleration card. The hostcode is compiled into the high_perf_mat_mult executable. The executable takes 3 arguments, namely number of rows of matrix A, number of columns of matrix B, and the common dimension representing number of columns in matrix A and number of rows in matrix B.
```
high_perf_mat_mult <rowsA> <colsB> <commonDim>
```
The testbench of the example reports the kernel execution time, the total number of operations (sum of matrix element multiplications and additions), as well as the efficiency expressed by number of operations per second. Please note, the testbench also compares the kernel results with a pure software matrix multiplication and reports potential differences.
 
The test is based on an encrypted RTL kernel. This kernel can also be configured to run with int8 data values, which effectively doubles the number of operations and throughput.
 
Before explaining the details about the structure of the example and it's compilation, here is a brief step by step instruction on how to run it on the AWS-F1 platform:
* Building the executable in the example directory (high_perf_mat_mult) by calling:
```
make all DEVICES=$AWS_PLATFORM
```
* Creating and registering the AFI
 
Please note, the angle bracket directories need to be replaced according to the user setup.
```
mkdir pack
cd pack
cp ../xclbin/high_perf_mat_mult0.hw.xilinx_aws-vu9p-f1_4ddr-xpr-2pr_4_0.xclbin .
$SDACCEL_DIR/tools/create_sdaccel_afi.sh -xclbin=high_perf_mat_mult0.hw.xilinx_aws-vu9p-f1_4ddr-xpr-2pr_4_0.xclbin -o=high_perf_mat_mult0.hw.xilinx_aws-vu9p-f1_4ddr-xpr-2pr_4_0 -s3_bucket=<bucket> -s3_dcp_key=<f1-dcp-folder> -s3_logs_key=<f1-logs>
```
* Check AFI registration
```
more *afi_id.txt
aws ec2 describe-fpga-images --fpga-image-ids <afi-id from file>
```
* Setup and execute
```
cd ..
mkdir run
cd run
cp ../pack/high_perf_mat_mult0.hw.xilinx_aws-vu9p-f1_4ddr-xpr-2pr_4_0.awsxclbin high_perf_mat_mult0.hw.xilinx_aws-vu9p-f1_4ddr-xpr-2pr_4_0.xclbin
cp ../high_perf_mat_mult .
sudo sh
source /opt/Xilinx/SDx/2017.1.rte/setup.sh
./high_perf_mat_mult 500 500 500
exit
```

## 2. HOW TO DOWNLOAD THE REPOSITORY
To get a local copy of the SDAccel example repository, clone this repository to the local system with the following command:
```
git clone https://github.com/Xilinx/SDAccel_Examples examples
```
where examples is the name of the directory where the repository will be stored on the local system.This command needs to be executed only once to retrieve the latest version of all SDAccel examples. The only required software is a local installation of git.

## 3. SOFTWARE AND SYSTEM REQUIREMENTS
Board | Device Name | Software Version
------|-------------|-----------------
AWS VU9P F1|xilinx:aws-vu9p-f1:4ddr-xpr-2pr|SDAccel 2017.1
Xilinx KU115|xilinx:xil-accel-rd-ku115:4ddr-xpr|SDAccel 2017.1


*NOTE:* The board/device used for compilation can be changed by adding the DEVICES variable to the make command as shown below
```
make DEVICES=<device name>
```
where the *DEVICES* variable accepts either 1 device from the table above or a comma separated list of device names.

## 4. DESIGN FILE HIERARCHY
Application code is located in the src directory. Accelerator binary files will be compiled to the xclbin directory. The xclbin directory is required by the Makefile and its contents will be filled during compilation. A listing of all the files in this example is shown below

```
.gitignore
Makefile
README.md
description.json
src/high_perf_mat_mult.cpp
src/kernelShigh_perf_mat_mult_0.xo
src/ku115/ku115-constraints-pblock-1kernel.tcl
src/ku115/presynth.tcl
src/xilinx_aws-vu9p-f1/f1-constraints-pblock-1kernel.tcl
src/xilinx_aws-vu9p-f1/postopt.tcl
src/xilinx_aws-vu9p-f1/presynth.tcl
```

## 5. COMPILATION AND EXECUTION
### Compiling for Application Emulation
As part of the capabilities available to an application developer, SDAccel includes environments to test the correctness of an application at both a software functional level and a hardware emulated level.
These modes, which are named sw_emu and hw_emu, allow the developer to profile and evaluate the performance of a design before compiling for board execution.
It is recommended that all applications are executed in at least the sw_emu mode before being compiled and executed on an FPGA board.
```
make TARGETS=<sw_emu|hw_emu> all
```
where
```
	sw_emu = software emulation
	hw_emu = hardware emulation
```
*NOTE:* The software emulation flow is a functional correctness check only. It does not estimate the performance of the application in hardware.
The hardware emulation flow is a cycle accurate simulation of the hardware generated for the application. As such, it is expected for this simulation to take a long time.
It is recommended that for this example the user skips running hardware emulation or modifies the example to work on a reduced data set.
### Executing Emulated Application 
***Recommended Execution Flow for Example Applications in Emulation*** 

The makefile for the application can directly executed the application with the following command:
```
make TARGETS=<sw_emu|hw_emu> check

```
where
```
	sw_emu = software emulation
	hw_emu = hardware emulation
```
If the application has not been previously compiled, the check makefile rule will compile and execute the application in the emulation mode selected by the user.

***Alternative Execution Flow for Example Applications in Emulation*** 

An emulated application can also be executed directly from the command line without using the check makefile rule as long as the user environment has been properly configured.
To manually configure the environment to run the application, set the following
```
export LD_LIBRARY_PATH=$XILINX_SDX/runtime/lib/x86_64/:$LD_LIBRARY_PATH
export XCL_EMULATION_MODE=<sw_emu|hw_emu>
emconfigutil --xdevice 'xilinx:xil-accel-rd-ku115:4ddr-xpr' --nd 1
```
Once the environment has been configured, the application can be executed by
```
./high_perf_mat_mult 32 64 64
```
This is the same command executed by the check makefile rule
### Compiling for Application Execution in the FPGA Accelerator Card
The command to compile the application for execution on the FPGA acceleration board is
```
make all
```
The default target for the makefile is to compile for hardware. Therefore, setting the TARGETS option is not required.
*NOTE:* Compilation for application execution in hardware generates custom logic to implement the functionality of the kernels in an application.
It is typical for hardware compile times to range from 30 minutes to a couple of hours.

## 6. Execution in Cloud Environments
FPGA acceleration boards have been deployed to the cloud. For information on how to execute the example within a specific cloud, take a look at the following guides.
* [AWS F1 Application Execution on Xilinx Virtex UltraScale Devices] (Coming Soon)
* [Nimbix Application Execution on Xilinx Kintex UltraScale Devices]
* [IBM SuperVessel Research Cloud on Xilinx Virtex Devices]


## 7. SUPPORT
For more information about SDAccel check the [SDAccel User Guides][]

For questions and to get help on this project or your own projects, visit the [SDAccel Forums][].

To execute this example using the SDAccel GUI, follow the setup instructions in [SDAccel GUI README][]


## 8. LICENSE AND CONTRIBUTING TO THE REPOSITORY
The source for this project is licensed under the [3-Clause BSD License][]

To contribute to this project, follow the guidelines in the [Repository Contribution README][]

## 9. ACKNOWLEDGEMENTS
This example is written by developers at
- [Xilinx](http://www.xilinx.com)

## 10. REVISION HISTORY
Date | README Version | Description
-----|----------------|------------
Sept2017|1.0|Initial Xilinx Release

[3-Clause BSD License]: ../../LICENSE.txt
[SDAccel Forums]: https://forums.xilinx.com/t5/SDAccel/bd-p/SDx
[SDAccel User Guides]: http://www.xilinx.com/support/documentation-navigation/development-tools/software-development/sdaccel.html?resultsTablePreSelect=documenttype:SeeAll#documentation
[Nimbix Getting Started Guide]: http://www.xilinx.com/support/documentation/sw_manuals/xilinx2016_2/ug1240-sdaccel-nimbix-getting-started.pdf
[Walkthrough Video]: http://bcove.me/6pp0o482
[Nimbix Application Submission README]: ../../utility/nimbix/README.md
[Repository Contribution README]: ../../CONTRIBUTING.md
[SDaccel GUI README]: ../../GUIREADME.md
[AWS F1 Application Execution on Xilinx Virtex UltraScale Devices]: ../../README.md
[Nimbix Application Execution on Xilinx Kintex UltraScale Devices]: ../../utility/nimbix/README.md
[IBM SuperVessel Research Cloud on Xilinx Virtex Devices]: http://bcove.me/6pp0o482