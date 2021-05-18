---
autogenerated: true
title: National Instruments
layout: page
---

## National Instruments ("NI100X") Adapter

<table>
<tr>
<td markdown="1">

**Summary:**

</td>
<td markdown="1">

Controls NI I/O boards through NI-DAQmx interface

</td>
</tr>
<tr>
<td markdown="1">

**Author:**

</td>
<td markdown="1">

Nenad Amodaj, 100X Imaging Inc

</td>
</tr>
<tr>
<td markdown="1">

**License:**

</td>
<td markdown="1">

LGPL License

</td>
</tr>
<tr>
<td markdown="1">

**Platforms:**

</td>
<td markdown="1">

Windows only

</td>
</tr>
</table>

This adapter provides digital and analog signal control for National
Instruments cards. The version of the NI-DAQmx library used to build
this adapter is 9.2. Most (or all) of the newer NI boards should be
compatible. NI software (drivers) must be installed. LabView is not
required.

Our NI adapter allows for two different "device types": digital and
analog I/O.

### Digital I/O

This device type is used to send binary digital signals. NI cards have
one or more "ports" of digital lines, and each port contains one or more
digital output lines. The easiest way to get a list of the ports you
have on your card is to bring up NI-MAX, right-click on the card in the
device list, and look at the pin diagram. You will also want to know the
device name from NI-MAX (typically "Dev1", but if you have installed a
previous NI card, then it may be "Dev2", etc., and devices can also be
renamed in NI-MAX).

When adding a DigitalIO device in the Hardware Configuration Wizard, you
will need to provide the IOChannel property. This is the port that the
device is allowed to send output to. If you have multiple ports on your
card, then you will have to create a separate DigitalIO device for each
port you want to control. As of 1.4.22, the wizard will automatically
determine which ports are available and provide you with a dropdown
menu. In prior versions, you will need to type in the name manually (try
"Dev1/port0", for example).

Once you have set up the device, you can manipulate the digital output
signal by changing the "State" property. This is an unsigned 32-bit
integer, with each bit corresponding to one of the output lines of the
selected port (up to the limit of the number of lines in the port). So
for example, if you have an 8-bit port, then the values of 0 through 255
(2^8 - 1) would be valid. Setting State to 1 would turn on the first
line; setting it to 2 would turn on the second line; setting it to 3
would turn on the first two lines; et cetera. You can also set it to -1
to turn on all lines simultaneously.

### Analog I/O

The AnalogIO device can be used to control an analog output line from
the card. As with DigitalIO devices, you are required to specify the
output line when creating the device in the Hardware Configuration
Wizard, as the "IOChannel" property. You can see a list of available
pins to use for analog output by running NI-MAX, opening the "Devices
and Interfaces" section, finding your card, right-clicking on it, and
selecting the "Device Pinouts" option. Analog lines will have names like
"AO 0". However, for use in µManager, no spaces are used, and the device
name must be provided as a prefix, so for example, you could use
"Dev1/ao0" as the IOChannel property value.

Once properly configured, you can adjust the output voltage of the
device by setting the "Volts" property.

As of 1.4.22, the Hardware Configuration Wizard will provide you with a
convenient dropdown list of valid analog lines, instead of requiring you
to manually type a value in. However, as a side-effect of this change,
the IOChannel property is now case-sensitive. Configuration files with
names like "Dev1/AO1" will fail to load because the device is expecting
a name like "Dev1/ao1" (with lower-case "ao" instead of "AO"). You can
open the configuration file in a text editor and replace the string, and
the modified configuration file should load correctly.

You can use the "DA Z Stage" device (under the Utilities section of the
Hardware Configuration Wizard) to use an AnalogIO device as a stage
controller. This can be useful, for example, for piezo-driven stages.
The DA stage simply maps the output voltage range of the AnalogIO device
linearly to a micron range (e.g. 0 volts = 0µm, 5 volts = 200µm). This
range can be configured when setting up the DA stage device.

### Hardware Triggering

As of version 1.4.22, the NI100x adapter supports analog and digital
hardware triggering for fast acquisitions, for certain NI cards/boxes
only. Some cards support triggering of analog output but not of digital
output.

It is sometimes tricky to determine what is supported from the NI
website, but the following should help. For triggering of analog output,
you need a device that supports Analog Waveform Generation, and you need
to be able to set the AO Sample Clock source to one of the PFI pins on
the device. For triggering of digital output, you need a device that
supports Digital Waveform Generation, and you need to be able to set the
DO Sample Clock source to one of the PFI pins on the device. These
capabilities are normally described in the User Manual for the device,
which is usually available on the NI website. If you don't see any
description of waveform generation, the device \_probably\_ doesn't
support hardware triggering.

To enable hardware triggering, the card must support triggering, and you
must tell it which input pin to "listen" on for the control signal, by
setting the "TriggerInputLine" property. The device adapter will give
you a list of valid input pins to listen on for this property. You
should find a "PFI" pin that is *not* on the port that you want to use
for output; certain of the pins on NI cards may do double-duty as both
input and output pins, but they can only be used as one at a time. For
example, if you want to use Dev1/port1 as your output port, and the
Dev1/PFI0 input line is also the Dev1/port1/line0 output line, then you
cannot use Dev1/PFI0 as the trigger input, because it is on a port that
you are already using for output. Refer to the pin diagram in NI-MAX to
see which PFI inputs are valid for your setup.

Additionally, a single trigger input line may not be used for multiple
devices simultaneously. So for example, if you wish to use a DigitalIO
device to control lights, and an AnalogIO device to control your stage,
and you want both to be triggered at the same time from the same signal,
you will have to split that signal to go to two different "PFI" pins,
one for each device.

Once you have set a valid input line, and assuming the card is capable
of doing hardware triggering, the "SupportsTriggering" property should
be set to 1. This indicates that the card is capable of performing
hardware triggering. See
[Hardware-based\_Synchronization\_in\_Micro-Manager](Hardware-based_Synchronization_in_Micro-Manager "wikilink")
for more information on setting up triggering with the rest of your
system.

There is a "TriggerSequenceLength" property that you can adjust in the
Hardware Configuration Wizard when initially setting up the device. This
is the maximum number of entries usable when using the card for hardware
triggering (note that the sequence will repeat as needed, with only 2
actions needed to e.g. turn a light source on and then off again
regardless of how many times this is done). We expect that the default
value of 1000 will be more than sufficient for all users, but it is
configurable just in case.
