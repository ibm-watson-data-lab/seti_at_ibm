# Science Goals

The goal of this project is to help improve observations performed by SETI Institute at the Allen Telescope Array (ATA). 
We are aiming for citizen scientists to make significant contributions in the following ways.

### Buried Treasure

Has the SETI Institute missed an ET signal already? Are there patterns in the ATA data that have gone unnoticed? 
For the most part, the SETI search at the ATA is a "real-time" narrow-band signal search process and 
there is less emphasis and work done looking for patterns in the data after acquisition. 
This is a challenge to search for interesting signals and patterns in the current data set. 
Some of this work can be done by analyzing the contents of the SignalDB alone. 
It could also be done via a newly-developed signal classification algorithm applied to the raw data. We should note that 
this is a high-risk, high-reward project, meaning the likelihood of finding a signal in this data is very small. 
However, developing new signal classification would dovetail with projects described in the next section. 

 * Does there exist a short-term signal that is found days, weeks or month's apart?
 * Is there a consistent signal found at a specific frequency across the sky (need to consider Doppler drift)?
 * Is there a signal other than narrow-band types of signal found consistently anywhere?
 * Is there a narrow-band signal that from a particular location that is very compelling to reobserve?

### Improved Signal Detection

The ATA data acquisition system software, [SonATA](https://github.com/setiQuest/SonATA), which stands for SETI on the ATA,
makes subsequent observations of the same RA/DEC celestial location after detecting an initial signal. 
The SonATA software performs signal classification by looking for narrow-band signals. 
However, it also sometimes records signals that are not narrow-band, but at a much lower efficiency 
(false-positive narrow-band signals). If real-time signal classification can be improved, the ATA would 
spend less time observing where no signal 
exists, and more time where a real signal is present, increasing the rate of survey across the sky and 
improving the probability of locating an ET signal.  

Basically, we are looking for a broader signal classification tool, signal classification tools for specific signal types, 
or tools for narrow-band and non-narrow-band signals. We hope to install a classification tool at the ATA to assist
SonATA in its real-time signal classification. As such, we are looking for a robust classification tool that can 
execute quickly.

  * Can a ML-based model, based on features from spectograms, produce a better signal recognition or 
  better signal-to-noise for narrow-band signals? Can image recognition / neural nets be used here effectively? 

  * Can the complex Fourier transform of the signals (and not just the power) be used for classification? Is the phase important?

  * Does using a different orthonormal basis 
  ([KLT](https://en.wikipedia.org/wiki/Karhunen%E2%80%93Lo%C3%A8ve_theorem)) improve signal detection? 
  
  * Can we identify the non narrow-band signals in the data and use them to construct a set of algorithms to specifically detect them?

This task can also work well with the search for Buried Treasure, above. In particular, a 
highly efficient narrow-band signal detection algorithm may find interesting signals buried in the data.

#### Feedback

If you develop a serious model and wish to try it out on the entire SETI dataset available at IBM (not all data are accessible through the public API), **[let us know.](contact_us.md)**

  
