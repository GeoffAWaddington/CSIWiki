# December 13th, 2022

## Changes to Feedback behavior and Widget Mode
CSI now ignores feedback for Reaper Actions that do not properly report their status in the Action List (these are actions that do not show "on" or "off" in the Reaper action list). The Feedback [[Widget Mode|Widget Modes]] behavior was now changed as a result, so that you no longer have to enter "Feedback=No" to disable feedback in those instances.

CSI will default to only providing feedback for the first action in a CSI macro action (i.e. one button assigned to trigger more than action). If you want to report the feedback state of anything other than the first action in a macro list, you simply add the word "Feedback" to the end of the action you want to provide the feedback. There can only be one of these for each CSI macro action or the last in the list will be used.

Example 1. I only want feedback on the first action (default behaviour - no additional syntax required)....
```
    SomeButton SomeAction
    SomeButton AnotherAction
    SomeButton YetAnotherAction
```

Example 2. I want feedback on the middle action only; so I use the new Feedback widget mode to specify which action gets it....
```
    SomeButton SomeAction
    SomeButton AnotherAction Feedback 
    SomeButton YetAnotherAction
```

Example 3. I want feedback on the last action only....
```
    SomeButton SomeAction
    SomeButton AnotherAction
    SomeButton YetAnotherAction Feedback
```

Example 4. Erroneous definition. In this case only the 3rd Action gets feedback.
```
    SomeButton SomeAction
    SomeButton AnotherAction Feedback
    SomeButton YetAnotherAction Feedback
```

## Fixes for Extraneous Messages Being Sent / Flickering
The above referenced change to the feedback behavior should also resolve any problems with extraneous messages being sent or button flickering.

## New CSI Support Files Posted
The CSI support files were updated to reflect some of the latest changes.

# December 10th, 2022

## Crash Fix When Using GoFolder in Project With "Orphaned Folders"
If you had a Reaper project where tracks that have been left as "last track in folder" when the original folder no longer exists (i.e. "orphaned folder tracks"), Reaper would crash when using CSI's GoFolder action. This crash condition has been resolved.

## BREAKING CHANGES: EZFXZones Removed
The code for EZFXZones has been removed in order to keep just a single way of creating FX.zon's. While the EZFXZones used less syntax in some instances, the benefit of the alternate syntax doesn't seem great enough to warrant maintaining.

## "Only the First Action in a Series Provides Feedback to the Widget" Constraint Has Been Removed
Prior builds have CSI have had a workaround in the code to prevent runaway feedback when multiple actions were assigned to the same widget. The workaround was to impose a limitation that only the first action would provide feedback to the widget. Example:

```
    SomeButton SomeAction     // only this action would provide feedback to the widget
    SomeButton AnotherAction  // this action would not provide feedback
```

While this worked reasonably well, you were out of luck if you wanted SomeAction to occur before AnotherAction, but wanted the feedback from AnotherAction.

Now, with feedback available on a per Action basis you can do this:

```
    SomeButton SomeAction Feedback=No
    SomeButton AnotherAction
```

...or even something like this:
```
    SomeButton SomeAction Feedback=No
    SomeButton AnotherAction 
    SomeButton YetAnotherAction Feedback=No
```
As a result, that "only the first action provides feedback" constraint is no longer necessary.

## "CSI Toggle Show Input from Surfaces" Action Now Shows Zone File Path
To help facilitate troubleshooting of .zon files, particularly when you may have multiple zone folders or aren't sure which zone is currently active, the "CSI Toggle Show Input from Surfaces" action in Reaper will now show the full zone file path in addition to the action. 

Example:
```
IN <- XTouchOne Play 1.000000
Zone -- C:\Users\[UserName]\AppData\Roaming\REAPER/CSI/Zones/X-Touch_One_FB/Buttons.zon
```
...looking at this, I can see the full path to the zone file that was active when I pressed the Play button. 

## BREAKING CHANGES: WidgetMode and Property+NoFeedback Replaced With New Syntax
Previously, WidgetMode was used to set the encoder ring style and FaderPort display types. In addition, Property+NoFeedback was used for disabling feedback. This old implementation required multiple lines for these additional commands. 

New functionality has been implemented to replace WidgetMode and Property+NoFeedback, keeping everything on a single line. Because of this new syntax, existing .zon files will need to be updated to function properly.

For example, this [old syntax] has been replaced...
```
    Rotary|          	  	TrackPanAutoLeft
    Rotary|			WidgetMode Dot
```

...with just this [new syntax]:
```
Rotary|  TrackPanAutoLeft RingStyle=Dot
```

This [old syntax]...
```
    Up              Reaper _XENAKIOS_TVPAGEUP                  // Xenakios/SWS: Scroll track view up (page)
    Property+Up     NoFeedback
```

Has been replaced with this [new syntax]:
```
    Up              Reaper _XENAKIOS_TVPAGEUP Feedback=No     // Xenakios/SWS: Scroll track view up (page)

```

The new functionality is similarly dependent on the type of feedback processor used in your .mst for a given widget.

If using **FB_Encoder**, then the available options are:
```
RingStyle=Dot
RingStyle=BoostCut
RingStyle=Fill
RingStyle=Spread
```

If using **FB_FaderportValueBar**, then the available options are"
```
BarStyle=Normal
BarStyle=BiPolar
BarStyle=Fill
BarStyle=Spread
```

If using **FB_FP8ScribbleStripMode**, then the available options are:
```
Mode=0
Mode=1
Mode=2
....
Mode=8
```

If using **FB_FP8ScribbleLine1**, etc., then the available options are:
```
TextAlign=Center
TextAlign=Left
TextAlign=Right

TextInvert=Yes
TextInvert=No
```
# November 1st, 2022 Build
## Added special case logic for X32 Select button
Code changes deployed to get the Select buttons working correctly on the X32/MIDAS series consoles. Thanks to forum user jacksoonbrowne for the code.

## EZFXZones now support Fixed Text for -1 param number [EZFXZones Have Been Deprecated]
Using the new EZFXZones, you can now have fixed text appear in a display with no parameter assigned to it (FX Param -1). See the example below...

```
Zone "VST: UAD Fairchild 660 (Universal Audio, Inc.)" "Fairchild 660"
     FXParams                 1            2         3               7             4           6           9        8
     FXParamNames             "Input Gain" Threshold "Time Constant" "Output Gain" "SC Filter" "DC Thresh" Headroom Mix
     FXValueWidgets           RotaryA|
     FXParamNameDisplays      DisplayUpperA|
     FXParamValueDisplays     DisplayLowerA|

     FXParams                 5     0       -1      -1     -1         -1      -1     12 
     FXParamNames             Bal   Meter   "Fixed" "Text" "Example"  "Here"  ""     Wet
     FXValueWidgets           RotaryB|
     FXParamNameDisplays      DisplayUpperB|
     FXParamValueDisplays     DisplayLowerB|
ZoneEnd
```

## Removed string feedback for GoVCA and GoFolder, FXBypassDisplay and FXOfflineDisplay provide string feedback
Changes to feedback on OSC to prevent issues with two messages of different types being sent simultaneousy.

## Fixed bug in negative measures display
Small change to further improve negative measure display on TimeDisplay widgets in Measure mode.

## New Navigation Action: GoMasterTrack
GoMasterTrack was added to activate the MasterTrack zone for fader surfaces that do not have a dedicated master track fader. Be sure to include this as an AssociatedZone in your Home.zon.

```
Zone "Buttons"
    SomeButton    GoMasterTrack
ZoneEnd
```

## Fix for TrackBank Not Updating Channel Count to Account for Hidden Tracks
After hiding a track in Reaper, CSI was not updating it's internal track count to adjust for hidden tracks. This has been resolved.

## Fix for Zoom Modifier
The Zoom modifier was broken in a recent build. This has been fixed.

## Fix for Touch Modifier Not Working Outside of Track Zones
The Touch modifier recently stopped working outside of Track Zones. So for example, Touch would do nothing when used as a modifier in an FX.zon. This has ben resolved.

## Fix for Negative Measure Displays
Fix for the MCUTimeDisplay action incorrectly displaying negative measures when in Bars/Beats/Ticks mode. Thanks to Reaper forum user StereoDidi for the code contribution for this fix!

## Added Track Number to SpeakTrackSendDestination and SpeakTrackReceiveSource Actions
For visually impaired users, SpeakTrackSendDestination and SpeakTrackReceiveSource actions will now also speak the Track #. 

## Possible Fix for CSI Speaking Twice with Some Screen Readers
There is a possible fix for an issue with CSI speaking twice when using certain screen readers. 

## New Actions: SpeakFXMenuName, SpeakTrackSendDestination, SpeakTrackReceiveSource
The following new actions were added for OSARA users in order to allow CSI to speak the FX Menu Name of a plugin (SpeakFXMenuName) in the menu, or the track send destination (SpeakTrackSendDestination), or the receive source (SpeakTrackReceiveSource). These can be assigned to widgets to trigger when these actions take place.

```
Zone "SelectedTrackFXMenu"
    RecArm     SpeakFXMenuName
...
```

```
Zone "SelectedTrackSend"
    RecArm     SpeakTrackSendDestination
...
```

```
Zone "SelectedTrackReceive"
    RecArm     SpeakTrackReceiveSource
...
```

## Hex Colors for OSC
Users can now transmit colors to OSC surfaces using the standard Hex color format. These messages are transmitted to **OSCAddress + "/Color"** and formatted as #rrggbbaa. You will need to have your OSC widget setup to receive and respond to colors at that address. Note: as of the time of this writing, TouchOSC support for Hex colors with the hash characters is currently in beta (build 149). 

## Turn Off MIDI Fighter Twister Button LEDs by Sending 0 0 0 Color Message
Thanks to Reaper forum user Diesel, users of the MIDI Fighter Twister can turn off LED button lights on the device by sending RGB messages of 0 0 0 to the device. Thanks Diesel!

```
RotaryPushB5 FXParam 999 { 0 0 0 }
```

## New Action: SendOSCMessage
SendOSCMessage is designed to send arbitrary OSC messages to the address specified in the action. The syntax is **[Widget/Virtual Widget] SendOSCMessage "[OSC address] [Value]"** as shown in the examples below...

```
    OnInitialization SendOSCMessage "/Displays/UpperDisplay1 aString".   // String 
    OnInitialization SendOSCMessage "/Displays/LowerDisplay1 -123"       // 32-bit integer
    OnInitialization SendOSCMessage "/Displays/ValueDisplay1 24.98".     // Float
```

## Default Step Size and Default Encoder Acceleration Can Be Used in All Zones
If you're using a MIDI surface with encoders, you can now define the default StepSize (resolution) and Acceleration in your .mst file and these settings will be carried into all Zone types. This allows you to define these values once, and avoid having to repeatedly define them in zone files.

A refresher on what this might look like in an .mst...
```
StepSize
    RotaryWidgetClass 0.003
StepSizeEnd

AccelerationValues
    RotaryWidgetClass Dec 41     42    43    44    45    46    47     48
    RotaryWidgetClass Inc 01     02    03    04    05    06    07     08
    RotaryWidgetClass Val 0.005  0.01  0.02  0.04  0.05  0.06  0.08   0.1
AccelerationValuesEnd

/...

Widget Rotary1 RotaryWidgetClass
	Encoder b0 10 7f
	FB_Encoder b0 10 7f
WidgetEnd

Widget Rotary2 RotaryWidgetClass
	Encoder b0 11 7f
	FB_Encoder b0 11 7f
WidgetEnd

Widget Rotary3 RotaryWidgetClass
	Encoder b0 12 7f
	FB_Encoder b0 12 7f
WidgetEnd

Widget Rotary4 RotaryWidgetClass
	Encoder b0 13 7f
	FB_Encoder b0 13 7f
WidgetEnd

Widget Rotary5 RotaryWidgetClass
	Encoder b0 14 7f
	FB_Encoder b0 14 7f
WidgetEnd

Widget Rotary6 RotaryWidgetClass
	Encoder b0 15 7f
	FB_Encoder b0 15 7f
WidgetEnd

Widget Rotary7 RotaryWidgetClass
	Encoder b0 16 7f
	FB_Encoder b0 16 7f
WidgetEnd

Widget Rotary8 RotaryWidgetClass
	Encoder b0 17 7f
	FB_Encoder b0 17 7f
WidgetEnd
```

## New Feature: Auto-Tick-Count Generation for Encoders+Stepped Parameters
If you're utilizing the new [ZoneStepSizes](https://github.com/GeoffAWaddington/CSIWiki/wiki/CSI-Changelog#zonestepsizes-and-stp-files) feature, CSI will automatically create a tick-count list for comfortably cycling through stepped parameters so they're not going by too quickly to be useful or too slowly to be enjoyable. First, you need to define your encoders using the RotaryWidgetClass, StepSize, and DefaultAcceleration features in the .mst. Once you have that, you just need a .stp file (ZoneStepFile) for any fx you map. CSI does the rest!

## Fixes for FXZone Crash and DisplayMode Spread [EZFXZones Have Been Deprecated]
Bug fixes for an EZFXZone crash when parsing an incorrect syntax and a fix for the DisplayMode "Spread" setting not working as expected.

## New Feature: Local Modifiers
Previously, modifiers such as Shift, Alt, Control, and Option were strictly "Global Modifiers": engaging the modifier on one surface, would enable that modifier on all devices. Now, a new CSI.ini preference has been added that will allow surfaces to set their modifiers locally (i.e. modifiers will not impact any other surface than the one where they were engaged). To make the change, add the word **LocalModifiers** prior to the surface name under each Page where you'd like to enable local modifiers.
```
Version 2.0

MidiSurface "XTouchOne" 7 9 
MidiSurface "MFTwister" 6 8 
OSCSurface "iPad Pro" 8003 9003 10.0.0.146
OSCSurface "TouchOSCLocal" 8002 9002 10.0.0.100
MidiSurface "CMC-QC" 23 24 

Page "HomePage"
LocalModifiers "XTouchOne" 1 0 "X-Touch_One.mst" "X-Touch_One_FB" 
"MFTwister" 8 0 "MIDIFighterTwisterEncoder.mst" "FXTwisterFocusedFX" 
"iPad Pro" 8 0 "MCU-Pad.ost" "MCU" 
"TouchOSCLocal" 8 0 "FXTwister.ost" "FXTwisterFocusedFX" 
"CMC-QC" 0 0 "Stienberg_CMC-QC-2.mst" "Steinberg_CMC-QC-2" 
```

# September 23, 2022

## Support for the X32/M32/XAir Series of Behringer/MIDAS Consoles via OSC
Thanks to absolutely huge contributions from Reaper forum user jacksoonbrowne, the Behringer X32 is now supported in CSI via OSC. These devices have a very unique OSC implementation that required substantial code contributions.

The X32 implementation is an on-going work in progress and many more features will be added to the support files.

For these to work:
1. Add a new OSC device as per the Installation and Setup instructions.
2. In the CSI Device setup, the "CSI sends to port" and "CSI recieves on port" must **both** be set to port 10023 for the X32/M32 or port 10024 for the XAir series.
3. If you're creating your own .ost, the .ost must contain either "X32" or "x32" somewhere in filename. Note: a set of files for the Behringer X32 has been added to the CSI Support Files.

Examples:
```
BehringerX32.ost
X32Surface.ost
SurfaceX32.ost
Surface_x32Intance1.ost
SurfaceX32Intance2.ost
```
An additional special thanks goes out to Patrick-Gilles Maillot for having documented the X32 OSC protocol [here.](https://drive.google.com/file/d/1Snbwx3m6us6L1qeP1_pD6s8hbJpIpD0a/view)

```
//-----------------------------------------------------------------------------------------
// Reaper CSI X32/Xair/Midas32 support files installation notes.
//-----------------------------------------------------------------------------------------

Go to the below URL to download and retrieve the most recent versions of support files and place them in the CSI file structure appropriately:
https://www.dropbox.com/scl/fo/upw6bjujgm8wp1gjz3bel/h?dl=0&rlkey=n3japvturdtzbeqjyob5e69d2

Having done that, load the X32 scene file from "CSI\Behringer X32 Scene\CSI-X32-Config.scn", and place it on a USB thumb drive that is 32Gbytes or less, and formatted as "FAT32".
Follow the instructions from Behringer web site for the X32 on how to download a scene from a USB stick and install the scene.
And then load that scene on the X32.
This scene will configure the "Assignable sections: rotaries and buttons" to control the "Transport" as per the current X32 .ost and .zon files.
```

## Fix for Reaper Tracks Not Scrolling Into View When Selected on Surface
If the number of channels in Reaper was fewer than the number of tracks on your surface, selecting a track on the surface would not scroll that track into view in Reaper. This should fix that problem.

# September 19, 2022

## MIDI Surface Template (.mst) File Updates
Several enhancements in this build pertain to work happening around EZFXZones and helping build up to auto-mapping of FX. As a result, there are some changes that can be made to MIDI Surface Template (mst) files to streamline how encoders are used in EZFXZones, creating some default behaviors, and eventually help allow for auto-mapping FX. **Note:** these tweaks are optional and fully backwards compatible with prior .mst files so no changes are required.

### RotaryWidgetClass and JogWheelWidgetClass
RotaryWidgetClass is designed to help streamline how encoders are defined in .mst files and tell CSI which widgets are encoders. There are multiple elements to how this is used in an .mst file. 

First, by putting the word RotaryWidgetClass after the widget name in the .mst file, you are telling CSI, "this widget belongs in the rotary widget class" (as shown below). In a moment, we'll show you what that allows for.
```
Widget RotaryA1 RotaryWidgetClass
    Encoder b0 00 7f
    FB_Fader7Bit b0 00 00
WidgetEnd
```

The same can be done with your JogWheel using the new JogWheelWidgetClass.
```
Widget JogWheel JogWheelWidgetClass
	Encoder b0 3c 7f
WidgetEnd
```

### Defining "StepSize" for All Encoders in the RotaryWidgetClass
Now that the RotaryWidgetClass and/or JogWheelWidgetClass is defined for our encdoers, we can set the encoder StepSize globally by adding this to the top of the .mst file. This represents how fine the resolution will be for each encoder "tick". A value of 0.001 will be very fine and move parameters one-tenth of one-percent, which is very fine. If you find that resolution a little too fine, resulting in slow encoders, you may have better luck with a value of 0.003 or some other higher value. It will really depend on your hardware surfaces and preferences.

Here is an example from the X-Touch.mst showing both class types, each using a StepSize of 0.003.
```
StepSize
    RotaryWidgetClass   0.003
    JogWheelWidgetClass 0.003
StepSizeEnd
```

The encoders on MIDI Fighter Twister are very sensitive, so I might use a StepSize of 0.001 for that surface.
```
StepSize
    RotaryWidgetClass 0.001
StepSizeEnd
```

### AccelerationValues
Next, we can then remove the Encoder Acceleration step values from each individual widget and just create a global set of acceleration values at the top of the .mst file. The benefit of this approach is that rather than being required to define the Encoder Acceleration in each EZFXZone, CSI can now use a default for all encoders in the RotaryWidgetClass. 

Here we are defining the Decrease values ("Dec") from slowest encoder turns to fastest, and the same for the Increase ("Inc") values. Below that, you will find one encoder acceleration value ("Val") for each encoder acceleration step. The actual values used will depend on what your encoder transmits. The values below are from a MIDI Fighter Twister and my own personal acceleration curve.
```
AccelerationValues
    RotaryWidgetClass Dec 3f     3e    3d    3c    3b    3a    39     38    36    33    2f     
    RotaryWidgetClass Inc 41     42    43    44    45    46    47     48    4a    4d    51
    RotaryWidgetClass Val 0.001  0.002 0.003 0.004 0.005 0.006 0.0075 0.01  0.02  0.03  0.04
AccelerationValuesEnd
```
So in the past, each of my MFTEncoder widgets looked like the this...
```
Widget RotaryA1
	MFTEncoder b0 00 7f [ < 3f 3e 3d 3c 3b 3a 39 38 36 33 2f > 41 42 43 44 45 46 47 48 4a 4d 51 ]
	FB_Fader7Bit b0 00 00
WidgetEnd
```

Now we've got this text at the top of the .mst
```
StepSize
    RotaryWidgetClass 0.001
StepSizeEnd

AccelerationValues
    RotaryWidgetClass Dec 3f     3e    3d    3c    3b    3a    39     38    36    33    2f      
    RotaryWidgetClass Inc 41     42    43    44    45    46    47     48    4a    4d    51
    RotaryWidgetClass Val 0.001  0.002 0.003 0.004 0.005 0.006 0.0075 0.01  0.02  0.03  0.04
AccelerationValuesEnd
```

And the encoder widgets look like this (Note: MFTEncoder has been replaced with the standard Encoder widget since all the instructions are now up top).
```
Widget RotaryA1 RotaryWidgetClass
    Encoder b0 00 7f
    FB_Fader7Bit b0 00 00
WidgetEnd
```

Here is an example from the X-Touch.mst where the step size is 0.003 and the AccelerationValues line up with MCU-style devices.
```
StepSize
    RotaryWidgetClass   0.003
    JogWheelWidgetClass 0.003
StepSizeEnd

AccelerationValues
    RotaryWidgetClass   Dec 41     42    43    44    45    46    47  
    RotaryWidgetClass   Inc 01     02    03    04    05    06    07  
    RotaryWidgetClass   Val 0.0006 0.001 0.002 0.003 0.008 0.04  0.08 

    JogWheelWidgetClass Dec 41     42    43    44    45    46    47  
    JogWheelWidgetClass Inc 01     02    03    04    05    06    07  
    JogWheelWidgetClass Val 0.0006 0.001 0.002 0.003 0.008 0.04  0.08 
AccelerationValuesEnd
```

### EWidget (or "Eligible Widgets")
Another feature instituted now as part of the roadmap to auto-mapping FX is the addition of the "Ewidget" option in .mst files. This will eventually be used to tell CSI which widgets you'd like automatically included for automatic FX.zon mapping and which widgets you'd like excluded from that. Anything defined as an EWidget will be eligible for mapping. Here are some examples from an X-Touch.mst where we are using Displays, Rotarypush, Rotary, and Faders for FX mapping.

```
EWidget DisplayUpper1
	FB_XTouchDisplayUpper 0
EWidgetEnd
```

```
EWidget Fader1
	Fader14Bit e0 7f 7f
	FB_Fader14Bit e0 7f 7f
	Touch 90 68 7f 90 68 00
EWidgetEnd
```

```
EWidget RotaryPush1
	Press 90 20 7f 90 20 00
EWidgetEnd
```

```
EWidget Rotary1 RotaryWidgetClass
	Encoder b0 10 7f
	FB_Encoder b0 10 7f
EWidgetEnd
```

## ZoneStepSizes and .stp Files
The CSI Support Files now include a "ZoneStepSizes" sub-folder within the Zones folder. These files will be used by CSI in EZFXZones to determine which FX Parameters are stepped, how many steps each parameter has, and what the exact step values are. Again, this is another feature meant to simplify EZFXZone creation. Once a ZoneStepFile exists for a plugin, it shouldn't need to change (unless the developer adds new automation parameters) and can be shared. The CSI Support Files currently include ZoneFXFiles for almost 500 FX to get users started.

If you'd like to create some .stp files for your own use, you can add the below "AutoScan" line to your CSI.ini. The AutoScan process only attempts to create the .stp files for fx you already have a .zon for. It will not create .stp files for all FX. Note: this is an experimental feature and works better on Mac right now. If you run into issues, I'd encourage you to turn the AutoScan off by commenting out that line. When the AutoScan is complete and ZoneStepSize files created, you should also comment out (with a forward slash) or delete that line in your CSI.
```
Version 2.0

AutoScan

MidiSurface "XTouchOne" 7 9 
MidiSurface "MFTwister" 6 8 
OSCSurface "iPad Pro" 8003 9003 10.0.0.146
OSCSurface "TouchOSCLocal" 8002 9002 10.0.0.100
MidiSurface "CMC-QC" 23 24 

Page "HomePage"
"XTouchOne" 1 0 "X-Touch_One.mst" "X-Touch_One_FB" 
"MFTwister" 8 0 "MIDIFighterTwisterEncoder.mst" "FXTwisterMenu" 
"iPad Pro" 8 0 "FXTwister.ost" "FXTwisterMenu" 
"TouchOSCLocal" 8 0 "FXTwister.ost" "FXTwisterMenu" 
"CMC-QC" 0 0 "Stienberg_CMC-QC-2.mst" "Steinberg_CMC-QC-2" 
```

When a .stp file exists for a plugin, things like the below example are no longer required in EZFXZones. CSI will already know the step values and will automatically create a curve for your surface to move through each step if that parameter is assigned to an encoder. 
```
     FXParamStepValues   1    0.0 0.05 0.11 0.16 0.21 0.26 0.32 0.37 0.42 0.47 0.53 0.58 0.63 0.68 0.74 0.79 0.84 0.89 0.95 1.0
```

## EZFXZone Changes [EZFXZones Have Been Deprecated]
If you've got Step Files for your FX, and made the changes outlined above, EZFXZones get much easier. The block of text at the bottom dictating the default acceleration and FX Step sizes are no longer required.

Before...
```
Zone "VST: UAD Fairchild 660 (Universal Audio, Inc.)" "Fair660"
     FXParams                 1            2         3               7             4           6           9        8
     FXParamNames             "Input Gain" Threshold "Time Constant" "Output Gain" "SC Filter" "DC Thresh" Headroom Mix
     FXValueWidgets           RotaryA|
     FXParamNameDisplays      DisplayUpperA|
     FXParamValueDisplays     DisplayLowerA|

     FXParams                 5     0       -1     -1     -1      -1     -1     12 
     FXParamNames             Bal   Meter   ""     ""     ""      ""     ""     Wet
     FXValueWidgets           RotaryB|
     FXParamNameDisplays      DisplayUpperB|
     FXParamValueDisplays     DisplayLowerB|

     FXParams                 -1   -1   -1   -1   -1   -1   -1     -1
     FXParamNames             ""   ""   ""   ""   ""   ""   ""     "" 
     FXValueWidgets           RotaryPushA|
     FXParamNameDisplays      DisplayRotaryPushA|

     FXParams                 -1   -1   -1   -1   -1   -1   13     11
     FXParamNames             ""   ""   ""   ""   ""   ""   Delta  Bypass
     FXValueWidgets           RotaryPushB|
     FXParamNameDisplays      DisplayRotaryPushB|

     DefaultAcceleration      0.001 0.002 0.003 0.004 0.005 0.006 0.0075 0.01 0.02 0.035 0.04     
     FXParamStepValues   0    0.0 0.50 1.0
     FXParamStepValues   1    0.0 0.05 0.11 0.16 0.21 0.26 0.32 0.37 0.42 0.47 0.53 0.58 0.63 0.68 0.74 0.79 0.84 0.89 0.95 1.0
     FXParamStepValues   3    0.0 0.20 0.40 0.60 0.80 1.0 
     FXParamStepValues   9    0.0 0.17 0.33 0.50 0.67 0.83 1.0
ZoneEnd
```

After...
```
Zone "VST: UAD Fairchild 660 (Universal Audio, Inc.)" "Fair660"
     FXParams                 1            2         3               7             4           6           9        8
     FXParamNames             "Input Gain" Threshold "Time Constant" "Output Gain" "SC Filter" "DC Thresh" Headroom Mix
     FXValueWidgets           RotaryA|
     FXParamNameDisplays      DisplayUpperA|
     FXParamValueDisplays     DisplayLowerA|

     FXParams                 5     0       -1     -1     -1      -1     -1     12 
     FXParamNames             Bal   Meter   ""     ""     ""      ""     ""     Wet
     FXValueWidgets           RotaryB|
     FXParamNameDisplays      DisplayUpperB|
     FXParamValueDisplays     DisplayLowerB|

     FXParams                 -1   -1   -1   -1   -1   -1   -1     -1
     FXParamNames             ""   ""   ""   ""   ""   ""   ""     "" 
     FXValueWidgets           RotaryPushA|
     FXParamNameDisplays      DisplayRotaryPushA|

     FXParams                 -1   -1   -1   -1   -1   -1   13     11
     FXParamNames             ""   ""   ""   ""   ""   ""   Delta  Bypass
     FXValueWidgets           RotaryPushB|
     FXParamNameDisplays      DisplayRotaryPushB|
ZoneEnd
```

## New Modifiers: Marker, Nudge, Scrub, and Zoom
CSI has added new "radio-button style" modifiers designed to allow for expanded functionality with MCU-style surfaces. By radio-button style, that means you cannot combine these modifiers like you can Global Modifiers (e.g. you can't combine Zoom+Scrub, but you can combine Zoom+Shift). 

In the below example, Marker, Nudge and Zoom modifiers are being used with the arrow buttons to expand how they function in a logical way, without the need for separate sub-zones for these tasks. If you prefer to use SubZones in lieu of these modifiers, you may still continue to do so - the new modifiers are entirely optional. The new modifiers are also not limited to the arrow keys and could of course be used elsewhere.

```
Zone "Buttons"
    
    Marker                      Marker
    Nudge                       Nudge
    Zoom                        Zoom
    
    Up                          Reaper _XENAKIOS_TVPAGEUP              // Xenakios/SWS: Scroll track view up (page)
    Down                        Reaper _XENAKIOS_TVPAGEDOWN            // Xenakios-SWS: Scroll track view down 
    Left                        Reaper _SWS_SCROLL_L10                 // SWS: Scroll left 10%
    Right                       Reaper _SWS_SCROLL_R10                 // SWS: Scroll right 10%                                                                 
    
    Zoom+Up                     Reaper 40111                           // Zoom in vertical                                            
    Zoom+Down                   Reaper 40112                           // Zoom out vertical                                                       
    Zoom+Right                  Reaper 1012                            // Zoom in horizontal                                      
    Zoom+Left                   Reaper 1011                            // Zoom out horizontal                                     

    Marker+Up                   Reaper 40613                           // Delete marker near cursor                         
    Marker+Down                 Reaper 40157                           // Insert marker at current or edit position                           
    Marker+Right                Reaper 40173                           // Go to next marker or project end                           
    Marker+Left                 Reaper 40172                           // Go to previous marker or project start

    Nudge+Up                    Reaper 41925                           // Item: Nudge items volume +1dB
    Nudge+Down                  Reaper 41924                           // Item: Nudge items volume -1dB
    Nudge+Left                  Reaper 41279                           // Item edit: Nudge left by saved nudge dialog settings 1
    Nudge+Right                 Reaper 41275                           // Item edit: Nudge right by saved nudge dialog settings 1
ZoneEnd                                                               
```

## New "GlobalModeDisplay" Action and "Global" Modifier
GlobalModeDisplay will display the current state of the new Global modifier. When the modifier is off, it will show "SE" for "Selected Track" and when the modifier is enabled, it will show "GL" for "Global". It is designed with MCU-style surfaces in mind so that when you see the Plugin button lit, indicating you are in an FXMenu zone, the SE/GL text will let you know if you're using the SelectedTrack or Track (aka "Global") variant. 

In the example below we see the GlobalModeDisplay on the MCU AssignmentDisplay widget, and we also see how the new Global modifier can be used to load the "Track/Global" variants of the various zone types.
```
Zone "Buttons"

//  Encoder Assign Buttons

    Track                       GoHome
    Pan                         GoVCA
    EQ                          GoFolder
    Send                        GoSelectedTrackSend
    Global+Send                 GoTrackSend
    Plugin                      GoSelectedTrackFXMenu
    Global+Plugin               GoTrackFXMenu
    Instrument                  GoSelectedTrackReceive
    Global+Instrument           GoTrackReceive

    GlobalView                  Global
    AssignmentDisplay           GlobalModeDisplay 
```
Note: the new Global modifier is a "Global Modifier" like Shift, Control, and Alt, and not a "radio-button style" modifier like Marker and Zoom.

Below, we also see the Global modifier being used on the automation buttons to indicate whether you're changing the SelectedTrack (SE) variant of those actions or the Global Automation Modes.
```
    Read                        TrackAutoMode 1
    Write                       TrackAutoMode 3
    Trim                        TrackAutoMode 0
    Touch                       TrackAutoMode 2
    Latch                       TrackAutoMode 4
    Group                       TrackAutoMode 5
    Global+Read                 GlobalAutoMode 1
    Global+Write                GlobalAutoMode 3
    Global+Trim                 GlobalAutoMode 0
    Global+Touch                GlobalAutoMode 2
    Global+Latch                GlobalAutoMode 4  
    Global+Group                GlobalAutoMode 5   
```

## New Action: ClearAllModifiers
This action was designed to allow a way to easily clear all CSI global modifiers (e.g. Shift, Alt, Option, Control) with a single button press or automatically based on a certain trigger using [[Virtual Widgets]]. This action will not clear the Toggle modifier (or Touch - which makes sense if you're currently touching something). One example use-case for this new action: a user may want to clear all modifiers whenever the Home Zone is activated as shown below...
```
Zone "Home"
    OnZoneActivation     ClearAllModifiers
     IncludedZones
          "Buttons"
          "SelectedTrack"
          "MasterTrack"
     IncludedZonesEnd
     AssociatedZones
          "SelectedTrackSend"
          "SelectedTrackReceive"   
     AssociatedZonesEnd
ZoneEnd
Zone
```

# September 12, 2022

## Two-Way Encoder Behavior (Requires Updates to Your JogWheel in the .mst File)
Now you can assign two different Reaper actions to a single encoder based on which way the encoder is being turned, Counter-Clockwise (CCW) or Clockwise (CW). We do this via the Decrease and Increase modifiers. Note: these modifiers only work with Encoders. The examples below all show the JogWheel but this functionality will work with any encoder.

```
Zone "Zoom"
    Decrease+JogWheel      Reaper 41326   // Decrease track height
    Increase+JogWheel      Reaper 41325   // Increase track height
ZoneEnd
```

**Important Note:** for this to work on your JogWheel, you will need to update your .mst file. If you're coming from a prior version of CSI, you probably have two or more separate JogWheel widgets, and those widgets probably are defined using Press instead of Encoder. So replace your JogWheel widgets to look like this (assuming MCU-style surface):
```
Widget JogWheel 
	Encoder b0 3c 7f
WidgetEnd
```

The following native CSI actions support this same syntax:
```
TrackBank
VCABank
FolderBank
SelectedTrackSendBank
SelectedTrackReceiveBank
SelectedTrackFXMenuBank
TrackSendBank
TrackReceiveBank
TrackFXMenuBank
```

Example:
```
Zone "Buttons"
    Decrease+JogWheel      TrackBank -1
    Increase+JogWheel      TrackBank  1
ZoneEnd
```

## EZFXZones: New FX Zone Syntax Option [EZFXZones Have Been Deprecated]
**Note: consider this feature extra-experimental. This is not ready for production projects yet. Also, some actions described below are currently still being coded.**

EZFXZones are a new way of writing FX Zones that uses far fewer lines, and saves a lot of the tedious repetition of the legacy fx.zon format (which is not going away). These new FX zones follow a spreadhseet-like format where you read both across and down.

Diving right into a real-world example, the first and last lines of any fx.zon remain unchanged. After that, the first block of text starting with "FXParams" is saying FXParam 9, which, reading down, we are giving the alias "HeadRoom", gets assigned to Rotary1. You will see the FXParamName on DisplayUpper1, and you will see the FXParamValue on DisplayLower1. Reading across on line 2 again, FXParam 1, "Input", will get assigned to Rotary2 (we're reading down again), with the FXParamName name showing up on DisplayUpper2 and FXParamValueName on DisplayLower2. And on and on.
```
Zone "VST: UAD Fairchild 660 (Universal Audio, Inc.)" "Fair660"
     FXParams                 9      1     2      3     6    7      0     8
     FXParamNames             HdRoom Input Thresh Ratio Knee Output Meter WetDry
     FXValueWidgets           Rotary|
     FXParamNameDisplays      DisplayUpper|
     FXParamValueDisplays     DisplayLower|
ZoneEnd
```
**Note:** using the legacy FX.zon syntax, this simple mapping would've required something like 40 lines of code. The new EZFXZone syntax accomplishes all the same functionality in only 5 lines! If you wanted, you could stop right there; that could be an entire fx.zon!

Now, if we want to add a modifier, we add the next block of text shown below. Very similar layout, except there is one more row, with the addition of the **FXWidgetModifiers Shift** line. This says, show FXParam 5, "DC Bal", on Shift+Rotary1 and the corresponding Shift-Displays. Also note that "DC Bal" is in quotes - these are only required because there is a space in what we want to appear on the display. Next, if you continue reading across, you see a series of -1 entries in the FXParams row and double quotes in the corresponding FXParamNames rows. This tells CSI, "No Action here and clear out the displays".
```
Zone "VST: UAD Fairchild 660 (Universal Audio, Inc.)" "Fair660"
     FXParams                 9      1     2      3     6    7      0     8
     FXParamNames             HdRoom Input Thresh Ratio Knee Output Meter WetDry
     FXValueWidgets           Rotary|
     FXParamNameDisplays      DisplayUpper|
     FXParamValueDisplays     DisplayLower|

     FXParams		      5	 -1 -1 -1 -1 -1 -1 4
     FXParamNames	      "DC Bal" "" "" "" "" "" "" SidChn
     FXValueWidgets  	      Rotary|
     FXParamNameDisplays      DisplayUpper|
     FXParamValueDisplays     DisplayLower|
     FXWidgetModifiers	      Shift
ZoneEnd
```

Now, let's say I want RotaryPush8 to toggle the FX Bypass state, but it's the only toggle-style action I need and I don't necessarily need to see that on a display. You can simply add another block with that specific instruction on the one widget:
```
Zone "VST: UAD Fairchild 660 (Universal Audio, Inc.)" "Fair660"
     FXParams                 9      1     2      3     6    7      0     8
     FXParamNames             HdRoom Input Thresh Ratio Knee Output Meter WetDry
     FXValueWidgets           Rotary|
     FXParamNameDisplays      DisplayUpper|
     FXParamValueDisplays     DisplayLower|

     FXParams		      5	 -1 -1 -1 -1 -1 -1 4
     FXParamNames	      "DC Bal" "" "" "" "" "" "" SidChn
     FXValueWidgets  	      Rotary|
     FXParamNameDisplays      DisplayUpper|
     FXParamValueDisplays     DisplayLower|
     FXWidgetModifiers	      Shift

     FXParams                 11
     FXValueWidgets           RotaryPush8
ZoneEnd
```

Now, how do we deal with stepped params, encoder acceleration, step sizes, colors, etc.? We simply add those to a new block of text at the bottom and define them on a per-parameter basis (except DefaultAcceleration, which is not at a per-parameter level). 
```
Zone "VST: UAD Fairchild 660 (Universal Audio, Inc.)" "Fair660"
     FXParams                 9      1     2      3     6    7      0     8
     FXParamNames             HdRoom Input Thresh Ratio Knee Output Meter WetDry
     FXValueWidgets           Rotary|
     FXParamNameDisplays      DisplayUpper|
     FXParamValueDisplays     DisplayLower|

     FXParams		      5	 -1 -1 -1 -1 -1 -1 4
     FXParamNames	      "DC Bal" "" "" "" "" "" "" SidChn
     FXValueWidgets  	      Rotary|
     FXParamNameDisplays      DisplayUpper|
     FXParamValueDisplays     DisplayLower|
     FXWidgetModifiers	      Shift

     FXParams                 11
     FXValueWidgets           RotaryPush8

     Left                     FXParamsBank -1
     Right                    FXParamsBank 1

     DefaultAcceleration      0.001 0.002 0.003 0.004 0.005 0.006 0.0075 0.01 0.02 0.035 0.05   
     FXParamAcceleration 7    0.001 0.002 0.003 0.004 0.005 0.006 0.0075 0.01 0.02 0.025 0.03 
     FXParamStepSize     0    0.5
     FXParamStepValues   1    0.0 0.05 0.11 0.16 0.21 0.26 0.32 0.37 0.42 0.47 0.53 0.58 0.63 0.68 0.74 0.79 0.84 0.89 0.95 1.0
     FXParamStepValues   3    0.0 0.20 0.40 0.60 0.80 1.0 
     FXParamStepValues   9    0.0 0.17 0.33 0.50 0.67 0.83 1.0
     FXParamTickCounts   3    3
     FXWidgetModes       1    BoostCut
     FXWidgetModes       7    BoostCut  
     FXParamColors       11   #ff0000 #00ff007f
ZoneEnd
```
So let's look at the new additions: what happens if you have more than 8 FXParams you want to assign to your 8-channel unit? You can continue adding additional FXParams in lines 2 and 3, then CSI will allow you to bank to the next FXParam via the **FXParamsBank** actions. Banking will also allow for mapping fx on 1-fader surfaces.

**DefaultAcceleration** is optional but allows you to set a custom acceleration curve to all encoders (there are per-parameter curves as well). 

The next set of actions are all on per-parameter basis. Notice how the syntax is the action type, the FXParam #, then the instructions with no need for brackets as used in traditional fx.zon files. So you can further still create parameter-level custom acceleration curves using the **FXParamAcceleration** action. **FXParamStepSize** is used to set the step size for an encoder tick on a FX Param. **FXParamStepValues** is used for stepped params. **FXParamTickCounts** sets the number of encoder ticks CSI must receive before advancing the FXParamValue. **FXWidgetModes** allow you to set display [[WidgetModes|WidgetMode]] on a per-parameter baseis. Lastly, **FXParamColors** can be used by supported surfaces to send RGBA colors (in hex format) to a supported widget. **Note:** CSI will be moving away from RGB to Hex everywhere.

At the time of this writing, the following actions have been implemented
```
FXParams
FXParamNames
FXValueWidgets
FXParamNameDisplays
FXParamValueDisplays

FXParamAcceleration
FXParamRange
FXParamStepSize
FXParamStepValues
FXParamTickCounts
FXParamColors (now uses Hex colors)
DefaultAcceleration
```

The following actions are coming soon.
```
FXParamsBank
FXWidgetModes
```

## New Implementation for Track, Folder, VCA Modes With New Actions
There is a new implementation and set of corresponding zones (see the X-Touch folder in the CSI Support Files for examples), of Folder and VCA modes. Track is your basic Track.zon, there's no change there. When you use a GoFolder action to activate Folder mode, only Reaper's Folder tracks become visible, and you can press the Select [or any user-defined] button to drill down into the folder. After drilling down, the parent folder track is the first track on the left, and child tracks are on the right. There's an identical implementation for VCA leaders and followers via the GoVCA navigation action.

The new actions for these modes are:
```
    GoTrack
    GoVCA
    GoFolder
    TrackVCALeaderDisplay
    TrackFolderParentDisplay
    TrackToggleVCASpill
    TrackToggleFolderSpill
```

And here is an example of the new Folder.zon:
```
Zone "Folder"
    DisplayUpper|                  TrackNameDisplay
    Fader|Touch+DisplayLower|      TrackVolumeDisplay
    DisplayLower|                  TrackFolderParentDisplay
    Shift+DisplayLower|            TrackAutoModeDisplay
    Toggle+DisplayLower|           TrackPanAutoRightDisplay
    Alt+DisplayLower|              TrackInputMonitorDisplay
    VUMeter|                       TrackOutputMeterMaxPeakLR
    Fader|                         TrackVolume 
    Flip+Fader|                    TrackPan 
    Rotary|                        TrackPanAutoLeft
    Rotary|                        WidgetMode Dot
    Toggle+Rotary|                 TrackPanAutoRight
    Toggle+Rotary|                 WidgetMode Dot
    RotaryPush|                    ToggleChannel
    Shift+RotaryPush|              TrackPan [ 0.5 ]
    Option+RotaryPush|             TrackPanWidth [ 1.0 ]
    Alt+RotaryPush|                CycleTrackInputMonitor
    RecordArm|                     TrackRecordArm
    Shift+RecordArm|               CycleTrackAutoMode
    Solo|                          TrackSolo
    Mute|                          TrackMute
    Select|                        TrackToggleFolderSpill
    Shift+Select|                  TrackRangeSelect
    Control+Select|                TrackSelect
                                   
    BankLeft                       FolderBank -8
    BankRight                      FolderBank 8
    ChannelLeft                    FolderBank -1
    ChannelRight                   FolderBank 1

ZoneEnd
```

And here's a sample VCA.zon:
```
Zone "VCA"
    DisplayUpper|               TrackNameDisplay
    Fader|Touch+DisplayLower|   TrackVolumeDisplay
    DisplayLower|               TrackVCALeaderDisplay
    Shift+DisplayLower|         TrackAutoModeDisplay
    Toggle+DisplayLower|        TrackPanAutoRightDisplay
    Alt+DisplayLower|           TrackInputMonitorDisplay
    VUMeter|                    TrackOutputMeterMaxPeakLR
    Fader|                      TrackVolume 
    Flip+Fader|                 TrackPan 
    Rotary|                     TrackPanAutoLeft
    Rotary|                     WidgetMode Dot
    Toggle+Rotary|              TrackPanAutoRight
    Toggle+Rotary|              WidgetMode Dot
    RotaryPush|                 ToggleChannel
    Shift+RotaryPush|           TrackPan [ 0.5 ]
    Option+RotaryPush|          TrackPanWidth [ 1.0 ]
    Alt+RotaryPush|             CycleTrackInputMonitor
    RecordArm|                  TrackRecordArm
    Shift+RecordArm|            CycleTrackAutoMode
    Solo|                       TrackSolo
    Mute|                       TrackMute
    Select|                     TrackToggleVCASpill
    Shift+Select|               TrackRangeSelect
    Control+Select|             TrackSelect

    BankLeft                    VCABank -8
    BankRight                   VCABank 8
    ChannelLeft                 VCABank -1
    ChannelRight                VCABank 1

ZoneEnd
```

## Depreciated CycleTrackVCAFolderModes Action
With the addition of the GoTrack, GoVCA, GoFolder actions, the CycleTrackVCAFolderModes was removed. Please transition to using the new zones and zone activation actions.

## Hex Color Support
CSI has been expanded to support Hex colors on actions. One can currently use RGB (legacy) or Hex (do not mix and match). For instance, if you wanted a particular button on your OSC surface to change colors based on whether or not the new VCA zone was active, you could do that as shown below.
```
     ButtonM31     GoVCA      { #6437017f #FA95017f }
```

## AnyPress Widget Now Available for OSC Surfaces
The [[AnyPress|Message-Generators#anypress]] Message Generator is now available to be used in .ost files for OSC surfaces such as the Behringer X32/Midas series.

# August 31, 2022

## Bug Fixes for Zone On/Off Colors on OSC, Track Selection, ScrollLink
There were some recent fixes to Zone on/off colors on OSC surfaces, as well as fixes to track selection and ScrollLink introduced due to recent changes to Track Visibility.

## Preliminary Test Implentation for OSARA Integration
[OSARA](https://osara.reaperaccessibility.com/) is described as "a Reaper extension that aims to make Reaper accessible to screen reader users." CSI has added preliminary support for OSARA with the goal of improving CSI with these screen readers. A new "Speak" action was added that can be triggered in various scenarios. See the example below which would speak the phrase "UAD Fairchild 660 Compressor" when the FX.zon was activated.
```
Zone "VST: UAD Fairchild 660 (Universal Audio, Inc.)" "Fair660"
	OnZoneActivation	Speak "UAD Fairchild 660 Compressor"

	DisplayUpper1		FixedTextDisplay "HdRoom"
 	DisplayLower1		FXParamValueDisplay 9
	Rotary1			FXParam 9 [ 0.0 0.17 0.33 0.50 0.67 0.83 1.0 ]
   ...
ZoneEnd
``` 

## New Virtual Widgets: OnPlayStart, OnPlayStop, OnRecordStart, OnRecordStop, OnZoneActivation, OnZoneDeactivation
A slew of new virtual widgets have been added to allow you to automatically fire off actions based on Play or Record state, or when Zones are activated or deactivated. This adds a LOT of flexibility to Zones and can create a "smarter" experience.

Here are some example use-cases:
```
Zone "Home"
OnRecordStart SendMIDIMessage "B5 0F 04"     // Makes button B8 strobe on record start
OnRecordStop  SendMIDIMessage "B5 0F 00"     // Makes button B8 stop strobing on record stop
OnPlayStart   SendMIDIMessage "B5 0E 04"     // Makes button B8 strobe on play start
OnPlayStop    SendMIDIMessage "B5 0E 00"     // Makes button B8 stop strobing on play start
...
ZoneEnd
```

```
Zone "SelectedTrackSend"
	OnZoneActivation	    SetXTouchDisplayColors Cyan
	OnZoneDeactivation	    RestoreXTouchDisplayColors
 ...
ZoneEnd
```

```
Zone "SelectedTrackFXMenu"
	OnZoneActivation	SetXTouchDisplayColors Yellow
	OnZoneDeactivation	RestoreXTouchDisplayColors
...
ZoneEnd
```

## New X-Touch Exclusive Actions: SetXTouchDisplayColors, RestoreXTouchDisplayColors
SetXTouchDisplayColors and RestoreXTouchDisplayColors are highly specialized actions for the X-Touch Universal and X-Touch Extenders to set the colors of all displays at once. When combined with the new OnZoneActivation, OnZoneDeactivation virtual widgets, these allow you to set all of the surface displays to the same color when you enter a SelectedTrackFXMenu zone, and restore the prior colors when you exit that Zone...
```
Zone "SelectedTrackFXMenu"
	OnZoneActivation	SetXTouchDisplayColors Yellow
	OnZoneDeactivation	RestoreXTouchDisplayColors
...
ZoneEnd
```

On the X-Touch, you can also set all 8 colors to any arbitrary value you'd like, but it MUST be all 8 colors. You include the color name for each of the 8 channels in a string with quotes. The syntax for that is shown below:
```
      OnZoneActivation     SetXTouchDisplayColors "Red Red Magenta Blue Yellow Green Cyan Red"
```

See [[FB_XTouchDisplayUpper|Feedback-Processors#fb_xtouchdisplayupper]] for a list of available X-Touch colors. 

## New Action: SendMIDIMessage
SendMIDIMessage allows you to send arbitrary MIDI message to any CSI device based on whatever conditions you'd like to setup. This is great for devices like the MIDIFighterTwister, the Launch Pads, and other MIDI surfaces that will change colors or functionality based on MIDI messages they receive. For example, I'm doing this in my Home.zon to turn on strobing and change colors of buttons on my MIDI Fighter Twister based on the playback and record states in Reaper.
```
Zone "Home"
OnRecordStart SendMIDIMessage "B1 0F 50"     // Makes button B8 red on record start
OnRecordStart SendMIDIMessage "B5 0F 04"     // Makes button B8 strobe on record start
OnRecordStop  SendMIDIMessage "B1 0F 5F"     // Makes button B8 pink on record stop
OnRecordStop  SendMIDIMessage "B5 0F 00"     // Makes button B8 stop strobing on record stop
OnPlayStart   SendMIDIMessage "B1 0E 2D"     // Makes button B8 green on play start
OnPlayStart   SendMIDIMessage "B5 0E 04"     // Makes button B8 strobe on play start
OnPlayStop    SendMIDIMessage "B1 0E 5F"     // Makes button B8 pink on play stop
OnPlayStop    SendMIDIMessage "B5 0E 00"     // Makes button B8 stop strobing on play start
     IncludedZones
          "SelectedTrack"r
          "Buttons"
          "SelectedTrackFXMenu"
          "SelectedTrackSend"
          "SelectedTrackReceive"
     IncludedZonesEnd
ZoneEnd
```

You can, of course, assign this to a button.
```
Zone "Buttons"
    SommeButton     SendMIDIMessage "B5 0E 04"     // Makes button B8 strobe on play start
ZoneEnd
```

## CSI No Longer Saves the Bank Location in Your Reaper Project
Due to changes in track visibility, Reaper state, etc., saving the banking information in the Reaper .rpp could lead to issues and was eliminated.

## Improvements to Track Visibility
To improve Track Visibility and update behavior, CSI now recalculates the Track list on every Run, about 33 times a second -- don't worry, the recalculation takes microseconds.

## Follow TCP Functionality
By default CSI will follow Reaper's MCP view. ou can override this behavior if you add FollowTCP to the line in the csi.ini with the page name as shown below.
```
Version 2.0

MidiSurface "XTouchOne" 7 9
MidiSurface "MFTwister" 6 8 
OSCSurface "iPad Pro" 8003 9003 10.0.0.146 

Page "HomePage" FollowTCP
"XTouchOne" 1 0 "X-Touch_One.mst" "X-Touch_One_SelectedTrack"
"MFTwister" 8 0 "MIDIFighterTwisterEncoder.mst" "FXTwisterMenu"
"iPad Pro" 8 0 "FXTwister.ost" "FXTwisterMenu"

Page "MixPage" 
"XTouchOne" 1 0 "X-Touch_One.mst" "X-Touch_One_SelectedTrack"
"MFTwister" 8 0 "MIDIFighterTwisterEncoder.mst" "FXTwisterFocusedFX"
"iPad Pro" 8 0 "FXTwister.ost" "FXTwisterFocusedFX"
```

## New Action: GlobalAutoModeDisplay
If you wanted to dedicate a display to showing the global auomation mode within Reaper (example: on an OSC device), there is now a CSI action that will display that.
```
Zone "Buttons"
     AutoModeDisplay     GlobalAutoModeDisplay
ZoneEnd
```

## TrackVCAFolderModeDisplay, ToggleFXOffline, and ToggleFXBypass Now Display String and Integer (Based on Feedback Processor Type)
Depending on what type of Feedback Processor these actions are assigned to, CSI will either return the surface an integer (e.g. 0 for off, 1 for on) or return a string value (e.g. 'Offline'). Example: on an OSC surface, you may want ToggleFXBypass to return "Bypased/Active" if using the FB_Processor, or 0/1 if using the FB_IntProcessor.

## Bug-Fix: ToggleFXBypass Now Respects FX Chain Bypass Status
ToggleFXBypass would return the incorrect status if the plugin was enabled by the entire FX Chain was bypassed. This has been corrected.

## New Action: OSCTimeDisplay
Use OSCTimeDisplay for displaying Reaper's time, including the various modes, on an OSC surface. This is basically the OSC equivalent of MCUTimeDisplay. Use the same, pre-existing, CycleTimeDisplayModes action to change modes.
```
Zone "Buttons"
    TimeDisplay                 OSCTimeDisplay
    SomeButton                  CycleTimeDisplayModes    
ZoneEnd
```

## New Actions: ToggleFXOffline, FXOfflineDisplay
Use ToggleFXOffline to change the FX status to "offline" in Reaper. Offline FX is similar to Bypass, but it removes the plugin from memory and additional processing. In the below example, it's assigned to Shift+Mute. The corresponding display action is FXOfflineDisplay.
```
Zone "SelectedTrackFXMenu"
        DisplayUpper|         FXMenuNameDisplay
        DisplayLower|         FXBypassDisplay
        Shift+DisplayLower|   FXOfflineDisplay
        Rotary|               NoAction
        RotaryPush|           GoFXSlot
        Mute|                 ToggleFXBypass
        Shift+Mute|           ToggleFXOffline
        Left                  SelectedTrackFXMenuBank -1
        Right                 SelectedTrackFXMenuBank 1

        RecordArm|            NoAction
        Solo|                 NoAction
        Select|               NoAction
     Fader|                   NoAction    
ZoneEnd
```

## Additions to Broadcast/Receive Functionality
[[Broadcast and Receive|Broadcast-and-Receive]] has been expanded with some additional functioanlity:

* **ToggleEnableFocusedFXMapping** is automatically included if **ToggleEnableFocusedFXMapping** is in Broadcast/Receive list
* **SelectedTrackSendBank** is automatically included if **SelectedTrackSend** is in Broadcast/Receive list
* **SelectedTrackReceiveBank** is automatically included if **SelectedTrackReceive** is in Broadcast/Receive list
* **SelectedTrackFXMenuBank** is automatically included if **SelectedTrackFX** is in Broadcast/Receive list

## New Actions: CycleTrackInputMonitor, TrackInputMonitorDisplay
Use CyleTrackInputMonitor to cycle through the input monitoring modes in a Track or SelectedTrack zone. The corresponding display action in TrackInputMonitorDisplay.
```
Zone "Track"
    Alt+RotaryPush|        	CycleTrackInputMonitor
    Alt+DisplayLower|   	TrackInputMonitorDisplay
ZoneEnd
```

## Subzones Can Now Contain | Character
In prior CSI builds, a SubZone called from a Track context with a | character could not inherit the full track context with all the channels (example: channels 1-8). Now they can. For instance, here's a basic use-case using a DualPan SubZone that wasn't possible prior to this build.
```
Zone "Track"
    SubZones
        "DualPan"
    SubZonesEnd
     RotaryPush|                        ToggleChannel

     DisplayLower|                      TrackPanDisplay
     Rotary|                            TrackPan
     Rotary|                            WidgetMode Dot
     
     Toggle+Rotary|                     TrackPanWidth
     Toggle+Rotary|                     WidgetMode BoostCut
     Toggle+DisplayLower|               TrackPanWidthDisplay
     
     Shift+RotaryPush|                  GoSubZone "DualPan"
ZoneEnd
```

And the DualPan zone itself...
```
Zone "DualPan"
     RotaryPush|                        ToggleChannel
     
     DisplayLower|                      TrackPanLDisplay
     Rotary|                            TrackPanL
     Rotary|                            WidgetMode Dot
     
     Toggle+DisplayLower|               TrackPanRDisplay
     Toggle+Rotary|                     TrackPanR
     Toggle+Rotary|                     WidgetMode Dot

     Shift+RotaryPush|                  LeaveSubZone
ZoneEnd
```

## New Action: ToggleChannel
ToggleChannel allows you to define a widget, such as RotaryPush, to toggle functionality assigned to that action. Example: this allows you to toggle between TrackPan + TrackPanDisplay and TrackPanWidth + TrackPanWidth display on the same channel. To do this, first you would define "RotaryPush|" to the ToggleChannel action. Next, you would use Toggle+as a modifier. 
```
    RotaryPush|                 ToggleChannel

    Rotary|                     TrackPan
    Rotary|			WidgetMode Dot
    DisplayLower|      		TrackPanDisplay

    Toggle+Rotary|              TrackPanWidth
    Toggle+Rotary|		WidgetMode BoostCut
    Toggle+DisplayLower| 	TrackPanWidthDisplay
```

You can now also modify your .mst files and remove the Toggle part of a rotary definition. So this...
```
Widget Rotary1
	Encoder b0 10 7f [ > 01-0f < 41-4f ]
	FB_Encoder b0 10 7f
        Toggle 90 20 7f
WidgetEnd
```

Becomes...
```
Widget Rotary1
	Encoder b0 10 7f [ > 01-0f < 41-4f ]
	FB_Encoder b0 10 7f
WidgetEnd
```

## New Actions: TrackPanAutoLeft, TrackPanAutoRight, TrackPanAutoLeftDisplay, TrackPanAutoRightDisplay
TrackPanAutoLeft will control TrackPan or TrackPanL (if using Dual Pans). TrackPanAutoRight will control TrackPanWidth or TrackPanR (if using Dual Pans). This adds considerable convenience in that you can use Stereo Balance Pans or Dual Pans even in the same Reaper project and control them in CSI without having to change zones. The one difference is that the WidgetMode for TrackPanAutoRight must be fixed (i.e. you can't use the Spread mode for PanWidth and Dot mode for PanR - you have to pick one).
```
Zone "Track"
    RotaryPush|                 ToggleChannel

    Rotary|                     TrackPanAutoLeft
    Rotary|			WidgetMode Dot
    Toggle+Rotary|              TrackPanAutoRight
    Toggle+Rotary|		WidgetMode Dot

    DisplayLower|      		TrackPanAutoLeftDisplay
    Toggle+DisplayLower|   	TrackPanAutoRightDisplay
ZoneEnd
```

**Note:** When using Dual Pans, TrackL and TrackR automation does not get written from a control surface. This appears to require a change to the Reaper API's. 

## New Action: WidgetMode
[[WidgetMode]] is designed to send additional, specific, instructions to a given widget. For instance, on a typical MCU-style device, you can set the Rotary encoder feedback to vary between Dot, BoostCut, Fill, and Spread modes.
```
    Rotary|                     TrackPan
    Rotary|			WidgetMode Dot
    DisplayLower|      		TrackPanDisplay

    Toggle+Rotary|              TrackPanWidth
    Toggle+Rotary|		WidgetMode BoostCut
    Toggle+DisplayLower| 	TrackPanWidthDisplay
```

## New Action: SetWidgetMode
SetWidgetMode exists because you may want to set a Faderport display ScribbleStripMode, therefore, you will need SetWidgetMode since there is no other Action that updates the Widget, as there would be with, say, Rotary values and LED ring style.
```
    FPDisplay   SetWidgetMode
    FPDisplay   WidgetMode SomeWidgetMode
```

## Depreciated Actions: MCUTrackPan, ToggleMCUTrackPanWidth, MCUTrackPanDisplay
The MCUTrackPan actions have been removed in favor of the new, more flexible, "ToggleChannel" functionality.

## New Message Generator: Encoder7Bit
[[Encoder7Bit|Message-Generators#Encoder7Bit]] was created to address 7-Bit absolute encoders that continue to transmit 00 values when turned counter-clockwise after the minimum has been reached, and send 7f values when turned clockwise even after the maximum value has been reached. The X-Touch Compact and X-Touch Mini encoders can be configured to behave this way.

# New FaderPortDisplay functionality (FB_FP8ScribbleStripMode Feedback Processors, new widget modes)
CSI supports all the display functionality of the Presonus FaderPort8 and FaderPort16. The FaderPorts have multiple (9) display types to choose from, contains a ValueBar and can, depending on the display type, show the VU Meter. 

### Display Type
Display type is a per display setting and consists of a widget and an action to set the actual display type. In your .mst file this will look like:
```
// ===========================================
// SCRIBBLE STRIP MODE
// ===========================================
Widget ScribbleStripMode1
	FB_FP8ScribbleStripMode 0
WidgetEnd

Widget ScribbleStripMode2
	FB_FP8ScribbleStripMode 1
WidgetEnd
etc.....
```
The image below shows all the display types and the Id

![FaderPortDisplayTypes](https://user-images.githubusercontent.com/52307138/185772386-6217f370-9564-43b4-8ca3-79463b374c9d.jpg)

The default scribble strip mode is id **2** This is a version with 4 lines and in most cases the most versatile one.

For implementing the scribble strip mode you need to add 2 lines of code. The first one is adding the scribblescript widget to the file. It's value should be `SetWidgetMode`. This tells CSI the only value to use will be the WidgetMode. The second line sets the WidgetMode value. In this case that is the Scribble strip mode. The value is the ID of one of the layouts shown above.

In your zone file this will look like this:
```
Zone "Track"
  ScribbleStripMode|        SetWidgetMode
  ScribbleStripMode|        WidgetMode 8
```

### Scribble lines

Each of the 4 scribble lines requires it’s own widget in the .mst file.
For your .mst, here are the names for the FB generators that correspond to each line on the surface.

|  | **FaderPort 8** | **FaderPort 16** |
| --- | --- | --- |
| Line 1 | FB_FP8ScribbleLine1 | FB_FP16ScribbleLine1 |
| Line 2 | FB_FP8ScribbleLine2 | FB_FP16ScribbleLine2 |
| Line 3 | FB_FP8ScribbleLine3 | FB_FP16ScribbleLine3 |
| Line 4 | FB_FP8ScribbleLine4 | FB_FP16ScribbleLine4 |

And here is an example of how the `.mst` would be mapped out for Display 1 on the Faderport8.
```
// ===========================================
// SCRIBBLE LINES CHANNEL 1
// ===========================================
Widget ScribbleLine1_1
	FB_FP8ScribbleLine1 0
WidgetEnd

Widget ScribbleLine2_1
	FB_FP8ScribbleLine2 0
WidgetEnd

Widget ScribbleLine3_1
	FB_FP8ScribbleLine3 0
WidgetEnd

Widget ScribbleLine4_1
	FB_FP8ScribbleLine4 0
WidgetEnd

```

Per scribble line you're able to set ythe text align and you can set a line to inverted (changing back and foreground colour). These values are set with a WidgetMode.
* **TextAlign**: Possible values are **Center**, **Left** and **Right**. The default when not set is **Center**
* **Invert**: Pass the value **Invert** to invert. It will be normal when not passed.

Using this in a zone file will look like this:
```
Zone "Track"
  ScribbleLine1_|      TrackNameDisplay
  ScribbleLine1_|      WidgetMode "Left"

  ScribbleLine2_|      TrackNameDisplay
  ScribbleLine2_|      WidgetMode "Center Invert"
ZoneEnd
```
_In this example line 1 in the scribble text will be left aligned. Line 2 will be right aligned and inverted._

### Valuebar
The FaderPort 8 and FaderPort 16 have a Valuebar available in the scribble display. The ValueBar can be used for a visual representation of pan value, volume, pan width, etc. The valuebar has 5 different modes.

![FaderPortValueBar](https://user-images.githubusercontent.com/52307138/185798368-b404f2a3-945f-4c06-88e5-3088c663faed.jpg)

In the mst file it looks like:
```
// ===========================================
// VALUE BAR
// ===========================================
Widget ValueBar1
	FB_FPValueBar 0
WidgetEnd

Widget ValueBar2
	FB_FPValueBar 1
WidgetEnd
```
Setting the mode is the zone file is handled by a property. For this reason it is not possible changing this with a modifier key. In the zone file it looks like:
```
Zone "Track"
  ValueBar|     TrackPan
  ValueBar|     WidgetMode BiPolar
```

# August 15, 2022 Update
Reaper forum user [Navelpluisje](https://forum.cockos.com/member.php?u=139512) has made some contributions to improve FaderPort8/16 functionality in this build, including developing a TrackNumberDisplay action. Thanks to Navelpluisje for the additions!

### BREAKING CHANGE: FB_FaderportRGB7bit is now FB_FaderportRGB
Anyone using an existing set of files for the Presonus Faderport8 or Faderport16 will need to update their .mst files to rename FB_FaderportRGB7bit to FB_FaderportRGB.

### New Action: TrackNumberDisplay
This new display action will show the Reaper track number on a display. This can be useful for multi-line displays like the Faderport8/16 or SCE24 where you may want to spare a line for the track number as well as OSC surfaces.

### New Feedback Processors: FB_FaderportTwoStateRGB, FB_FPVUMeter
FB_FaderportTwoStateRGB is used to allow two different colors for the Select buttons on the Faderport8/16 (track selected, track not selected), and FB_FPVUMeter allows one of the lines on the Faderport8/16 to function as a VU meter.

# August 14, 2022 Update

### Bug Fix: Custom Deltas
The 'Custom Delta' functionality for [[Encoders|Message-Generators#encoders]] was not working previously working and fixed in this build.

# August 5, 2022 Update

### New Feature: X-Touch Color Support (FB_XTouchDisplayUpper)
Color support for the X-Touch Universal and X-Touch Extender has been added. See [[FB_XTouchDisplayUpper|Feedback-Processors#fb_xtouchdisplayupper]] for details.

### Removed ForceScrollLink Action
The ForceScrollLink action has been removed.

### ToggleScrollLink now defaults to Off
The [[ToggleScrollLink|Navigation-Actions#togglescrolllink]] action now defaults to Off. Previously defaulted to on.

### Renamed FixedRGBColourDisplay to FixedRGBColorDisplay to standardize spelling
Standardizing CSI to consistently utilize US spelling. The FixedRGBColourDisplay action is now [[FixedRGBColorDisplay|Other-Actions#FixedRGBColorDisplay]].

### Renamed FB_GainReductionMeter to FB_ConsoleOneGainReductionMeter for clarity
Name changed to clarify this widget is specific to the Softube Console One.

### Renamed FB_VUMeter to FB_ConsoleOneVUMeter for clarity
Name changed to clarify this widget is specific to the Softube Console One.

# CSI Version 2.0 - July 10, 2022

CSI version 2.0 made a number of under-the-hood changes to zone loading to improve results and simplify some processes, while also expanding capabilities and improving features in various areas. This page is designed towards users with familiarity with CSI v1/1.1 authoring and is meant to assist in migration to version 2.0 by summarizing and consolidating the changes. 

### No more Navigators and fixed Zone names
In order to simplify .zon creation, the concept of Navigators have been removed (at least from the .zon files), and CSI inherits the Zone type based on a fixed set of Zone names. The fixed Zone names are:


```
Home
Buttons
Track
SelectedTrack
MasterTrack
SelectedTrackFXMenu
SelectedTrackSend
SelectedTrackReceive
TrackFXMenu
TrackReceive
TrackSend
```

### Changes to the Home.zon syntax (AssociatedZones)
CSI version 2 introduced the concept of AssociatedZones. These are zones like Sends, Receives, and FX Menus that are not activated as part of the home.zon, but will be called from this zone.

Below is an example of a typical MCU-style home.zon. The "Track" zone will use the displays and widgets when the Home zone is active, but if you want to call up an FX menu, Sends, or Receives to then takeover over some of the widgets, they need to be listed as AssociatedZones as shown below. When configured like this, the AssociatedZones function as "radio-button" style zones, where only one can be active at a given time (example: SelectedTrackSends or SelectedTrackReceives - not both simultaneously).
```
Zone Home
    IncludedZones
        "Buttons"
        "Track"
        "MasterTrack"
    IncludedZonesEnd
    AssociatedZones
       "SelectedTrackFXMenu"
       "SelectedTrackSend"
       "SelectedTrackReceive"
       "TrackFXMenu"
       "TrackSend"
       "TrackReceive"
    AssociatedZonesEnd
ZoneEnd
```

### No more “GoZone”
Fixed zone names mean we no longer need the GoZone action. Instead, users can activate fixed zones directly from the following list of actions:

* [GoHome](https://github.com/GeoffAWaddington/CSIWiki/wiki/GoHome)
* [GoSubZone](https://github.com/GeoffAWaddington/CSIWiki/wiki/FX-Sub-Zones)
* [LeaveSubZone](https://github.com/GeoffAWaddington/CSIWiki/wiki/LeaveSubZone)
* [GoTrackFXMenu](https://github.com/GeoffAWaddington/CSIWiki/wiki/GoTrackFXMenu)
* [GoTrackSend](https://github.com/GeoffAWaddington/CSIWiki/wiki/GoTrackSend)
* [GoTrackReceive](https://github.com/GeoffAWaddington/CSIWiki/wiki/GoTrackReceive)
* [GoSelectedTrackFXMenu](https://github.com/GeoffAWaddington/CSIWiki/wiki/GoSelectedTrackFXMenu)
* [GoSelectedTrackSend](https://github.com/GeoffAWaddington/CSIWiki/wiki/GoSelectedTrackSend)
* [GoSelectedTrackReceive](https://github.com/GeoffAWaddington/CSIWiki/wiki/GoSelectedTrackReceive)
* [GoSelectedTrack](https://github.com/GeoffAWaddington/CSIWiki/wiki/GoSelectedTrack)

### Zone feedback
Now, active zones will send feedback to surfaces that support this like the Behringer X-Touch, X-Touch One, etc. Example: if the Home zone is active, the button dedicated to this zone on these types of surfaces will light up. Same for the Send/Receive/FXMenu type zones. 

### Broadcast and Receive syntax improved
In CSI, you can instruct one surface to "broadcast" zone changes to another surface to keep them in sync, as long as that other surface is set to "receive" and listen to those broadcast changes. This is not new functionality, however, rather than having a bunch of separate actions for this behavior, you can now list all your broadcast message types on a single row in the Home.zon, and same for Receive messages. 

Broadcast/Receive only works for "Home" and "Associated Zones". If you have SelectedTrackFXMenu or FXMenu set to broadcast/receive, that includes the GoFXSlot messages that activates those zones.

The below example shows what Broadcast/Receive would look like with all currently supported options:
```
Zone Home
OnInitialization Broadcast Home SelectedTrackSend SelectedTrackReceive SelectedTrackFXMenu TrackSend TrackReceive TrackFXMenu
OnInitialization Receive Home SelectedTrackSend SelectedTrackReceive SelectedTrackFXMenu TrackSend TrackReceive TrackFXMenu
    IncludedZones
        "Buttons"
        "Track"
        "MasterTrack"
    IncludedZonesEnd
    AssociatedZones
       "SelectedTrackFXMenu"
       "SelectedTrackSend"
       "SelectedTrackReceive"
       "TrackFXMenu"
       "TrackSend"
       "TrackReceive"
    AssociatedZonesEnd
ZoneEnd
```

### FocusedFX changes
The elimination of navigators mentioned above applies to FX.zon files too! By default, CSI version 2 will have FocusedFX mapping enabled. This means if you have a fx.zon file for a particular FX/instrument plugin, and open the GUI in Reaper, that mapping will become activated by default. You can toggle this behavior off and on by assigning a button to the ToggleEnableFocusedFXMapping action as shown below:
```
   SomeButton     ToggleEnableFocusedFXMapping
```

But what if you don't want FocusedFXMapping on by default?  Since the default toggle state is "on", you can add "OnInitialization ToggleFocusedFXMapping" to flip it to off when CSI starts up as shown below:
```
Zone Home
OnInitialization ToggleEnableFocusedFXMapping
OnInitialization Broadcast Home SelectedTrackSend SelectedTrackReceive SelectedTrackFXMenu TrackSend TrackReceive TrackFXMenu
OnInitialization Receive Home SelectedTrackSend SelectedTrackReceive SelectedTrackFXMenu TrackSend TrackReceive TrackFXMenu
    IncludedZones
        "Buttons"
        "Track"
        "MasterTrack"
    IncludedZonesEnd
    AssociatedZones
       "SelectedTrackFXMenu"
       "SelectedTrackSend"
       "SelectedTrackReceive"
       "TrackFXMenu"
       "TrackSend"
       "TrackReceive"
    AssociatedZonesEnd
ZoneEnd
```

### Rewind and FastForward improvements
Big improvements have been made to the Rewind and FastForward actions. They will now latch and seek the play/edit cursor. The first press of either button will result in a slower rewind/forward speed. If you press the button again, you get a second, faster speed. In addition, feedback has been added to these actions for use in OSC surfaces or any MIDI surfaces that can provide rewind/fastforward feedback.

### New "Flip" Modifier
A new modifier called "Flip" has been introduced with the intention of being assigned to the Flip button on MCU-style control surfaces. A common use-case for this would be to temporarily put track pans onto the faders for quick panning adjustments. And because it's a modifier, a quick press of the button this is assigned to will allow for latching.

Here's how you'd define the modifier in a Buttons zone.
```
Zone "Buttons"
    Flip      Flip
ZoneEnd
```

And an example of the intended use-case can be seen here where Flip gets used as a modifier to put TrackPan on the faders....
```
Zone "Track"
    DisplayUpper|               TrackNameDisplay
    Fader|Touch+DisplayLower|   TrackVolumeDisplay
    DisplayLower|               MCUTrackPanDisplay
    VUMeter|                    TrackOutputMeterMaxPeakLR
    Fader|                      TrackVolume 
    Flip+Fader|                 TrackPan 
    Rotary|                     MCUTrackPan
    RotaryPush|                 ToggleMCUTrackPanWidth
    RecordArm|                  TrackRecordArm
    Solo|                       TrackSolo
    Mute|                       TrackMute
    Select|                     TrackUniqueSelect
    Shift+Select|               TrackRangeSelect
    Control+Select|             TrackSelect
ZoneEnd
```

### SubZones are for more than just FX now
SubZones are custom zones (i.e. they don’t have a fixed, pre-defined name) that can be called up from other zones. Common use-cases for SubZones would be create a custom Zoom zone, or a custom Marker zone that can re-purpose some widget assignments and change the functionality of the surface. They are also commonly used FX SubZones, which existed in CSI version 1.1. Think of an example of a Mastering Suite that may have more parameters than you have controls for on your surface. With FX SubZones, you could create one zone for the compressor section, another for the limiter section, then activate those different surfaces via button presses.

### FX SubZone crashes on Mac have been resolved
In CSI v1.1, using a SubZone on Mac would cause CSI to crash Reaper [there was a workaround]. These now work as intended without crashing.

### Banking actions can exist in the buttons zone
In CSI v1.1, Send, Receive and FX Menu banking actions needed to exist in those zones. In CSI v2, there are unique banking actions for each type. Those banking actions are no longer constrained to the Send, Receive, FX Menu zone files…you can use those in the buttons.zon

### Track Pin features have been removed
Track pinning has been removed [as of now] from CSI v2 in order to clean up and simplify the code behind the scenes.

### New CSI Preferences screen
To accommodate some under the hood changes how to surfaces get assigned to Pages, the CSI preferences screen has been redesigned. It now has a left to right flow where first you configure Surfaces on the left, then create one or more [[Pages]] in the middle, then use the Assignments section to assign surfaces to each page. See [[Setting up CSI for the first time|Installation-and-Setup#setting-up-your-csi-devices-for-the-first-time]] for more details. 

![CSI Preferences Screen Print](https://i.imgur.com/3gqL16s.png)

### CSI Preferences: removed the FX Menu and Send/Receive channel count fields
In the CSI Device Preferences, you'll no longer see the Send/Receive channel count fields. They inherit the count from the channel count.

### New CSI.ini format, new file error handling
Version 2.0 introduces changes to the file [[CSI.ini|CSI.INI]] file format. Additionally, now CSI will check for an incorrectly formatted CSI.ini and warn users when Reaper starts, notifying users of the version mismatch (rather than crashing Reaper). In general, CSI will warn users if there's a missing folder or incorrectly formated CSI.ini and should not crash Reaper.

### MCUTrackPan action no longer requires toggle row in .mst file
If the Rotary encoders in your .mst file contained the toggle line that was required for the MCUTrackPan action, that line can now be removed.

So if you previously had this in your .mst file...
```
Widget Rotary1
	Encoder b0 10 7f [ < 41-48 > 01-08 ]
	FB_Encoder b0 10 7f
	Toggle 90 20 7f
WidgetEnd
```

You can delete the toggle row so it looks like this...
```
Widget Rotary1
	Encoder b0 10 7f [ < 41-48 > 01-08 ]
	FB_Encoder b0 10 7f
WidgetEnd
```
### RGB Color Fixes
Reaper handles RGB colors differently between Mac and PC. A bug in CSI has been fixed to correct the RGB behavior on both platforms.

### TouchOSC can now run locally
You can now use the TouchOSC [mk II] application running on your local machine as a control surface in CSI.

### Native Apple Silicon (ARM) support, universal binary
Mac users with M1 [or more recent] can now use CSI natively in the Reaper ARM version. The .dylib is now a universal binary that includes ARM and Intel versions. You will need to allow the CSI .dylib to have access within Security and Privacy settings in Mac OS, then restart Reaper.

### Catalina or greater required for MacOS
The minimum version of MacOS supported is 10.15. This was changed in order to support new processors.

### No more 32-bit CSI builds
CSI no longer includes 32-bit builds on Windows and Mac. This was dropped in order to support new processors. 

### New actions
Here’s a list of new actions introduced in CSI v2:

* SaveProject
* Undo
* Redo
* MCUTimeDisplay (previously existed and named TimeDisplay)
* CycleTrackVCAFolderModes
* TrackVCAFolderModeDisplay
* Broadcast
* Receive
* GoHome
* GoSubZone
* LeaveSubZone
* ToggleEnableFocusedFXMapping
* ToggleEnableFocusedFXParamMapping
* GoSelectedTrackFX
* GoTrackSend
* GoTrackReceive
* GoTrackFXMenu
* GoSelectedTrackSend
* GoSelectedTrackReceive
* GoSelectedTrackFXMenu
* TrackSendBank
* TrackReceiveBank
* TrackFXMenuBank
* SelectedTrackSendBank
* SelectedTrackReceiveBank
* SelectedTrackFXMenuBank
* Flip
* ToggleFXBypass
* FXBypassDisplay

### Deprecated actions
Here’s a list of legacy actions that have been removed. Many of these actions had to do with how zones were activated.

* TimeDisplay (still exists - was renamed to MCUTimeDisplay)
* MapSelectedTrackSendsToWidgets
* UnmapSelectedTrackSendsFromWidgets
* MapTrackSendsSlotToWidgets
* UnmapTrackSendsSlotFromWidgets
* MapSelectedTrackSendsSlotToWidgets
* UnmapSelectedTrackSendsSlotFromWidgets
* MapSelectedTrackReceivesToWidgets
* UnmapSelectedTrackReceivesFromWidgets
* MapTrackReceivesSlotToWidgets
* UnmapTrackReceivesSlotFromWidgets
* MapSelectedTrackReceivesSlotToWidgets
* UnmapSelectedTrackReceivesSlotFromWidgets
* FXParamRelative
* MapSelectedTrackFXToWidgets
* UnmapSelectedTrackFXFromMenu
* MapSelectedTrackFXToMenu
* UnmapSelectedTrackFXFromMenu
* MapTrackFXMenusSlotToWidgets
* UnmapTrackFXMenusSlotFromWidgets
* GoCurrentFXSlot
* SendSlotBank
* ReceiveSlotBank
* FXMenuSlotBank
* TogglePin
* GoZone
* SetBroadcastGoZone
* SetReceiveGoZone
* SetBroadcastGoFXSlot
* SetReceiveGoFXSlot
* SetBroadcastMapSelectedTrackSendsToWidgets
* SetReceiveMapSelectedTrackSendsToWidgets
* SetBroadcastMapSelectedTrackReceivesToWidgets
* SetReceiveMapSelectedTrackReceivesToWidgets
* SetBroadcastMapSelectedTrackFXToWidgets
* SetReceiveMapSelectedTrackFXToWidgets
* SetBroadcastMapSelectedTrackFXToMenu
* SetReceiveMapSelectedTrackFXToMenu
* SetBroadcastMapTrackSendsSlotToWidgets
* SetReceiveMapTrackSendsSlotToWidgets
* SetBroadcastMapTrackReceivesSlotToWidgets
* SetReceiveMapTrackReceivesSlotToWidgets
* SetBroadcastMapTrackFXMenusSlotToWidgets
* SetReceiveMapTrackFXMenusSlotToWidgets
* ToggleVCAMode
