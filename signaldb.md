# SignalDB

| Field        | Description           |
| :------------- |:--------------|
| uniqueid   |  ID based on other info                            |
| time       |  Date/Time of observation                          | 
| acttyp     |  Activity Type: initial or followup                |
| catalog    |  Target catalog name                               |
| tgtid      |  Target Number in Catalog                          |  
| ra2000hr   |  Right Ascension (hours)                           |
| dec2000deg |  Delination (degrees)                              |
| power      |  Raw Power Measure                                 |
| snr        |  Signal/Noise                                      |
| freqmhz    |  Start frequency of observed signal (MHz)          |
| drifthzs   |  Signal Drift (Hz/s)                               |
| widhz      |  Signal Bandwidth (Hz)                             |
| pol        |  Polarization of Signal                            |
| sigtyp     |  Signal Type                                       |
| pperiods   |  Pulse Period (s)                                  |   
| npul       |  Number of pulses                                  |   
| inttimes   |  Integration Time                                  |   
| tscpazdeg  |  Primary telescope azimuth direction (degrees)     |
| tscpeldeg  |  Primary telescope elevation direction (degrees)   |
| beamno     |  Beam signal number                                |
| sigclass   |  Signal Class                                      | 
| sigreason  |  Signal Reason                                     | 
| candreason |  Candidate Reason

#### Additional Comments on Fields

##### `time`
The date-time of the observation is recorded in ISO format 'yyyy-mm-ddThh:mm:ssZ'

##### `acttyp`

This is the "Activity Type" of the observation. There are a number of possible values:

  * target
  * target`N`-on
  * target`N`off
  
for N = 1, 2, 3, 4.  When the ATA makes an initial observation of a candidate signal, the ATA
decides to look again for the signal. This second observation is called 'target1-on'. The ATA 
then looks *elsewhere*, called target1-off. If a signal is **not** observed in target1-off, 
the ATA then again looks at the original location. An observation of a candidate signal would 
then be called target2-on. This happens repeatedly. Each time a potential candidate signal is 
re-observed, the N is increased. So, look in your data for "large" N.
More information is found on the "How Observing Works" tab at http://setiquest.info/.


##### `catalog` and `tgtid`
 
Each observed location in the sky corresponds to a particular known object, or target. Each target 
has an identification number, which is stored in a particular external database or catalog. T

##### `power`

?? what are the units? 

##### `snr`

The signal-to-noise ratio is only reported for candidate events.

##### `freqmhz`

This is the measured frequency, in MHz, of the signal observed in the spectrogram at `t=0`. 

##### `drifthz`

This is the Dopper drift of the observed signal. This is due to an acceleration between the ATA
and the source of the signal. 

##### `widhz`

This is the size of the signal bandwidth -- it's how wide the signal appears on the spectrogram. 

##### `pol`

** More info. If it's left, does it mean only observed in left circularly polarized signal? what does 'mixed' mean - signal found in both, but not at the exact same time and frequency

Values can be

  * left
  * right
  * both
  * mixed

##### `sigtyp`

?? More info

The signal type can be 

  * CwP = carrier wave power (0.7 Hz)
  * CwC = carrier wave coherent (0.01 Hz)
  * Pul = pulsed signal

##### `pperiods`

?? Does this pertain only to `sigtyp` = Pul? 

##### `npul`

?? Also, does this pertain only to sigtyp=Pul?

##### `inttimes`

?? Does this pretain just to the spectrogram? Can I get an example? If the signal is observed over the whole spectrogram, would inttimes = 92 seconds? And if the signal only showed up for a few seconds within the spectrogram would inttimes = 3 seconds? Or can `inttimes` be larger than a single spectrogram? 

Total time of observation that resulted in this signal.

##### `tscpazdeg` and `tscpeldeg`

?? More info. 

##### `beamno`

The ATA can record signals from three different locations in the sky, called beams. This is a record of 
which beam produced the signal. 

##### `sigclass`

This is the Signal Classification. For achive-compamp files, which are the only types of files we are currently
making available, this value will always be `Candidate`. If we make other file types available in the future, 
they will be labeled `RFI` (radio frequency interference) or `Unkn` (unknown). 

##### `sigreason`

This is the justification for the classification. The values could be

  * PsPwrT. Passed power threshold, passed on to candidate status
  * Dft2Hi. Drift too high, rejected
  * 2MnyCnd. Too many candidates, the system flooded and aborted, rejected
  * ZeroDft. Signal has essentially zero drift, therefore most likely terrestrial, rejected
  * SNtInCh. Signal drifted out of channel, could not follow up, unresolved
  * RctRFI. Signal was found in recent RFI database, rejected

Since all of the archive-compamp files are `Candidate` events, the `sigreason` for those files `PsPwrT`. 

##### `candreason`

This is sub-categorization for `Candidate` signals

* PsPwrT. For pulses, signal passes on to next follow up
* PsCohD. For carrier wave, signal passes on to a follow up
* ZeroDft. See above, rejected
* SnMulBm. Seen in multiple beams, rejected
* Confrm. Signal seen in only one beam above threshold, pass on to a follow up
* Dft2Hi. See above, rejected
* SNoSigl. Signal disappeared and was not seen in secondary beam either. No follow up, but not rejected. 
* SSawSig. Signal disappeared and was seen in secondary beam, rejected.
* NoSignl. Signal disappeared in followup. No follow up, but not rejected 
* RConfrm. Signal was confirmed in followup, passes on to next follow up.
* SeenOff. Sognal was seen in an off observation, rejected 
* RctRFI. See above


