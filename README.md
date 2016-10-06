# Getting Started with SETI@IBMCloud Data Analysis

This document serves as an introductory guide to accessing the SETI data available from <span>SETI</span>@IBMCloud. 


#### Brief Introduction to SET<span>I@</span>IBMCloud Data

The SETI Institute utilizes the Allen Telescope Array (ATA) to search for radio signals from intelligent life
beyond our Solar System. Nearly each night, the ATA observes radio frequencies in the ~1-10 GHz 
frequency range from multiple locations in the sky. 

Observation of a potential signal results in three pieces of data

  * two raw data files, called a `compamp` or `archive-compamp`
  * preliminary analysis of the signal, stored as a row in the `SignalDB` table 

On each ATA telescope, the horizontal and vertical polarization 
components of the radio signal are measured separately. 
For each polarization, the raw, time-series signals from the entire ATA array are digitized and 
combined into a single data file. Additionally, the time-series data have been bandpass filtered, meaning the
frequencies observed in the data only cover a small range, called the bandwidth. 
The exact frequency range may be recovered from information found
in the header of the raw data file. (The [`ibmseti`](https://github.com/ibm-cds-labs/ibmseti) python package will calculate this for you, along 
with reading the data file and providing the necessary signal processing to get you started.)

A [SignalDB row contains the conditions and characteristics of the observation](signaldb.md),
such as the Right Ascension (RA) and Declination (DEC) celestial coordinates of the signal, 
an estimate of the power of the signal, primary carrier frequency, carrier frequency drift, 
signal classification, etc. The RA/DEC are the coordinates in the sky of the target being observed 
(referenced from the J2000 equinox). 

The raw data for a signal that is categorized as a `Candidate` is stored as an `archive-compamp` file, 
while all other non-candidate signals are stored as `compamp` files. 

Right now, we are only making the `Candidate`/`archive-compamp` files and their associated
rows in the SignalDB available. There are 456717 `archiv-compamp` files currently available, obtained
during observations from 2013 to 2015. 

Further updates will provide access to the non-candidate `compamp` files and other ways of retrieving the data.

### Basic Intro to HTTP API

Before getting into the details of the [raw signal data analysis](#raw-data-analysis), you can get a peek using just your browser. This is purely for demonstration, however. The [HTTP API](setigopublic.md) is designed to be consumed programmatically. 

#### SignalDB Introduction 

You can grab as much of the SETI SignalDB data as you like without needing a IBM Bluemix or 
Data Science Experience account. 

  1. Find a Celestial Coordinate with data
    * https://setigopublic.mybluemix.net/v1/coordinates/aca
    * Returns the number of SignalDB rows for Candidate E.T. signals for each RA/DEC
    * Select an RA,DEC coordinate  
      * For example:  RA = 0.03, DEC = 66.306  
  2. Find meta-data for Candidate E.T. signals from a coordinate
    * https://setigopublic.mybluemix.net/v1/aca/meta/0.03/66.306
    * Returns the SignalDB data for the Candidate events. 

The SignalDB data includes signal classifications, 
[metrics such as central frequency, central frequency drift rate, power, signal/noise, etc.](signaldb.md), 
with which you can already do some interesting visualization and analysis. 

#### Raw Data Introduction

In order to get at the raw data, **you will need Bluemix or Data Science Experience**.

  1. Get Access Token 
    * https://setigopublic.mybluemix.net/token
  2. Get Raw Data temporary URL 
    * using a `container` and `objectname` returned above
    * using your `access_token` 
    * Build URL: `https://setigopublic.mybluemix.net/v1/data/url/{container}/{objectname}?access_token={access_token}`
    * Example: 
      * container: `setiCompAmp`
      * objectname: `2013-03-14/act10/2013-03-14_20-37-32_UTC.act10.dx1000.id-0.R.archive-compamp`
      * access_token: `abcdefg1234567890`
      * URL: `https://setigopublic.mybluemix.net/v1/data/url/setiCompAmp/2013-03-14/act10/2013-03-14_20-37-32_UTC.act10.dx1000.id-0.R.archive-compamp?access_token=abcdefg1234567890`
  3. Get Raw Data for Candidate E.T. signal
    * Use the URL returned in the previous step to get the raw `archive-compamp` file.
    * Example:
      * `https://dal.objectstorage.open.softlayer.com/v1/AUTH_cdbef52bdf7a449c96936e1071f0a46b/setiCompAmp/2013-03-14/act10/2013-03-14_20-37-32_UTC.act10.dx1000.id-0.R.archive-compamp?temp_url_sig=2e4e981c7a14b899394e4bde6a9d6d53e238f56b&temp_url_expires=1475256287`

## Raw Data Analysis

The following set of instructions and notebooks will introduce you to doing analysis with SE<span>TI</span>@IBMCloud. We  will use

  * the HTTP API at https://setigopublic.mybluemix.net
  * IBM Spark and Object Store services
  * the [`ibmseti`](https://github.com/ibm-cds-labs/ibmseti) python package

As previously noted, you must create an [IBM Data Science Experience](http://datascience.ibm.com/) account or sign up for [IBM Bluemix](http://www.ibm.com/cloud-computing/bluemix/) in order to get an `access_token` to the data. The example notebooks below use Spark and Object Storage. The free 30-day Bluemix/DSX trial will allow you to provision a Spark service and associated Object Store. 


### Choose DSX or Bluemix Account

We highly recommend using a DSX account. DSX is focused on the data scientist, obviously, and helps you to 
use the analytics and data storage tools. DSX uses Bluemix to provision services on your 
behalf and provides an overall better project management experience. 

### Data Science Experience (DSX)

  1. [Sign up for DSX](http://datascience.ibm.com/)
  2. Create a new notebook. You have the option to create a notebook that copies one of our [example notebooks](notebooks). This is done by selecting the 'From URL' tab when configuring the new notebook and pasting in a URL. Or, if you like, you can create a new blank notebook and type in the code by hand. 
  3. In order to save raw SETI data, click on 'Data Sources' on the right pallete and follow the instructions to provision an Object Storage instance.
    * create a container called "seti_raw_data"


### Bluemix

  1. [Sign Up for Bluemix](https://console.ng.bluemix.net/registration/?Target=https%3A%2F%2Fconsole.ng.bluemix.net%2Flogin%3Fstate%3D%2Fhome%2Fonboard)
  2. [Provision a Spark Service](https://console.ng.bluemix.net/catalog/services/apache-spark/) from the Catalog
  3. In order to save raw SETI data, connect an Object Store during the setup.
    * Click on the Apache Spark Service card from your Dashboard
    * Click on "Notebooks"
    * Click on "Object Store" near the top
    * Click on "Add Object Storage" and follow the instructions
        * create a container called "seti_raw_data"
  4. From your Spark Service, start a new notebook. You have the option to create a notebook that copies one of our [example notebooks](notebooks). This is done by selecting the 'From URL' tab when configuring the new notebook and pasting in a URL. **For Bluemix, you'll need to link to the raw version of the file.** Or, if you like, you can create a new blank notebook and type in the code by hand. 




## Introduction Notebooks

After you work through these notebooks, you'll have used the HTTP API
to access the data, saved that data to your IBM Object Storage service, produced a spectrogram
from the raw SETI data, and extracted some features, which may be used in a machine-learning analysis.


  * [Introduction to the HTTP API](notebooks/ibmseti_intro_to_http_api.ipynb) 
  * [How to store data to your Bluemix Object Store](notebooks/ibmseti_get_data_tutorial.ipynb)
  * [Retrieve the data from Object Store and calculate a spectrogram](notebooks/ibmseti_my_first_spectrogram.ipynb)
  * [Retrieve the data from Object Store and calculate features from the spectrogram](notebooks/ibmseti_intro_features.ipynb)


You may use the links to these notebooks to create new notebooks in your Spark service. 
Do this by selecting "New Notebook" in your Spark service, and then provide the URL to 
these notebooks before you create it. 

## Other Documents

  * [Description of SignalDB](signaldb.md)
  * [Science Goals](science_goals.md)
  * [What Is an E.T. Signal?](what_is_an_et_signal.md)


#### Check out SETI:

  * [Facebook/SETIInstitute](https://www.facebook.com/SETIInstitute)
  * [SETI.org](http://www.seti.org/)
  * [SETIQuest.info](http://setiquest.info/)
  * [SETIQuest.org](http://setiquest.org/)
  * [Jon Richard's Twitter feed (SETI engineer at ATA)](https://twitter.com/jrseti)

## Contact

We intend for this SETI+IBM collboration to extend to the general public. We want to empower citizen 
scientists to analyze this data and contribute their analysis and code back to SETI. You can influence
future analysis and future observation planning! So, please contact us should you have an interesting
analysis. We can provide you with direction, answers to questions, and access to more data! 

  * [Contact Info](contact_us.md)

If you have any problems with the service, submit an Issue or [contact me directly](https://github.com/gadamc).



