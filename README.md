# Getting Started with SETI Data Analysis

This document serves as an introductory guide to accessing the SETI data available at IBM. 

## Setup

There are four parts to working with SETI data

  * IBM Spark and Object Store services
  * [The HTTP API](setigopublic.md)
  * The [`ibmseti` python package](https://github.com/ibm-cds-labs/ibmseti)
  * Understanding the data

> Please read the [privacy warning](https://github.com/ibm-cds-labs/ibmseti#privacy-warning) when using the `ibmseti` package. 

You must create an account on [IBM Bluemix](http://www.ibm.com/cloud-computing/bluemix/) and provision a Spark service and associated Object Store. 

  1. [Sign Up for Bluemix](https://console.ng.bluemix.net/registration/?Target=https%3A%2F%2Fconsole.ng.bluemix.net%2Flogin%3Fstate%3D%2Fhome%2Fonboard)
  2. [Select the Spark Service](https://console.ng.bluemix.net/catalog/services/apache-spark/) from the Catalog
  3. Connect an Object Store during the setup.
    * Click on the Apache Spark Service card from your Dashboard
    * Click on "Notebooks"
    * Click on "Object Store" near the top
    * Click on "Add Object Storage" and follow the instructions
        * create a container called "seti_raw_data"

## Introduction Notebooks

Please read [the beginning section of the HTTP API](setigopublic.md). It provides a 
basic introduction to the SETI data that is being provided. 

After you work through these notebooks, you'll have used the HTTP API
to access the data, saved that data to your IBM Object Storage service, produced a spectrogram
from the raw SETI data, and extracted some features, which may be used in a machine-learning analysis.

  * [Introduction to the HTTP API](notebooks/ibmseti_intro_to_http_api.ipynb) 
  * [How to store data to your Bluemix Object Store](notebooks/ibmseti_get_data_tutorial.ipynb)
  * [Retrieve the data from Object Store and calculate a spectrogram](notebooks/ibmseti_spectrogram_tutorial.ipynb)
  * [Retrieve the data from Object Store and calculate features from the spectrogram](notebooks/ibmseti_intro_features.ipynb)

You may use the links to these notebooks to create new notebooks in your Spark service. 
Do this by selecting "New Notebook" in your Spark service, and then provide the URL to 
these notebooks before you create it. 

## Other Documents

  * [Science Goals](science_goals.md)
  * [What Is an E.T. Signal?](what_is_an_et_signal.md)


## Contact

We intent for this SETI+IBM collboration to extend to the general public. We want to empower citizen 
scientists to analyze this data and contribute their analysis and code back to SETI. You can influence
future analysis and future observation planning. So, please contact us should you have an interesting
analysis. We can provide you with direction, answers to questions, and access to more data! 

  * [Contact Info](contact_us.md)




