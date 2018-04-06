---
layout: post
title: Spectrum Analyzer Saga
tags:
    - radio
    - electronics
    - spectrum analyzer
    - tools
excerpt: "A story of building and calibraring the W7ZOI spectrum analyzer, Kanga edition."
---

## What is a spectrum analyzer and why do I want one?

A _spectrum analyzer_ allows one to clearly *see* with naked eye the frequency components that make up a signal. As a result it displays a frequency to amplitude graphic to the user where the horizontal axis represents the frequency whereas the vertical one represents the amplitude.

Most signals are not pure (that is they are not formed of a single frequency) but made of a bunch of other signals with different amplitudes and phases at different frequencies.

Whenever we perform processing of a signal we may cause side effects such as creating other (un)needed signal components.

## How I got the kit

emrfd -> qrp-l -> got a W7ZOI SA Kanga edition kit.


## First run

Once all modules got completed and individually tested as best I could, I wanted to quickly set up the system so I can actually *see* whether I got anything out. I dugged out a frequency reference kit which I got from W8DIZ and hooked it up to the 1st LO and 1st Mixer module directly (since the 70MHz LPF was not built yet), powered everything, set the scope to XY mode and bam! Here is the output of the first sweep ever to be ran on the spectrum analyzer. 

![First SA sweep](/assets/first-sweep.png)

Shown in the graphic is the output of a [W8DIZ frequency reference](http://www.kitsandparts.com/fref.php) described to output a 3V PP in 50 Ohm which translates to somewhere around 13.5dBm. The frequency reference is set up to output a 10MHz square wave signal. We can see in the picture above the [predominant odd harmonics characteristic to square waves](https://en.wikipedia.org/wiki/Square_wave#Examining_the_square_wave). 

## Problem
The problem at this point is that while the whole system works, it does not work as it should. 

## Debugging

I will attempt to describe my process of thought as I go about attempting to debug the beast. The test conditions for the graphic above are the following:

* +13.5dBm input signal
* no 70MHz LPF in the chain
* IF GAIN ADJ from IF LOG set to +15V directly
* scope Y channel sampling R321 directly, that is we bypass the LOG AMP CAL control, taking the maximum available signal
* both X and Y scope channels set to 0.5V/div

Theoretically we should be getting signals that are so high that get out of screen but clearly we do not; the most we can get is shown in the picture above. This leads me to think that it is either a tuning or an amplification problem somewhere in the chain. 

Here is a schematic of the current test setup:

![Signal Path](/assets/first-sweep-setup.svg)

I would start from right to left in the signal path and test that each module performs according to the expectations. Thus, first on the list is the IF+LOG Amp module.

As the name puts it, there are actually two modules in there, namely an IF amplifier and a LOG detector. Let's take them one by one.

### LOG detector

After re-reading the original articles and updates, the calibration assertion is the following: if we inject a clean -10dBm signal directly in the LOG detector input, we should be able to get a 4V signal at the output in order to confirm it is working as expected. That is not to say that we should be getting a fixed 4V signal but a larger one which can be later reduced if needed through the use of LOG AMP CAL control.

Sounds like a plan, let's go! Only problem is I do not possess a clean -10dBm signal source. Time to build one.

#### -10dBm 10MHz signal source

I decided to select 10MHz as XTAL frequency because this will come in hand when testing the IF AMP and peaking the resolution bandwidth filters too. 

The basic idea is to start off with a 10MHz XTAL oscillator whose output will be amplified, filtered and brought to the appropriate level with an attenuator in order to establish the needed 50 ohm output impedance.

![Signal Source](/assets/-10dbm-signal-source.svg)

As luck would have it, while looking around for a 10MHz crystal I ran into an old G3UUR oscillator made for xtal characterisation and decided to canibalize that. I hooked a crystal to it and read 9.997MHz. 3KHz off but good enough.
Next I needed to measure the output level in a 50 ohm load.

I hooked the output of the oscillator to a 50 ohm load and used the scope on x10 to read the peak to peak voltage. I had 515 mV PP which roughly translates to -2.5 dBm. The output however varied with input voltage and was somewhat distorted on the lower side of the sinusoid, a sign of harmonic content.

I decided to add a MAR-6 monolithic amplifier to pump out the output and power both the oscillator and the amplifier from a 7806 in order to keep a stable amplitude on the output.

These changes had the effect of an output of 1.71V PP in a 50 ohm load which roughly translates to +8.6 dBm. It looks like I only have to make sure I get a clean signal by filtering away the harmonics and add attenuation to meet the desired output level which in our case is -10 dBm.

I knew I ordered long ago some LPF kits from Dieter _"The toroid king"_ W8DIZ and after a hasty run through my unbuilt kit box I came out victorious with an unbuilt [30M 7th order Chebyshev](http://www.kitsandparts.com/univlpfilter.php) low pass filter. I couldn't wait to build this one up and plug it in to see how output changes.

Two hours later I happily watched the beautiful sinusoidal signal unfolding on the scope. Its amplitude was 1.35V PP in a 50 ohm which equates to about +6.5 dBm. Now I only need to lose 16.5 dB in order to match the required -10 dBm output level.

A PI attenuator pad with 68 ohm shunt resistors and 154 ohm series resistor took care of that.


---------------------

Edit: amp got cracked (mind the P1dB of the MAR-6 which was 1dBm !!!11) so I decided to leave it out and only attack the output with the PI filter.

265mV PP in 50 ohms => -7.55 dBm. Measured with the signal source plugged into a 50 ohm load and scope measuring the voltage on the load with 10x probe.

Next I coupled the signal source to the chinese power meter and it read -13.4dBm. Wow, someone is in error. I used the 10x scope probe to measure the voltage on the powermeter input leads and sure enough it was the same 265mV. 

Next I coupled the signal source to the W7ZOI power meter (KangaUS edition) and read the DC voltage. I got 4.72 volts. Comparing that value to the chart from figure 5, "Build an RF power meter" article from June 2001 QST article put me in the same ballpark as the chinese digital power meter reading, somewhere around -13.5dBm.

Time to do some head scratching. Could the initial calculation be wrong? I used 
















5.8V -> VCO straight to powermeter
4.72V -> signal source 10MHz straight to powermeter

if 5.8V ~= 10dBm
   4.72V would be 18 dB lower -> 8dBm
