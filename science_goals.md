# Science Goals

The goal of this project is to help improve the work being done at the SETI Institute and at the ATA. 
We are aiming for some bright citizen scientists to make significant contributions in the following ways.

### Improved Signal Detection

Have we missed a SETI signal already? The SETI institute is always looking for new a better ways to increase signal detection. 

  * Can a ML-based model based on features from spectograms produce a better signal recognition or 
  better signal-to-noise? Can image recognition / neural nets be used here effectively? There are many possible features to extract that have not yet been explored and lots of opportunity, overall. Combine new features from spectrograms with information from [SignalDB](signaldb.md)

  * Does using a different orthonormal basis 
  ([KLT](https://en.wikipedia.org/wiki/Karhunen%E2%80%93Lo%C3%A8ve_theorem)) improve signal detection? 
  This is completely unexplored territory.

  

### Real-time signal classification at ATA

The ATA DAQ software, [SonATA](https://github.com/setiQuest/SonATA) data acquisition system software, 
makes subsequent observations of the same RA/DEC celestial location after detecting a signal. 
The SonATA software uses it's own signal classification methodology. However, 
if real-time signal classification can be improved, the ATA woudl spend less time observing where no signal 
exists, and more time where a real signal is present. 

An accurate ML model, trained from previously taken data, could provide a better signal classifier 
for the ATA to make its re-observation decisions. We hope to install a ML model to assist SonATA in its
real-time signal classification. 

**[Please submit to us your models!](contact_us.md)**
  