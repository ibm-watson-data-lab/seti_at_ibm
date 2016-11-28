# Getting Started with SETI@IBMCloud Data Analysis

This document serves as an introductory guide to accessing the SETI data available from <span>SETI</span>@IBMCloud. 


#### Brief Introduction to SET<span>I@</span>IBMCloud Data

The SETI Institute utilizes the Allen Telescope Array (ATA) to search for radio signals from intelligent life
beyond our Solar System. Nearly each night, the ATA observes radio frequencies in the ~1-10 GHz 
frequency range from multiple locations in the sky. 

Observation of a potential signal results the following data

  * two raw data files, either two `compamp` or two `archive-compamp` files, depending on signal classification
  * real-time analysis of the signal, stored as a row in the `SignalDB` table 

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

Extraction of the SignalDB rows via the HTTP API are based on locations of objects in the sky.

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

In order to get at the raw data, we ask that you first attain an access token. 
We do this to track usage of the project. This will, for the moment, require you to create a
30-day free IBM Bluemix account (no credit-card required). 
Even after your free trial period ends, you can still access the SETI data with your access-token. 

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

As previously noted, you must create an [IBM Bluemix](http://www.ibm.com/cloud-computing/bluemix/) account
in order to get an `access_token` to the data. The trial period let's you provision most servies
completely free, such as Spark service and Object Store, which we use heavily in the example
analysis below.

### Bluemix Setup

[Sign Up for Bluemix](https://console.ng.bluemix.net/registration/?Target=https%3A%2F%2Fconsole.ng.bluemix.net%2Flogin%3Fstate%3D%2Fhome%2Fonboard)

  1. [Provision a Spark Service](https://console.ng.bluemix.net/catalog/services/apache-spark/) from the Catalog
  3. In order to save raw SETI data, connect an Object Store during the setup.
    * Click on the Apache Spark Service card from your Dashboard
    * Click on `Notebooks`
    * Click on `Object Store` near the top
    * Click on `Add Object Storage` and follow the instructions
        * create a container called `seti_raw_data` (or other name if you wish)
  4. From your Spark Service, create a new notebook. You have the option to create a notebook that is a duplicate of one of our [example notebooks](notebooks). This is done by selecting the `From URL` tab when configuring the new notebook and pasting in the URL. **For Bluemix, you'll need to link to the raw version of the example notebook ([for example](https://raw.githubusercontent.com/ibm-cds-labs/seti_at_ibm/master/notebooks/ibmseti_get_data_tutorial.ipynb)). Look for the `Raw` button on the GitHub page.** You can also create a new blank notebook and type in the code by hand. 



### Data Science Experience 

While you can get by with just a Bluemix account, we also recommend trying
[Data Science Experience (DSX)](http://datascience.ibm.com/). DSX is a new platform
from IBM that is focused on the data scientist.
DSX helps you to organize your
use of the analytics and data storage tools available in Bluemix.

[Sign up for DSX](http://datascience.ibm.com/)

With a DSX account, you can create a new project (Select `My Projects` from the upper left menu) and add
Spark and data management services. If you've already provisioned those services in your Bluemix account,
you should see the options to use those services when you initially configure your project. If you have not
provisioned those services in Bluemix, you can provision them from within DSX, though it will take a few more steps.

Once your project is set up, select `create notebook`. You have the option to create a notebook that copies
one of our [example notebooks](notebooks). This is done by selecting the `From URL` tab when configuring
the new notebook and pasting in a URL. You can also create a new blank notebook and type in
the code by hand.  


## Introduction Notebooks

After you work through these notebooks, you will have used the HTTP API
to access the SETI data, saved that data to your IBM Object Storage service, produced a spectrogram
from the raw SETI data, and extracted some features, which may be used in a machine-learning analysis.


  * [Introduction to the HTTP API](notebooks/ibmseti_intro_to_http_api.ipynb) 
  * [How to store data to your Bluemix Object Store](notebooks/ibmseti_get_data_tutorial.ipynb)
  * [Retrieve the data from Object Store and calculate a spectrogram](notebooks/ibmseti_my_first_spectrogram.ipynb)
  * [Retrieve the data from Object Store and calculate features from the spectrogram](notebooks/ibmseti_intro_features.ipynb)



## Other Documents

Reading these documents will be helpful to analyzing the SETI data. 

  * [Description of SignalDB](signaldb.md)
  * [Brief description of the raw data](rawdata.md)
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

### License 

Copyright 2016 IBM Cloud Data Services

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

