# Science Goals

The goal of this project is to help improve the work being done at the SETI Institute and at the ATA. 
We are aiming for citizen scientists to make significant contributions in the following ways.

### Buried Treasure

Have we missed an ET signal already? Are there patterns in the ATA data that have gone unnoticed? 
For the most part, the SETI search at the ATA is a "real-time" narrow-band signal search process. 
There is less emphasis and work done looking for patterns in the data after acquisition. Help us to look for 
long-term patterns in the data. Some of this work can be done by analyzing the contents of the SignalDB alone. 
It could also be via a different signal classification of the raw data.

 * Does there exist a short-term signal that is found days, weeks or month's apart?
 * Is there a signal found at a specific frequency across the sky (need to correct for drift)?
 * Is there a signal other than narrow-band found consistently anywhere?

### Improved Signal Detection

The ATA data acquisition system software, [SonATA](https://github.com/setiQuest/SonATA), 
makes subsequent observations of the same RA/DEC celestial location after detecting a signal. 
The SonATA software performs signal classification by looking for narrow-band signals. However, it also sometimes records signals that are not narrow-band, but at a much lower efficiency (false-positive narrow-band signals). If real-time signal classification can be improved, the ATA would spend less time observing where no signal 
exists, and more time where a real signal is present. But there's a catch -- the need to be as open as possible to what an ET signal might look like!

We basically need a broader signal classification tool -- a tool that can detect narrow-band signals, as well algorithms that separate out other classes. We hope to install a classification tool at the ATA to assist SonATA in its real-time signal classification. We are looking for citizen scientists to build a robust classification tool. 

  * Can a ML-based model, based on features from spectograms, produce a better signal recognition or 
  better signal-to-noise for narrow-band signals? Can image recognition / neural nets be used here effectively? There are many possible features to extract that have not yet been explored and lots of opportunity, overall. 

  * Does using a different orthonormal basis 
  ([KLT](https://en.wikipedia.org/wiki/Karhunen%E2%80%93Lo%C3%A8ve_theorem)) improve signal detection? 
  This is completely unexplored territory.
  
  * Can we identify the non narrow-band signals in the data and use them to construct an algorithm to specifically detect them?


This task can also work well with the search for Buried Treasure, above. In particular, a highly efficient narrow-band signal detection algorithm may find interesting signals in the data.

#### Feedback

If you develop a serious model and wish to try it out on the entire SETI dataset available at IBM (not all data are accessible through the public API), **[let us know.](contact_us.md)**

  
