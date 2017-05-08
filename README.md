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


##### Important Update

The new endpoint `/v1/aca/meta/all` will return the entire SignalDB table in a single CSV file.
You should use this file to select your data of interest. Loading this data into a dataframe will
make data selection significantly easier and more flexible. Once you've found the subset of data that
you find interesting, you can then make queries to find the associated raw data files. The example notebooks
in this documentation, unfortunately, will not be immediately updated to include these instructions. 
Any instructions you see below where you query the meta-data can be replaced by appropriate filter and
selection in a dataframe created from the SignalDB CSV file. 


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
30-day free IBM Data Science Experience account. (This will create a Bluemix account for you automatically during this process.) Even after your free trial period ends, you can still access the SETI data with your access-token. 

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

As previously noted, you must create an [IBM Data Science Experience](https://datascience.ibm.com) account
in order to get an `access_token` to the data. The trial period let's you provision most servies
completely free, such as Spark service and Object Store. 

### Data Science Experience Setup

When you [signed up for a DSX account](http://datascience.ibm.com/), the onboarding process of DSX should
have automatically instantiated a Spark service, an Object Storage service and a new sample project space. 
This is all you need to get started.  

1. Select `Projects` from the links at the top of the DSX landing page
2. If a `Default Project` does not exist for you:
  * Create a new project by clicking the ![+](img/plus_button.png "+") button (or [click here](https://apsportal.ibm.com/projects/new-project?context=analytics))
  * Provide a project name and select the Spark and Object Storage instances available to you.
3. Within your project, select "add notebook". 
4. To directly import one of the tutorial notebooks, select `From URL`
  * Use the URL to a notebook in the GH repo, such as https://github.com/ibm-cds-labs/seti_at_ibm/blob/master/notebooks/ibmseti_get_data_tutorial.ipynb
5. Optionally, create a new notebook from scratch and copy/paste the python code from the tutorials.

#### Object Storage Credentials

In order to use the Object Storage instance that is automatically generated in your DSX account 
programmatically, you will need to obtain the credentials. [For example, the second tutorial in this project steps 
you through the process of saving data to your Object Storage](notebooks/ibmseti_get_data_tutorial.ipynb). 
There are two ways to obtain the credentials. 

##### Feed Object Storage some data

1. From within any notebook, select the ![10/01 icon](img/1001.png "10/01") icon near the top right.
2. If you already have data in your Object Storage, you should see files listed in the side panel. 
  * If there are no data, click the `browse` button to upload a file. 
  * Choose a small text file for quick upload.
  * Click `Apply`
3. Select an open cell in your notebook.
4. Under one of your data files, select the `Insert to code` button and choose `Insert Credentials`.

##### Copy from Bluemix

Alternatively, the credentials are found in your Bluemix account, which was created for you automatically when you signed up for DSX.

1. Log in to https://bluemix.net
2. Scroll down and click the Object Storage instance you are using in DSX
3. Select the `Service Credentials` tab and `View Credentials`
4. Copy these into your notebooks where appropriate.

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

