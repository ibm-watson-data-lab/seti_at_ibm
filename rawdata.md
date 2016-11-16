# SETI Raw Data

The raw data from the ATA are the time-series signals from the combined signal of the entire
telescope array. This signal has been demodulated (quadrature demodulation), bandpass limited,
digitized and packed into a time-series of the real and imaginary parts of the signal. 
The raw data are the complex amplitude values and the name of the files are called "compamp" files 
to reflect this. Each time sample is stored as a single byte. The
first four bits are the real part and the second four bits are imaginary part.  

When SonATA (the SETI data
acquisition system software at the ATA) observse a signal that is deemed to be radio frequency interference
(RFI), only the subband in which the signal is seen is stored to disk in file
called with an extension of '.compamp'. Each subband is, effectively, 533 Hz wide (though there is
some over-sampling to ensure all frequencies are sufficiently measured). Furthermore, due to technical
details of bandpass filtering, the data appear on disk as a set observations that last for roughly a
full second. Each of these observations are sometimes called a 'raster line' or a 'half frame'.
There are typically 129 half-frames in a single file. Each half-frame contains a header of 40 bytes, followed by the raw data

However, when SonATA classifies a signal as a potential candidate, it packages the 16 subbands nearest
to the central frequency of the observed signal. These are files with an extension of '.archive-compamp'. 

The `ibmseti` python package will read both of these files types for you. It also contains some basic 
digital signal processing tools (FFT, spectrogram) on the data and code to extract potential
machine-learning features. You can learn to use this package from the [documentation found on the `ibmseti` Github page](https://github.com/ibm-cds-labs/ibmseti)
or through the [example notebooks](README.md/#introduction-notebooks). 
