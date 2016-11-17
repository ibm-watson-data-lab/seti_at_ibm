# SETI Raw Data

The raw data available in this project are time-series signals created from a combination of signals from the individual telescopes of the Allen Telescope Array (ATA). This signal has been demodulated
(quadrature demodulation), bandpass limited,
digitized and packed into a time-series of complex values -- the real and
imaginary parts of the demodulated signal. 
The raw data are the complex amplitude values and the name of the files are called "compamp" files 
to reflect this. In these data files, each time sample is stored in a single byte. The
first four bits are the real part and the second four bits are imaginary part.  

When SonATA (the SETI data
acquisition system software at the ATA) observes a signal that is deemed to be radio frequency interference
(RFI), only the subband in which the signal is seen is stored to disk in file
called with an extension of '.compamp'. Each subband is, effectively, 533 Hz wide (though there is
some over-sampling to ensure all frequencies are sufficiently measured). Furthermore, due to technical
details of bandpass filtering, the data appear on disk as a set of smaller observations that last for roughly a
full second. Each of these observations are sometimes called a 'raster line' or a 'half frame'.
There are typically 129 half-frames in a single file. The 129 half-frames consitute an observation 
of the frequency spectrum for about 90 seconds. On disk, each half-frame is laid out separately and contains a header of 40 bytes, followed by the raw data. 

However, when SonATA classifies a signal as a potential candidate, it packages the 16 subbands nearest
to the central frequency of the observed signal. These are files with an extension of '.archive-compamp'. These 
files are, essentially, 16 of the regular compamp files packaged together. 

In the SETI@IBMCloud project, we only make the candidate, 'archive-compamp' files available right now.
There are 456k raw data files for you to look at. 

The `ibmseti` python package will read both of these files types for you so you do not need to know
the specific details. It also contains some basic 
digital signal processing tools (FFT, spectrogram) on the data and code to extract potential
machine-learning features. You can learn to use this package from the [documentation found on the `ibmseti` Github page](https://github.com/ibm-cds-labs/ibmseti)
or through the [example notebooks](README.md/#introduction-notebooks). 
