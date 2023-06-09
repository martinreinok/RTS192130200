# RTS192130200
Real-time Systems 1

# Info about simulavr simulator
  - public domain atmega cycle true instruction set simulator
  - some documentation http://reprap.org/wiki/SimulAVR
  - allows to simulate the atmega328 processor which is used on the Arduino uno
    - use info on web for programming examples of the arduino, 
      e.g. setting output pins, use of serial library
    - documentation atmega328 processor in $ cd /home/rttools/cprog/rts_avr/simulavr/examples/Blink/doc
  - simultator does not only include the core but also peripherals, 
    e.g. timers, ADCs
    - some peripherals are not implemented, e.g. watchdog timer 
  - atmega328 has only 32Kbyte program memory and 2K data memory
    - I have changed it to 64 Kbyte and 4k data memory in the simulator in atmega668base.h file
    - usefull to run code compiled with optimization level O0 instead of Os. 
      We use O0 to simplify WCET program analysis 
  - debugging with gdb is supported (gdb is connected to simulator)
    - in one xterm start:
      $ cd /home/rttools/cprog/rts_avr/simulavr/examples/Blink  
      $ . profile %sets ld_library_path   
      $ ../../src/simulavr -g -v -F16000000 -d atmega328 -c vcd:tracein.txt:trace.vcd -f Blink.elf  
    - in other xterm start: $ avr-gdb -p 1212  
    - follow other instructions on the webpage http://reprap.org/wiki/SimulAVR
    - use of debugger is not required for the exercises

# Compilation to create the elf file for the simulavr
- uses the gcc/g++ cross-compiler toolchain for Atmel cores
- code is compile using a Makefile
- program file must end with .ino e.g. Blink.ino
- $ cd /home/rttools/cprog/rts_avr/ArduinoMake/Arduino-Makefile-master/examples/Blink  
- $ gedit Blink.ino  
- $ . profile  
- $ make clean; make  
- $ cd /home/rttools/cprog/rts_avr/ArduinoMake/Arduino-Makefile-master/examples/Blink/build-uno
- $ ls %Blink.elf should be listed 
- $ avr-nm Blink.elf %lists all function names/code-segments in elf file
- $ cd /home/rttools/cprog/rts_avr/simulavr/examples/Blink
- $ . profile
- $ ./run_simulavr.sh
- $ gtkwave trace.vcd %observe switching signal on port B, and measure on-time
- tracein.txt contains the signals that can be viewed with the gtkwave viewer
- tracelist.txt contains all signals that can be traced
 
# FreeRTOS:
- See http://www.freertos.org/ for info
- manual in /home/rttools/cprog/rts_avr/ArduinoMake/Arduino-Makefile-master/examples/Taskswitching/doc/FreeRTOS_Real_Time_Kernel-A_Hands-On_Tutorial_Guide.pdf
- RTOS which supports many types of cores
- contains functionality for task creation, preemptive task scheduling and communication between tasks
- runs on atmega328 but only for small examples due to small instruction memory
  - for this reason the memory size in simulavr have been doubled
- some changes in the FreeRTOS where made to be able to run it on simulavr
  - simulavr does not contain a watchdog timer therefore timer1 is used instead
  - modification has been made in /home/rttools/cprog/rts_avr/arduino-1.6.13/Arduino_FreeRTOS-master_arduinoSim/src/port.c
    - interupt/task-switch frequency can be adapted in port.c file

# Exercise Taskswitching using FreeRTOS
- create three tasks T1,T2, and T3. T3 has the highest priority, T1 the lowest and T2 the intermediate priority 
- establish functionally deterministic communication using FIFOs from T1 to T2 and from T2 to T3 using FreeRTOS functions. Task T2 copies values from its input to its output. Send an endless stream of values 1,2,3,4,1,2,3,4,... from T1 to T3. Show using gtkwave the streams produced and consumed by the tasks.
- give the tasks an execution time of 5 ms using the "delay(5);" command
- use only communication with ports (e.g. PORTB) because communication with serial port is not thread-safe (simulator reports erros)
- $ cd /home/rttools/cprog/rts_avr/ArduinoMake/Arduino-Makefile-master/examples/Taskswitching
- $ gedit Taskswitching.ino %make necessary modifications
- $ . profile
- $ make clean; make
- verify the result using simulavr
- $ cd /home/rttools/cprog/rts_avr/simulavr/examples/Taskswitching
- $ . profile
- $ ./run_simulavr.sh
- $ gtkwave trace.vcd


# Boundt WCET analysis tool /home/rttools/cprog/rts_avr/boundt
- open source tool
- http://www.bound-t.com/
- manuals in /home/rttools/cprog/rts_avr/boundt/doc 

# Exercise WCETanalysis1
- $ cd /home/rttools/cprog/rts_avr/ArduinoMake/Arduino-Makefile-master/examples/WCETanalysis1
- $ . profile
- $ make clean; make
- $ cd /home/rttools/cprog/rts_avr/boundt/experiments/WCETanalysis1
- $ . profile
- $ ./make_graphs.sh %adapt assertions.txt to obtain tight WCET, be aware of small mistakes in names of functions and loops
- $ gv graph.ps
- $ find way to verify WCET result with the avr-simulator: 
    - how large is the difference between computed and simulated result?
    - $ cd /home/rttools/cprog/rts_avr/simulavr/examples/WCETanalysis1
    - $ . profile
    - $ ./run_simulavr.sh
    - $ gtkwave trace.vcd
    - why is the computed WCET bound pessimistic    
- $ find a way to obtain a tighter WCET bound

# Exercise WCETanalysis2
- $ cd /home/rttools/cprog/rts_avr/ArduinoMake/Arduino-Makefile-master/examples/WCETanalysis2
- $ . profile
- $ make clean; make
- $ cd /home/rttools/cprog/rts_avr/boundt/experiments/WCETanalysis2
- $ . profile
- $ ./make_graphs.sh %adapt assertions.txt to obtain tight WCET, use combination of manual computation and boundt results
- $ gv graph.ps
- $ find way to verify WCET result with simulator: 
    - is the compute WCET pessimistic
    - how large is the difference between computed and simulated result?
    - $ cd /home/rttools/cprog/rts_avr/simulavr/examples/WCETanalysis2
    - $ . profile
    - $ ./run_simulavr.sh
    - $ gtkwave trace.vcd

# Exercises with HAPI DF simulator
- The HAPI simulator is on the same virtual machine as the ARM simulator
- The documentation of the HAPI simulator can be found in: $cd /home/rttools/omphale_tools/progs/HapiLight/doc
  - to build an example
  - $make clean; make 
  - to run an example: $./ex1
  - to view the results: $gtkwave ex1.vcd
  - Usage of the simulator can be found in: hapi_user_guide.pdf, the code of HAPI is in $cd /home/rttools/omphale_tools/progs/HapiLight
  - Simulator can report max response times. Execution times should vary to evaluate many cases.
  
# HAPI exercise 1
- Consider a single processor system with two periodic task \tau_1 and \tau_2 with computation time C_1=2, and period P_1=5 and C_2=3, P_2=7 scheduled by a fixed priority preemptive scheduler according to the rate-monotonic rule
  - verify the schedulability of this task set in 3 different ways. 
  - verify the results with HAPI by adapting the rma1.cc file, include code and simulation traces in report. Executed with $cd /home/rttools/omphale_tools/progs/HapiLight/doc; make; ./rma1 and then $gtkwave rma1.vcd
  - change the computation time of C_2 into 5  and P_2 into 10. Determine whether this task set is schedulable based on utilization only. Is this confirmed by simulation? 



# Report describing the results of the exercises:
- Small report ($<15$ pages) describing the implementations, and the evaluation results of the implementations.
- Hand-in date: before 1 week after the written examination.

