# Lesson 01 - HackRF Introduction

## Setting Up

The software used to control the HackRF is called __GNU Radio Companion__. This tutorial run this on Windows.
If you decide to use Pentoo, do __NOT__ use a virtual machine.

Plug in the HackRF to your host computer, you should see the LEDs on the side of the device illuminate. This
indicates that the HackRF is getting power.

## Exploring the GUI

When opening the GNU Radio Companion for the first time, you should get something that looks like this:

![](https://github.com/dtokita/hackrf_wiki/blob/master/pics/PIC1.PNG)

## Starting Off

The way that you can interact with blocks is by double-clicking them. Double-click on the __Generate Options__
block in the top left corner and change the property from __QT GUI__ to __WX GUI__.

We first want to create a source block which tells the HackRF to receive signals. This can be done by creating
a block called __osmocom Source__. This block can be found at __(no module specified)__->__Sources__->__osmocom Source__.

We can create a GUI that allows us to see the FFT of the signals that are coming in from the RF. Create a
__WX GUI FFT Sink__ block that can be found at __Core__->__Instrumentation__->__WX__->__WX GUI FFT Sink__.

Connect the output of the __osmocom Source__ to the input of the __WX GUI FFT Sink__.

We now want to modify some of the properties of the __osmocom Source__. We can change the global sampling rate
to __10 MHz__ by changing the property __samp_rate__ to __10e6__. We can change the hardware frequency that it
will tune to by changing the property __Ch0: Frequency (Hz)__ to whichever radio station you want to tune to
(__102.7e6__). Change the __RF Gain (dB)__ to __0__. Your properties should look as follows:

![alt text](https://github.com/dtokita/hackrf_wiki/blob/master/pics/PIC3.PNG "osmocom Source properties")

Modify the property __Average__ to __On__ in the __WX GUI FFT Sink__ block. This will smooth the graph out.
The properties should look like:

![alt text](https://github.com/dtokita/hackrf_wiki/blob/master/pics/PIC4.PNG "WX GUI FFT Sink properties")

Generate the flow to check for errors using __F5__ and then execute the flow using __F6__. A window showing
the FFT should pop up. Notice that the graph is centered around 0 MHz, we can change this to reflect the 
frequency it is tuned to.

![alt text](https://github.com/dtokita/hackrf_wiki/blob/master/pics/PIC2.PNG "FFT Plot")

Kill the foreground window using __F7__.

We can create a variable by creating a variable block which is found at __Core__->__Variables__->__Variable__.
Name the variable __center_freq__ and give it the value that you tuned to (__102.7e6__). We can now change the
property __Ch0: Frequency (Hz)__ in __osmocom Source__ to __center_freq__. We can also change the property
__Baseband Freq__ in __WX GUI FFT Sink__ to __center_freq__. The flow should reflect these changes.

## Bigger Picture

Understand that we've only simply tuned the radio by tuning the hardware in the HackRF. However, often times
the strength of a software radio is being able to "tune" the radio using software. Recall from signals that
we can use sinusoidal signals to manipulate the signal that we are receiving from the HackRF to get tune
to a particular frequency, completely in software.

We can generate a cosine value by creating a __Signal Source__ block found at __Core__->__Waveform Generators__
->__Signal Source__. We know that if we multiply the signal receive from the HackRF by our generated cosine
sinusoid, we will shift the HackRF signal by the frequency of the generated source.

Create a new variable (by creating a variable block) called __channel_freq__. Set this value to a frequency
you want to tune into (__104.3e6__). Now set the property __Frequency__ in the __Signal Source__ block to
__center_freq - channel_freq__. It is important to not that we can use Python as property values.

Create a multipication block found at __Core__->__Math Operators__->__Multiply__. Connect the output of
__osmocom Source__ to the first input of __Multiply__. Connect the output of __Signal Source__ to the second
input of __Multiply__.

To verify that this has worked, create a copy of __WX GUI FFT Sink__ by copying and pasting and have the output
of the __Multiply__ block connect to the input of this new block that was just created. Change the property
__Baseband Freq__ to __channel_freq__ as this new FFT should be tuned accordingly. Run to confirm:

![alt text](https://github.com/dtokita/hackrf_wiki/blob/master/pics/PIC5.PNG "Shifted FFT")

## Demodulation of the Signal

In order to hear the signal that we've been tuning into, we need to demodulate this. We first can pass this 
through a __Low Pass Filter__ which can be found at __Core__->__Filters__->__Low Pass Filter__. Connect the
output of the __Multiply__ block to the input of the __Low Pass Filter__.

Create another variable (by creating a block) and name it __channel_width__. Set this value to __200e3__.

Set the properties of the __Low Pass Filter__ as follows:

__Cutoff Freq__: 75e3,
__Transition Width__: 25e3,
__Decimation__: int(samp_rate/channel_width)

Notice that we can use a Python statement again to force the float into an int, the reason being is that this
particular block can only decimate by an integer. Decimation reduces the sample rate, so we will get 200e3
samples per second out of the LPF.

We can circumvent the issue of an integer only decimation by using a __Rational Resampler__ which can be found
at __Core__->__Resamplers__->__Rational Resampler__. Set the __Interpolation__ to __12__ and the __Decimation__
to __5__. This allows us to decimate by __12/5__. Connect the output of the __Low Pass Filter__ to the input
of the __Rational Resampler__.

We now need a __WBFM Receiver__ (Wide Band Frequency Modulation) found at __Core__->__Modulators__->__WBFM Receive__.
Set the __Quadrature Rate__ (input frequency) to __480e3__ and __Audio Decimation__ to __10__. This should give us 
a final sample rate of __48e3__ which most sound cards support.

Create an __Audio Sink__ found at __Core__->__Audio__->__Audio Sink__. Set the __sample_rate__ to __48kHz__.
Connect the output of __WBFM Receive__ to the input of __Audio Sink__.

Your flow should look something similar to this:

![alt text](https://github.com/dtokita/hackrf_wiki/blob/master/pics/PIC6.PNG "Modulation Flow")

Run this flow and make sure your volume is up, you should be hearing audio.

You can play with the __channel_freq__ variable in order to use software to "tune" into different frequencies
without changing the frequency that the hardware is tuned to. A combination that I found that works well is
__center_freq__: 91e6, __channel_freq__: 94.7e6, __sample_rate__: 20e6).

## Gain Slider

We currently have no way to adjust the volume of the incoming HackRF signal (besides changing the volume slider
on your computer). This can be accomplished by multipling the amplitude of the signal by a constant value.
This is done using a __Multiply Const__ block found at __Core__->__Math Operators__->__Multiply Const__.

Delete the connection between __WBFM Receive__ and __Audio Sink__ and place the __Multiply Const__ block
between them. Connect the output from __WBFM Receive__ to the input of __Multiply Const__ and the output of
__Multiply Const__ to __Audio Sink__. Change the property __IO Type__ to __Float__ in the __Multiply Const__ block.

We can the augment the GUI of the window that pops up by inserting a slider. This is done by __Core__->
__GUI Widgets__->__WX__->__WX GUI Slider__. This will give us a simple way to change the __Constant__ property of
__Multiply Const__. Change the __ID__ to __audio_gain__, __default_value__ to __1__, __minimum__ to __0__, and
__maximum__ to __10__. Finally, change the property __Constant__ in __Multiply Const__ to __audio_gain__. Now you 
can use the slider on the GUI to change the volume:

![alt text](https://github.com/dtokita/hackrf_wiki/blob/master/pics/PIC7.PNG "GUI Slider")

## Conclusion

You can use this flow in order to explore the idea of using a software radio in order to tune into different
frequencies. Because we used software, it is possible to tune into two radion stations at once. This should be done
as an exercise. 

