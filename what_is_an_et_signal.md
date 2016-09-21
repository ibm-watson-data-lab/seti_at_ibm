# What is an E.T. Signal?

The short answer to this question is "we don't know" and we aim to keep assumptions to a minimum when
searching for a potential signal.

However, there are characteristics of a signal that would be reasonable to expect. 

  * Persistent. We observe the same signal multiple times from the same location in the sky.
  * Unique. Simultanously, we don't observe the signal at another location.
  * Reproducible. Should a signal be found, confirmation should be obtained from an independent observation.
  * Stable, non-zero Doppler drift. Any signal with a zero frequency drift is probably local radio frequency interference (RFI).
  * Not RFI. Internally at SETI, there exists a database of RFI signals to compare against before classification. The data provided in this project have passed this check.
  * Sign of intelligence. A signal appears to contain some type of encoding or chirping, or displays the statistical properties of a signal (not noise). In the SETI context, decoding symbols from signals will likely not be possible. A signal with a bandwidth smaller than 300 Hz would be sufficient evidence since there is no evidence that astrophysical objects can produce smaller bandwidths<sup>1</sup>.


Yet, things aren't always so simple, as explained by Dr. Jeff Scargle of the NASA Ames 
Research Center (a member of this SETI+IBM collaboration).  He makes strong arguments for 
avoiding as many assumptions as you can when searching for potential signals.

The first three points are also characteristics of signals from astrophysical objects, such as a pulsars. 
In fact, the scientists that discovered the first pulsar [briefly speculated the signal was 
a radio beacon from an extraterrestrial civilization](http://www.bigear.org/vol1no1/burnell.htm)! 

Additionally, E.T. signal necessarily have a stable drift-rate -- the E.T. signal source could orbit a 
binary star system, causing shifts in the Doppler drift.  These requirements would also rule out any E.T. 
probe orbiting our solar system since it would not appear in the same location in the sky!

Some of these (and other) criteria have already been applied to the SETI data at the ATA before being 
written to disk (and hence, available to you). 
Short-term versions of the 'persistence' and 'unique' requirements are applied 
at the time of observation. Signals that appear from one 'beam' at the ATA, but not on others and are observed
multiple times get tagged with ['Activity' values of 'targetN-on/off'](signaldb.md#acttyp). 
Yet, suppose that E.T. were broadcasting a beacon for only a few minutes toward Earth a few times 
per week or month (because it's also broadcasting toward other habitable planets). That signal 
might show up at SETI for a few N observations, but would then disappear. 
But perhaps it showed up again at a later date?

Nevertheless, given that we need some set of criteria by which to make decisions about our search for E.T., 
should you find a signal that satisfies some of these requirements, 
it would be easy to argue for further observations. But don't limit yourself. 
If you find something interesting, [report it](contact_us.md) and perhaps we can make 
further observations at the ATA. 

<hr>

1. R.J.Cohen, et. al., Mon. Not. R. astr. Soc. (1987) **225**, 491-498
