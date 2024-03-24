---
title: "Fan Speed Supermicro IPMI"
description: "Provides instructions on setting the thresholds of fans with the ipmitool utility on supermicro boards."
weight: 15
tags:
- hardware
draft: true
---



# Fan Speed Control on Supermicro Mainboards

On Supermicro mainboards with a BMC, the fans are not controlled by the operating system or the BIOS, but the BMC. These boards are made for industrial applications. In those environments fan speeds are often above 5000 RPM, resulting in high fan noise. While this is not a problem for server racks, in home use, this is often not acceptable. 
When using consumer grade fans attached to the mainboard, this leads to issues and event logs reporting faults. 

This guide shows how to set the thresholds for fans correctly. 
## Prerequisits

- Shell on the server with [**IPMICFG**](https://www.supermicro.com/en/solutions/management-software/ipmi-utilities)  [**ipmitool**](https://github.com/ipmitool/ipmitool) installed to set sensor values.
- IPMIView for monitoring the sensor values (not strictly necessary). 

## Checking the sensor readings in the Web IPMI tool

To check the real max values read by the IPMI set the **Fan Mode** to **Full Speed**. 
{{< trueimage src="/images/SCALE/Hardware/SupermicroFanModeFullSpeed.png" alt="Fan Mode" id="Fan Mode" >}}



In the Sensor readings tab select Fan sensors and enable **Auto Refresh** and **Show Thresholds** 
{{ < trueimage src="/images/SCALE/Hardware/SupermicroFanSensorsNR.png" alt="Fan Sensor NR" id="Fan Sensor NR" >}}

The color indicates the state of the Sensor 
- Green: Sensor is reading normal 
- Red: Sensor has reached a critical state. Immediate action is required to resolve the problem.

FAN1 reports a Reading of 2000 RPM. Since the High NR (Non Recoverable) is also set to 2000 RPM. The Health Event Log shows:

{{< trueimage src="/images/SCALE/Hardware/SupermicroFanEventLog.png" alt="Fan Health Event Log" id="Fan Health Event Log" >}}


The color indicates the state of the Sensor 
- Green: Sensor is reading normal 
- Red: Sensor has reached a critical state. Immediate action is required to resolve the problem.

## Setting the upper thresholds 
In a Console on the operating system itself, the `ipmitool` utility can be used to set the values

- using the commands 
```
ipmitool senosr thresh FAN1 upper <NORMAL> <CT> <NR> 
ipmitool senosr thresh FAN1 lower <NORMAL> <CT> <NR>
```

The three values for the upper and lower values are: 
- **Normal**: Fan speed is within the Non Critical range  
- **Critical Threshold (CT)**: reported fan speed is below or above the critical value and  has reached a critical state.
- **Non Recoverable (NR)**: The BMC is not able to set the fan to a non critical Value and assumes a critical failure of the sensor.

in the terminal use the command:
```
ipmitool senosr thresh FAN1 upper 2000 2100 2200 
```

in this case the values are set to: 
- **Normal** is set to 2000, since the max speed of the fan is 2000 RPM. 
- **CT** is set 2100 RPM, its a value above the max speed of FAN1
- **NR** is set to 2200 this speed can not be reached by the fan under normal circumstances. And A **NR** event would indicate a failure of the fan or sensor. 

It might be useful to set **CT** to the max speed of the fan to report this state. 

This resolves the warning from **FAN1**

{{ < trueimage src="/images/SCALE/Hardware/SupermicroFanSensorNormalHighRPM.png" alt="Fan Sensor Normal High RPM" id="Fan Sensor Normal High RPM" >}}


Even though the state is **Normal** the fans are running on max RPM. In the **Fan Mode** tab, set the mode to **Standard Speed** or **Optimal Speed**. If the Fans are not spinning down, this is because FAN1 had a **NR** event. 
To resolve this issue, to go **Maintanace**, under **Unit Reset** perform a reboot of the IPMI device. 
    
{{ < trueimage src="/images/SCALE/Hardware/SupermicroRebootIPMI.png" alt="Reboot IPMI" id="Reboot IPMI" >}}

After logging back into the IPMI web view, check the sensor readings again. The IPMI should have adjusted the fan speeds according to the Fan mode. 

{{ < trueimage src="/images/SCALE/Hardware/SupermicroFanSensorNormalLowRPM.png" alt="Fan Sensor Normal Low RPM" id="Fan Sensor Normal Low RPM" >}}


Setting the lower is similar to setting the upper speed. Using consumer PC fans has a view caveats though, since most fans stop below a certain RPM value. Since stopping would lead to a **NR** event the the values should be adjusted accordingly.

For example FAN1 is a Noctua A12 that would stop below 300 RPM. The values would be set with the command

```
ipmitool senosr thresh FAN1 lower 0 100 200
```

This allows for low fan speeds and not triggering a **CT** or **NR** event. 