# SETI Public Data Server

The SETI Institute utilizes the Allen Telescope Array (ATA) to search for radio signals from intelligent life
beyond our Solar System. Nearly each night, the ATA observes radio frequencies in the ~1-10 GHz 
frequency range from particular localtions in the sky. 

Observation of a potential signal results in two pieces of data: a raw data file, called a `compamp`
or `archive-compamp`, and preliminary analysis of the signal, which is stored as a row
in the `SignalDB` table. The raw data are the time-series signal recorded from the ATA, 
decomposed as left- and right-circularly polarized signals into two separate files, for a 
particular location in the sky and bandwidth. A SignalDB row contains the conditions of the observation,
such as the Right Ascension (RA) and Declination (DEC) celestial coordinates of the observation, and 
signal characteristics such as estimates of the power of the signal, primary carrier frequency, drift, 
etc., which are derived from the raw data signals.  All RA/DEC coordinates are referenced from 
the J2000 equinox. A full description of the values in SignalDB are found [here](). For this 
project, we have stored the SignalDB on
[dashDB](http://www.ibm.com/analytics/us/en/technology/cloud-data-services/dashdb) and the raw
`compamp` and `archive-compamp` files in 
[IBM Object Store](https://console.ng.bluemix.net/catalog/object-storage/).

We have constructed an HTTP API that allows you to access these data, which is described in this document. 

The raw data for a signal that is categorized as a `Candidate` is stored as an `archive-compamp` file, 
while all other types signals are stored as `compamp` files. The only difference between these file 
types is that the `archive-compamp` contains signal across 16 subbands nearest to the signal observed 
by the ATA, while the `compamp` files contain just the single subband where `non-Candidate`
signals was observed. 

Right now, we are making only the `Candidate`/`archive-compamp` files and their associated
row in the SignalDB available. 

Further enhancements to this API will provide access to `compamp` and other ways of retrieving the data.

### Data License

This data is licensed by the SETI Institute of Mountain View, CA under the Create Commons BY 4.0 license.

IBM has been granted non-exclusive permission by the SETI Institute to reproduce, 
prepare derivative works, and distribute copies of data, which have been provided to IBM, 
or will be provided in the future, by the SETI Institute under the Creative Commons BY 4.0 license.


## API Reference

### Endpoints:

  * [**/v1/coordinates/aca**](#celestial-coordinates-of-candidate-events)
  * [**/v1/aca/meta/{ra}/{dec}**](#meta-data-and-location-of-candidate-events)
  * [**/v1/aca/meta/spacecraft**](#meta-data-and-location-of-candidate-events-for-pacecraft)
  * [**/v1/token/{username}/{email address}**](#token-for-raw-data-access) **Not Yet Implemented**
  * [**/v1/data/url/{container}/{objectname}**](#temporary-url-for-raw-data)


## Full Example Usage with Python

### Introduction

The following example, done in Python, shows a typical way to use this API. 

The general flow should be something like: 

  * Find an interesting target in the sky and note the coordinates.
  * Determine if data is available for a particular point or region in the sky.
  * Get the SignalDB rows and raw data file information for that region. 
  * Get a temporary URL for the raw data file.
  * Download the data and store it.

Though it's not necessary, typically one will start by selecting an interesting target or
interesting region in the sky.  

Once the authorization token system is implemented, you will be required to 
have an [IBM Bluemix](https://bluemix.net)
account and will be limited to 10k temporary URL requests per month. 

You may read below or [click here to see a full example in a shared IBM Spark notebook](https://console.ng.bluemix.net/data/notebooks/b05446cc-8303-4b1d-8150-2d55e49691ef/view?access_token=e70e82feeae786d3e57f2f918b4d22439723dd619e3059143942f0bdb4b9504c).

### Select an Interesting Target

Use some kind of [source](http://phl.upr.edu/projects/habitable-exoplanets-catalog)
to find interesting expolanet coordinates. There are other websites that carry this information.
You can also just pick a region in the sky. 
[This map](http://www.hpcf.upr.edu/~abel/phl/exomaps/all_stars_with_exoplanets_white.png), 
along with the `Observing Progress` link found on [https://setiquest.info](http://setiquest.info/),
may be helpful.

You'll need to know the right ascension (RA, in hours from 0.0 to 24) and declination 
(DEC, in degrees from -90.0 to 90.0) of your exoplanet/target of interest. However, the RA/DEC
values recorded in the database may not exactly match the known position of the object (at most, 
these values differ by 0.002 degrees). Or you may choose a wider region of the sky. Either way, 
we use the API to search for data within an enclosed region of the sky. 

#### Kepler 1229b

Let's look at Kepler 1229b, found here http://phl.upr.edu/projects/habitable-exoplanets-catalog.
This candidate planet has an Earth similarity index (ESI) of 0.73 (Earth = 1.0, Mars = 0.64, Jupiter = 0.12) 
and is 770 light-years away. It is the 5th highest-ranked planet by ESI.

You can take a look at this object in the sky: 
[http://simbad.cfa.harvard.edu/simbad/sim-id?Ident=%408996422&Name=KOI-2418.01&submit=submit](http://simbad.cfa.harvard.edu/simbad/sim-id?Ident=%408996422&Name=KOI-2418.01&submit=submit)

The celestial coordinates for Kepler 1229b is RA = 19.832 and DEC = 46.997

### Query the SignalDB

Here is the example code to query our API for a region about the position of Kepler 1229b. The API
endpoint to use is [`/v1/coordinates/aca`](#celestial-coordinates-of-candidate-events). In this case,
we define a 0.004 x 0.004 box around the RA/DEC position.

```python
import requests
RA=19.832
DEC=46.997
box = 0.002

# We want this query
# http://setigopublic.mybluemix.net/v1/coordinates/aca?ramin=19.830&ramax=19.834&decmin=46.995&decmax=46.999

params = {
  'ramin':RA-box, 'ramax':RA+box, 'decmin':DEC-box, 'decmax':DEC+box
}
r = requests.get('https://setigopublic.mybluemix.net/v1/coordinates/aca',
    params = params)

import json
print json.dumps(r.json(), indent=1)
```

We get the following output:

```json
{
  "returned_num_rows": 1,   
  "skipped_num_rows": 0, 
  "rows": [
    {
      "dec2000deg": 46.997, 
      "number_of_rows": 392, 
      "ra2000hr": 19.832
    }
  ], 
  "total_num_rows": 1
}
```

This means we found 392 'Candidate' signals recorded while observing this planet in the SignalDB. 

##### Aside

If we expand our range of allowed RA/DEC values, we find 'Candidate' events for some nearby positions.

```
GET http://setigopublic.mybluemix.net/v1/coordinates/aca?ramin=19.800&ramax=19.90&decmin=46.95&decmax=47.02
```

returns

```json
{
  "total_num_rows": 4,
  "skipped_num_rows": 0,
  "returned_num_rows": 4,
  "rows": [
    {
      "ra2000hr": 19.832,
      "dec2000deg": 46.997,
      "number_of_candidates": 392
    },
    { 
      "ra2000hr": 19.834,
      "dec2000deg": 46.961,
      "number_of_candidates": 370
    },
    {
      "ra2000hr": 19.856,
      "dec2000deg": 46.968,
      "number_of_candidates": 44
    },
    {
      "ra2000hr": 19.861,
      "dec2000deg": 46.965,
      "number_of_candidates": 32
    }
  ]
}
```

You may wish to extend your box further so see what you find. 

### Get SignalDB Rows

Given a particular celestial coordinate, we can obtain all of the 'Candidate' signal meta data,
as found in the SignalDB.

The endpoint to use is 
[/v1/aca/meta/{ra}/{dec}](#meta-data-and-location-of-candidate-events).

Continuing from the example above


```python
ra = r.json()['rows'][0]['ra2000hr']  #19.832 from above query
dec = r.json()['rows'][0]['dec2000deg'] 46.997 

r = requests.get('https://setigopublic.mybluemix.net/v1/aca/meta/{}/{}'.format(ra, dec)

print json.dumps(r.json(), indent=1)
```

Here's what the output will look like:

```
{
 "returned_num_rows": 200, 
 "skipped_num_rows": 0, 
 "rows": [
  {
   "inttimes": 94, 
   "pperiods": 27.36000061, 
   "pol": "mixed", 
   "tgtid": 150096, 
   "sigreason": "PsPwrT", 
   "freqmhz": 6113.461883333, 
   "dec2000deg": 46.997, 
   "container": "setiCompAmp", 
   "objectname": "2014-05-20/act14944/2014-05-20_13-00-01_UTC.act14944.dx2016.id-0.L.archive-compamp", 
   "ra2000hr": 19.832, 
   "npul": 3, 
   "acttype": null, 
   "power": 50.652, 
   "widhz": 2.778, 
   "catalog": "keplerHZ", 
   "snr": 32.397, 
   "uniqueid": "kepler8ghz_14944_2016_0_2208930", 
   "beamno": 2, 
   "sigclass": "Cand", 
   "sigtyp": "Pul", 
   "tscpeldeg": 78.558, 
   "drifthzs": -1.055, 
   "candreason": "Confrm", 
   "time": "2014-05-20T12:59:55Z", 
   "tscpazdeg": 307.339
  }, 
  ...
  "total_num_rows": 392
}
```

The maximum return limit is 200 rows per query. The `total_num_rows` tells you there are 392 rows. To get the rest, you'll need to use the `?skip=200` optional
URL parameter. For example

```python 
r = requests.get('https://setigopublic.mybluemix.net/v1/aca/meta/{}/{}?skip=200'.format(ra, dec)
newrows = r.json()['rows']
```

Searching through the results from this particular example, one thing you'll notice is that while there are 392
'Candidate' signals found in the SignalDB, it doesn't mean there are 392 raw data files. *There
will be duplicates.* So you'll need to sort through them appropriately. If you do not and you use
the same raw data multiple times within a machine-learning algorithm (to extract a set of features, 
for example), you'll likely corrupt your results. 

An easy way to reorder your results by the raw data file is with a `groupby`. For example

```python
rows # list of signalDB rows returned from above
rows = sorted(rows, key=lambda row:row['container'] + '-' + row['objectname'])

grl = []  #a list of (key, [rows])
for k, g in itertools.groupby(rows, lambda row:row['container'] + '-' + row['objectname']):
  grl.append((k, list(g))

# OR in Spark

rdd = sc.parallelize(rows)
rdd = rdd.groupBy(lambda row: row['container'] + '-' + row['objectname']) 
```

You'll also notice there multiple files that have the same SignalDB, but with slightly
different file names. The antenna data are decomposed into left- and right-circularly polarized
complex data signals, which are stored in separate files; hence, the `L` and `R` components of the names.

The location of the raw data is stored in an
[IBM Object Store available on Bluemix](https://developer.ibm.com/bluemix/2015/10/20/getting-started-with-bluemix-object-storage/).

In order to access those  data files, one must first request a temporary URL.

NB: We separate the steps between getting the SignalDB row and container/objectname for the raw data
file and getting a URL to the raw data file for a reason. Since downloading and analyzing the
raw data can be an expensive operation, making these steps separate gives your analysis the
opportunity to select only data, based on the values in SignalDB, that may be interesting. For example,
you may wish to consider only data that have a particular Signal-To-Noise ratio (SNR is one of the values
in SignalDB). Secondly, we issue temporary URLs that expire one hour after you've requested
the URL, which allows us to control usage. 

##### Spacecraft

Since spacecraft do not have a fixed RA/DEC value, we have a special endpoint, 
[/v1/aca/meta/spacecraft](#meta-data-and-location-of-candidate-events-for-pacecraft).

Use this endpoint just as you would the `/v1/aca/meta/{ra}/{dec}`. The returned JSON will have
the same schema. 


### Temporary URLs and Data Storage

The temporary URLs are obtained with the
[`/v1/data/url/{containter}/{objectname}`](#temporary-url-for-raw-data) 
endpoint. *These temporary URLs, by default, are valid for only one hour.* You must consider this
when obtaining the URLs and retrieving the data. 

We separate the stages of querying for the SignalDB rows and the raw data in order to allow for an
analysis of the information in the SignalDB before making requests for the raw data. This cuts down on
unnecessary requests and traffic.


```python
def get_temp_url(row):
  r = requests.get('https://setigopublic.mybluemix.net/v1/data/url/{}/{}'.format(row[0], row[1]))
  return (r.status_code, r.json()['temp_url'], row[0], row[1])

temp_urls = map(get_temp_url, data_paths)
```

For each temporary URL, we can now download the file and store it, or analyze it. 
[Here is a complete example](https://console.ng.bluemix.net/data/notebooks/6818fe79-84f5-4c22-a800-b80aa7696ef4/view?access_token=c1a0bf78689d45d01425a5b8a59c886da55a16eb17c4708791c14551c3c17b21)
using the IBM Spark Service to download and place data in
[IBM Object Storage](https://developer.ibm.com/bluemix/2015/10/20/getting-started-with-bluemix-object-storage/).


## API Reference


### Celestial Coordinates of Candidate Events
##### GET /v1/coordinates/aca

**Description**: Returns a JSON object that lists the exact RA and DEC coordinates
for 'Candidate' observations available in the Public SETI database. 
The structure of the returned JSON object is

```
{
 "returned_num_rows": 200, 
 "skipped_num_rows": 0, 
 "rows": [
  {
   "dec2000deg": 46.997, 
   "number_of_candidates": 392, 
   "ra2000hr": 19.832
  },
  ... 
 ], 
 "total_num_rows": 1124
}
```

Each element of `rows` contains the exact coordinates and the number of candidate
events found for that position (`number_of_candidates`). 

  * **Optional Parameters**

    **ramin, ramax, decmin, decmax**: any or all of these parameters may be used. 
    They define allowed ranges to be returned. 

    **skip**: number of results to skip
  
    **limit**: number of results to return (maximum is 200)

  * **Examples**:

    Get candidate events found within an area of the sky.
    
    ```
    GET /v1/coordinates/aca?ramin=19.0&ramax=20.0&decmin=35.0&decmax=55.0
    ```

    Paginate through candidate coordinates

    ```
    GET /v1/coordinates/aca?skip=200
    GET /v1/coordinates/aca?skip=400
    ... until `returned_num_rows` == 0
    ```



### Meta-data and location of Candidate Events
##### GET /v1/aca/meta/{ra}/{dec}

**Description**: Given the RA and DEC celestial coordinates for a position in the sky,
returns a JSON object containing the meta-data and file location of 
each candidate event for that RA/DEC coordinate. The meta-data are the data found
in the [SignalDB](https://github.com/ibmjstart/SETI/docs/signaldb.md). 
There may be tens to thousands of candidate events for a particular position.
The structure of the JSON document is

```
{
 "returned_num_rows": 200, 
 "skipped_num_rows": 0, 
 "rows": [
  {
   "inttimes": 94, 
   "pperiods": 27.36000061, 
   "pol": "mixed", 
   "tgtid": 150096, 
   "sigreason": "PsPwrT", 
   "freqmhz": 6113.461883333, 
   "dec2000deg": 46.997, 
   "container": "setiCompAmp", 
   "objectname": "2014-05-20/act14944/2014-05-20_13-00-01_UTC.act14944.dx2016.id-0.L.archive-compamp", 
   "ra2000hr": 19.832, 
   "npul": 3, 
   "acttype": null, 
   "power": 50.652, 
   "widhz": 2.778, 
   "catalog": "keplerHZ", 
   "snr": 32.397, 
   "uniqueid": "kepler8ghz_14944_2016_0_2208930", 
   "beamno": 2, 
   "sigclass": "Cand", 
   "sigtyp": "Pul", 
   "tscpeldeg": 78.558, 
   "drifthzs": -1.055, 
   "candreason": "Confrm", 
   "time": "2014-05-20T12:59:55Z", 
   "tscpazdeg": 307.339
  }, 
  ...
  "total_num_rows": 392
}
```

Only 200 rows may be returned in a single query. Use the `skip` and `limit` options to paginate through
the results. 



  * **Optional Parameters**

    **skip**: number of results to skip
  
    **limit**: number of results to return (maximum is 200)

  * **Examples**:

    Get candidate events found within an area of the sky.
    
    ```
    GET /v1/aca/meta/19.832/46.997
    ```

    Paginate through candidate coordinates

    ```
    GET /v1/aca/meta/19.832/46.997?skip=200
    GET /v1/aca/meta/19.832/46.997?skip=400
    ... until `returned_num_rows` == 0
    ```


### Meta-data and location of Candidate Events For Spacecraft
##### GET /v1/aca/meta/spacecroft

**Description**: This method returns results exactly like [/v1/aca/meta/{ra}/{dec}](),
except that it returns all data for observed spacecraft (mainly satellites). 
This returns a JSON object containing the meta-data and file location of 
each candidate event. The meta-data are the data found
in the [SignalDB](https://github.com/ibmjstart/SETI/docs/signaldb.md). 
There may be tens to thousands of candidate events for a particular position.
The structure of the JSON document is

```
{
 "returned_num_rows": 10, 
 "skipped_num_rows": 100, 
 "rows": [
  {
    "uniqueid": "exoplanets8ghz_10640_2009_0_559913",
    "time": "2014-03-14T04:02:04-07:00",
    "acttype": "target",
    "tgtid": 175,
    "catalog": "spacecraft",
    "ra2000hr": -99,
    "dec2000deg": -99,
    "power": 1354.566,
    "snr": -1.881,
    "freqmhz": 2217.520738889,
    "drifthzs": -0.09,
    "widhz": 5.556,
    "pol": "both",
    "sigtyp": "CwC",
    "pperiods": null,
    "npul": null,
    "inttimes": 94,
    "tscpazdeg": 0,
    "tscpeldeg": 0,
    "beamno": 2,
    "sigclass": "Cand",
    "sigreason": "Confrm",
    "candreason": "Confrm",
    "container": "setiCompAmp",
    "objectname": "2014-03-14/act10640/2014-03-14_04-02-04_UTC.act10640.dx2009.id-0.R.archive-compamp"
  }, 
  ...
  "total_num_rows": 17394
}
```

Only 200 rows may be returned in a single query. Use the `skip` and `limit` options to paginate through
the results. 


  * **Optional Parameters**

    **skip**: number of results to skip
  
    **limit**: number of results to return (maximum is 200)

  * **Examples**:

    Paginate through candidate coordinates

    ```
    GET /v1/aca/meta/spacecraft?skip=200
    GET /v1/aca/meta/spacecraft?skip=400
    ... until `returned_num_rows` == 0
    ```


### Token for raw data access
##### GET /v1/token/{username}/{email address}

**Description**: This is not yet implemented. Once implemented, this will create a 
token for the user to access the data. 


### Temporary URL for raw data
##### GET /v1/data/url/{container}/{objectname}

**Description**: Given a container and object name, returns a JSON object 
containing a temporary URL to the data file.
The temporary URL will be valid for 60 minutes from the time it was issued. 
The container and object name are obtained from the results of 
[`/v1/aca/meta/{ra}/{dec}'](#meta-data-and-location-of-candidate-events)

*When the `token` parameter is implemented, you will be rate-limited to 10k 
temporary URLs per month. This is equivalent to 10 GB of raw data.*

*If you can make a compelling argument, you may obtain
an increase in the number of temporary URLs per month. Contact
adamcox@us.ibm.com*



  * **Required Parameters**

    **token**: Not yet implemented. Once implemented, a token will be required for
    data to be returned.

  * **Examples**:


    ```python
    import requests

    cont = 'setiCompAmp'
    objname = '2014-05-20/act14944/2014-05-20_13-00-01_UTC.act14944.dx2016.id-0.L.archive-compamp'

    data_url = 'https://setigopublic.mybluemix.net/v1/data/url/{}/{}'.format(cont, objname)  
    r_data_url = requests.get(data_url)

    print json.dumps(r.json(), indent=1)
    ```

    Prints 

    ```
    {
     "temp_url": "https://dal05.objectstorage.softlayer.net/v1/AUTH_4f10e4df-4fb8-44ab-8931-1529a1035371/setiCompAmp/2014-05-20/act14944/2014-05-20_13-00-01_UTC.act14944.dx2016.id-0.L.archive-compamp?temp_url_sig=b228c472264ff49fc869ada18c3ac4a0cc96817d&temp_url_expires=1468279610"
    }
    ```

    Use URL to get data.

    ```python
    r_data = requests.get(r_data_url.json()['temp_url'])

    rawdata = r_data.content
    print 'file size: ', len(rawdata)
    ```

    ```
    file size:  1061928
    ```
