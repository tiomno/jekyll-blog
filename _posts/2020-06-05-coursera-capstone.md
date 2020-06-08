---
layout: post
title: Family-friendly Neighbourhoods in Melbourne, AU
permalink: /coursera-capstone/
---

![_config.yml]({{ site.baseurl }}/images/aerial-shot-of-city-2097616.jpg)
>Photo by Tiff Ng from Pexels

## Introduction

> Disclaimer: Melbourne city does have programs to make it <a href="https://whatson.melbourne.vic.gov.au/whatson/forthekids/pages/forthekids.aspx">family-friendly<a>. However, this post is not an actual request of the local government of Melbourne. It was made up as an exercise for the Course Course <a href="https://www.coursera.org/learn/applied-data-science-capstone/home/welcome">Applied Data Science Capstone</a>

The concept of family in Australia is a pillar of society. The success of this country as one of the most liveable in the world depends on how each household thrives by achieving a safe and prosper lifestyle. Families raise and educate the new generations that will take over the future development of the country. Melbourne is the most liveable city of Australia, and the local government is not thinking of losing this title. No wonders one of its programs is to focus on developing the outdoor activities for families with children under 15 years. The final objective is to improve local communities' public and family-friendly areas. This study pretends to help in the finding of the suburbs that could need more resources to meet the requirements of this program.

Merging demographics from the population than comprise each suburb with family-firndly places, we can build a map that show the distribution of the best and worst suburbs in term of suitability for families with small children. Thus, for this analysis we can use data from the Australian Bureau of Statistics (<a href="https://www.abs.gov.au/">ABS</a>) to compile information about families per suburb and the API of <a href="https://foursquare.com/">Foursquare</a> to gather the venues of each locality.

## Data Description

The ABS publishes datasets from the lastest census in Australia. One if its tables has aggregations of family composition and other statistics grouped by the suburbs of the State of Victoria in Australia. It comes with several columns, but we will use the three we need to extract information about families. These columns gives us the number of couple families with children under 15 years `CF_ChU15_a_Total_F`, one-parent families with children under 15 years `OPF_ChU15_a_Total_F` and the total of families `Total_F`.

Each row of this table represents the grouping of data per suburb with an ID of 5 digits named SSC (State Suburb Code) which is assigned by the ABS.

An example of the data for this table looks like this:

| SSC_CODE_2016 | ...  | CF_ChU15_a_Total_F | ...  | OPF_ChU15_a_Total_F | ... | Total_F | ...  |
|---------------|------|--------------------|------|---------------------|-----|---------|------|
| ...           | ...  | ...                | ...  | ...                 | ... | ...     | ...  |
| SSC20002      | ...  | 353                | ...  | 58                  | ... | 1886    | ...  |
| SSC20003      | ...  | 356                | ...  | 39                  | ... | 1034    | ...  |
| ...           | ...  | ...                | ...  | ...                 | ... | ...     | ...  |

This informatio will help us to get the ratio of families with children per suburb with the following equation:

> (`CF_ChU15_a_Total_F` + `OPF_ChU15_a_Total_F`) / `Total_F`

The Victorian government also provides another table `CG_SSC_2016_SA4_2016.csv` with the correspondence between SSC and SA4. The latter is a large statistic area defined by the ABS. Those SA4 areas with 'Melbourne' as part of its name (`SA4_NAME_2016`) represent the zones of the city with their corresponding suburbs. With this dataset, we can filter out from the previous one any suburb out of the Melbourne city. This table looks like this:

| SSC_CODE_2016 | SSC_NAME_2016       | SA4_CODE_2016 | SA4_NAME_2016          | RATIO | PERCENTAGE |
|---------------|---------------------|---------------|------------------------|-------|------------|
| 20172         | Bayswater (Vic.)    | 211           | Melbourne - Outer East | 1     | 100        |
| 20173         | Bayswater North     | 211           | Melbourne - Outer East | 1     | 100        |
| 20174         | Beaconsfield (Vic.) | 212           | Melbourne - South East | 1     | 100        |
| 20175         | Beaconsfield Upper  | 212           | Melbourne - South East | 1     | 100        |
| 20176         | Bealiba             | 201           | Ballarat               | 1     | 100        |
| 20177         | Bearii              | 216           | Shepparton             | 1     | 100        |
| 20178         | Bears Lagoon        | 202           | Bendigo                | 1     | 100        |

Now we can go through each suburb and find the venues that are suitable for families with children. To only get those venues we are interested in, we will pass 50 categories chosen from the full list of Foursquare categories <a href="https://developer.foursquare.com/docs/resources/categories">here</a>.

Knowing how many venues per category every suburb of Melbourne has and adding this information to the ratio of families, we can run k-means clustarting starting with 5 clusters until we find the optimum value for k using the elbow method. After we have the best k parameter, we can finally get that many clusters of suburbs for the local government to analyse and support depending on the outcome and classification.

## Data Preparation

The first step is to download the demographics data from ABS with the aggregations of families with children in the State of Victoria. With this dataset, we want to obtain only the data for Melbourne and transform the numbers to use only the ratio of families per suburb. <a href="https://pandas.pydata.org/">Pandas</a> is a great Python library that can help us to analyse and transform the data.

The following snippet of code loads this table into a Pandas data frame.

```python
import pandas as pd

df_families = pd.read_csv('https://raw.githubusercontent.com/tiomno/Coursera_Capstone/master/datasets/2016Census_G25_VIC_SSC.csv')
```

And the first 5 rows of the *df_families* data frame is like so:

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
      <th>SSC_CODE_2016</th>
      <th>CF_no_children_F</th>
      <th>CF_no_children_P</th>
      <th>CF_ChU15_a_DSs_a_NdCh_F</th>
      <th>CF_ChU15_a_DSs_a_NdCh_P</th>
      <th>CF_ChU15_a_DSs_a_no_NdCh_F</th>
      <th>CF_ChU15_a_DSs_a_no_NdCh_P</th>
      <th>CF_ChU15_a_no_DSs_a_NdCh_F</th>
      <th>CF_ChU15_a_no_DSs_a_NdCh_P</th>
      <th>CF_ChU15_a_no_DSs_a_no_NdCh_F</th>
      <th>...</th>
      <th>OPF_no_ChU15_no_DSs_a_NdCh_F</th>
      <th>OPF_no_ChU15_no_DSs_a_NdCh_P</th>
      <th>OPF_no_ChU15_a_Total_F</th>
      <th>OPF_no_ChU15_a_Total_P</th>
      <th>OPF_Total_F</th>
      <th>OPF_Total_P</th>
      <th>Other_family_F</th>
      <th>Other_family_P</th>
      <th>Total_F</th>
      <th>Total_P</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>SSC20001</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>1</td>
      <td>SSC20002</td>
      <td>1164</td>
      <td>2281</td>
      <td>0</td>
      <td>7</td>
      <td>25</td>
      <td>111</td>
      <td>4</td>
      <td>10</td>
      <td>327</td>
      <td>...</td>
      <td>66</td>
      <td>134</td>
      <td>104</td>
      <td>208</td>
      <td>154</td>
      <td>342</td>
      <td>81</td>
      <td>171</td>
      <td>1886</td>
      <td>4539</td>
    </tr>
    <tr>
      <td>2</td>
      <td>SSC20003</td>
      <td>287</td>
      <td>561</td>
      <td>13</td>
      <td>66</td>
      <td>84</td>
      <td>390</td>
      <td>9</td>
      <td>32</td>
      <td>256</td>
      <td>...</td>
      <td>57</td>
      <td>131</td>
      <td>91</td>
      <td>213</td>
      <td>129</td>
      <td>328</td>
      <td>13</td>
      <td>32</td>
      <td>1034</td>
      <td>3282</td>
    </tr>
    <tr>
      <td>3</td>
      <td>SSC20004</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>4</td>
      <td>SSC20005</td>
      <td>21</td>
      <td>44</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>8</td>
      <td>0</td>
      <td>0</td>
      <td>9</td>
      <td>...</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>3</td>
      <td>3</td>
      <td>11</td>
      <td>0</td>
      <td>0</td>
      <td>40</td>
      <td>109</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 47 columns</p>
</div>

Now we can clean the data and get the ratio of families in every suburb:

```python
# Get rid of the prefix 'SSC' for SSC_CODE_2016
df_families['SSC_CODE_2016'] = df_families['SSC_CODE_2016'].str[-5:].astype(int)

# Calculate the ratios of families with children per suburb. Remove any area with no population declared.
df_families.drop(df_families[df_families['Total_F'] == 0].index, inplace=True)
df_families['ratio'] = (
    df_families['CF_ChU15_a_Total_F'] + df_families['OPF_ChU15_a_Total_F']
) / df_families['Total_F']

# Remove unnecessary columns and rename SSC_CODE_2016 to ssc
df_families = df_families[['SSC_CODE_2016', 'ratio']]
df_families.columns = ['ssc', 'ratio']
```

The first 5 rows of the transformed *df_families* data frame looks like so:

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
      <th>ssc</th>
      <th>ratio</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>20002</td>
      <td>0.217922</td>
    </tr>
    <tr>
      <td>2</td>
      <td>20003</td>
      <td>0.382012</td>
    </tr>
    <tr>
      <td>4</td>
      <td>20005</td>
      <td>0.300000</td>
    </tr>
    <tr>
      <td>6</td>
      <td>20007</td>
      <td>0.187500</td>
    </tr>
    <tr>
      <td>7</td>
      <td>20008</td>
      <td>0.200000</td>
    </tr>
  </tbody>
</table>
</div>

Let's load the table that has the correspondence between SSC and SA4 areas

```python
# Obtain the correspondence between ABS larger areas SA4 and pick only the suburbs in and around the CBD of Melbourne
df_correspondence = pd.read_csv('https://raw.githubusercontent.com/tiomno/Coursera_Capstone/master/datasets/CG_SSC_2016_SA4_2016.csv')
```

Which looks like so:

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
      <th>SSC_CODE_2016</th>
      <th>SSC_NAME_2016</th>
      <th>SA4_CODE_2016</th>
      <th>SA4_NAME_2016</th>
      <th>RATIO</th>
      <th>PERCENTAGE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>10001</td>
      <td>Aarons Pass</td>
      <td>103</td>
      <td>Central West</td>
      <td>1.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <td>1</td>
      <td>10002</td>
      <td>Abbotsbury</td>
      <td>127</td>
      <td>Sydney - South West</td>
      <td>1.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <td>2</td>
      <td>10003</td>
      <td>Abbotsford (NSW)</td>
      <td>120</td>
      <td>Sydney - Inner West</td>
      <td>1.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <td>3</td>
      <td>10004</td>
      <td>Abercrombie</td>
      <td>103</td>
      <td>Central West</td>
      <td>1.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <td>4</td>
      <td>10005</td>
      <td>Abercrombie River</td>
      <td>103</td>
      <td>Central West</td>
      <td>1.0</td>
      <td>100.0</td>
    </tr>
  </tbody>
</table>
</div>

Now we can obtain only those suburbs from the SA4 *Melbourne - Inner* which is the most populated of Melbourne City and the focus of this study:

```python
# Get only the list of suburbs from Melbourne - Inner
df_melbourne = df_correspondence[df_correspondence['SA4_NAME_2016'].str.contains('Melbourne - Inner')][['SSC_CODE_2016', 'SSC_NAME_2016']]
df_melbourne.columns = ['ssc', 'name']
```

And we get a data frame looking like so:

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
      <th>ssc</th>
      <th>name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>4595</td>
      <td>20002</td>
      <td>Abbotsford (Vic.)</td>
    </tr>
    <tr>
      <td>4596</td>
      <td>20003</td>
      <td>Aberfeldie</td>
    </tr>
    <tr>
      <td>4610</td>
      <td>20017</td>
      <td>Albert Park (Vic.)</td>
    </tr>
    <tr>
      <td>4626</td>
      <td>20033</td>
      <td>Alphington</td>
    </tr>
    <tr>
      <td>4660</td>
      <td>20065</td>
      <td>Armadale (Vic.)</td>
    </tr>
  </tbody>
</table>
</div>

At this point we only want the demographics data for the suburbs of *Melboune - Inner*. To to this we can merge the *fd_families* and *df_melbourne* data frames.

```python
# Get rid of any suburb from the df_families other than Melbourne's
df_melbourne_families = pd.merge(df_families, df_melbourne, on=['ssc'])
```

The new data frame looks like so:

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
      <th>ssc</th>
      <th>ratio</th>
      <th>name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>20002</td>
      <td>0.217922</td>
      <td>Abbotsford (Vic.)</td>
    </tr>
    <tr>
      <td>1</td>
      <td>20003</td>
      <td>0.382012</td>
      <td>Aberfeldie</td>
    </tr>
    <tr>
      <td>2</td>
      <td>20017</td>
      <td>0.362508</td>
      <td>Albert Park (Vic.)</td>
    </tr>
    <tr>
      <td>3</td>
      <td>20033</td>
      <td>0.363782</td>
      <td>Alphington</td>
    </tr>
    <tr>
      <td>4</td>
      <td>20065</td>
      <td>0.325092</td>
      <td>Armadale (Vic.)</td>
    </tr>
  </tbody>
</table>
</div>


Then, we can load the categories we will pass to Foursquare to search for venues suitable for families with children.

```python
# Load the Foursquare categories in pandas
df_categories = pd.read_csv('https://raw.githubusercontent.com/tiomno/Coursera_Capstone/master/datasets/Foursquare_categories.csv')
```

The categories are pairs of name and ID, the first 5 are the following:

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
      <th>Category Name</th>
      <th>Category ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>Aquarium</td>
      <td>4fceea171983d5d06c3e9823</td>
    </tr>
    <tr>
      <td>1</td>
      <td>Art Gallery</td>
      <td>4bf58dd8d48988d1e2931735</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Exhibit</td>
      <td>56aa371be4b08b9a8d573532</td>
    </tr>
    <tr>
      <td>3</td>
      <td>General Entertainment</td>
      <td>4bf58dd8d48988d1f1931735</td>
    </tr>
    <tr>
      <td>4</td>
      <td>Historic Site</td>
      <td>4deefb944765f83613cdba6e</td>
    </tr>
  </tbody>
</table>
</div>

At this point we can create a variable *CATEGORIES* with all the categories we want to pass to the Forusquare API. But before we can start requesting this service we need the geolocation of each suburb. 

```python
# And create a comma separated string with the category IDs to pass in the Query String of the Foursquare URL request.
CATEGORIES = ','.join(df_categories['Category ID'].values.tolist())
```

The resulting string is the following:

    '4fceea171983d5d06c3e9823,4bf58dd8d48988d1e2931735,56aa371be4b08b9a8d573532,4bf58dd8d48988d1f1931735,4deefb944765f83613cdba6e,5642206c498e4bfca532186c,4bf58dd8d48988d17f941735,4bf58dd8d48988d181941735,4bf58dd8d48988d1e3931735,507c8c4091d498d9fc8c67a9,4bf58dd8d48988d184941735,4bf58dd8d48988d182941735,4bf58dd8d48988d193941735,4bf58dd8d48988d17b941735,5267e4d9e4b0ec79466e48c7,52741d85e4b0d5d1e3c6a6d9,5bae9231bedf3950379f89c5,5267e4d8e4b0ec79466e48c5,5bae9231bedf3950379f89c3,4bf58dd8d48988d128941735,4bf58dd8d48988d16d941735,4bf58dd8d48988d1e0931735,4bf58dd8d48988d120951735,4bf58dd8d48988d1ca941735,52e81612bcbc57f1066b7a2c,4cce455aebf7b749d5e191f5,52e81612bcbc57f1066b7a2e,52e81612bcbc57f1066b7a28,56aa371be4b08b9a8d573544,4bf58dd8d48988d1e2941735,56aa371be4b08b9a8d57355e,52e81612bcbc57f1066b7a22,4bf58dd8d48988d1df941735,4bf58dd8d48988d1e5941735,52e81612bcbc57f1066b7a23,56aa371be4b08b9a8d573547,4bf58dd8d48988d15a941735,5744ccdfe4b0c0459246b4b5,4bf58dd8d48988d161941735,52e81612bcbc57f1066b7a21,52e81612bcbc57f1066b7a13,4bf58dd8d48988d163941735,4bf58dd8d48988d1e7941735,4bf58dd8d48988d164941735,4bf58dd8d48988d15e941735,52e81612bcbc57f1066b7a26,56aa371be4b08b9a8d573541,4eb1d4dd4b900d56c88a45fd,4bf58dd8d48988d159941735,56aa371be4b08b9a8d5734c3'

Now we need to get the geolocation for each suburb. For this, we can use geopy. This is an open source and free service that converts an address into a pair of geospatial coordinates latitude and longitude. Foursquare needs those as a start point to search for venues depending on keywords, or in our case, categories.

The following code is an example of how to gather the geolocation, it can take a few minutes as this service may fail and we need to try the request again. At the end, it saves the results a CSV file for further processing.

```python
from geopy.geocoders import Nominatim
geolocator = Nominatim(user_agent="ny_explorer")

# Define a function that gets the geolocation of a suburb.
def get_suburb_geolocation(suburb, geolocation):
    try:
        if geolocation[0] is None:
            location = geolocator.geocode(suburb + ',VIC,Australia')
            if location is None:
                print('x ', end='')
                return None, None
            else:
                print('o ', end='')
                return location.latitude, location.longitude
        else:
            return geolocation
    except:
        return None, None

# Go through the dataframe of suburbs and add the geolocation
# Repeat the process until all the suburb are filled in.
df_melbourne_families["geolocation"] = [(None, None)] * len(df_melbourne_families)
i = 0
while len(df_melbourne_families[df_melbourne_families['geolocation'] == (None, None)]) > 0 and i < 20:
    df_melbourne_families['geolocation'] = df_melbourne_families.apply(
        lambda df: get_suburb_geolocation(df['name'], df['geolocation']), axis=1
    )
    i += 1

df_melbourne_families.to_csv('df_melbourne_families.csv')

# Remove SSC duplication due to the correspondence with SA4 areas
bool_series = df_melbourne_families["ssc"].duplicated() 
df_melbourne_families = df_melbourne_families[~bool_series] 
```

The data frame *df_melbourne_families* look like so:

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
      <th>ssc</th>
      <th>ratio</th>
      <th>name</th>
      <th>geolocation</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>20002</td>
      <td>0.217922</td>
      <td>Abbotsford (Vic.)</td>
      <td>(-37.8045508, 144.9988542)</td>
    </tr>
    <tr>
      <td>1</td>
      <td>20003</td>
      <td>0.382012</td>
      <td>Aberfeldie</td>
      <td>(-37.7596196, 144.8974571)</td>
    </tr>
    <tr>
      <td>2</td>
      <td>20017</td>
      <td>0.362508</td>
      <td>Albert Park (Vic.)</td>
      <td>(-37.8477725, 144.96200797154074)</td>
    </tr>
    <tr>
      <td>3</td>
      <td>20033</td>
      <td>0.363782</td>
      <td>Alphington</td>
      <td>(-37.7783953, 145.0312823)</td>
    </tr>
    <tr>
      <td>4</td>
      <td>20065</td>
      <td>0.325092</td>
      <td>Armadale (Vic.)</td>
      <td>(-37.8567619, 145.0206905)</td>
    </tr>
  </tbody>
</table>
</div>

With the suburbs geolocation we can request Foursquare to gather the venues suitable for families with children. This is as easy as building the corresponding URL and using the *requests* module of Python:

```python
# Foursquare parameters
CLIENT_ID = 'Foursquare API Client ID here'
CLIENT_SECRET = 'Foursquare API Secret here'
VERSION = '20180604'
LIMIT = 30
RADIUS = 500

url_template = 'https://api.foursquare.com/v2/venues/search?client_id={}&client_secret={}&ll={}&v={}&categoryId={}&radius={}&limit={}'.format(CLIENT_ID, CLIENT_SECRET, '{}', VERSION, CATEGORIES, RADIUS, LIMIT)
url_template

import requests # library to handle requests

# Add new categories as clumns to df_melbourne_families
for category in df_categories['Category Name']:
    df_melbourne_families[category] = 0

def get_category_venues(ssc, geolocation):
    lst_venues=[]

    for ssc, location in zip(ssc, geolocation):
        lat_lng = location.strip('()')
            
        # create the API request URL
        url = url_template.format(lat_lng)
            
        # make the GET request
        venues = requests.get(url).json()["response"]['venues']
        
        # Append SSC and category to the venues list to group them in further steps
        lst_venues.append([
            (ssc, venue['categories'][0]['name']) for venue in venues
        ])

    return pd.DataFrame([item for lst in lst_venues for item in lst], columns=['ssc', 'category'])

    
df_categories_ssc = get_category_venues(
    ssc=df_melbourne_families['ssc'],
    geolocation=df_melbourne_families['geolocation']
)
```

The compiled data frame looks like so:

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
      <th>ssc</th>
      <th>category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>20002</td>
      <td>Park</td>
    </tr>
    <tr>
      <td>1</td>
      <td>20002</td>
      <td>Park</td>
    </tr>
    <tr>
      <td>2</td>
      <td>20002</td>
      <td>Cultural Center</td>
    </tr>
    <tr>
      <td>3</td>
      <td>20002</td>
      <td>Coffee Shop</td>
    </tr>
    <tr>
      <td>4</td>
      <td>20002</td>
      <td>Café</td>
    </tr>
  </tbody>
</table>
</div>

Now it's time to count the venues per categories by suburb. Then, we can merge the list of suburbs with the counts of venues per categories and start playing with clustering algorithm.

```python
# Add dummy value to count 1 per each row and venue
df_categories_ssc['venues'] = 1

# Count venues per category and total by SSC
df_categories_ssc['venues_count'] = df_categories_ssc.groupby(['ssc', 'category'])['venues'].transform('sum')

# Remove extra columns and row duplications
df_categories_ssc = df_categories_ssc[['ssc', 'category', 'venues_count']].drop_duplicates()
# Convert categories into columns with venues counts as cell values
df_categories_ssc = df_categories_ssc.pivot_table(
    values='venues_count', index=df_categories_ssc.ssc, columns='category', aggfunc='first', fill_value=0
)
df_categories_ssc.reset_index(level=0, inplace=True)

# Merge families data frame and the category aggregations
df_melbourne_venues = pd.merge(df_melbourne_families, df_categories_ssc, on=['ssc'])
```

And the result is something like the following:

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
      <th>ssc</th>
      <th>ratio</th>
      <th>name</th>
      <th>geolocation</th>
      <th>American Restaurant</th>
      <th>Art Gallery</th>
      <th>Art Museum</th>
      <th>Art Studio</th>
      <th>Asian Restaurant</th>
      <th>Athletics &amp; Sports</th>
      <th>...</th>
      <th>Thai Restaurant</th>
      <th>Theater</th>
      <th>Theme Park</th>
      <th>Trail</th>
      <th>Turkish Restaurant</th>
      <th>Vegetarian / Vegan Restaurant</th>
      <th>Water Park</th>
      <th>Wine Bar</th>
      <th>Zoo</th>
      <th>Zoo Exhibit</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>20002</td>
      <td>0.217922</td>
      <td>Abbotsford (Vic.)</td>
      <td>(-37.8045508, 144.9988542)</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>1</td>
      <td>20003</td>
      <td>0.382012</td>
      <td>Aberfeldie</td>
      <td>(-37.7596196, 144.8974571)</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>2</td>
      <td>20017</td>
      <td>0.362508</td>
      <td>Albert Park (Vic.)</td>
      <td>(-37.8477725, 144.96200797154074)</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>3</td>
      <td>20033</td>
      <td>0.363782</td>
      <td>Alphington</td>
      <td>(-37.7783953, 145.0312823)</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>4</td>
      <td>20065</td>
      <td>0.325092</td>
      <td>Armadale (Vic.)</td>
      <td>(-37.8567619, 145.0206905)</td>
      <td>0</td>
      <td>3</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 119 columns</p>
</div>

This is all we need to start playing with K-means to find out the clusters of suburbs.

## Methodology

With the necessary data at hand, we can use a clustering algorithm to identify a pattern in the dataset that shows us the relation between the family ratio and the venues suitable for families in a suburb.

We can use K-means ML algorithm for this purpose and play with different numbers of clusters until we get a value suitable for the work of the local government. A small number of clusters would give us too many different suburbs in a single group which is not of much help. On the other hand, too many groups would make difficult the task of identifying patterns and allocating the necessary resources and team of experts to each group.

Clustering is the perfect solution to discover segments of suburbs in the city. Still, we needed to focus the attention on specific variables. To find those variables, we had discussions with the local government to know which group of the population we needed to target. Then, based on the final goal, we can define the kind of venues suitable for families in this study.

Knowing the segment of the population to target and the categories of the venues we needed to find, we can start the search for the right tools and information available. Gathering aggregations from the Australian Bureau of Statistics website, we obtain insight into the shape of the families' distribution in Melbourne. With the Foursquare API, we search for the venues suitable for families in each suburb.

We merged this information and are ready to run the clustering algorithm and analyse the results.

## Analysis

Let's try to find the optimum number of clusters from a range of 5 - 20 using the Elbow method.

The following code, which is a version of the code you can find in the article <a href="https://pythonprogramminglanguage.com/kmeans-elbow-method/">kmeans elbow method</a>, helps us to fit the ML algorithm with different values of *k* and produc a graph showing the Distortion of clusters vs the number of clusters.

```python
from sklearn.cluster import KMeans
from scipy.spatial.distance import cdist
import numpy as np
import matplotlib.pyplot as plt

%matplotlib inline

# create new plot and data
plt.plot()
colors = ['b', 'g', 'r']
markers = ['o', 'v', 's']

df_melbourne_venues_clustering = df_melbourne_venues.drop(columns=['ssc', 'name', 'geolocation'])

# k means determine k
distortions = []
K = range(5, 20)
for k in K:
    kmeanModel = KMeans(n_clusters=k).fit(df_melbourne_venues_clustering)
    kmeanModel.fit(df_melbourne_venues_clustering)
    distortions.append(sum(np.min(
        cdist(df_melbourne_venues_clustering, kmeanModel.cluster_centers_, 'euclidean'), axis=1
    )) / df_melbourne_venues_clustering.shape[0])

# Plot the elbow
plt.plot(K, distortions, 'bx-')
plt.xlabel('k')
plt.ylabel('Distortion')
plt.title('The Elbow Method showing the optimal k')
plt.show()
```

The graph produced is similar to this: 

![_config.yml]({{ site.baseurl }}/images/elbow1.png)

Since the previous method doesn't show a clear "knee", let's apply an even further automated Elbow method thta meatured the time the K-means take to run and find the optimal cluster number:

```python
from yellowbrick.cluster.elbow import kelbow_visualizer

# Use the quick method and immediately show the figure
kelbow_visualizer(KMeans(random_state=4), df_melbourne_venues_clustering, k=(5, 20))
```

This the graph this other method produce. Notice that the verticar dashed line indicates the optimum number for the *k* parameter.

![_config.yml]({{ site.baseurl }}/images/elbow2.png)

Let's pick 13 clusters then, and continue with the anlysis of Melbourne suburbs.
> note: see Results and Discussion section at the bottom of this post for thoughts about this method.

Running K-means with 13 clusters

```python
# set number of clusters
kclusters = 13

# run k-means clustering
kmeans = KMeans(n_clusters=kclusters, random_state=0).fit(df_melbourne_venues_clustering)

# add clustering labels to the Melbourne venues dataframe
df_melbourne_venues.insert(0, 'Cluster Labels', kmeans.labels_)
```

Now the data frame *df_melbourne_venues* has a new column indicating what's the cluster each suburb belongs to:

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
      <th>Cluster Labels</th>
      <th>ssc</th>
      <th>ratio</th>
      <th>name</th>
      <th>geolocation</th>
      <th>American Restaurant</th>
      <th>Art Gallery</th>
      <th>Art Museum</th>
      <th>Art Studio</th>
      <th>Asian Restaurant</th>
      <th>...</th>
      <th>Thai Restaurant</th>
      <th>Theater</th>
      <th>Theme Park</th>
      <th>Trail</th>
      <th>Turkish Restaurant</th>
      <th>Vegetarian / Vegan Restaurant</th>
      <th>Water Park</th>
      <th>Wine Bar</th>
      <th>Zoo</th>
      <th>Zoo Exhibit</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>9</td>
      <td>20002</td>
      <td>0.217922</td>
      <td>Abbotsford (Vic.)</td>
      <td>(-37.8045508, 144.9988542)</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>1</td>
      <td>6</td>
      <td>20003</td>
      <td>0.382012</td>
      <td>Aberfeldie</td>
      <td>(-37.7596196, 144.8974571)</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>2</td>
      <td>12</td>
      <td>20017</td>
      <td>0.362508</td>
      <td>Albert Park (Vic.)</td>
      <td>(-37.8477725, 144.96200797154074)</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>3</td>
      <td>4</td>
      <td>20033</td>
      <td>0.363782</td>
      <td>Alphington</td>
      <td>(-37.7783953, 145.0312823)</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>4</td>
      <td>5</td>
      <td>20065</td>
      <td>0.325092</td>
      <td>Armadale (Vic.)</td>
      <td>(-37.8567619, 145.0206905)</td>
      <td>0</td>
      <td>3</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 120 columns</p>
</div>

Showing the clusters on a map will give us an idea of the distribution by kind of venues. Folium + Matplotlib are great tools for such a task.

```python
import folium # map rendering library

# Matplotlib and associated plotting modules
import matplotlib.cm as cm
import matplotlib.colors as colors

# Find the geolocation of Melbourne to use it as the centre of the map.
geolocator = Nominatim(user_agent="ny_explorer")
location = geolocator.geocode('Melbourne, Australia')
latitude = location.latitude
longitude = location.longitude

# create map
map_clusters = folium.Map(location=[latitude, longitude], zoom_start=10)

# set color scheme for the clusters
colors_array = cm.rainbow(np.linspace(0, 1, kclusters))
rainbow = [colors.rgb2hex(i) for i in colors_array]

# add markers to the map
markers_colors = []
for location, name, cluster in zip(
    df_melbourne_venues['geolocation'], df_melbourne_venues['name'], df_melbourne_venues['Cluster Labels']
):
    label = folium.Popup(str(name) + ' Cluster ' + str(cluster), parse_html=True)
    [lat, lng] = map(float, location.strip('()').split(', '))
    folium.CircleMarker(
        [lat, lng],
        radius=3,
        popup=label,
        color=rainbow[cluster-1],
        fill=True,
        fill_color=rainbow[cluster-1],
        fill_opacity=0.7).add_to(map_clusters)
       
map_clusters
```

![_config.yml]({{ site.baseurl }}/images/map.png)

But, let's take a closer look at the data of each cluster. The following piece of code will iterate the clusters and show:
- How many suburbs the cluster has been shown in the first row.
- The mean value for the families ratio and the occurrence of each type of venue.

```python
for i in range(kclusters):
    print('Cluster {}:'.format(i))
    df = df_melbourne_venues.loc[
        df_melbourne_venues['Cluster Labels'] == i,
        df_melbourne_venues.columns[[2] + list(range(5, df_melbourne_venues.shape[1]))]
    ]
    # remove columns (categories) with no venues
    df = df.loc[:, (df > 0).any(axis=0)]
    display(df.describe().head(2).T)
```

> Cluster 0:

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
      <th>count</th>
      <th>mean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ratio</td>
      <td>7.0</td>
      <td>0.372318</td>
    </tr>
    <tr>
      <td>Bowling Green</td>
      <td>7.0</td>
      <td>0.142857</td>
    </tr>
    <tr>
      <td>Breakfast Spot</td>
      <td>7.0</td>
      <td>0.285714</td>
    </tr>
    <tr>
      <td>Café</td>
      <td>7.0</td>
      <td>2.571429</td>
    </tr>
    <tr>
      <td>Coffee Shop</td>
      <td>7.0</td>
      <td>0.285714</td>
    </tr>
    <tr>
      <td>Dog Run</td>
      <td>7.0</td>
      <td>0.285714</td>
    </tr>
    <tr>
      <td>Garden</td>
      <td>7.0</td>
      <td>0.142857</td>
    </tr>
    <tr>
      <td>General Entertainment</td>
      <td>7.0</td>
      <td>1.285714</td>
    </tr>
    <tr>
      <td>Historic Site</td>
      <td>7.0</td>
      <td>0.428571</td>
    </tr>
    <tr>
      <td>Park</td>
      <td>7.0</td>
      <td>3.428571</td>
    </tr>
    <tr>
      <td>Pizza Place</td>
      <td>7.0</td>
      <td>1.285714</td>
    </tr>
    <tr>
      <td>Playground</td>
      <td>7.0</td>
      <td>0.857143</td>
    </tr>
    <tr>
      <td>Plaza</td>
      <td>7.0</td>
      <td>0.142857</td>
    </tr>
    <tr>
      <td>Public Art</td>
      <td>7.0</td>
      <td>0.142857</td>
    </tr>
    <tr>
      <td>Sports Club</td>
      <td>7.0</td>
      <td>0.428571</td>
    </tr>
    <tr>
      <td>Zoo Exhibit</td>
      <td>7.0</td>
      <td>0.285714</td>
    </tr>
  </tbody>
</table>
</div>

> Cluster 1:

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
      <th>count</th>
      <th>mean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ratio</td>
      <td>10.0</td>
      <td>0.315437</td>
    </tr>
    <tr>
      <td>American Restaurant</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Art Gallery</td>
      <td>10.0</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <td>Art Studio</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Australian Restaurant</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Bakery</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Bar</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Bridge</td>
      <td>10.0</td>
      <td>0.200000</td>
    </tr>
    <tr>
      <td>Cafeteria</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Café</td>
      <td>10.0</td>
      <td>11.600000</td>
    </tr>
    <tr>
      <td>Coffee Shop</td>
      <td>10.0</td>
      <td>1.700000</td>
    </tr>
    <tr>
      <td>Deli / Bodega</td>
      <td>10.0</td>
      <td>0.200000</td>
    </tr>
    <tr>
      <td>Dessert Shop</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Diner</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Dog Run</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Exhibit</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Fast Food Restaurant</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Food Court</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Football Stadium</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>French Restaurant</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Furniture / Home Store</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Garden</td>
      <td>10.0</td>
      <td>0.300000</td>
    </tr>
    <tr>
      <td>Garden Center</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Gastropub</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>General Entertainment</td>
      <td>10.0</td>
      <td>0.400000</td>
    </tr>
    <tr>
      <td>Grocery Store</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Gym</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Historic Site</td>
      <td>10.0</td>
      <td>0.200000</td>
    </tr>
    <tr>
      <td>History Museum</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Indie Movie Theater</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Italian Restaurant</td>
      <td>10.0</td>
      <td>0.400000</td>
    </tr>
    <tr>
      <td>Multiplex</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Park</td>
      <td>10.0</td>
      <td>1.900000</td>
    </tr>
    <tr>
      <td>Performing Arts Venue</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Pizza Place</td>
      <td>10.0</td>
      <td>2.700000</td>
    </tr>
    <tr>
      <td>Planetarium</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Playground</td>
      <td>10.0</td>
      <td>1.300000</td>
    </tr>
    <tr>
      <td>Plaza</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Pool</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Pool Hall</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Recreation Center</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>River</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Sculpture Garden</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Soccer Field</td>
      <td>10.0</td>
      <td>0.400000</td>
    </tr>
    <tr>
      <td>Sports Club</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Supermarket</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Thai Restaurant</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Theater</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Theme Park</td>
      <td>10.0</td>
      <td>0.200000</td>
    </tr>
    <tr>
      <td>Trail</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Vegetarian / Vegan Restaurant</td>
      <td>10.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Zoo Exhibit</td>
      <td>10.0</td>
      <td>0.200000</td>
    </tr>
  </tbody>
</table>
</div>

> Cluster 2:

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
      <th>count</th>
      <th>mean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ratio</td>
      <td>9.0</td>
      <td>0.361478</td>
    </tr>
    <tr>
      <td>Art Gallery</td>
      <td>9.0</td>
      <td>0.666667</td>
    </tr>
    <tr>
      <td>Bakery</td>
      <td>9.0</td>
      <td>0.111111</td>
    </tr>
    <tr>
      <td>Beach</td>
      <td>9.0</td>
      <td>0.111111</td>
    </tr>
    <tr>
      <td>Bike Shop</td>
      <td>9.0</td>
      <td>0.111111</td>
    </tr>
    <tr>
      <td>Breakfast Spot</td>
      <td>9.0</td>
      <td>0.222222</td>
    </tr>
    <tr>
      <td>Cafeteria</td>
      <td>9.0</td>
      <td>0.111111</td>
    </tr>
    <tr>
      <td>Café</td>
      <td>9.0</td>
      <td>8.111111</td>
    </tr>
    <tr>
      <td>Coffee Shop</td>
      <td>9.0</td>
      <td>1.222222</td>
    </tr>
    <tr>
      <td>Dessert Shop</td>
      <td>9.0</td>
      <td>0.111111</td>
    </tr>
    <tr>
      <td>Dog Run</td>
      <td>9.0</td>
      <td>0.111111</td>
    </tr>
    <tr>
      <td>Food Court</td>
      <td>9.0</td>
      <td>0.222222</td>
    </tr>
    <tr>
      <td>Football Stadium</td>
      <td>9.0</td>
      <td>0.111111</td>
    </tr>
    <tr>
      <td>Garden</td>
      <td>9.0</td>
      <td>0.222222</td>
    </tr>
    <tr>
      <td>General Entertainment</td>
      <td>9.0</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <td>Indie Movie Theater</td>
      <td>9.0</td>
      <td>0.222222</td>
    </tr>
    <tr>
      <td>Italian Restaurant</td>
      <td>9.0</td>
      <td>0.333333</td>
    </tr>
    <tr>
      <td>Movie Theater</td>
      <td>9.0</td>
      <td>0.555556</td>
    </tr>
    <tr>
      <td>Multiplex</td>
      <td>9.0</td>
      <td>0.222222</td>
    </tr>
    <tr>
      <td>Park</td>
      <td>9.0</td>
      <td>0.888889</td>
    </tr>
    <tr>
      <td>Pizza Place</td>
      <td>9.0</td>
      <td>0.666667</td>
    </tr>
    <tr>
      <td>Playground</td>
      <td>9.0</td>
      <td>1.666667</td>
    </tr>
    <tr>
      <td>Plaza</td>
      <td>9.0</td>
      <td>0.111111</td>
    </tr>
    <tr>
      <td>Polish Restaurant</td>
      <td>9.0</td>
      <td>0.111111</td>
    </tr>
    <tr>
      <td>Public Art</td>
      <td>9.0</td>
      <td>0.111111</td>
    </tr>
    <tr>
      <td>Recreation Center</td>
      <td>9.0</td>
      <td>0.111111</td>
    </tr>
    <tr>
      <td>Soccer Field</td>
      <td>9.0</td>
      <td>0.222222</td>
    </tr>
    <tr>
      <td>Sports Club</td>
      <td>9.0</td>
      <td>0.111111</td>
    </tr>
    <tr>
      <td>Trail</td>
      <td>9.0</td>
      <td>0.111111</td>
    </tr>
    <tr>
      <td>Water Park</td>
      <td>9.0</td>
      <td>0.111111</td>
    </tr>
  </tbody>
</table>
</div>

> Cluster 3:

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
      <th>count</th>
      <th>mean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ratio</td>
      <td>8.0</td>
      <td>0.380283</td>
    </tr>
    <tr>
      <td>American Restaurant</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Art Gallery</td>
      <td>8.0</td>
      <td>0.375000</td>
    </tr>
    <tr>
      <td>Beach</td>
      <td>8.0</td>
      <td>0.500000</td>
    </tr>
    <tr>
      <td>Bowling Green</td>
      <td>8.0</td>
      <td>0.250000</td>
    </tr>
    <tr>
      <td>Café</td>
      <td>8.0</td>
      <td>18.625000</td>
    </tr>
    <tr>
      <td>Car Wash</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>City Hall</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Coffee Shop</td>
      <td>8.0</td>
      <td>1.500000</td>
    </tr>
    <tr>
      <td>Cricket Ground</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Deli / Bodega</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Dog Run</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Eastern European Restaurant</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Food &amp; Drink Shop</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Football Stadium</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Garden</td>
      <td>8.0</td>
      <td>0.250000</td>
    </tr>
    <tr>
      <td>General Entertainment</td>
      <td>8.0</td>
      <td>0.750000</td>
    </tr>
    <tr>
      <td>Indie Movie Theater</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Italian Restaurant</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Kebab Restaurant</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Middle Eastern Restaurant</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Park</td>
      <td>8.0</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <td>Pizza Place</td>
      <td>8.0</td>
      <td>2.625000</td>
    </tr>
    <tr>
      <td>Playground</td>
      <td>8.0</td>
      <td>0.625000</td>
    </tr>
    <tr>
      <td>Plaza</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Pool Hall</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Recreation Center</td>
      <td>8.0</td>
      <td>0.250000</td>
    </tr>
    <tr>
      <td>Soccer Field</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Sports Club</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
  </tbody>
</table>
</div>

> Cluster 4:

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
      <th>count</th>
      <th>mean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ratio</td>
      <td>20.0</td>
      <td>0.352785</td>
    </tr>
    <tr>
      <td>Art Gallery</td>
      <td>20.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Australian Restaurant</td>
      <td>20.0</td>
      <td>0.050000</td>
    </tr>
    <tr>
      <td>Bakery</td>
      <td>20.0</td>
      <td>0.050000</td>
    </tr>
    <tr>
      <td>Bar</td>
      <td>20.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Beach</td>
      <td>20.0</td>
      <td>0.300000</td>
    </tr>
    <tr>
      <td>Breakfast Spot</td>
      <td>20.0</td>
      <td>0.050000</td>
    </tr>
    <tr>
      <td>Bridge</td>
      <td>20.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Burger Joint</td>
      <td>20.0</td>
      <td>0.050000</td>
    </tr>
    <tr>
      <td>Café</td>
      <td>20.0</td>
      <td>5.250000</td>
    </tr>
    <tr>
      <td>Car Wash</td>
      <td>20.0</td>
      <td>0.050000</td>
    </tr>
    <tr>
      <td>Cheese Shop</td>
      <td>20.0</td>
      <td>0.050000</td>
    </tr>
    <tr>
      <td>Coffee Shop</td>
      <td>20.0</td>
      <td>1.050000</td>
    </tr>
    <tr>
      <td>Dessert Shop</td>
      <td>20.0</td>
      <td>0.150000</td>
    </tr>
    <tr>
      <td>Dog Run</td>
      <td>20.0</td>
      <td>0.200000</td>
    </tr>
    <tr>
      <td>Exhibit</td>
      <td>20.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Falafel Restaurant</td>
      <td>20.0</td>
      <td>0.050000</td>
    </tr>
    <tr>
      <td>Fish &amp; Chips Shop</td>
      <td>20.0</td>
      <td>0.050000</td>
    </tr>
    <tr>
      <td>Flower Shop</td>
      <td>20.0</td>
      <td>0.050000</td>
    </tr>
    <tr>
      <td>Food Court</td>
      <td>20.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Garden</td>
      <td>20.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>General Entertainment</td>
      <td>20.0</td>
      <td>0.300000</td>
    </tr>
    <tr>
      <td>Greek Restaurant</td>
      <td>20.0</td>
      <td>0.050000</td>
    </tr>
    <tr>
      <td>Historic Site</td>
      <td>20.0</td>
      <td>0.150000</td>
    </tr>
    <tr>
      <td>History Museum</td>
      <td>20.0</td>
      <td>0.050000</td>
    </tr>
    <tr>
      <td>Indie Movie Theater</td>
      <td>20.0</td>
      <td>0.050000</td>
    </tr>
    <tr>
      <td>Indoor Play Area</td>
      <td>20.0</td>
      <td>0.050000</td>
    </tr>
    <tr>
      <td>Italian Restaurant</td>
      <td>20.0</td>
      <td>0.300000</td>
    </tr>
    <tr>
      <td>Memorial Site</td>
      <td>20.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Monument / Landmark</td>
      <td>20.0</td>
      <td>0.050000</td>
    </tr>
    <tr>
      <td>Movie Theater</td>
      <td>20.0</td>
      <td>0.050000</td>
    </tr>
    <tr>
      <td>Museum</td>
      <td>20.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Nature Preserve</td>
      <td>20.0</td>
      <td>0.050000</td>
    </tr>
    <tr>
      <td>Outdoor Event Space</td>
      <td>20.0</td>
      <td>0.050000</td>
    </tr>
    <tr>
      <td>Park</td>
      <td>20.0</td>
      <td>1.500000</td>
    </tr>
    <tr>
      <td>Pier</td>
      <td>20.0</td>
      <td>0.050000</td>
    </tr>
    <tr>
      <td>Pizza Place</td>
      <td>20.0</td>
      <td>2.250000</td>
    </tr>
    <tr>
      <td>Playground</td>
      <td>20.0</td>
      <td>0.500000</td>
    </tr>
    <tr>
      <td>Plaza</td>
      <td>20.0</td>
      <td>0.050000</td>
    </tr>
    <tr>
      <td>Pool</td>
      <td>20.0</td>
      <td>0.050000</td>
    </tr>
    <tr>
      <td>Pool Hall</td>
      <td>20.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Recreation Center</td>
      <td>20.0</td>
      <td>0.150000</td>
    </tr>
    <tr>
      <td>Restaurant</td>
      <td>20.0</td>
      <td>0.050000</td>
    </tr>
    <tr>
      <td>Soccer Field</td>
      <td>20.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Sports Club</td>
      <td>20.0</td>
      <td>0.150000</td>
    </tr>
    <tr>
      <td>Surf Spot</td>
      <td>20.0</td>
      <td>0.050000</td>
    </tr>
    <tr>
      <td>Tattoo Parlor</td>
      <td>20.0</td>
      <td>0.050000</td>
    </tr>
    <tr>
      <td>Trail</td>
      <td>20.0</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <td>Vegetarian / Vegan Restaurant</td>
      <td>20.0</td>
      <td>0.050000</td>
    </tr>
  </tbody>
</table>
</div>

> Cluster 5:

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
      <th>count</th>
      <th>mean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ratio</td>
      <td>8.0</td>
      <td>0.289903</td>
    </tr>
    <tr>
      <td>Art Gallery</td>
      <td>8.0</td>
      <td>2.750000</td>
    </tr>
    <tr>
      <td>Art Museum</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Bakery</td>
      <td>8.0</td>
      <td>0.250000</td>
    </tr>
    <tr>
      <td>Bar</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Breakfast Spot</td>
      <td>8.0</td>
      <td>0.500000</td>
    </tr>
    <tr>
      <td>Bridge</td>
      <td>8.0</td>
      <td>0.625000</td>
    </tr>
    <tr>
      <td>Bubble Tea Shop</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Café</td>
      <td>8.0</td>
      <td>16.125000</td>
    </tr>
    <tr>
      <td>Coffee Shop</td>
      <td>8.0</td>
      <td>2.750000</td>
    </tr>
    <tr>
      <td>Concert Hall</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Cupcake Shop</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Dog Run</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Exhibit</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Food Court</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>French Restaurant</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Garden</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>General Entertainment</td>
      <td>8.0</td>
      <td>0.250000</td>
    </tr>
    <tr>
      <td>Grocery Store</td>
      <td>8.0</td>
      <td>0.250000</td>
    </tr>
    <tr>
      <td>Hookah Bar</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Italian Restaurant</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Park</td>
      <td>8.0</td>
      <td>0.625000</td>
    </tr>
    <tr>
      <td>Pizza Place</td>
      <td>8.0</td>
      <td>1.125000</td>
    </tr>
    <tr>
      <td>Playground</td>
      <td>8.0</td>
      <td>0.250000</td>
    </tr>
    <tr>
      <td>Plaza</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Pool</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Pub</td>
      <td>8.0</td>
      <td>0.250000</td>
    </tr>
    <tr>
      <td>Recreation Center</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Strip Club</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Theme Park</td>
      <td>8.0</td>
      <td>0.250000</td>
    </tr>
    <tr>
      <td>Vegetarian / Vegan Restaurant</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
    <tr>
      <td>Wine Bar</td>
      <td>8.0</td>
      <td>0.125000</td>
    </tr>
  </tbody>
</table>
</div>

> Cluster 6:

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
      <th>count</th>
      <th>mean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ratio</td>
      <td>28.0</td>
      <td>0.374008</td>
    </tr>
    <tr>
      <td>Athletics &amp; Sports</td>
      <td>28.0</td>
      <td>0.035714</td>
    </tr>
    <tr>
      <td>Bakery</td>
      <td>28.0</td>
      <td>0.071429</td>
    </tr>
    <tr>
      <td>Beach</td>
      <td>28.0</td>
      <td>0.321429</td>
    </tr>
    <tr>
      <td>Café</td>
      <td>28.0</td>
      <td>1.392857</td>
    </tr>
    <tr>
      <td>Coffee Shop</td>
      <td>28.0</td>
      <td>0.250000</td>
    </tr>
    <tr>
      <td>Deli / Bodega</td>
      <td>28.0</td>
      <td>0.035714</td>
    </tr>
    <tr>
      <td>Dessert Shop</td>
      <td>28.0</td>
      <td>0.035714</td>
    </tr>
    <tr>
      <td>Dog Run</td>
      <td>28.0</td>
      <td>0.107143</td>
    </tr>
    <tr>
      <td>Food Court</td>
      <td>28.0</td>
      <td>0.035714</td>
    </tr>
    <tr>
      <td>Garden</td>
      <td>28.0</td>
      <td>0.107143</td>
    </tr>
    <tr>
      <td>General Entertainment</td>
      <td>28.0</td>
      <td>0.178571</td>
    </tr>
    <tr>
      <td>Italian Restaurant</td>
      <td>28.0</td>
      <td>0.035714</td>
    </tr>
    <tr>
      <td>Office</td>
      <td>28.0</td>
      <td>0.035714</td>
    </tr>
    <tr>
      <td>Other Great Outdoors</td>
      <td>28.0</td>
      <td>0.035714</td>
    </tr>
    <tr>
      <td>Park</td>
      <td>28.0</td>
      <td>0.428571</td>
    </tr>
    <tr>
      <td>Pizza Place</td>
      <td>28.0</td>
      <td>0.464286</td>
    </tr>
    <tr>
      <td>Playground</td>
      <td>28.0</td>
      <td>0.464286</td>
    </tr>
    <tr>
      <td>Pool</td>
      <td>28.0</td>
      <td>0.071429</td>
    </tr>
    <tr>
      <td>Recreation Center</td>
      <td>28.0</td>
      <td>0.178571</td>
    </tr>
    <tr>
      <td>Soccer Field</td>
      <td>28.0</td>
      <td>0.142857</td>
    </tr>
    <tr>
      <td>Sports Club</td>
      <td>28.0</td>
      <td>0.071429</td>
    </tr>
    <tr>
      <td>Surf Spot</td>
      <td>28.0</td>
      <td>0.035714</td>
    </tr>
    <tr>
      <td>Tennis Stadium</td>
      <td>28.0</td>
      <td>0.035714</td>
    </tr>
    <tr>
      <td>Trail</td>
      <td>28.0</td>
      <td>0.071429</td>
    </tr>
  </tbody>
</table>
</div>

> Cluster 7:

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
      <th>count</th>
      <th>mean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ratio</td>
      <td>1.0</td>
      <td>0.255165</td>
    </tr>
    <tr>
      <td>Café</td>
      <td>1.0</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <td>Dog Run</td>
      <td>1.0</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <td>Garden</td>
      <td>1.0</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <td>General Entertainment</td>
      <td>1.0</td>
      <td>3.000000</td>
    </tr>
    <tr>
      <td>Hockey Arena</td>
      <td>1.0</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <td>Park</td>
      <td>1.0</td>
      <td>4.000000</td>
    </tr>
    <tr>
      <td>Sports Club</td>
      <td>1.0</td>
      <td>2.000000</td>
    </tr>
    <tr>
      <td>Zoo</td>
      <td>1.0</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <td>Zoo Exhibit</td>
      <td>1.0</td>
      <td>16.000000</td>
    </tr>
  </tbody>
</table>
</div>

> Cluster 8:

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
      <th>count</th>
      <th>mean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ratio</td>
      <td>1.0</td>
      <td>0.369052</td>
    </tr>
    <tr>
      <td>Café</td>
      <td>1.0</td>
      <td>26.000000</td>
    </tr>
    <tr>
      <td>Coffee Shop</td>
      <td>1.0</td>
      <td>4.000000</td>
    </tr>
    <tr>
      <td>Cricket Ground</td>
      <td>1.0</td>
      <td>2.000000</td>
    </tr>
    <tr>
      <td>Dog Run</td>
      <td>1.0</td>
      <td>2.000000</td>
    </tr>
    <tr>
      <td>Garden</td>
      <td>1.0</td>
      <td>2.000000</td>
    </tr>
    <tr>
      <td>Italian Restaurant</td>
      <td>1.0</td>
      <td>2.000000</td>
    </tr>
    <tr>
      <td>Park</td>
      <td>1.0</td>
      <td>2.000000</td>
    </tr>
    <tr>
      <td>Pizza Place</td>
      <td>1.0</td>
      <td>4.000000</td>
    </tr>
    <tr>
      <td>Playground</td>
      <td>1.0</td>
      <td>4.000000</td>
    </tr>
  </tbody>
</table>
</div>

> Cluster 9:

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
      <th>count</th>
      <th>mean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ratio</td>
      <td>7.0</td>
      <td>0.280073</td>
    </tr>
    <tr>
      <td>Art Gallery</td>
      <td>7.0</td>
      <td>1.857143</td>
    </tr>
    <tr>
      <td>Asian Restaurant</td>
      <td>7.0</td>
      <td>0.142857</td>
    </tr>
    <tr>
      <td>Bakery</td>
      <td>7.0</td>
      <td>0.142857</td>
    </tr>
    <tr>
      <td>Bar</td>
      <td>7.0</td>
      <td>0.285714</td>
    </tr>
    <tr>
      <td>Bridge</td>
      <td>7.0</td>
      <td>0.285714</td>
    </tr>
    <tr>
      <td>Cafeteria</td>
      <td>7.0</td>
      <td>0.142857</td>
    </tr>
    <tr>
      <td>Café</td>
      <td>7.0</td>
      <td>12.285714</td>
    </tr>
    <tr>
      <td>Coffee Shop</td>
      <td>7.0</td>
      <td>3.285714</td>
    </tr>
    <tr>
      <td>Cultural Center</td>
      <td>7.0</td>
      <td>0.142857</td>
    </tr>
    <tr>
      <td>Dog Run</td>
      <td>7.0</td>
      <td>0.142857</td>
    </tr>
    <tr>
      <td>Exhibit</td>
      <td>7.0</td>
      <td>0.142857</td>
    </tr>
    <tr>
      <td>Garden</td>
      <td>7.0</td>
      <td>0.714286</td>
    </tr>
    <tr>
      <td>Gastropub</td>
      <td>7.0</td>
      <td>0.142857</td>
    </tr>
    <tr>
      <td>General Entertainment</td>
      <td>7.0</td>
      <td>1.714286</td>
    </tr>
    <tr>
      <td>Grocery Store</td>
      <td>7.0</td>
      <td>0.142857</td>
    </tr>
    <tr>
      <td>Gym / Fitness Center</td>
      <td>7.0</td>
      <td>0.142857</td>
    </tr>
    <tr>
      <td>Historic Site</td>
      <td>7.0</td>
      <td>0.142857</td>
    </tr>
    <tr>
      <td>History Museum</td>
      <td>7.0</td>
      <td>0.285714</td>
    </tr>
    <tr>
      <td>Hotel Bar</td>
      <td>7.0</td>
      <td>0.142857</td>
    </tr>
    <tr>
      <td>Indie Movie Theater</td>
      <td>7.0</td>
      <td>0.142857</td>
    </tr>
    <tr>
      <td>Italian Restaurant</td>
      <td>7.0</td>
      <td>0.142857</td>
    </tr>
    <tr>
      <td>Movie Theater</td>
      <td>7.0</td>
      <td>0.428571</td>
    </tr>
    <tr>
      <td>Park</td>
      <td>7.0</td>
      <td>2.142857</td>
    </tr>
    <tr>
      <td>Performing Arts Venue</td>
      <td>7.0</td>
      <td>0.142857</td>
    </tr>
    <tr>
      <td>Pet Store</td>
      <td>7.0</td>
      <td>0.142857</td>
    </tr>
    <tr>
      <td>Pizza Place</td>
      <td>7.0</td>
      <td>1.285714</td>
    </tr>
    <tr>
      <td>Playground</td>
      <td>7.0</td>
      <td>0.285714</td>
    </tr>
    <tr>
      <td>Plaza</td>
      <td>7.0</td>
      <td>0.428571</td>
    </tr>
    <tr>
      <td>Pool</td>
      <td>7.0</td>
      <td>0.285714</td>
    </tr>
    <tr>
      <td>Public Art</td>
      <td>7.0</td>
      <td>0.142857</td>
    </tr>
    <tr>
      <td>Recreation Center</td>
      <td>7.0</td>
      <td>0.142857</td>
    </tr>
    <tr>
      <td>River</td>
      <td>7.0</td>
      <td>0.142857</td>
    </tr>
    <tr>
      <td>Street Art</td>
      <td>7.0</td>
      <td>0.142857</td>
    </tr>
    <tr>
      <td>Strip Club</td>
      <td>7.0</td>
      <td>0.142857</td>
    </tr>
    <tr>
      <td>Theme Park</td>
      <td>7.0</td>
      <td>0.142857</td>
    </tr>
    <tr>
      <td>Zoo Exhibit</td>
      <td>7.0</td>
      <td>0.142857</td>
    </tr>
  </tbody>
</table>
</div>

> Cluster 10:

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
      <th>count</th>
      <th>mean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ratio</td>
      <td>6.0</td>
      <td>0.321004</td>
    </tr>
    <tr>
      <td>Art Gallery</td>
      <td>6.0</td>
      <td>0.500000</td>
    </tr>
    <tr>
      <td>Bakery</td>
      <td>6.0</td>
      <td>0.500000</td>
    </tr>
    <tr>
      <td>Bar</td>
      <td>6.0</td>
      <td>0.333333</td>
    </tr>
    <tr>
      <td>Bowling Green</td>
      <td>6.0</td>
      <td>0.166667</td>
    </tr>
    <tr>
      <td>Breakfast Spot</td>
      <td>6.0</td>
      <td>0.166667</td>
    </tr>
    <tr>
      <td>Bridge</td>
      <td>6.0</td>
      <td>0.333333</td>
    </tr>
    <tr>
      <td>Cafeteria</td>
      <td>6.0</td>
      <td>0.166667</td>
    </tr>
    <tr>
      <td>Café</td>
      <td>6.0</td>
      <td>8.166667</td>
    </tr>
    <tr>
      <td>Coffee Shop</td>
      <td>6.0</td>
      <td>1.666667</td>
    </tr>
    <tr>
      <td>Design Studio</td>
      <td>6.0</td>
      <td>0.166667</td>
    </tr>
    <tr>
      <td>Dog Run</td>
      <td>6.0</td>
      <td>0.333333</td>
    </tr>
    <tr>
      <td>Exhibit</td>
      <td>6.0</td>
      <td>0.166667</td>
    </tr>
    <tr>
      <td>Football Stadium</td>
      <td>6.0</td>
      <td>0.166667</td>
    </tr>
    <tr>
      <td>Furniture / Home Store</td>
      <td>6.0</td>
      <td>0.166667</td>
    </tr>
    <tr>
      <td>Garden</td>
      <td>6.0</td>
      <td>0.833333</td>
    </tr>
    <tr>
      <td>Garden Center</td>
      <td>6.0</td>
      <td>0.166667</td>
    </tr>
    <tr>
      <td>General Entertainment</td>
      <td>6.0</td>
      <td>0.666667</td>
    </tr>
    <tr>
      <td>Ice Cream Shop</td>
      <td>6.0</td>
      <td>0.166667</td>
    </tr>
    <tr>
      <td>Japanese Restaurant</td>
      <td>6.0</td>
      <td>0.166667</td>
    </tr>
    <tr>
      <td>Park</td>
      <td>6.0</td>
      <td>5.000000</td>
    </tr>
    <tr>
      <td>Pizza Place</td>
      <td>6.0</td>
      <td>1.500000</td>
    </tr>
    <tr>
      <td>Playground</td>
      <td>6.0</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <td>Pool</td>
      <td>6.0</td>
      <td>0.500000</td>
    </tr>
    <tr>
      <td>Pool Hall</td>
      <td>6.0</td>
      <td>0.166667</td>
    </tr>
    <tr>
      <td>Sandwich Place</td>
      <td>6.0</td>
      <td>0.166667</td>
    </tr>
    <tr>
      <td>Soccer Field</td>
      <td>6.0</td>
      <td>0.500000</td>
    </tr>
    <tr>
      <td>Sports Club</td>
      <td>6.0</td>
      <td>0.166667</td>
    </tr>
    <tr>
      <td>Stadium</td>
      <td>6.0</td>
      <td>0.166667</td>
    </tr>
    <tr>
      <td>Strip Club</td>
      <td>6.0</td>
      <td>0.166667</td>
    </tr>
    <tr>
      <td>Tennis Stadium</td>
      <td>6.0</td>
      <td>0.166667</td>
    </tr>
    <tr>
      <td>Theme Park</td>
      <td>6.0</td>
      <td>0.166667</td>
    </tr>
    <tr>
      <td>Trail</td>
      <td>6.0</td>
      <td>0.500000</td>
    </tr>
    <tr>
      <td>Vegetarian / Vegan Restaurant</td>
      <td>6.0</td>
      <td>0.166667</td>
    </tr>
  </tbody>
</table>
</div>

> Cluster 11:

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
      <th>count</th>
      <th>mean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ratio</td>
      <td>1.0</td>
      <td>0.154968</td>
    </tr>
    <tr>
      <td>Bar</td>
      <td>1.0</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <td>Café</td>
      <td>1.0</td>
      <td>3.000000</td>
    </tr>
    <tr>
      <td>Coffee Shop</td>
      <td>1.0</td>
      <td>12.000000</td>
    </tr>
    <tr>
      <td>Donut Shop</td>
      <td>1.0</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <td>Food Court</td>
      <td>1.0</td>
      <td>2.000000</td>
    </tr>
    <tr>
      <td>General Entertainment</td>
      <td>1.0</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <td>Juice Bar</td>
      <td>1.0</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <td>Library</td>
      <td>1.0</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <td>Movie Theater</td>
      <td>1.0</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <td>Multiplex</td>
      <td>1.0</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <td>Plaza</td>
      <td>1.0</td>
      <td>2.000000</td>
    </tr>
    <tr>
      <td>Pub</td>
      <td>1.0</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <td>Shopping Mall</td>
      <td>1.0</td>
      <td>3.000000</td>
    </tr>
  </tbody>
</table>
</div>

> Cluster 12:

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
      <th>count</th>
      <th>mean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ratio</td>
      <td>12.0</td>
      <td>0.311555</td>
    </tr>
    <tr>
      <td>Art Gallery</td>
      <td>12.0</td>
      <td>0.333333</td>
    </tr>
    <tr>
      <td>Australian Restaurant</td>
      <td>12.0</td>
      <td>0.083333</td>
    </tr>
    <tr>
      <td>Bakery</td>
      <td>12.0</td>
      <td>0.166667</td>
    </tr>
    <tr>
      <td>Bar</td>
      <td>12.0</td>
      <td>0.083333</td>
    </tr>
    <tr>
      <td>Beach</td>
      <td>12.0</td>
      <td>0.083333</td>
    </tr>
    <tr>
      <td>Breakfast Spot</td>
      <td>12.0</td>
      <td>0.166667</td>
    </tr>
    <tr>
      <td>Bridge</td>
      <td>12.0</td>
      <td>0.416667</td>
    </tr>
    <tr>
      <td>Café</td>
      <td>12.0</td>
      <td>14.750000</td>
    </tr>
    <tr>
      <td>Coffee Shop</td>
      <td>12.0</td>
      <td>1.166667</td>
    </tr>
    <tr>
      <td>Dance Studio</td>
      <td>12.0</td>
      <td>0.083333</td>
    </tr>
    <tr>
      <td>Deli / Bodega</td>
      <td>12.0</td>
      <td>0.166667</td>
    </tr>
    <tr>
      <td>Dessert Shop</td>
      <td>12.0</td>
      <td>0.083333</td>
    </tr>
    <tr>
      <td>Dog Run</td>
      <td>12.0</td>
      <td>0.166667</td>
    </tr>
    <tr>
      <td>Fast Food Restaurant</td>
      <td>12.0</td>
      <td>0.083333</td>
    </tr>
    <tr>
      <td>Food Court</td>
      <td>12.0</td>
      <td>0.083333</td>
    </tr>
    <tr>
      <td>Football Stadium</td>
      <td>12.0</td>
      <td>0.083333</td>
    </tr>
    <tr>
      <td>French Restaurant</td>
      <td>12.0</td>
      <td>0.083333</td>
    </tr>
    <tr>
      <td>Garden</td>
      <td>12.0</td>
      <td>0.416667</td>
    </tr>
    <tr>
      <td>General Entertainment</td>
      <td>12.0</td>
      <td>0.750000</td>
    </tr>
    <tr>
      <td>Historic Site</td>
      <td>12.0</td>
      <td>0.166667</td>
    </tr>
    <tr>
      <td>Indie Movie Theater</td>
      <td>12.0</td>
      <td>0.083333</td>
    </tr>
    <tr>
      <td>Indoor Play Area</td>
      <td>12.0</td>
      <td>0.083333</td>
    </tr>
    <tr>
      <td>Italian Restaurant</td>
      <td>12.0</td>
      <td>0.166667</td>
    </tr>
    <tr>
      <td>Lake</td>
      <td>12.0</td>
      <td>0.166667</td>
    </tr>
    <tr>
      <td>Park</td>
      <td>12.0</td>
      <td>1.583333</td>
    </tr>
    <tr>
      <td>Pizza Place</td>
      <td>12.0</td>
      <td>1.583333</td>
    </tr>
    <tr>
      <td>Playground</td>
      <td>12.0</td>
      <td>0.916667</td>
    </tr>
    <tr>
      <td>Plaza</td>
      <td>12.0</td>
      <td>0.250000</td>
    </tr>
    <tr>
      <td>Pool</td>
      <td>12.0</td>
      <td>0.250000</td>
    </tr>
    <tr>
      <td>Pool Hall</td>
      <td>12.0</td>
      <td>0.166667</td>
    </tr>
    <tr>
      <td>Radio Station</td>
      <td>12.0</td>
      <td>0.083333</td>
    </tr>
    <tr>
      <td>Recreation Center</td>
      <td>12.0</td>
      <td>0.166667</td>
    </tr>
    <tr>
      <td>River</td>
      <td>12.0</td>
      <td>0.166667</td>
    </tr>
    <tr>
      <td>Sports Club</td>
      <td>12.0</td>
      <td>0.083333</td>
    </tr>
    <tr>
      <td>Trail</td>
      <td>12.0</td>
      <td>0.416667</td>
    </tr>
    <tr>
      <td>Turkish Restaurant</td>
      <td>12.0</td>
      <td>0.083333</td>
    </tr>
  </tbody>
</table>
</div>

## Results and Discussion

Looking at the data of each cluster, we can get some insight to share with the experts that will run this program.

Clusters with suburbs showing a mean of 0.3 or more tell us that 30% or more of the families there have children; these are prospective candidates to work with. If the suburbs don't have enough venues or an irregular distribution among the types of venues, they are definitely candidates for the work of the local government.

It's important to note the limitations of the analysis in this study:
> We only get venues in a radius of 500 meters from the centre of the suburb and limit the search to only 30 venues. A substantial improvement would be not to limit the number of venues we can gather. Also, to request the venues in a radius big enough to go out of each suburb, then filter out those venues not located in the suburb by matching the address.

> The elbow method to select the number of clusters is not the most appropriate way to select groups of clusters in this case. There is no evident change in the curve of the graph that indicates an optimal number. This may be due to the high diversity of data among the suburbs in Melbourne. A more realistic approach would be for the local government to define how many groups they want to work with, depending on budget, resources and time to tackle the field studies and projects. It can be 13, as shown here or a different number. Having said that, it is also possible to play with KElbowVisualizer – silhouette and calinski_harabasz. For more information on those algorithms check this <a href="https://www.scikit-yb.org/en/latest/api/cluster/elbow.html">link</a>.

> Another point to be aware of is that this study was done only for the SA4 area 'Melbourne - Inner' which is the most populated of the SA4 areas conforming Melbourne. However, the local government will be interested in replicating this study to the rest of the suburbs of the city: *Melbourne - North West, Melbourne - West, Melbourne - North East, Melbourne - Inner East, Melbourne - South East, Melbourne - Inner South and Melbourne - Outer East*.

One possible approach to minimise the limitations is to select three different number of clusters as long as they're suitable for the work. Then, build three reports for the local government to choose the most feasible for the task.

## Conclusion

The study presented here is a good start point for the local government of Melbourne to solve the problem of suitable venues for families with children. With a full licence of Foursquare to request data about venues, plus gathering information from government archives and other online services like Google Places, the dataset of venues would be much more valuable to yield better outcome.

It is critical to have discussions with stakeholders and contractors in charge of the project and logistics to define what number of clusters is the most appropriate.

After having a plan base on this study, further discussion with local councils can be of great feedback to make it more accurate and find the best possible distribution of the local resources in the area.

### Thank you for reviewing this work!

This post was created by [Israel Tiomno](https://www.linkedin.com/in/tiomno/)
