

# Capstone Project - The Battle of Neighborhoods

This file, and other associated files, make up my contribution to the final Peer Reviewed Assignment for the Coursera Capstone Project for Applied Data Science Capstone. This was my final module in the [IBM Data Science Professional Certificate](https://www.coursera.org/specializations/ibm-data-science-professional-certificate) programme.

For reference I include the original definition for each part of the assignment.

## Part 1

Clearly define a problem or an idea of your choice, where you would need to leverage the Foursquare location data to solve or execute. Remember that data science problems always  target an audience and are meant to help a group of stakeholders solve a  problem, so make sure that you explicitly describe your audience and  why they would care about your problem.

This submission will eventually become your **Introduction / Business Problem**  section in your final report. So I recommend that you push the report  (having your Introduction/Business Problem section only for now) to your  Github repository and submit a link to it.

## Part 2

Describe the data that you will be  using to solve the problem or execute your idea. Remember that you will  need to use the Foursquare location data to solve the problem or execute  your idea. You can absolutely use other datasets in combination with  the Foursquare location data. So make sure that you provide adequate  explanation and discussion, with examples, of the data that you will be  using, even if it is only Foursquare location data.

This submission will eventually become your **Data** section in your final report. So I recommend that you push the report (having your **Data** section) to your Github repository and submit a link to it.



# Section 1: Introduction

In this section I will clearly define the idea of my choosing, where I leverage the Foursquare location data to solve the imagined business opportunity. 

## Background

There are 100's, maybe even 1000's, of travel sites on the Internet, including [FourSquare](www.foursquare.com), that will tell you all about places to go, things to see, restaurants to eat at, bars to drink in, nightclubs to part the night away in and then where to go in the morning to get breakfast and a strong coffee. The problems with these sites is that they are one dimensional. If you want to find out all this information about a city you plan to visit next month, **you** have to do the hard work. Also, just because a venue is the hottest place to go for a night out does not always mean that the unwitting tourist should just ramble in unprepared. The areas surrounding this new venue might be riddled with crime including muggings, car theft and assault, for example. Approach the venue from any direction other than from the north and you could be putting your life in danger. This is when my idea comes in.

Imagine the following scenario:

1. You like to plan ahead and always review your options and make your choices about where you will visit and eat up front before you travel. 
2. You are flying to Chicogo for a Data Science Conference.
3. You arrive in Chicago the day the conference starts but you've managed to convince your boss to delay your return by a few days giving you time to explore.
4. But you know no one in Chicago to show you around to all the top sites and to bring you to the best restaurants.
5. Also the last time you went to a conference you were mugged and had you passport. money and credit cards stolen so you're now nervous of going somewhere without first researching the venue and the surrounding area.
6. The conference is next week and you don't have time to do all the research you'd like.

**What do you do ... ?**

## Project Idea

My idea for the Capstone Project is to show that when driven by venue and location data from FourSquare, backed up with open source crime data, that it is possible to present the cautious and nervous traveller with a list of attractions to visit supplementd with a graphics showing the occurance of crime in the region of the venue.

A high level approach is as follows:

1. The travellers decides on a city location [in this case Chicago]
2. The ForeSquare website is scrapped for the top venues in the city
3. From this list of top venues the list is augmented with additional grographical data
4. Using this additional geographical data the top nearby restaurents are selects
5. The historical crime within a predetermined distance of all venues are obtained
6. A map is presented to the to the traveller showing the selected venues and crime statistics of the area. 

Now that the conference is over the Data Sceintist can explore Chigago and feel much safer.



# Section 2: Methodology

In this section, I will describe the data used to solve the problem as described previously. 

As noted below in the Further Development Section, it is possible to attempt quite complex and sophisticated scenarios when approaching this problem. However, given the size of the project and for simplicity only the following scenario will be addressed:

1. Query the FourSqaure website for the top sites in Chicago
2. Use the FourSquare API to get supplemental geographical data about the top sites
3. Use the FourSquare API to get top restaurent recommendations closest to each of the top site 
4. Use open source Chicago Crime data to provide the user with additional crime data

### Top Sites from FourSquare Website

Although FourSquare provides a comprehensive API, one of the things that API does not easily support is a mechanism to directly extract the top N sites / venues in a given city. This data, however, is easily available directly from the FourSquare Website. To do this simply go to www.foursquare.com, enter the city of your choise and select Top Picks from _I'm Looking For_ selection field.

Using BeautifulSoup and Requests the results of the Top Pick for Chicago was retrieved. A sample venue is shown below:

```html
<div class="venueDetails">
    <div class="venueName">
        <h2>
    <a href="/v/millennium-park/42b75880f964a52090251fe3" target="_blank">Millennium Park
    </a>
</h2>
    </div>
    <div class="venueMeta">
        <div class="venueScore positive" style="background-color: #00B551;" title="9.7/10 - People like this place">9.7</div>
        <div class="venueAddressData">
            <div class="venueAddress">201 E Randolph St (btwn Columbus Dr &amp; Michigan Ave), Chicago</div>
            <div class="venueData"><span class="venueDataItem"><span class="categoryName">Park</span><span class="delim"> • </span></span>
            </div>
        </div>
    </div>
</div>
```

From this HTML the following data can be extracted:

* Venue Name
* Venue Score
* Venue Category 
* Venue HREF
* Venue ID [Extracted from the HREF]

A sample of the extracted data is given below:

| id                       | score | category      | name                         | href                                              |
| ------------------------ | ----- | ------------- | ---------------------------- | ------------------------------------------------- |
| 42b75880f964a52090251fe3 | 9.7   | Park          | Millennium Park              | /v/millennium-park/42b75880f964a52090251fe3       |
| 4b9511c7f964a520f38d34e3 | 9.6   | Trail         | Chicago Lakefront Trail      | /v/chicago-lakefront-trail/4b9511c7f964a520f38... |
| 49e9ef74f964a52011661fe3 | 9.6   | Art Museum    | The Art Institute of Chicago | /v/the-art-institute-of-chicago/49e9ef74f964a5... |
| 4f2a0d0ae4b0837d0c4c2bc3 | 9.6   | Deli / Bodega | Publican Quality Meats       | /v/publican-quality-meats/4f2a0d0ae4b0837d0c4c... |
| 4aa05f40f964a520643f20e3 | 9.6   | Theater       | The Chicago Theatre          | /v/the-chicago-theatre/4aa05f40f964a520643f20e3   |

###Supplemental Geographical Data

Using the `id` field extracted from the HTML it is then possible to get further supplemental geographical details about each of the top sites from FourSquare using the following sample API call:

```python
# Get the properly formatted address and the latitude and longitude
url = 'https://api.foursquare.com/v2/venues/{}?client_id={}&client_secret={}&v={}'.format(
    venue_id, 
    cfg['client_id'],
    cfg['client_secret'],
    cfg['version'])
    
result = requests.get(url).json()
result['response']['venue']['location']
```
The requests returns a JSON object which can then be queried for the details required. The last line in the sample code above returns the following sample JSON:

```json
{  
   'city':'Chicago',
   'lng':-87.62323915831546,
   'crossStreet':'btwn Columbus Dr & Michigan Ave',
   'neighborhood':'The Loop',
   'postalCode':'60601',
   'cc':'US',
   'formattedAddress':[  
      '201 E Randolph St (btwn Columbus Dr & Michigan Ave)',
      'Chicago, IL 60601',
      'United States'
   ],
   'state':'IL',
   'address':'201 E Randolph St',
   'lat':41.8826616030636,
   'country':'United States'
}
```

From this the following attributes are extracted:

* Venue Address
* Venue Postalcode
* Venue City
* Venue Latitude
* Venue Longitude

### Final FourSquare Top Sites Data

A sample of the final FourSquare Top Sites data is shown below:

| id                       | score | category      | name                         | address                 | postalcode | city    | href                                              | latitude  | longitude  |
| ------------------------ | ----- | ------------- | ---------------------------- | ----------------------- | ---------- | ------- | ------------------------------------------------- | --------- | ---------- |
| 42b75880f964a52090251fe3 | 9.7   | Park          | Millennium Park              | 201 E Randolph St       | 60601      | Chicago | /v/millennium-park/42b75880f964a52090251fe3       | 41.882662 | -87.623239 |
| 4b9511c7f964a520f38d34e3 | 9.6   | Trail         | Chicago Lakefront Trail      | Lake Michigan Lakefront | 60611      | Chicago | /v/chicago-lakefront-trail/4b9511c7f964a520f38... | 41.967053 | -87.646909 |
| 49e9ef74f964a52011661fe3 | 9.6   | Art Museum    | The Art Institute of Chicago | 111 S Michigan Ave      | 60603      | Chicago | /v/the-art-institute-of-chicago/49e9ef74f964a5... | 41.879665 | -87.623630 |
| 4f2a0d0ae4b0837d0c4c2bc3 | 9.6   | Deli / Bodega | Publican Quality Meats       | 825 W Fulton Market     | 60607      | Chicago | /v/publican-quality-meats/4f2a0d0ae4b0837d0c4c... | 41.886642 | -87.648718 |
| 4aa05f40f964a520643f20e3 | 9.6   | Theater       | The Chicago Theatre          | 175 N State St          | 60601      | Chicago | /v/the-chicago-theatre/4aa05f40f964a520643f20e3   | 41.885578 | -87.627286 |

We are now ready to get the top 5 restaurents within 250 meters of each of the top sites.

### FourSquare Restaurent Recommendation Data



```json
  {  
     'referralId':'v-1538424503',
     'hasPerk':False,
     'venuePage':{  
        'id':'135548807'
     },
     'id':'55669b9b498ee34e5249ea61',
     'location':{  
        'labeledLatLngs':[  
           {  
              'label':'display',
              'lng':-87.62460021795313,
              'lat':41.88169538551873
           }
        ],
        'crossStreet':'btwn E Madison & E Monroe St',
        'postalCode':'60603',
        'formattedAddress':[  
           '12 S Michigan Ave (btwn E Madison & E Monroe St)',
           'Chicago, IL 60603',
           'United States'
        ],
        'distance':155,
        'city':'Chicago',
        'lng':-87.62460021795313,
        'neighborhood':'The Loop',
        'cc':'US',
        'state':'IL',
        'address':'12 S Michigan Ave',
        'lat':41.88169538551873,
        'country':'United States'
     },
     'name':"Cindy's",
     'categories':[  
        {  
           'pluralName':'Gastropubs',
           'id':'4bf58dd8d48988d155941735',
           'name':'Gastropub',
           'primary':True,
           'icon':{  
              'prefix':'https://ss3.4sqi.net/img/categories_v2/food/gastropub_',
              'suffix':'.png'
           },
           'shortName':'Gastropub'
        }
     ]
  },
```




###Chicago Crime Data

This dataset can be download from the [Chicago Data Portal](https://data.cityofchicago.org/) and reflects reported incidents of crime (with the exception of murders where data exists for each victim) that occurred in the City of Chicago in the last year, minus the most recent seven days. A full desription of the data is available on the site.

Data is extracted from the Chicago Police Department's CLEAR (Citizen Law Enforcement Analysis and Reporting) system. In order to protect the privacy of crime victims, addresses are shown at the block level only and specific locations are not identified.

| Column Name           | Type        | Description                                                  |
| :-------------------- | :---------- | :----------------------------------------------------------- |
| CASE#                 | Plain Text  | The Chicago Police Department RD Number (Records Division Number), which is unique to the incident. |
| DATE OF OCCURRENCE    | Date & Time | Date when the incident occurred. this is sometimes a best estimate. |
| BLOCK                 | Plain Text  | The partially redacted address where the incident occurred, placing it on the same block as the actual address. |
| IUCR                  | Plain Text  | The Illinois Unifrom Crime Reporting code. This is directly linked to the Primary Type and Description. See the list of IUCR codes at https://data.cityofchicago.org/d/c7ck-438e. |
| PRIMARY DESCRIPTION   | Plain Text  | The primary description of the IUCR code.                    |
| SECONDARY DESCRIPTION | Plain Text  | The secondary description of the IUCR code, a subcategory of the primary description. |
| LOCATION DESCRIPTION  | Plain Text  | Description of the location where the incident occurred.     |
| ARREST                | Plain Text  | Indicates whether an arrest was made.                        |
| DOMESTIC              | Plain Text  | Indicates whether the incident was domestic-related as defined by the Illinois Domestic Violence Act. |
| BEAT                  | Plain Text  | Indicates the beat where the incident occurred. A beat is the smallest police geographic area – each beat has a dedicated police beat car. Three to five beats make up a police sector, and three sectors make up a police district. The Chicago Police Department has 22 police districts. See the beats at https://data.cityofchicago.org/d/aerh-rz74. |
| WARD                  | Number      | The ward (City Council district) where the incident occurred. See the wards at https://data.cityofchicago.org/d/sp34-6z76. |
| FBI CD                | Plain Text  | Indicates the crime classification as outlined in the FBI's National Incident-Based Reporting System (NIBRS). See the Chicago Police Department listing of these classifications at http://gis.chicagopolice.org/clearmap_crime_sums/crime_types.html. |
| X COORDINATE          | Plain Text  | The x coordinate of the location where the incident occurred in State Plane Illinois East NAD 1983 projection. This location is shifted from the actual location for partial redaction but falls on the same block. |
| Y COORDINATE          | Plain Text  | The y coordinate of the location where the incident occurred in State Plane Illinois East NAD 1983 projection. This location is shifted from the actual location for partial redaction but falls on the same block. |
| LATITUDE              | Number      | The latitude of the location where the incident occurred. This location is shifted from the actual location for partial redaction but falls on the same block. |
| LONGITUDE             | Number      | The longitude of the location where the incident occurred. This location is shifted from the actual location for partial redaction but falls on the same block. |
| LOCATION              | Location    | The location where the incident occurred in a format that allows for creation of maps and other geographic operations on this data portal. This location is shifted from the actual location for partial redaction but falls on the same block. |

Not all of the attributes are required so on the following data was imported:

* Date of Occurance
* Block
* Primary Description
* Ward
* Latitude
* Longitude

A sample of the imported data is shown.

| CASE#    | DATE  OF OCCURRENCE    | BLOCK                | PRIMARY DESCRIPTION | WARD | LATITUDE  | LONGITUDE  |
| -------- | ---------------------- | -------------------- | ------------------- | ---- | --------- | ---------- |
| JB241987 | 04/28/2018 10:05:00 PM | 009XX N LONG AVE     | NARCOTICS           | 37.0 | 41.897895 | -87.760744 |
| JB241350 | 04/28/2018 08:00:00 AM | 008XX E 53RD ST      | CRIMINAL DAMAGE     | 5.0  | 41.798635 | -87.604823 |
| JB245397 | 04/28/2018 09:00:00 AM | 062XX S MICHIGAN AVE | THEFT               | 20.0 | 41.780946 | -87.621995 |
| JB241444 | 04/28/2018 12:15:00 PM | 046XX N ELSTON AVE   | THEFT               | 39.0 | 41.965404 | -87.736202 |
| JB241667 | 04/28/2018 04:28:00 PM | 022XX S KENNETH AVE  | ARSON               | 22.0 | 41.850673 | -87.735597 |

This data was then processed as follows:

1. Move September 2017 dates to September 2018
   The extract of data used was taken mid September which meant that there was half a months data for September 2017 and half a months data for september 2018. These were combined to create a single month.
2. Clean up the column names:
   1. Strip leading & trailing whitespace
   2. Replace multiple spaces with a single space
   3. Remove # characters
   4. Replace spaces with _
   5. Convert to lowercase
3. Change the date of occurance field to a date / time object
4. Add new columns for:
   1. Hour
   2. Day
   3. Month
   4. Year
   5. etc.
5. Split Block into zip_code and street
6. Verify that all rows have valid data

#### Data Visualisation

To get a better understanding of the data we will now visualise it. The number of crimes per month, day and hour were calculated:

![image-20181001162714277](./capstone_images/cases_month.jpg)

 ![image-20181001162714277](./capstone_images/cases_day.jpg)

![image-20181001162714277](./capstone_images/cases_hour.jpg)

Unsuprisingly there little obvious variation in the number of crimes committed per month other than an apparent drop-off in February. There is a small increase in crime reported at the weekend, Saturday and
Sunday, but nothing that couldbe considered significant. There is an expected fall-off in reported crime rates after midnight and before eight in the morning.



# Further Development

Best time to visit each venue

Suggestions for morning, afternoon, evening and night time

Daily itineraries

Route planning and transportation

Time lapse of the crime in the area of the venue

Favourite dining preferences could be used to choose the restaurants

