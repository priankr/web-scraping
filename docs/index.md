# Web Scraping

Web scraping is used for extracting data from websites. 

The process begins by inspecting the website's html and identifying the elements that contain the data we are interested in. The data can be located in different types of html elements such as div, span etc. and depending on how it is organized we approach it differently.  

- Grouped Data (contact details, product details etc.) will be stored in similar himl elements. We will have to loop over these similar elements and extract the desired information. 
- Ungrouped Data may be stored in an html element that is shared across other pieces of data. If there is no unique class or id for the desired data, we can store all the instances of this html element in a list or data frame and use indexing to access the desired information.

#### Below are examples of web scraping on different types of data.


```python
import requests
import pandas as pd
from bs4 import BeautifulSoup
import requests
import re
```

## Search Result Data

### Indeed Job Postings


```python
#Indeed search for Business Analyst roles in Toronto
page = requests.get("https://ca.indeed.com/jobs?q=Business+Analyst&l=Toronto%2C+ON")

#Checking to see what response code we get from the page we requested
#The HTTP 200 OK success status response code indicates that the request has succeeded
page
```




    <Response [200]>




```python
#We can use the BeautifulSoup library to parse this document
soup = BeautifulSoup(page.content, 'html.parser')
```


```python
#We can use the find_all method to search for items by class or by id
jobs_html = soup.find_all('td', class_="resultContent")

#Creating an empty dataframe with a column to store all values we are interested in
jobs_df = pd.DataFrame(columns=['Job Title', 'Company Name', 'Location'])

for jobs in jobs_html:
    #Some jobs have a label called "new" to indicate a new posting. We need to check for this to identify which span element contains the job title
    if (jobs.find_all('span')[0]).text.strip() == "new":
        job_title_html = jobs.find_all('span')[1]
    else:
        job_title_html = jobs.find_all('span')[0]
    
    company_name_html = jobs.find('span', class_="companyName")
    location_html = jobs.find('div', class_="companyLocation")
   
    job_title = job_title_html.text.strip()
    company_name = company_name_html.text.strip()
    location = location_html.text.strip()
    
    df = {'Job Title':job_title, 'Company Name':company_name, 'Location':location}
    
    jobs_df = jobs_df.append(df, ignore_index = True)

jobs_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Job Title</th>
      <th>Company Name</th>
      <th>Location</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Category Business Analyst</td>
      <td>Canadian Tire</td>
      <td>Toronto, ON</td>
    </tr>
    <tr>
      <th>1</th>
      <td>business management analyst</td>
      <td>MSG GLOBAL SOLUTIONS CANADA INC</td>
      <td>Toronto, ON</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Business Analyst</td>
      <td>IG Wealth Management</td>
      <td>Toronto, ON</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Business Analyst</td>
      <td>Accenture</td>
      <td>Toronto, ON</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Business Analyst</td>
      <td>Orion Health</td>
      <td>Toronto, ON</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Intern, Business Analyst</td>
      <td>Equitable Bank</td>
      <td>Toronto, ON</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Business Analyst / QA</td>
      <td>MEDCAN</td>
      <td>Toronto, ON</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Business Analyst Intern (4 months)</td>
      <td>IBM Canada</td>
      <td>Toronto, ON+1 location</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Business Analyst</td>
      <td>ThreePDS Inc</td>
      <td>Toronto, ON</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Analyst, Business Intelligence</td>
      <td>Bell Canada</td>
      <td>Don Mills, ON+1 location</td>
    </tr>
    <tr>
      <th>10</th>
      <td>business management analyst</td>
      <td>ShoreWise Consulting LLC</td>
      <td>Mississauga, ON•Remote</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Junior Business Analyst (6 month contract)</td>
      <td>H&amp;M</td>
      <td>Toronto, ON</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Junior Business Operations Analyst, Ebooks</td>
      <td>Rakuten Kobo Inc.</td>
      <td>Toronto, ON</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Jr. / Int. Business Analyst</td>
      <td>JLL</td>
      <td>Toronto, ON</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Research and Business Analyst (Business Planni...</td>
      <td>Metrolinx</td>
      <td>Toronto, ON</td>
    </tr>
  </tbody>
</table>
</div>



### Yelp Listings


```python
#Yelp search for Coffee Shops in Los Angeles
page = requests.get("https://www.yelp.com/search?find_desc=Coffee+Shop&find_loc=Los+Angeles%2C+CA")

#Checking to see what response code we get from the page we requested
#The HTTP 200 OK success status response code indicates that the request has succeeded
page
```




    <Response [200]>




```python
#We can use the BeautifulSoup library to parse this document
soup = BeautifulSoup(page.content, 'html.parser')
```


```python
#We can use the find_all method to search for items by class or by id
shops_html = soup.find_all('div', class_="arrange-unit__09f24__eFC_S arrange-unit-fill__09f24__1bMmp border-color--default__09f24__3Epto")

#Creating an empty dataframe with a column to store all values we are interested in
shops_df = pd.DataFrame(columns=['Coffee Shop Name', 'Number of Reviews', 'Location'])


for shop in shops_html:
    #Extracting the desired html elements
    shop_name_html = shop.find('a', class_="css-og60gk")
    number_of_reviews_html = shop.find('span', class_="reviewCount__09f24__3GsGY css-e81eai")
    location_html = (shop.find('p', class_="css-1j7sdmt"))
    
    #We get an issue with some Nonetype objects appearing when we select the html elements. 
    #The if statement below allows us to avoid any Nonetype values 
    if None not in (shop_name_html, number_of_reviews_html, location_html):
       
        
        #Extracting the text from the html elements
        shop_name = shop_name_html.text.strip()
        number_of_reviews = number_of_reviews_html.text.strip()
        
        #We need to use .find() a second time to access the <span> tag within the <p> tag
        #The class "css-e81eai" is shared by another element on the page. In those cases the class="some_class css-e81eai", but we are not interested in those values
        #We use regular expressions to identify if there are additional classes present
        # '\\b class_name' allows us to check "css-e81eai" has a whitespace before it indicating another class'
        location_exact = location_html.find('span', class_=re.compile("\\b css-e81eai"))
       
        #Again, we get an issue with some Nonetype objects appearing when we select the html elements. 
        #The if statement below allows us to avoid any Nonetype values 
        if location_exact is None:
            location = location_html.find('span', class_=("css-e81eai")).text.strip()
        else:
            #For Coffee Shops do not have an area listed so for those we set a default value
            location =  "Area Not Listed"
        
        df = {'Coffee Shop Name':shop_name, 'Number of Reviews':number_of_reviews, 'Location':location}
        
        shops_df = shops_df.append(df, ignore_index = True)

shops_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Coffee Shop Name</th>
      <th>Number of Reviews</th>
      <th>Location</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Alibi Coffee</td>
      <td>157</td>
      <td>Larchmont</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Coffee Connection</td>
      <td>627</td>
      <td>Mar Vista</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Alchemist Coffee Project</td>
      <td>1165</td>
      <td>Wilshire Center</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Coffee For Sasquatch</td>
      <td>312</td>
      <td>Hancock Park</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Document Coffee Bar</td>
      <td>592</td>
      <td>Koreatown</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Blackwood Coffee Bar</td>
      <td>229</td>
      <td>Hollywood</td>
    </tr>
    <tr>
      <th>6</th>
      <td>The Palm Coffee Bar</td>
      <td>342</td>
      <td>Area Not Listed</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Coffee Signal</td>
      <td>17</td>
      <td>Koreatown</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Maru Coffee</td>
      <td>287</td>
      <td>Los Feliz</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Bungalow 40</td>
      <td>87</td>
      <td>Hollywood</td>
    </tr>
  </tbody>
</table>
</div>



### Yelp Listings: Multiple Pages

We are interested in getting the data for the top 50 coffee shops. The desired information is located on multiple pages.


```python
# Each successive results page follows the pattern below
#'https://www.yelp.com/search?find_desc=Coffee%20Shop&find_loc=Los%20Angeles%2C%20CA&start=10' where 10 indicates coffee shops ranked 11-20
# Therefore a value of 20 would indicate coffee shops ranked 21-30 and so on
url = 'https://www.yelp.com/search?find_desc=Coffee%20Shop&find_loc=Los%20Angeles%2C%20CA&start='

#We are interested in the search results for the following pages so we will loop over a range and add the value we are interested to the string representing the url
#The loop below will allow us to get results for the top 50 coffee shops in Los Angeles
for i in range (1,5):
    url_page = url+str(i*10)
    page = requests.get(url_page)
    
    soup = BeautifulSoup(page.content, 'html.parser')
    
    shops_html = soup.find_all('div', class_="arrange-unit__09f24__eFC_S arrange-unit-fill__09f24__1bMmp border-color--default__09f24__3Epto")
    
    for shop in shops_html:
        #Extracting the desired html elements
        shop_name_html = shop.find('a', class_="css-og60gk")
        number_of_reviews_html = shop.find('span', class_="reviewCount__09f24__3GsGY css-e81eai")
        location_html = (shop.find('p', class_="css-1j7sdmt"))

        #We get an issue with some Nonetype objects appearing when we select the html elements. 
        #The if statement below allows us to avoid any Nonetype values 
        if shop_name_html is not None and number_of_reviews_html is not None and location_html is not None:

            #Extracting the text from the html elements
            shop_name = shop_name_html.text.strip()
            number_of_reviews = number_of_reviews_html.text.strip()
           
            #We need to use .find() a second time to access the <span> tag within the <p> tag
            #The class "css-e81eai" is shared by another element on the page. In those cases the class="some_class css-e81eai", but we are not interested in those values
            #We use regular expressions to identify if there are additonally classes present
            # '\\b class_name' allows us to check "css-e81eai" has a whitespace before it indicating another class is also present
            location_exact = location_html.find('span', class_=re.compile("\\b css-e81eai"))

            #Html elements without class ="some_class css-e81eai" will be Nonetypes. 
            #The if statement below allows us to avoid any Nonetype values 
            if location_exact is None:
                location = location_html.find('span', class_=("css-e81eai")).text.strip()
            else:
                #Some Coffee Shops do not have an area listed so for those we set a defualt value
                location =  "Area Not Listed"
            
            df = {'Coffee Shop Name':shop_name, 'Number of Reviews':number_of_reviews, 'Location':location}

            shops_df = shops_df.append(df, ignore_index = True)

shops_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Coffee Shop Name</th>
      <th>Number of Reviews</th>
      <th>Location</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Alibi Coffee</td>
      <td>157</td>
      <td>Larchmont</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Coffee Connection</td>
      <td>627</td>
      <td>Mar Vista</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Alchemist Coffee Project</td>
      <td>1165</td>
      <td>Wilshire Center</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Coffee For Sasquatch</td>
      <td>312</td>
      <td>Hancock Park</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Document Coffee Bar</td>
      <td>592</td>
      <td>Koreatown</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Blackwood Coffee Bar</td>
      <td>229</td>
      <td>Hollywood</td>
    </tr>
    <tr>
      <th>6</th>
      <td>The Palm Coffee Bar</td>
      <td>342</td>
      <td>Area Not Listed</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Coffee Signal</td>
      <td>17</td>
      <td>Koreatown</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Maru Coffee</td>
      <td>287</td>
      <td>Los Feliz</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Bungalow 40</td>
      <td>87</td>
      <td>Hollywood</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Cafe De Mama</td>
      <td>92</td>
      <td>Harvard Heights</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Blackwood Coffee Bar</td>
      <td>229</td>
      <td>Hollywood</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Boxx Coffee Roasters</td>
      <td>42</td>
      <td>Arts District</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Maru Coffee</td>
      <td>287</td>
      <td>Los Feliz</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Liberation Coffee House</td>
      <td>21</td>
      <td>Hollywood</td>
    </tr>
    <tr>
      <th>15</th>
      <td>CAFE/5</td>
      <td>39</td>
      <td>Jefferson Park</td>
    </tr>
    <tr>
      <th>16</th>
      <td>The Palm Coffee Bar</td>
      <td>342</td>
      <td>Area Not Listed</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Balcony Coffee and Tea</td>
      <td>385</td>
      <td>East Hollywood</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Fratelli Cafe</td>
      <td>1672</td>
      <td>Fairfax</td>
    </tr>
    <tr>
      <th>19</th>
      <td>I coffee bar</td>
      <td>60</td>
      <td>Koreatown</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Stereoscope Coffee Company</td>
      <td>48</td>
      <td>Echo Park</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Balcony Coffee and Tea</td>
      <td>385</td>
      <td>East Hollywood</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Awesome Coffee</td>
      <td>649</td>
      <td>Koreatown</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Stella Coffee Beverly Hills</td>
      <td>81</td>
      <td>Carthay</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Liberation Coffee House</td>
      <td>21</td>
      <td>Hollywood</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Sharp Specialty Coffee</td>
      <td>231</td>
      <td>Wilshire Center</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Groundwork Coffee Co.</td>
      <td>613</td>
      <td>Hollywood</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Bohemia</td>
      <td>239</td>
      <td>Hollywood</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Honey &amp; Bacon Coffee House</td>
      <td>61</td>
      <td>Larchmont</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Fratelli Cafe</td>
      <td>1672</td>
      <td>Fairfax</td>
    </tr>
    <tr>
      <th>30</th>
      <td>The Coffee Company</td>
      <td>1829</td>
      <td>Westchester</td>
    </tr>
    <tr>
      <th>31</th>
      <td>Coffee and Plants</td>
      <td>349</td>
      <td>Area Not Listed</td>
    </tr>
    <tr>
      <th>32</th>
      <td>Bolt</td>
      <td>429</td>
      <td>Hollywood</td>
    </tr>
    <tr>
      <th>33</th>
      <td>Bluestone Lane</td>
      <td>164</td>
      <td>Hancock Park</td>
    </tr>
    <tr>
      <th>34</th>
      <td>Coffee Coffee</td>
      <td>138</td>
      <td>Area Not Listed</td>
    </tr>
    <tr>
      <th>35</th>
      <td>Coffee Commissary</td>
      <td>738</td>
      <td>Beverly Grove</td>
    </tr>
    <tr>
      <th>36</th>
      <td>Alibi Coffee Co.</td>
      <td>146</td>
      <td>Harvard Heights</td>
    </tr>
    <tr>
      <th>37</th>
      <td>Rocketship Coffee</td>
      <td>29</td>
      <td>Fairfax</td>
    </tr>
    <tr>
      <th>38</th>
      <td>6xs Coffee</td>
      <td>93</td>
      <td>Koreatown</td>
    </tr>
    <tr>
      <th>39</th>
      <td>Coffee Dose</td>
      <td>125</td>
      <td>Beverly Grove</td>
    </tr>
    <tr>
      <th>40</th>
      <td>Haute Mess LA</td>
      <td>79</td>
      <td>Fairfax</td>
    </tr>
    <tr>
      <th>41</th>
      <td>Tilt Coffee Bar</td>
      <td>488</td>
      <td>Downtown</td>
    </tr>
    <tr>
      <th>42</th>
      <td>La La Land Kind Cafe</td>
      <td>105</td>
      <td>Area Not Listed</td>
    </tr>
    <tr>
      <th>43</th>
      <td>Verve Coffee Roasters</td>
      <td>411</td>
      <td>Area Not Listed</td>
    </tr>
    <tr>
      <th>44</th>
      <td>Intelligentsia Coffee</td>
      <td>1727</td>
      <td>Silver Lake</td>
    </tr>
    <tr>
      <th>45</th>
      <td>Bricks &amp; Scones</td>
      <td>1358</td>
      <td>Larchmont</td>
    </tr>
    <tr>
      <th>46</th>
      <td>Kumquat Coffee</td>
      <td>154</td>
      <td>Highland Park</td>
    </tr>
    <tr>
      <th>47</th>
      <td>Dam Good Coffee</td>
      <td>2</td>
      <td>Mid-Wilshire</td>
    </tr>
    <tr>
      <th>48</th>
      <td>Intelligentsia Coffee</td>
      <td>46</td>
      <td>Hollywood</td>
    </tr>
    <tr>
      <th>49</th>
      <td>Cafe Nemo</td>
      <td>8</td>
      <td>Arlington Heights</td>
    </tr>
  </tbody>
</table>
</div>



## Tabular Data

### IMDb Box Office Charts 


```python
#IMDb page with today's box office charts
page = requests.get("https://www.imdb.com/chart/boxoffice/")

#Checking to see what response code we get from the page we requested
#The HTTP 200 OK success status response code indicates that the request has succeeded
page
```




    <Response [200]>




```python
#We can use the BeautifulSoup library to parse this document
soup = BeautifulSoup(page.content, 'html.parser')
```


```python
#We can use the find_all method to search for items by class or by id
imdb_html = soup.find_all('tr')

#Creating an empty dataframe with a column to store all values we are interested in
imdb_df = pd.DataFrame(columns=['Title', 'Weekend Earnings', 'Gross Earnings', 'Weeks in Theaters'])

for movie in imdb_html:
    #Extracting the desired html elements
    title_html = movie.find('td', class_="titleColumn")
    weekend_html = movie.find('td', class_="ratingColumn")
    gross_html = movie.find('span', class_="secondaryInfo")
    weeks_html = movie.find('td', class_="weeksColumn")
    
    #We get an issue with some Nonetype objects appearing when we select the html elements. 
    #The if statement below allows us to avoid any Nonetype values 
    if None not in (title_html, weekend_html, gross_html,weeks_html):
        
        #Extracting the text from the html elements
        title = title_html.text.strip()
        weekend = weekend_html.text.strip()
        gross =  gross_html.text.strip()
        weeks = weeks_html.text.strip()

        df = {'Title':title, 'Weekend Earnings':weekend, 'Gross Earnings':gross, 'Weeks in Theaters':weeks}

        imdb_df = imdb_df.append(df, ignore_index = True)

imdb_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Title</th>
      <th>Weekend Earnings</th>
      <th>Gross Earnings</th>
      <th>Weeks in Theaters</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Venom: Let There Be Carnage</td>
      <td>$90.0M</td>
      <td>$90.0M</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>The Addams Family 2</td>
      <td>$17.3M</td>
      <td>$17.3M</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Shang-Chi and the Legend of the Ten Rings</td>
      <td>$6.1M</td>
      <td>$206.2M</td>
      <td>5</td>
    </tr>
    <tr>
      <th>3</th>
      <td>The Many Saints of Newark</td>
      <td>$4.7M</td>
      <td>$4.7M</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Dear Evan Hansen</td>
      <td>$2.5M</td>
      <td>$11.8M</td>
      <td>2</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Free Guy</td>
      <td>$2.3M</td>
      <td>$117.6M</td>
      <td>8</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Candyman</td>
      <td>$1.3M</td>
      <td>$58.9M</td>
      <td>6</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Jungle Cruise</td>
      <td>$703K</td>
      <td>$116.1M</td>
      <td>10</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Chal Mera Putt 3</td>
      <td>$644K</td>
      <td>$644K</td>
      <td>1</td>
    </tr>
    <tr>
      <th>9</th>
      <td>The Jesus Music</td>
      <td>$549K</td>
      <td>$549K</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



## Text Data

### Wikipedia Page


```python
#Wikipedia page on Televisions
page = requests.get("https://en.wikipedia.org/wiki/Television")

#Checking to see what response code we get from the page we requested
#The HTTP 200 OK success status response code indicates that the request has succeeded
page
```




    <Response [200]>




```python
#We can use the BeautifulSoup library to parse this document
soup = BeautifulSoup(page.content, 'html.parser')
```


```python
#We can use the find_all method to search for items by class or by id
tvs_html = soup.find_all('p')

#Creating an empty list to store all values we are interested in
tv_wiki = []

for tv in tvs_html:
    tv_wiki.append(tv.text.strip())    

#Removing any empty strings
tv_wiki.remove('')

#See the first three paragraphs
tv_wiki[0:3]
```




    ['Television, sometimes shortened to TV or telly, is a telecommunication medium used for transmitting moving images in monochrome (black and white), or in color, and in two or three dimensions and sound. The term can refer to a television set, a television show, or the medium of television transmission. Television is a mass medium for advertising, entertainment, news, and sports.',
     'Television became available in crude experimental forms in the late 1920s, but it would still be several years before the new technology would be marketed to consumers. After World War II, an improved form of black-and-white television broadcasting became popular in the United Kingdom and United States, and television sets became commonplace in homes, businesses, and institutions. During the 1950s, television was the primary medium for influencing public opinion.[1] In the mid-1960s, color broadcasting was introduced in the U.S. and most other developed countries. The availability of various types of archival storage media such as Betamax and VHS tapes, high-capacity hard disk drives, DVDs, flash drives, high-definition Blu-ray Discs, and cloud digital video recorders has enabled viewers to watch pre-recorded material—such as movies—at home on their own time schedule. For many reasons, especially the convenience of remote retrieval, the storage of television and video programming now also occurs on the cloud (such as the video on demand service by Netflix). At the end of the first decade of the 2000s, digital television transmissions greatly increased in popularity. Another development was the move from standard-definition television (SDTV) (576i, with 576 interlaced lines of resolution and 480i) to high-definition television (HDTV), which provides a resolution that is substantially higher. HDTV may be transmitted in different formats: 1080p, 1080i and 720p. Since 2010, with the invention of smart television, Internet television has increased the availability of television programs and movies via the Internet through streaming video services such as Netflix, Amazon Video, iPlayer and Hulu.',
     "In 2013, 79% of the world's households owned a television set.[2] The replacement of earlier bulky, high-voltage cathode ray tube (CRT) screen displays with compact, energy-efficient, flat-panel alternative technologies such as LCDs (both fluorescent-backlit and LED), OLED displays, and plasma displays was a hardware revolution that began with computer monitors in the late 1990s. Most television sets sold in the 2000s were flat-panel, mainly LEDs. Major manufacturers announced the discontinuation of CRT, DLP, plasma, and even fluorescent-backlit LCDs by the mid-2010s.[3][4] In the near future, LEDs are expected to be gradually replaced by OLEDs.[5] Also, major manufacturers have announced that they will increasingly produce smart TVs in the mid-2010s.[6][7][8] Smart TVs with integrated Internet and Web 2.0 functions became the dominant form of television by the late 2010s.[9]"]


