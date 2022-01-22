# RPI_PICO_TimerInterrupt Library

[![arduino-library-badge](https://www.ardu-badge.com/badge/RPI_PICO_TimerInterrupt.svg?)](https://www.ardu-badge.com/RPI_PICO_TimerInterrupt)
[![GitHub release](https://img.shields.io/github/release/khoih-prog/RPI_PICO_TimerInterrupt.svg)](https://github.com/khoih-prog/RPI_PICO_TimerInterrupt/releases)
[![GitHub](https://img.shields.io/github/license/mashape/apistatus.svg)](https://github.com/khoih-prog/RPI_PICO_TimerInterrupt/blob/master/LICENSE)
[![contributions welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg?style=flat)](#Contributing)
[![GitHub issues](https://img.shields.io/github/issues/khoih-prog/RPI_PICO_TimerInterrupt.svg)](http://github.com/khoih-prog/RPI_PICO_TimerInterrupt/issues)

---
---

## Table of Contents

* [Important Change from v1.2.0](#Important-Change-from-v120)
* [Why do we need this RPI_PICO_TimerInterrupt library](#why-do-we-need-this-rpi_pico_timerinterrupt-library)
  * [Features](#features)
  * [Why using ISR-based Hardware Timer Interrupt is better](#why-using-isr-based-hardware-timer-interrupt-is-better)
  * [Currently supported Boards](#currently-supported-boards)
  * [Important Notes about ISR](#important-notes-about-isr)
* [Changelog](changelog.md)
* [Prerequisites](#prerequisites)
* [Installation](#installation)
  * [Use Arduino Library Manager](#use-arduino-library-manager)
  * [Manual Install](#manual-install)
  * [VS Code & PlatformIO](#vs-code--platformio)
* [Packages' Patches](#packages-patches)
  * [1. For RP2040-based boards using Earle Philhower arduino-pico core](#1-for-rp2040-based-boards-using-earle-philhower-arduino-pico-core)
    * [1.1. To use BOARD_NAME](#11-to-use-board_name)
    * [1.2. To avoid compile error relating to microsecondsToClockCycles](#12-to-avoid-compile-error-relating-to-microsecondstoclockcycles)
* [HOWTO Fix `Multiple Definitions` Linker Error](#howto-fix-multiple-definitions-linker-error)
* [More useful Information](#more-useful-information)
* [Usage](#usage)
  * [1. Using only Hardware Timer directly](#1-using-only-hardware-timer-directly)
    * [1.1 Init Hardware Timer](#11-init-hardware-timer)
    * [1.2 Set Hardware Timer Interval and attach Timer Interrupt Handler function](#12-set-hardware-timer-interval-and-attach-timer-interrupt-handler-function)
    * [1.3 Set Hardware Timer Frequency and attach Timer Interrupt Handler function](#13-set-hardware-timer-frequency-and-attach-timer-interrupt-handler-function)
  * [2. Using 16 ISR_based Timers from 1 Hardware Timer](#2-using-16-isr_based-timers-from-1-hardware-timer)
    * [2.1 Important Note](#21-important-note)
    * [2.2 Init Hardware Timer and ISR-based Timer](#22-init-hardware-timer-and-isr-based-timer)
    * [2.3 Set Hardware Timer Interval and attach Timer Interrupt Handler functions](#23-set-hardware-timer-interval-and-attach-timer-interrupt-handler-functions)
* [Examples](#examples)
  * [  1. Argument_Complex](examples/Argument_Complex)
  * [  2. Argument_None](examples/Argument_None)
  * [  3. Argument_Simple](examples/Argument_Simple)
  * [  4. Change_Interval](examples/Change_Interval).
  * [  5. ISR_16_Timers_Array_Complex](examples/ISR_16_Timers_Array_Complex)
  * [  6. ISR_Timers_Array_Simple](examples/ISR_Timers_Array_Simple)
  * [  7. RPM_Measure](examples/RPM_Measure)
  * [  8. SwitchDebounce](examples/SwitchDebounce)
  * [  9. TimerInterruptTest](examples/TimerInterruptTest)
  * [ 10. **multiFileProject**](examples/multiFileProject) **New**
* [Example ISR_Timers_Array_Simple](#example-isr_timers_array_simple)
* [Debug Terminal Output Samples](#debug-terminal-output-samples)
  * [1. ISR_Timers_Array_Simple on RASPBERRY_PI_PICO](#1-isr_timers_array_simple-on-raspberry_pi_pico)
  * [2. TimerInterruptTest on RASPBERRY_PI_PICO](#2-timerinterrupttest-on-raspberry_pi_pico)
  * [3. Change_Interval on RASPBERRY_PI_PICO](#3-change_interval-on-raspberry_pi_pico)
  * [4. SwitchDebounce on RASPBERRY_PI_PICO](#4-switchdebounce-on-raspberry_pi_pico)
  * [5. ISR_Timers_Array_Simple on ADAFRUIT_FEATHER_RP2040](#5-isr_timers_array_simple-on-adafruit_feather_rp2040)
  * [6. ISR_16_Timers_Array_Complex on ADAFRUIT_ITSYBITSY_RP2040](#6-isr_16_timers_array_complex-on-adafruit_itsybitsy_rp2040)
* [Debug](#debug)
* [Troubleshooting](#troubleshooting)
* [Issues](#issues)
* [TO DO](#to-do)
* [DONE](#done)
* [Contributions and Thanks](#contributions-and-thanks)
* [Contributing](#contributing)
* [License](#license)
* [Copyright](#copyright)

---
---

### Important Change from v1.2.0

Please have a look at [HOWTO Fix `Multiple Definitions` Linker Error](#howto-fix-multiple-definitions-linker-error)

### Why do we need this [RPI_PICO_TimerInterrupt library](https://github.com/khoih-prog/RPI_PICO_TimerInterrupt)

## Features

This library enables you to use Interrupt from Hardware Timers on on RP2040-based boards such as RASPBERRY_PI_PICO, using [**Earle Philhower's arduino-pico** core](https://github.com/earlephilhower/arduino-pico). Support to [**Arduino-mbed RP2040** core](https://github.com/arduino/ArduinoCore-mbed) will be added in future releases.

As **Hardware Timers are rare, and very precious assets** of any board, this library now enables you to use up to **16 ISR-based Timers, while consuming only 1 Hardware Timer**. Timers' interval is very long (**ulong millisecs**).

Now with these new **16 ISR-based timers**, the maximum interval is **practically unlimited** (limited only by unsigned long miliseconds) while **the accuracy is nearly perfect** compared to software timers. 

The most important feature is they're ISR-based timers. Therefore, their executions are **not blocked by bad-behaving functions / tasks**. This important feature is absolutely necessary for mission-critical tasks. 

The [**ISR_Timers_Array_Simple**](examples/ISR_Timers_Array_Simple) example will demonstrate the nearly perfect accuracy compared to software timers by printing the actual elapsed millisecs of each type of timers.

Being ISR-based timers, their executions are not blocked by bad-behaving functions / tasks, such as connecting to WiFi, Internet and Blynk services. You can also have many `(up to 16)` timers to use.

This non-being-blocked important feature is absolutely necessary for mission-critical tasks.

You'll see blynkTimer Software is blocked while system is connecting to WiFi / Internet / Blynk, as well as by blocking task 
in loop(), using delay() function as an example. The elapsed time then is very unaccurate

### Why using ISR-based Hardware Timer Interrupt is better

Imagine you have a system with a **mission-critical** function, measuring water level and control the sump pump or doing something much more important. You normally use a software timer to poll, or even place the function in loop(). But what if another function is **blocking** the loop() or setup().

So your function **might not be executed, and the result would be disastrous.**

You'd prefer to have your function called, no matter what happening with other functions (busy loop, bug, etc.).

The correct choice is to use a Hardware Timer with **Interrupt** to call your function.

These hardware timers, using interrupt, still work even if other functions are blocking. Moreover, they are much more **precise** (certainly depending on clock frequency accuracy) than other software timers using millis() or micros(). That's necessary if you need to measure some data requiring better accuracy.

Functions using normal software timers, relying on loop() and calling millis(), won't work if the loop() or setup() is blocked by certain operation. For example, certain function is blocking while it's connecting to WiFi or some services.

The catch is **your function is now part of an ISR (Interrupt Service Routine), and must be lean / mean, and follow certain rules.** More to read on:

[**HOWTO Attach Interrupt**](https://www.arduino.cc/reference/en/language/functions/external-interrupts/attachinterrupt/)

---

### Currently supported Boards

1. RP2040-based boards such as **RASPBERRY_PI_PICO, ADAFRUIT_FEATHER_RP2040, Nano_RP2040_Connect, GENERIC_RP2040**, etc.

---

### Important Notes about ISR

1. Inside the attached function, **delay() won’t work and the value returned by millis() will not increment.** Serial data received while in the function may be lost. You should declare as **volatile any variables that you modify within the attached function.**

2. Typically global variables are used to pass data between an ISR and the main program. To make sure variables shared between an ISR and the main program are updated correctly, declare them as volatile.


---
---


## Prerequisites

1. [`Arduino IDE 1.8.19+` for Arduino](https://github.com/arduino/Arduino). [![GitHub release](https://img.shields.io/github/release/arduino/Arduino.svg)](https://github.com/arduino/Arduino/releases/latest)
2. [`Earle Philhower's arduino-pico core v1.9.14+`](https://github.com/earlephilhower/arduino-pico) for RP2040-based boards such as **RASPBERRY_PI_PICO, ADAFRUIT_FEATHER_RP2040 and GENERIC_RP2040**, etc. [![GitHub release](https://img.shields.io/github/release/earlephilhower/arduino-pico.svg)](https://github.com/earlephilhower/arduino-pico/releases/latest)
3. To use with certain example
   - [`SimpleTimer library`](https://github.com/jfturcot/SimpleTimer) for [ISR_Timers_Array_Simple](examples/ISR_Timers_Array_Simple) and [ISR_16_Timers_Array_Complex](examples/ISR_16_Timers_Array_Complex) examples.
   
   
---
---

## Installation

### Use Arduino Library Manager

The best and easiest way is to use `Arduino Library Manager`. Search for [**RPI_PICO_TimerInterrupt**](https://github.com/khoih-prog/RPI_PICO_TimerInterrupt), then select / install the latest version.
You can also use this link [![arduino-library-badge](https://www.ardu-badge.com/badge/RPI_PICO_TimerInterrupt.svg?)](https://www.ardu-badge.com/RPI_PICO_TimerInterrupt) for more detailed instructions.

### Manual Install

Another way to install is to:

1. Navigate to [**RPI_PICO_TimerInterrupt**](https://github.com/khoih-prog/RPI_PICO_TimerInterrupt) page.
2. Download the latest release `RPI_PICO_TimerInterrupt-master.zip`.
3. Extract the zip file to `RPI_PICO_TimerInterrupt-master` directory 
4. Copy whole `RPI_PICO_TimerInterrupt-master` folder to Arduino libraries' directory such as `~/Arduino/libraries/`.

### VS Code & PlatformIO

1. Install [VS Code](https://code.visualstudio.com/)
2. Install [PlatformIO](https://platformio.org/platformio-ide)
3. Install [**RPI_PICO_TimerInterrupt** library](https://platformio.org/lib/show/12177/RPI_PICO_TimerInterrupt) or [**RPI_PICO_TimerInterrupt** library](https://platformio.org/lib/show/12273/RPI_PICO_TimerInterrupt) by using [Library Manager](https://platformio.org/lib/show/12177/RPI_PICO_TimerInterrupt/installation). Search for **RPI_PICO_TimerInterrupt** in [Platform.io Author's Libraries](https://platformio.org/lib/search?query=author:%22Khoi%20Hoang%22)
4. Please visit documentation for the other options and examples at [Project Configuration File](https://docs.platformio.org/page/projectconf.html)


---
---

### Packages' Patches

#### 1. For RP2040-based boards using [Earle Philhower arduino-pico core](https://github.com/earlephilhower/arduino-pico)

#### Important: Only necessary if you use core v1.4.0-

#### 1.1 To use BOARD_NAME

 **To be able to automatically detect and display BOARD_NAME on RP2040-based boards (RASPBERRY_PI_PICO, ADAFRUIT_FEATHER_RP2040, GENERIC_RP2040, etc) boards**, you have to copy the file [RP2040 platform.txt](Packages_Patches/rp2040/hardware/rp2040/1.4.0) into rp2040 directory (~/.arduino15/packages/rp2040/hardware/rp2040/1.4.0). 

Supposing the rp2040 core version is 1.4.0. This file must be copied into the directory:

- `~/.arduino15/packages/rp2040/hardware/rp2040/1.4.0/platform.txt`

Whenever a new version is installed, remember to copy this file into the new version directory. For example, new version is x.yy.zz
This file must be copied into the directory:

- `~/.arduino15/packages/rp2040/hardware/rp2040/x.yy.zz/platform.txt`

With core after v1.4.0, this step is not necessary anymore thanks to the PR [Add -DBOARD_NAME="{build.board}" #136](https://github.com/earlephilhower/arduino-pico/pull/136).

#### 1.2 To avoid compile error relating to microsecondsToClockCycles

Some libraries, such as [Adafruit DHT-sensor-library](https://github.com/adafruit/DHT-sensor-library), require the definition of microsecondsToClockCycles(). **To be able to compile and run on RP2040-based boards**, you have to copy the files in [**RP2040 Arduino.h**](Packages_Patches/rp2040/hardware/rp2040/1.4.0/cores/rp2040/Arduino.h) into rp2040 directory (~/.arduino15/packages/rp2040/hardware/rp2040/1.4.0).

Supposing the rp2040 core version is 1.4.0. This file must be copied to replace:

- `~/.arduino15/packages/rp2040/hardware/rp2040/1.4.0/cores/rp2040/Arduino.h`

Whenever a new version is installed, remember to copy this file into the new version directory. For example, new version is x.yy.zz
This file must be copied to replace:

- `~/.arduino15/packages/rp2040/hardware/rp2040/x.yy.zz/cores/rp2040/Arduino.h`

With core after v1.4.0, this step is not necessary anymore thanks to the PR [Add defs for compatibility #142](https://github.com/earlephilhower/arduino-pico/pull/142).

---
---

### HOWTO Fix `Multiple Definitions` Linker Error

The current library implementation, using `xyz-Impl.h` instead of standard `xyz.cpp`, possibly creates certain `Multiple Definitions` Linker error in certain use cases.

You can include these `.hpp` or `.h` files

```
// Can be included as many times as necessary, without `Multiple Definitions` Linker Error
#include "RPi_Pico_TimerInterrupt.h"      //https://github.com/khoih-prog/RPI_PICO_TimerInterrupt

// Can be included as many times as necessary, without `Multiple Definitions` Linker Error
#include "RPi_Pico_ISR_Timer.hpp"         //https://github.com/khoih-prog/RPI_PICO_TimerInterrupt
```

in many files. But be sure to use the following `.h` file **in just 1 `.h`, `.cpp` or `.ino` file**, which must **not be included in any other file**, to avoid `Multiple Definitions` Linker Error

```
// To be included only in main(), .ino with setup() to avoid `Multiple Definitions` Linker Error
#include "RPi_Pico_ISR_Timer.h"           //https://github.com/khoih-prog/RPI_PICO_TimerInterrupt
```

Check the new [**multiFileProject** example](examples/multiFileProject) for a `HOWTO` demo.

---
---

## More useful Information

The RPI_PICO system timer peripheral provides a global microsecond timebase for the system, and generates interrupts based on this timebase. It supports the following features:
  • A single 64-bit counter, incrementing once per microsecond
  • This counter can be read from a pair of latching registers, for race-free reads over a 32-bit bus.
  • Four alarms: match on the lower 32 bits of counter, IRQ on match: **TIMER_IRQ_0-TIMER_IRQ_3**

---

Now with these new `16 ISR-based timers` (while consuming only **1 hardware timer**), the maximum interval is practically unlimited (limited only by unsigned long miliseconds). The accuracy is nearly perfect compared to software timers. The most important feature is they're ISR-based timers Therefore, their executions are not blocked by bad-behaving functions / tasks.
This important feature is absolutely necessary for mission-critical tasks. 

The `ISR_Timer_Complex` example will demonstrate the nearly perfect accuracy compared to software timers by printing the actual elapsed millisecs of each type of timers.
Being ISR-based timers, their executions are not blocked by bad-behaving functions / tasks, such as connecting to WiFi, Internet and Blynk services. You can also have many `(up to 16)` timers to use.
This non-being-blocked important feature is absolutely necessary for mission-critical tasks. 
You'll see blynkTimer Software is blocked while system is connecting to WiFi / Internet / Blynk, as well as by blocking task in loop(), using delay() function as an example. The elapsed time then is very unaccurate


---
---

## Usage

Before using any Timer, you have to make sure the Timer has not been used by any other purpose.
`TIMER_IRQ_0, TIMER_IRQ_1, TIMER_IRQ_2 and TIMER_IRQ_3` are supported for RP2040-based boards.


### 1. Using only Hardware Timer directly

### 1.1 Init Hardware Timer

```
// Select the timer you're using, from ITimer0(0)-ITimer3(3)
// Init RPI_PICO_Timer
RPI_PICO_Timer ITimer1(1);
```

### 1.2 Set Hardware Timer Interval and attach Timer Interrupt Handler function

Use one of these functions with **interval in unsigned long microseconds**

```   
// interval (in us), callback is ISR
bool setInterval(unsigned long interval, pico_timer_callback callback);

// interval (in us), callback is ISR
bool attachInterruptInterval(unsigned long interval, pico_timer_callback callback)
```

as follows

```
void TimerHandler()
{
  // Doing something here inside ISR
}

#define TIMER_INTERVAL_MS        5000L

// Init RPI_PICO_Timer
RPI_PICO_Timer ITimer(0);

void setup()
{
  ....
  
  // Interval in unsigned long microseconds
  if (ITimer.attachInterruptInterval(TIMER_INTERVAL_MS * 1000, TimerHandler))
    Serial.println("Starting ITimer OK, millis() = " + String(millis()));
  else
    Serial.println("Can't set ITimer. Select another freq. or timer");
}  
```

### 1.3 Set Hardware Timer Frequency and attach Timer Interrupt Handler function

Use one of these functions with **frequency in float Hz**

```
// frequency (in Hz), callback is ISR
bool setFrequency(float frequency, pico_timer_callback callback)

// frequency (in Hz), callback is ISR
bool attachInterrupt(float frequency, timer_callback callback);
```

as follows

```
void TimerHandler()
{
  // Doing something here inside ISR
}

#define TIMER_FREQ_HZ        5555.555

// Init RPI_PICO_Timer
RPI_PICO_Timer ITimer(0);

void setup()
{
  ....
  
  // Frequency in float Hz
  if (ITimer.attachInterrupt(TIMER_FREQ_HZ, TimerHandler))
    Serial.println("Starting ITimer OK, millis() = " + String(millis()));
  else
    Serial.println("Can't set ITimer. Select another freq. or timer");
}  
```


### 2. Using 16 ISR_based Timers from 1 Hardware Timer

### 2.1 Important Note

The 16 ISR_based Timers, designed for long timer intervals, only support using **unsigned long millisec intervals**. If you have to use much higher frequency or sub-millisecond interval, you have to use the Hardware Timers directly as in [1.3 Set Hardware Timer Frequency and attach Timer Interrupt Handler function](#13-set-hardware-timer-frequency-and-attach-timer-interrupt-handler-function)

### 2.2 Init Hardware Timer and ISR-based Timer

```
// Init RPI_PICO_Timer
RPI_PICO_Timer ITimer1(1);

// Init ISR_Timer
// Each ISR_Timer can service 16 different ISR-based timers
RPI_PICO_ISR_Timer ISR_timer;
```

### 2.3 Set Hardware Timer Interval and attach Timer Interrupt Handler functions

```
void TimerHandler()
{
  ISR_timer.run();
}

#define HW_TIMER_INTERVAL_MS          50L

#define TIMER_INTERVAL_2S             2000L
#define TIMER_INTERVAL_5S             5000L
#define TIMER_INTERVAL_11S            11000L
#define TIMER_INTERVAL_101S           101000L

// In AVR, avoid doing something fancy in ISR, for example complex Serial.print with String() argument
// The pure simple Serial.prints here are just for demonstration and testing. Must be eliminate in working environment
// Or you can get this run-time error / crash
void doingSomething2s()
{
  // Doing something here inside ISR every 2 seconds
}
  
void doingSomething5s()
{
  // Doing something here inside ISR every 5 seconds
}

void doingSomething11s()
{
  // Doing something here inside ISR  every 11 seconds
}

void doingSomething101s()
{
  // Doing something here inside ISR every 101 seconds
}

void setup()
{
  ....
  
  if (ITimer1.attachInterruptInterval(HW_TIMER_INTERVAL_MS * 1000, TimerHandler))
  {
    Serial.print(F("Starting ITimer1 OK, millis() = ")); Serial.println(millis());
  }
  else
    Serial.println(F("Can't set ITimer1. Select another freq. or timer"));

  // Just to demonstrate, don't use too many ISR Timers if not absolutely necessary
  // You can use up to 16 timer for each ISR_Timer
  ISR_timer.setInterval(TIMER_INTERVAL_2S, doingSomething2s);
  ISR_timer.setInterval(TIMER_INTERVAL_5S, doingSomething5s);
  ISR_timer.setInterval(TIMER_INTERVAL_11S, doingSomething11s);
  ISR_timer.setInterval(TIMER_INTERVAL_101S, doingSomething101s);
}  
```

---
---

### Examples 

 1. [Argument_Complex](examples/Argument_Complex)
 2. [Argument_None](examples/Argument_None)
 3. [Argument_Simple](examples/Argument_Simple) 
 4. [Change_Interval](examples/Change_Interval) 
 5. [ISR_16_Timers_Array_Complex](examples/ISR_16_Timers_Array_Complex)
 6. [ISR_Timers_Array_Simple](examples/ISR_Timers_Array_Simple)
 7. [RPM_Measure](examples/RPM_Measure)
 8. [SwitchDebounce](examples/SwitchDebounce) 
 9. [TimerInterruptTest](examples/TimerInterruptTest)
10. [**multiFileProject**](examples/multiFileProject). **New**

---
---

### Example [ISR_Timers_Array_Simple](examples/ISR_Timers_Array_Simple)

```
// These define's must be placed at the beginning before #include "TimerInterrupt_Generic.h"
// _TIMERINTERRUPT_LOGLEVEL_ from 0 to 4
// Don't define _TIMERINTERRUPT_LOGLEVEL_ > 0. Only for special ISR debugging only. Can hang the system.
#define TIMER_INTERRUPT_DEBUG         1
#define _TIMERINTERRUPT_LOGLEVEL_     4

// Can be included as many times as necessary, without `Multiple Definitions` Linker Error
#include "RPi_Pico_TimerInterrupt.h"

// To be included only in main(), .ino with setup() to avoid `Multiple Definitions` Linker Error
#include "RPi_Pico_ISR_Timer.h"

#include <SimpleTimer.h>              // https://github.com/jfturcot/SimpleTimer

// Init RPI_PICO_Timer
RPI_PICO_Timer ITimer1(1);

RPI_PICO_ISR_Timer ISR_timer;

#ifndef LED_BUILTIN
  #define LED_BUILTIN       25
#endif

#define LED_TOGGLE_INTERVAL_MS        1000L

// You have to use longer time here if having problem because Arduino AVR clock is low, 16MHz => lower accuracy.
// Tested OK with 1ms when not much load => higher accuracy.
#define TIMER_INTERVAL_MS            1L

volatile uint32_t startMillis = 0;

volatile uint32_t deltaMillis2s = 0;
volatile uint32_t deltaMillis5s = 0;

volatile uint32_t previousMillis2s = 0;
volatile uint32_t previousMillis5s = 0;


bool TimerHandler(struct repeating_timer *t)
{
  static bool toggle  = false;
  static int timeRun  = 0;

  ISR_timer.run();

  // Toggle LED every LED_TOGGLE_INTERVAL_MS = 2000ms = 2s
  if (++timeRun == ((LED_TOGGLE_INTERVAL_MS) / TIMER_INTERVAL_MS) )
  {
    timeRun = 0;

    //timer interrupt toggles pin LED_BUILTIN
    digitalWrite(LED_BUILTIN, toggle);
    toggle = !toggle;
  }

  return true;
}

void doingSomething2s()
{
  unsigned long currentMillis  = millis();

  deltaMillis2s    = currentMillis - previousMillis2s;
  previousMillis2s = currentMillis;
}

void doingSomething5s()
{
  unsigned long currentMillis  = millis();

  deltaMillis5s    = currentMillis - previousMillis5s;
  previousMillis5s = currentMillis;
}

/////////////////////////////////////////////////

#define SIMPLE_TIMER_MS        2000L

// Init SimpleTimer
SimpleTimer simpleTimer;

// Here is software Timer, you can do somewhat fancy stuffs without many issues.
// But always avoid
// 1. Long delay() it just doing nothing and pain-without-gain wasting CPU power.Plan and design your code / strategy ahead
// 2. Very long "do", "while", "for" loops without predetermined exit time.
void simpleTimerDoingSomething2s()
{
  static unsigned long previousMillis = startMillis;

  unsigned long currMillis = millis();

  Serial.print(F("SimpleTimer : programmed ")); Serial.print(SIMPLE_TIMER_MS);
  Serial.print(F("ms, current time ms : ")); Serial.print(currMillis);
  Serial.print(F(", Delta ms : ")); Serial.println(currMillis - previousMillis);

  Serial.print(F("Timer2s actual : ")); Serial.println(deltaMillis2s);
  Serial.print(F("Timer5s actual : ")); Serial.println(deltaMillis5s);
  
  previousMillis = currMillis;
}

////////////////////////////////////////////////

void setup()
{
  pinMode(LED_BUILTIN, OUTPUT);

  Serial.begin(115200);
  while (!Serial);

  Serial.print(F("\nStarting ISR_Timers_Array_Simple on ")); Serial.println(BOARD_NAME);
  Serial.println(RPI_PICO_TIMER_INTERRUPT_VERSION);
  Serial.print(F("CPU Frequency = ")); Serial.print(F_CPU / 1000000); Serial.println(F(" MHz"));

  if (ITimer1.attachInterruptInterval(TIMER_INTERVAL_MS * 1000, TimerHandler))
  {
    Serial.print(F("Starting ITimer1 OK, millis() = ")); Serial.println(millis());
  }
  else
    Serial.println(F("Can't set ITimer1. Select another freq. or timer"));

  previousMillis5s = previousMillis2s = millis();  

  ISR_timer.setInterval(2000L, doingSomething2s);
  ISR_timer.setInterval(5000L, doingSomething5s);

  // You need this timer for non-critical tasks. Avoid abusing ISR if not absolutely necessary.
  simpleTimer.setInterval(SIMPLE_TIMER_MS, simpleTimerDoingSomething2s);
}

#define BLOCKING_TIME_MS      10000L

void loop()
{
  // This unadvised blocking task is used to demonstrate the blocking effects onto the execution and accuracy to Software timer
  // You see the time elapse of ISR_Timer still accurate, whereas very unaccurate for Software Timer
  // The time elapse for 2000ms software timer now becomes 3000ms (BLOCKING_TIME_MS)
  // While that of ISR_Timer is still prefect.
  delay(BLOCKING_TIME_MS);

  // You need this Software timer for non-critical tasks. Avoid abusing ISR if not absolutely necessary
  // You don't need to and never call ISR_Timer.run() here in the loop(). It's already handled by ISR timer.
  simpleTimer.run();
}
```
---
---

### Debug Terminal Output Samples

### 1. ISR_Timers_Array_Simple on RASPBERRY_PI_PICO

The following is the sample terminal output when running example [ISR_Timers_Array_Simple](examples/ISR_Timers_Array_Simple) to demonstrate the accuracy of ISR Hardware Timer, **especially when system is very busy**. The ISR timer is **programmed for 2s, is activated exactly after 2.000s !!!**

While software timer, **programmed for 2s, is activated after more than 10.000s !!!**

```

Starting ISR_Timers_Array_Simple on RASPBERRY_PI_PICO
RPi_Pico_TimerInterrupt v1.2.0
CPU Frequency = 125 MHz
[TISR] _timerNo = 1, Clock (Hz) = 1000000.00, _fre (Hz) = 1000.00
[TISR] _count = 0-1000
[TISR] add_repeating_timer_us = 1000
Starting  ITimer3 OK, millis() = 1707
SimpleTimer : programmed 2000ms, current time ms : 11707, Delta ms : 11707
Timer2s actual : 2000
Timer5s actual : 5000
SimpleTimer : programmed 2000ms, current time ms : 21708, Delta ms : 10001
Timer2s actual : 2000
Timer5s actual : 5000
SimpleTimer : programmed 2000ms, current time ms : 31708, Delta ms : 10000
Timer2s actual : 2000
Timer5s actual : 5000
```

---

### 2. TimerInterruptTest on RASPBERRY_PI_PICO

The following is the sample terminal output when running example [TimerInterruptTest](examples/TimerInterruptTest) to demonstrate how to start/stop Hardware Timers on RP2040-based boards.

```
Starting TimerInterruptTest on RASPBERRY_PI_PICO
RPi_Pico_TimerInterrupt v1.2.0
CPU Frequency = 125 MHz
[TISR] _timerNo = 0, Clock (Hz) = 1000000.00, _fre (Hz) = 1.00
[TISR] _count = 0-1000000
[TISR] add_repeating_timer_us = 1000000
Starting ITimer0 OK, millis() = 882
[TISR] _timerNo = 1, Clock (Hz) = 1000000.00, _fre (Hz) = 0.33
[TISR] _count = 0-3000000
[TISR] add_repeating_timer_us = 3000000
Starting  ITimer1 OK, millis() = 1782
ITimer0 called, millis() = 2781
ITimer0 called, millis() = 3781
ITimer0 called, millis() = 4781
ITimer1 called, millis() = 4782
Stop ITimer0, millis() = 5001
ITimer1 called, millis() = 7782
Start ITimer0, millis() = 10002
ITimer1 called, millis() = 10782
ITimer0 called, millis() = 11002
ITimer0 called, millis() = 12002
ITimer0 called, millis() = 13002
ITimer1 called, millis() = 13782
ITimer0 called, millis() = 14002
Stop ITimer1, millis() = 15001
ITimer0 called, millis() = 15002
Stop ITimer0, millis() = 15003
Start ITimer0, millis() = 20004
ITimer0 called, millis() = 21004
ITimer0 called, millis() = 22004
ITimer0 called, millis() = 23004
ITimer0 called, millis() = 24004
ITimer0 called, millis() = 25004
Stop ITimer0, millis() = 25005
```

---


### 3. Change_Interval on RASPBERRY_PI_PICO

The following is the sample terminal output when running example [Change_Interval](examples/Change_Interval) to demonstrate how to change Timer Interval on-the-fly on RP2040-based boards.

```
Starting Change_Interval on RASPBERRY_PI_PICO
RPi_Pico_TimerInterrupt v1.2.0
CPU Frequency = 125 MHz
[TISR] _timerNo = 0, Clock (Hz) = 1000000.00, _fre (Hz) = 0.50
[TISR] _count = 0-2000000
[TISR] add_repeating_timer_us = 2000000
Starting  ITimer0 OK, millis() = 2363
[TISR] _timerNo = 1, Clock (Hz) = 1000000.00, _fre (Hz) = 0.20
[TISR] _count = 0-5000000
[TISR] add_repeating_timer_us = 5000000
Starting  ITimer1 OK, millis() = 1544
ITimer0: millis() = 3544
ITimer0: millis() = 5544
ITimer1: millis() = 6544
ITimer0: millis() = 7544
ITimer0: millis() = 9544
Time = 10001, Timer0Count = 4, Timer1Count = 1
ITimer0: millis() = 11544
ITimer1: millis() = 11544
ITimer0: millis() = 13544
ITimer0: millis() = 15544
ITimer1: millis() = 16544
ITimer0: millis() = 17544
ITimer0: millis() = 19544
Time = 20002, Timer0Count = 9, Timer1Count = 3
[TISR] RPI_PICO_TimerInterrupt: _timerNo = 0 , _fre = 1000000.00
[TISR] _count = 0 - 4000000
[TISR] add_repeating_timer_us = 4000000
[TISR] RPI_PICO_TimerInterrupt: _timerNo = 1 , _fre = 1000000.00
[TISR] _count = 0 - 10000000
[TISR] add_repeating_timer_us = 10000000
Changing Interval, Timer0 = 4000,  Timer1 = 10000
ITimer0: millis() = 24002
ITimer0: millis() = 28002
ITimer1: millis() = 30003
Time = 30003, Timer0Count = 11, Timer1Count = 4
ITimer0: millis() = 32002
ITimer0: millis() = 36002
ITimer0: millis() = 40002
ITimer1: millis() = 40003
Time = 40004, Timer0Count = 14, Timer1Count = 5
[TISR] RPI_PICO_TimerInterrupt: _timerNo = 0 , _fre = 1000000.00
[TISR] _count = 0 - 2000000
[TISR] add_repeating_timer_us = 2000000
[TISR] RPI_PICO_TimerInterrupt: _timerNo = 1 , _fre = 1000000.00
[TISR] _count = 0 - 5000000
[TISR] add_repeating_timer_us = 5000000
Changing Interval, Timer0 = 2000,  Timer1 = 5000
ITimer0: millis() = 42004
ITimer0: millis() = 44004
ITimer1: millis() = 45004
ITimer0: millis() = 46004
ITimer0: millis() = 48004
ITimer0: millis() = 50004
ITimer1: millis() = 50004
Time = 50005, Timer0Count = 19, Timer1Count = 7
ITimer0: millis() = 52004
ITimer0: millis() = 54004
ITimer1: millis() = 55005
ITimer0: millis() = 56004
ITimer0: millis() = 58004
ITimer0: millis() = 60004
ITimer1: millis() = 60005
Time = 60006, Timer0Count = 24, Timer1Count = 9
[TISR] RPI_PICO_TimerInterrupt: _timerNo = 0 , _fre = 1000000.00
[TISR] _count = 0 - 4000000
[TISR] add_repeating_timer_us = 4000000
[TISR] RPI_PICO_TimerInterrupt: _timerNo = 1 , _fre = 1000000.00
[TISR] _count = 0 - 10000000
[TISR] add_repeating_timer_us = 10000000
Changing Interval, Timer0 = 4000,  Timer1 = 10000
ITimer0: millis() = 64006
```

---

### 4. SwitchDebounce on RASPBERRY_PI_PICO

The following is the sample terminal output when running example [SwitchDebounce](examples/SwitchDebounce)

```
Starting SwitchDebounce on RASPBERRY_PI_PICO
RPi_Pico_TimerInterrupt v1.2.0
CPU Frequency = 125 MHz
[TISR] _timerNo = 1, Clock (Hz) = 1000000.00, _fre (Hz) = 50.00
[TISR] _count = 0-20000
[TISR] add_repeating_timer_us = 20000
Starting  ITimer1 OK, millis() = 1302
SW Press, from millis() = 77377
SW Released, from millis() = 78077
SW Pressed total time ms = 700
SW Press, from millis() = 78257
SW Released, from millis() = 78577
SW Pressed total time ms = 320
SW Press, from millis() = 79057
SW Released, from millis() = 80238
SW Pressed total time ms = 1181
```

---

### 5. ISR_Timers_Array_Simple on ADAFRUIT_FEATHER_RP2040

The following is the sample terminal output when running example [ISR_Timers_Array_Simple](examples/ISR_Timers_Array_Simple) on ADAFRUIT_FEATHER_RP2040

```
Starting ISR_Timers_Array_Simple on RASPBERRY_PI_PICO
RPi_Pico_TimerInterrupt v1.2.0
CPU Frequency = 125 MHz
[TISR] _timerNo = 1, Clock (Hz) = 1000000.00, _fre (Hz) = 1000.00
[TISR] _count = 0-1000
[TISR] add_repeating_timer_us = 1000
Starting ITimer1 OK, millis() = 1701
SimpleTimer : programmed 2000ms, current time ms : 11707, Delta ms : 11707
Timer2s actual : 2000
Timer5s actual : 5000
SimpleTimer : programmed 2000ms, current time ms : 21707, Delta ms : 10000
Timer2s actual : 2000
Timer5s actual : 5000
SimpleTimer : programmed 2000ms, current time ms : 31708, Delta ms : 10001
Timer2s actual : 2000
Timer5s actual : 5000
```

### 6. ISR_16_Timers_Array_Complex on ADAFRUIT_ITSYBITSY_RP2040

The following is the sample terminal output when running example [ISR_16_Timers_Array_Complex](examples/ISR_16_Timers_Array_Complex) on ADAFRUIT_ITSYBITSY_RP2040

```
Starting ISR_16_Timers_Array_Complex on RASPBERRY_PI_PICO
RPi_Pico_TimerInterrupt v1.2.0
CPU Frequency = 125 MHz
[TISR] _timerNo = 1, Clock (Hz) = 1000000.00, _fre (Hz) = 100.00
[TISR] _count = 0-10000
[TISR] add_repeating_timer_us = 10000
Starting ITimer OK, millis() = 1797
SimpleTimer : 2, ms : 11798, Dms : 10001
Timer : 0, programmed : 5000, actual : 5005
Timer : 1, programmed : 10000, actual : 0
Timer : 2, programmed : 15000, actual : 0
Timer : 3, programmed : 20000, actual : 0
Timer : 4, programmed : 25000, actual : 0
Timer : 5, programmed : 30000, actual : 0
Timer : 6, programmed : 35000, actual : 0
Timer : 7, programmed : 40000, actual : 0
Timer : 8, programmed : 45000, actual : 0
Timer : 9, programmed : 50000, actual : 0
Timer : 10, programmed : 55000, actual : 0
Timer : 11, programmed : 60000, actual : 0
Timer : 12, programmed : 65000, actual : 0
Timer : 13, programmed : 70000, actual : 0
Timer : 14, programmed : 75000, actual : 0
Timer : 15, programmed : 80000, actual : 0
SimpleTimer : 2, ms : 21800, Dms : 10002
Timer : 0, programmed : 5000, actual : 4995
Timer : 1, programmed : 10000, actual : 10010
Timer : 2, programmed : 15000, actual : 15005
Timer : 3, programmed : 20000, actual : 0
Timer : 4, programmed : 25000, actual : 0
Timer : 5, programmed : 30000, actual : 0
Timer : 6, programmed : 35000, actual : 0
Timer : 7, programmed : 40000, actual : 0
Timer : 8, programmed : 45000, actual : 0
Timer : 9, programmed : 50000, actual : 0
Timer : 10, programmed : 55000, actual : 0
Timer : 11, programmed : 60000, actual : 0
Timer : 12, programmed : 65000, actual : 0
Timer : 13, programmed : 70000, actual : 0
Timer : 14, programmed : 75000, actual : 0
Timer : 15, programmed : 80000, actual : 0
SimpleTimer : 2, ms : 31802, Dms : 10002
Timer : 0, programmed : 5000, actual : 4995
Timer : 1, programmed : 10000, actual : 10000
Timer : 2, programmed : 15000, actual : 15005
Timer : 3, programmed : 20000, actual : 20010
Timer : 4, programmed : 25000, actual : 25005
Timer : 5, programmed : 30000, actual : 0
Timer : 6, programmed : 35000, actual : 0
Timer : 7, programmed : 40000, actual : 0
Timer : 8, programmed : 45000, actual : 0
Timer : 9, programmed : 50000, actual : 0
Timer : 10, programmed : 55000, actual : 0
Timer : 11, programmed : 60000, actual : 0
Timer : 12, programmed : 65000, actual : 0
Timer : 13, programmed : 70000, actual : 0
Timer : 14, programmed : 75000, actual : 0
Timer : 15, programmed : 80000, actual : 0
SimpleTimer : 2, ms : 41804, Dms : 10002
Timer : 0, programmed : 5000, actual : 4995
Timer : 1, programmed : 10000, actual : 10000
Timer : 2, programmed : 15000, actual : 15005
Timer : 3, programmed : 20000, actual : 20010
Timer : 4, programmed : 25000, actual : 25005
Timer : 5, programmed : 30000, actual : 30010
Timer : 6, programmed : 35000, actual : 35005
Timer : 7, programmed : 40000, actual : 0
Timer : 8, programmed : 45000, actual : 0
Timer : 9, programmed : 50000, actual : 0
Timer : 10, programmed : 55000, actual : 0
Timer : 11, programmed : 60000, actual : 0
Timer : 12, programmed : 65000, actual : 0
Timer : 13, programmed : 70000, actual : 0
Timer : 14, programmed : 75000, actual : 0
Timer : 15, programmed : 80000, actual : 0
SimpleTimer : 2, ms : 51807, Dms : 10003
Timer : 0, programmed : 5000, actual : 4995
Timer : 1, programmed : 10000, actual : 10000
Timer : 2, programmed : 15000, actual : 14995
Timer : 3, programmed : 20000, actual : 20000
Timer : 4, programmed : 25000, actual : 25005
Timer : 5, programmed : 30000, actual : 30010
Timer : 6, programmed : 35000, actual : 35005
Timer : 7, programmed : 40000, actual : 40010
Timer : 8, programmed : 45000, actual : 45005
Timer : 9, programmed : 50000, actual : 50010
Timer : 10, programmed : 55000, actual : 0
Timer : 11, programmed : 60000, actual : 0
Timer : 12, programmed : 65000, actual : 0
Timer : 13, programmed : 70000, actual : 0
Timer : 14, programmed : 75000, actual : 0
Timer : 15, programmed : 80000, actual : 0
SimpleTimer : 2, ms : 61809, Dms : 10002
Timer : 0, programmed : 5000, actual : 5005
Timer : 1, programmed : 10000, actual : 10000
Timer : 2, programmed : 15000, actual : 15005
Timer : 3, programmed : 20000, actual : 20000
Timer : 4, programmed : 25000, actual : 25005
Timer : 5, programmed : 30000, actual : 30000
Timer : 6, programmed : 35000, actual : 35005
Timer : 7, programmed : 40000, actual : 40010
Timer : 8, programmed : 45000, actual : 45005
Timer : 9, programmed : 50000, actual : 50010
Timer : 10, programmed : 55000, actual : 55005
Timer : 11, programmed : 60000, actual : 60010
Timer : 12, programmed : 65000, actual : 0
Timer : 13, programmed : 70000, actual : 0
Timer : 14, programmed : 75000, actual : 0
Timer : 15, programmed : 80000, actual : 0
SimpleTimer : 2, ms : 71812, Dms : 10003
Timer : 0, programmed : 5000, actual : 5005
Timer : 1, programmed : 10000, actual : 10000
Timer : 2, programmed : 15000, actual : 15005
Timer : 3, programmed : 20000, actual : 20000
Timer : 4, programmed : 25000, actual : 25005
Timer : 5, programmed : 30000, actual : 30000
Timer : 6, programmed : 35000, actual : 35005
Timer : 7, programmed : 40000, actual : 40010
Timer : 8, programmed : 45000, actual : 45005
Timer : 9, programmed : 50000, actual : 50010
Timer : 10, programmed : 55000, actual : 55005
Timer : 11, programmed : 60000, actual : 60010
Timer : 12, programmed : 65000, actual : 65005
Timer : 13, programmed : 70000, actual : 70010
Timer : 14, programmed : 75000, actual : 0
Timer : 15, programmed : 80000, actual : 0
SimpleTimer : 2, ms : 81814, Dms : 10002
Timer : 0, programmed : 5000, actual : 5005
Timer : 1, programmed : 10000, actual : 10000
Timer : 2, programmed : 15000, actual : 14995
Timer : 3, programmed : 20000, actual : 20000
Timer : 4, programmed : 25000, actual : 24995
Timer : 5, programmed : 30000, actual : 30000
Timer : 6, programmed : 35000, actual : 35005
Timer : 7, programmed : 40000, actual : 40000
Timer : 8, programmed : 45000, actual : 45005
Timer : 9, programmed : 50000, actual : 50010
Timer : 10, programmed : 55000, actual : 55005
Timer : 11, programmed : 60000, actual : 60010
Timer : 12, programmed : 65000, actual : 65005
Timer : 13, programmed : 70000, actual : 70010
Timer : 14, programmed : 75000, actual : 75005
Timer : 15, programmed : 80000, actual : 80010
```

---
---

### Debug

Debug is enabled by default on Serial.

You can also change the debugging level (_TIMERINTERRUPT_LOGLEVEL_) from 0 to 4

```cpp
// These define's must be placed at the beginning before #include "RPI_PICO_TimerInterrupt.h"
// _TIMERINTERRUPT_LOGLEVEL_ from 0 to 4
// Don't define _TIMERINTERRUPT_LOGLEVEL_ > 0. Only for special ISR debugging only. Can hang the system.
#define TIMER_INTERRUPT_DEBUG         0
#define _TIMERINTERRUPT_LOGLEVEL_     0
```

---

### Troubleshooting

If you get compilation errors, more often than not, you may need to install a newer version of the core for Arduino boards.

Sometimes, the library will only work if you update the board core to the latest version because I am using newly added functions.


---
---

### Issues

Submit issues to: [RPI_PICO_TimerInterrupt issues](https://github.com/khoih-prog/RPI_PICO_TimerInterrupt/issues)

---

## TO DO

1. Search for bug and improvement.
2. Add support to RP2040-based boards such as RASPBERRY_PI_PICO, using [**Arduino-mbed RP2040** core](https://github.com/arduino/ArduinoCore-mbed)

---

## DONE

1. Basic hardware timers for **RP2040-based boards such as RASPBERRY_PI_PICO**, using [**Earle Philhower's arduino-pico** core](https://github.com/earlephilhower/arduino-pico)
2. More hardware-initiated software-enabled timers
3. Longer time interval
4. Add Version String 
5. Add Table of Contents
6. Add support to new boards (**ADAFRUIT_ITSYBITSY_RP2040, ADAFRUIT_QTPY_RP2040, ADAFRUIT_STEMMAFRIEND_RP2040, ADAFRUIT_TRINKEYQT_RP2040, ADAFRUIT_MACROPAD_RP2040, SPARKFUN_PROMICRO_RP2040, Nano_RP2040_Connect, etc.**) using the arduino-pico core
7. Fix `multiple-definitions` linker error
8. Optimize library code by using `reference-passing` instead of `value-passing`


---
---

### Contributions and Thanks

Many thanks for everyone for bug reporting, new feature suggesting, testing and contributing to the development of this library.


---

## Contributing

If you want to contribute to this project:

- Report bugs and errors
- Ask for enhancements
- Create issues and pull requests
- Tell other people about this library

---

### License

- The library is licensed under [MIT](https://github.com/khoih-prog/RPI_PICO_TimerInterrupt/blob/master/LICENSE)

---

## Copyright

Copyright 2021- Khoi Hoang


