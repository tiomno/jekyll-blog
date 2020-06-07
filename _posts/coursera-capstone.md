---
layout: post
title: Family-friendly Neighbourhoods in Melbourne, AU
permalink: /coursera-capstone/
---

Next you can update your site name, avatar and other options using the _config.yml file in the root of your repository (shown below).

![_config.yml]({{ site.baseurl }}/images/config.png)

The easiest way to make your first post is to edit this one. Go into /_posts/ and update the Hello World markdown file. For more instructions head over to the [Jekyll Now repository](https://github.com/barryclark/jekyll-now) on GitHub.

# Family-friendly Neighbourhoods in Melbourne, AU

## Table of contents
* [Introduction](#introduction)
* [Data Description](#description)
* [Data Preparation](#preparation)
* [Methodology](#methodology)
* [Analysis](#analysis)
* [Results and Discussion](#results)
* [Conclusion](#conclusion)

## Introduction <a name="introduction"></a>

The local government of Melbourne in Australia wants to compile information about the venues and facilities suitable for families with children under 15 years old. The final objective is to improve local communities' public areas for families with children. Thus the goal of this study is to find which suburbs need more resources for this purpose.

The analysis will use data from the Australian Bureau of Statistics (<a href="https://www.abs.gov.au/">ABS</a>) to compile data about families per suburb and the API of <a href="https://foursquare.com/">Foursquare</a> to gather the venues of each locality.

## Data Description <a name="description"></a>

The first dataset downloaded from ABS has aggregations of family composition and other statistics grouped by the suburbs of the State of Victoria in Australia. It comes with several columns, but we will use the three we need to extract information about families. These columns gives us the number of couple families with children under 15 years `CF_ChU15_a_Total_F`, one-parent families with children under 15 years `OPF_ChU15_a_Total_F` and the total of families `Total_F`.

Each row of this dataset is the aggregation data per suburb with an ID of 5 digits named SSC (State Suburb Code) and assigned by the ABS.

An example of the data looks like this:

| SSC_CODE_2016 | ...  | CF_ChU15_a_Total_F | ...  | OPF_ChU15_a_Total_F | ... | Total_F | ...  |
|---------------|------|--------------------|------|---------------------|-----|---------|------|
| ...           | ...  | ...                | ...  | ...                 | ... | ...     | ...  |
| SSC20002      | ...  | 353                | ...  | 58                  | ... | 1886    | ...  |
| SSC20003      | ...  | 356                | ...  | 39                  | ... | 1034    | ...  |
| ...           | ...  | ...                | ...  | ...                 | ... | ...     | ...  |

We can get a ratio of families with children per suburb with the equation:

> (`CF_ChU15_a_Total_F` + `OPF_ChU15_a_Total_F`) / `Total_F`

The Victorian government also provided a dataset `CG_SSC_2016_SA4_2016.csv` with the correspondence between SSC and SA4, which are large statistic areas defined by ABS. Those that have 'Melbourne' as part of its name `SA4_NAME_2016` represent the areas of the city with their corresponding suburbs. With this dataset, we can filter out from the previous one any suburb out of the Melbourne area. This dataset looks like this:

| SSC_CODE_2016 | SSC_NAME_2016       | SA4_CODE_2016 | SA4_NAME_2016          | RATIO | PERCENTAGE |
|---------------|---------------------|---------------|------------------------|-------|------------|
| 20172         | Bayswater (Vic.)    | 211           | Melbourne - Outer East | 1     | 100        |
| 20173         | Bayswater North     | 211           | Melbourne - Outer East | 1     | 100        |
| 20174         | Beaconsfield (Vic.) | 212           | Melbourne - South East | 1     | 100        |
| 20175         | Beaconsfield Upper  | 212           | Melbourne - South East | 1     | 100        |
| 20176         | Bealiba             | 201           | Ballarat               | 1     | 100        |
| 20177         | Bearii              | 216           | Shepparton             | 1     | 100        |
| 20178         | Bears Lagoon        | 202           | Bendigo                | 1     | 100        |

Now we will be able to go through each suburb and find the venues that are suitable for families with children. To only get those venues we are interested in, we will pass 50 categories chosen from the full list of Foursquare categories <a href="https://developer.foursquare.com/docs/resources/categories">here</a>.

At this point, we will know how many venues per category every suburb of Melbourne has. Adding this information to the ratio of families, we can run k-means clustarting starting with 5 clusters until we find the optimum value for k using the elbow method. When we have the best k parameter, we can finally get that many clusters of suburbs for the government to analyse and support depending on the outcome and classification.

#### Final recommendations on how to interpret the final result

- Suburbs with a low ratio of families with children should be of low priority.
- Suburbs with a high ratio of families with children and a high number of suitable venues should be of low priority to create new facilities. The focus here should be on maintenance of the current infrastructure and businesses.
- Suburbs with a high ratio of families with children and a low number of suitable venues should be the focus of the government plan and give top priority to these areas.

## Data Preparation <a name="preparation"></a>

The first step is to download the demographics data from ABS with the aggregations of families with children in the State of Victoria. With this dataset, we want to obtain only the data for Melbourne and transform the numbers to use only the ratio of families per suburb.


```python
import pandas as pd

df_families = pd.read_csv('https://raw.githubusercontent.com/tiomno/Coursera_Capstone/master/datasets/2016Census_G25_VIC_SSC.csv')
df_families.head()
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
df_families.head()
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




```python
# Obtain the correspondence between ABS larger areas SA4 and pick only the suburbs in and around the CBD of Melbourne

df_correspondence = pd.read_csv('https://raw.githubusercontent.com/tiomno/Coursera_Capstone/master/datasets/CG_SSC_2016_SA4_2016.csv')
df_correspondence.head()
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




```python
# Get only the list of suburbs from Melbourne
df_melbourne = df_correspondence[df_correspondence['SA4_NAME_2016'].str.contains('Melbourne - Inner')][['SSC_CODE_2016', 'SSC_NAME_2016']]
df_melbourne.columns = ['ssc', 'name']
df_melbourne.head()
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




```python
# Get rid of any suburb from the df_families other than Melbourne's
print('Families dataframe shape: {}'.format(df_families.shape))
print('Melbourne dataframe shape: {}'.format(df_melbourne.shape))
df_melbourne_families = pd.merge(df_families, df_melbourne, on=['ssc'])
print('Melbourne Families dataframe shape: {}'.format(df_melbourne_families.shape))
df_melbourne_families.head()
```

    Families dataframe shape: (2717, 2)
    Melbourne dataframe shape: (126, 2)
    Melbourne Families dataframe shape: (125, 3)





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



Then we can load the categories we will pass to Foursquare to search for venues suitable for families with children.


```python
# Load the Foursquare categories in pandas
df_categories = pd.read_csv('https://raw.githubusercontent.com/tiomno/Coursera_Capstone/master/datasets/Foursquare_categories.csv')
df_categories.head()
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




```python
# And create a comma separated string with the category IDs to pass in the Query String of the Foursquare URL request.
CATEGORIES = ','.join(df_categories['Category ID'].values.tolist())
CATEGORIES
```




    '4fceea171983d5d06c3e9823,4bf58dd8d48988d1e2931735,56aa371be4b08b9a8d573532,4bf58dd8d48988d1f1931735,4deefb944765f83613cdba6e,5642206c498e4bfca532186c,4bf58dd8d48988d17f941735,4bf58dd8d48988d181941735,4bf58dd8d48988d1e3931735,507c8c4091d498d9fc8c67a9,4bf58dd8d48988d184941735,4bf58dd8d48988d182941735,4bf58dd8d48988d193941735,4bf58dd8d48988d17b941735,5267e4d9e4b0ec79466e48c7,52741d85e4b0d5d1e3c6a6d9,5bae9231bedf3950379f89c5,5267e4d8e4b0ec79466e48c5,5bae9231bedf3950379f89c3,4bf58dd8d48988d128941735,4bf58dd8d48988d16d941735,4bf58dd8d48988d1e0931735,4bf58dd8d48988d120951735,4bf58dd8d48988d1ca941735,52e81612bcbc57f1066b7a2c,4cce455aebf7b749d5e191f5,52e81612bcbc57f1066b7a2e,52e81612bcbc57f1066b7a28,56aa371be4b08b9a8d573544,4bf58dd8d48988d1e2941735,56aa371be4b08b9a8d57355e,52e81612bcbc57f1066b7a22,4bf58dd8d48988d1df941735,4bf58dd8d48988d1e5941735,52e81612bcbc57f1066b7a23,56aa371be4b08b9a8d573547,4bf58dd8d48988d15a941735,5744ccdfe4b0c0459246b4b5,4bf58dd8d48988d161941735,52e81612bcbc57f1066b7a21,52e81612bcbc57f1066b7a13,4bf58dd8d48988d163941735,4bf58dd8d48988d1e7941735,4bf58dd8d48988d164941735,4bf58dd8d48988d15e941735,52e81612bcbc57f1066b7a26,56aa371be4b08b9a8d573541,4eb1d4dd4b900d56c88a45fd,4bf58dd8d48988d159941735,56aa371be4b08b9a8d5734c3'



Now we need to get the geolocation for each suburb. For this, we can use geopy.


```python
#!conda install -c conda-forge geopy --yes # uncomment this if geopy needs to be install in the environment
from geopy.geocoders import Nominatim # convert an address into latitude and longitude values
```

The following code is commented out not to gather the geolocation again as it may take a few minutes. At the end, the results were saved in a CSV file for further processing.


```python
# geolocator = Nominatim(user_agent="ny_explorer")

# # Define a function that gets the geolocation of a suburb.
# def get_suburb_geolocation(suburb, geolocation):
#     try:
#         if geolocation[0] is None:
#             location = geolocator.geocode(suburb + ',VIC,Australia')
#             if location is None:
#                 print('x ', end='')
#                 return None, None
#             else:
#                 print('o ', end='')
#                 return location.latitude, location.longitude
#         else:
#             return geolocation
#     except:
#         return None, None

# # Go through the dataframe of suburbs and add the geolocation
# # Repeat the preocess until all the suburb are filled in
# # Then, save it to a CSV file for further use.
# df_melbourne_families["geolocation"] = [(None, None)] * len(df_melbourne_families)
# i = 0
# while len(df_melbourne_families[df_melbourne_families['geolocation'] == (None, None)]) > 0 and i < 20:
#     df_melbourne_families['geolocation'] = df_melbourne_families.apply(
#         lambda df: get_suburb_geolocation(df['name'], df['geolocation']), axis=1
#     )
#     i += 1

# df_melbourne_families.to_csv('df_melbourne_families.csv')
```


```python
df_melbourne_families = pd.read_csv('https://raw.githubusercontent.com/tiomno/Coursera_Capstone/master/datasets/df_melbourne_families.csv')
# Remove SSC duplication due to the correspondence with SA4 areas
bool_series = df_melbourne_families["ssc"].duplicated() 
df_melbourne_families = df_melbourne_families[~bool_series] 
df_melbourne_families.head() 
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



With the suburbs geolocation we can request Foursquare to gather the venues suitable for families with children.


```python
# Foursquare parameters
CLIENT_ID = 'Foursquare API Client ID here'
CLIENT_SECRET = 'Foursquare API Secret here'
VERSION = '20180604'
LIMIT = 30
RADIUS = 500

url_template = 'https://api.foursquare.com/v2/venues/search?client_id={}&client_secret={}&ll={}&v={}&categoryId={}&radius={}&limit={}'.format(CLIENT_ID, CLIENT_SECRET, '{}', VERSION, CATEGORIES, RADIUS, LIMIT)
url_template
```




    'https://api.foursquare.com/v2/venues/search?client_id=Foursquare API Client ID here&client_secret=Foursquare API Secret here&ll={}&v=20180604&categoryId=4fceea171983d5d06c3e9823,4bf58dd8d48988d1e2931735,56aa371be4b08b9a8d573532,4bf58dd8d48988d1f1931735,4deefb944765f83613cdba6e,5642206c498e4bfca532186c,4bf58dd8d48988d17f941735,4bf58dd8d48988d181941735,4bf58dd8d48988d1e3931735,507c8c4091d498d9fc8c67a9,4bf58dd8d48988d184941735,4bf58dd8d48988d182941735,4bf58dd8d48988d193941735,4bf58dd8d48988d17b941735,5267e4d9e4b0ec79466e48c7,52741d85e4b0d5d1e3c6a6d9,5bae9231bedf3950379f89c5,5267e4d8e4b0ec79466e48c5,5bae9231bedf3950379f89c3,4bf58dd8d48988d128941735,4bf58dd8d48988d16d941735,4bf58dd8d48988d1e0931735,4bf58dd8d48988d120951735,4bf58dd8d48988d1ca941735,52e81612bcbc57f1066b7a2c,4cce455aebf7b749d5e191f5,52e81612bcbc57f1066b7a2e,52e81612bcbc57f1066b7a28,56aa371be4b08b9a8d573544,4bf58dd8d48988d1e2941735,56aa371be4b08b9a8d57355e,52e81612bcbc57f1066b7a22,4bf58dd8d48988d1df941735,4bf58dd8d48988d1e5941735,52e81612bcbc57f1066b7a23,56aa371be4b08b9a8d573547,4bf58dd8d48988d15a941735,5744ccdfe4b0c0459246b4b5,4bf58dd8d48988d161941735,52e81612bcbc57f1066b7a21,52e81612bcbc57f1066b7a13,4bf58dd8d48988d163941735,4bf58dd8d48988d1e7941735,4bf58dd8d48988d164941735,4bf58dd8d48988d15e941735,52e81612bcbc57f1066b7a26,56aa371be4b08b9a8d573541,4eb1d4dd4b900d56c88a45fd,4bf58dd8d48988d159941735,56aa371be4b08b9a8d5734c3&radius=500&limit=30'



Let's add the venue categories as columns to the Melbourne families dataframe. Then, fill in the number of venues per category in the neighbourhood.

The following code is commented out not to request Foursquare as it's a bit time consuming. At the end, the results were saved in a CSV file for further processing.


```python
# import requests # library to handle requests

# # Add new categories as clumns to df_melbourne_families
# for category in df_categories['Category Name']:
#     df_melbourne_families[category] = 0

# def get_category_venues(ssc, geolocation):
#     lst_venues=[]

#     for ssc, location in zip(ssc, geolocation):
#         lat_lng = location.strip('()')
            
#         # create the API request URL
#         url = url_template.format(lat_lng)
            
#         # make the GET request
#         venues = requests.get(url).json()["response"]['venues']
        
#         # Append ssc and category to the venues list to group them in further steps
#         lst_venues.append([
#             (ssc, venue['categories'][0]['name']) for venue in venues
#         ])

#     return pd.DataFrame([item for lst in lst_venues for item in lst], columns=['ssc', 'category'])

    
# df_categories_ssc = get_category_venues(
#     ssc=df_melbourne_families['ssc'],
#     geolocation=df_melbourne_families['geolocation']
# )
# df_categories_ssc.to_csv('categories_ssc.csv', index=False)
```


```python
df_categories_ssc = pd.read_csv('https://raw.githubusercontent.com/tiomno/Coursera_Capstone/master/datasets/categories_ssc.csv')
df_categories_ssc.head()
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




```python
# Group categories by SSC (suburb) and get the ratio from the total.

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
df_categories_ssc.head()
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
      <th>category</th>
      <th>ssc</th>
      <th>American Restaurant</th>
      <th>Art Gallery</th>
      <th>Art Museum</th>
      <th>Art Studio</th>
      <th>Asian Restaurant</th>
      <th>Athletics &amp; Sports</th>
      <th>Australian Restaurant</th>
      <th>Bakery</th>
      <th>Bar</th>
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
      <td>0</td>
      <td>3</td>
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
      <td>20003</td>
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
      <td>2</td>
      <td>20017</td>
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
      <td>3</td>
      <td>20033</td>
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
      <td>20065</td>
      <td>0</td>
      <td>3</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
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
<p>5 rows × 116 columns</p>
</div>



We'll merge the list of suburbs with the counts of venues per categories and start playing with clustering algorithm.


```python
df_melbourne_venues = pd.merge(df_melbourne_families, df_categories_ssc, on=['ssc'])
df_melbourne_venues.head()
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



## Methodology <a name="methodology"></a>

With the necessary data at hand, we can use a clustering algorithm to identify a pattern in the dataset that shows us the relation between the family ratio and the venues suitable for families in a suburb.

We can use K-means ML algorithm for this purpose and play with different numbers of clusters until we get a value suitable for the work of the local government. A small number of clusters would give us too many different suburbs in a single group which is not of much help. On the other hand, too many groups would make difficult the task of identifying patterns and allocate the necessary resources and team of experts to each group.

Clustering is the perfect solution to identify segments of suburbs in the city. Still, we needed to focus the attention on specific variables. To identify those variables, we had discussions with the local government to know which group of the population we needed to target. Then, based on the final goal, define the kind of venues suitable for families in this study.

Knowing the segment of the population to target and the categories of the venues we needed to find, we started the search for the right tools and information available. Gathering aggregations from the Australian Bureau of Statistics website, we obtained insight into the shape of the families' distribution in Melbourne. With the Foursquare API, we could search for the venues suitable for families in each suburb.

We merged this information and are ready to run the clustering algorithm and analyse the results.

## Analysis <a name="analysis"></a>

Let's try to find the optimun number of cluster from a range from 5 - 20 using the Elbow method.


```python
# source: https://pythonprogramminglanguage.com/kmeans-elbow-method/

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


![png](output_31_0.png)



```python
# Since the previous method doesn't show a clear "knee", let's apply an even further automated Elbow method
# to find the optimal cluster number.

#!conda install -c districtdatalabs yellowbrick --yes # uncomment this if yellowbrick needs to be install in the environment
from yellowbrick.cluster.elbow import kelbow_visualizer

# Use the quick method and immediately show the figure
kelbow_visualizer(KMeans(random_state=4), df_melbourne_venues_clustering, k=(5, 20))
```


![png](output_32_0.png)





    KElbowVisualizer(ax=<matplotlib.axes._subplots.AxesSubplot object at 0x7fce07767d10>,
                     k=None, locate_elbow=True, metric='distortion', model=None,
                     timings=True)



We can pick 13 clusters and continue with the anlysis of Melbourne suburbs.
> note: see Results and Discussion section at the bottom of this notebook for thoughts about this method.


```python
# set number of clusters
kclusters = 13

# run k-means clustering
kmeans = KMeans(n_clusters=kclusters, random_state=0).fit(df_melbourne_venues_clustering)

# check cluster labels generated for each row in the dataframe
kmeans.labels_[0:10]
```




    array([ 9,  6, 12,  4,  5,  4,  4,  6,  6,  9], dtype=int32)




```python
# add clustering labels to the Melbourne venues dataframe
df_melbourne_venues.insert(0, 'Cluster Labels', kmeans.labels_)
df_melbourne_venues.head()
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



Showing the clusters on a map will give us an idea of the distribution by kind of venues.


```python
#!conda install -c conda-forge folium=0.5.0 --yes # uncomment this if folium needs to be install in the environment
import folium # map rendering library

# Matplotlib and associated plotting modules
import matplotlib.cm as cm
import matplotlib.colors as colors
```


```python
geolocator = Nominatim(user_agent="ny_explorer")
location = geolocator.geocode('Melbourne, Australia')
latitude = location.latitude
longitude = location.longitude
print('The geograpical coordinates of Melbourne are {}, {}.'.format(latitude, longitude))

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

    The geograpical coordinates of Melbourne are -37.8142176, 144.9631608.





<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe src="about:blank" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" data-html=PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgPHNjcmlwdD5MX1BSRUZFUl9DQU5WQVMgPSBmYWxzZTsgTF9OT19UT1VDSCA9IGZhbHNlOyBMX0RJU0FCTEVfM0QgPSBmYWxzZTs8L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2FqYXguZ29vZ2xlYXBpcy5jb20vYWpheC9saWJzL2pxdWVyeS8xLjExLjEvanF1ZXJ5Lm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvanMvYm9vdHN0cmFwLm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9jZG5qcy5jbG91ZGZsYXJlLmNvbS9hamF4L2xpYnMvTGVhZmxldC5hd2Vzb21lLW1hcmtlcnMvMi4wLjIvbGVhZmxldC5hd2Vzb21lLW1hcmtlcnMuanMiPjwvc2NyaXB0PgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvY3NzL2Jvb3RzdHJhcC10aGVtZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vZm9udC1hd2Vzb21lLzQuNi4zL2Nzcy9mb250LWF3ZXNvbWUubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9yYXdnaXQuY29tL3B5dGhvbi12aXN1YWxpemF0aW9uL2ZvbGl1bS9tYXN0ZXIvZm9saXVtL3RlbXBsYXRlcy9sZWFmbGV0LmF3ZXNvbWUucm90YXRlLmNzcyIvPgogICAgPHN0eWxlPmh0bWwsIGJvZHkge3dpZHRoOiAxMDAlO2hlaWdodDogMTAwJTttYXJnaW46IDA7cGFkZGluZzogMDt9PC9zdHlsZT4KICAgIDxzdHlsZT4jbWFwIHtwb3NpdGlvbjphYnNvbHV0ZTt0b3A6MDtib3R0b206MDtyaWdodDowO2xlZnQ6MDt9PC9zdHlsZT4KICAgIAogICAgICAgICAgICA8c3R5bGU+ICNtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMgewogICAgICAgICAgICAgICAgcG9zaXRpb24gOiByZWxhdGl2ZTsKICAgICAgICAgICAgICAgIHdpZHRoIDogMTAwLjAlOwogICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICBsZWZ0OiAwLjAlOwogICAgICAgICAgICAgICAgdG9wOiAwLjAlOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICA8L3N0eWxlPgogICAgICAgIAo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzIiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGJvdW5kcyA9IG51bGw7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgdmFyIG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0MyA9IEwubWFwKAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgJ21hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0MycsCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB7Y2VudGVyOiBbLTM3LjgxNDIxNzYsMTQ0Ljk2MzE2MDhdLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgem9vbTogMTAsCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBtYXhCb3VuZHM6IGJvdW5kcywKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIGxheWVyczogW10sCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB3b3JsZENvcHlKdW1wOiBmYWxzZSwKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIGNyczogTC5DUlMuRVBTRzM4NTcKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgfSk7CiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciB0aWxlX2xheWVyXzgyZGQzZjMwZGViMjRjMmQ4MGJiNGVjNTFkM2YzMjE3ID0gTC50aWxlTGF5ZXIoCiAgICAgICAgICAgICAgICAnaHR0cHM6Ly97c30udGlsZS5vcGVuc3RyZWV0bWFwLm9yZy97en0ve3h9L3t5fS5wbmcnLAogICAgICAgICAgICAgICAgewogICJhdHRyaWJ1dGlvbiI6IG51bGwsCiAgImRldGVjdFJldGluYSI6IGZhbHNlLAogICJtYXhab29tIjogMTgsCiAgIm1pblpvb20iOiAxLAogICJub1dyYXAiOiBmYWxzZSwKICAic3ViZG9tYWlucyI6ICJhYmMiCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl82NTIzMzM3MDU0NmE0NWI0YjBjMjU2ZTI5YjdmNTk5YiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy44MDQ1NTA4LDE0NC45OTg4NTQyXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiNkNGRkODAiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjZDRkZDgwIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2JjMjIzMzYwMDI1MDQ5YjE5ZThkMDI0M2U4ZGE1MWUzID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2Q4NTJhODgzYjc1YjQ0OTdiZDBmNTUxYTY1MGFkNDZmID0gJCgnPGRpdiBpZD0iaHRtbF9kODUyYTg4M2I3NWI0NDk3YmQwZjU1MWE2NTBhZDQ2ZiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QWJib3RzZm9yZCAoVmljLikgQ2x1c3RlciA5PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9iYzIyMzM2MDAyNTA0OWIxOWU4ZDAyNDNlOGRhNTFlMy5zZXRDb250ZW50KGh0bWxfZDg1MmE4ODNiNzViNDQ5N2JkMGY1NTFhNjUwYWQ0NmYpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfNjUyMzMzNzA1NDZhNDViNGIwYzI1NmUyOWI3ZjU5OWIuYmluZFBvcHVwKHBvcHVwX2JjMjIzMzYwMDI1MDQ5YjE5ZThkMDI0M2U4ZGE1MWUzKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzA0NjQ0ODRkYzMxMjRmMjVhYmNjYmI4ZjZjZTNhYmUxID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3Ljc1OTYxOTYsMTQ0Ljg5NzQ1NzFdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzU0ZjZjYiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM1NGY2Y2IiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfOGYwMTUzM2ZkOTZlNDkwNDhhN2EzOWU4MmZiZDZmNzUgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMjBhOGUwMWFlZTM1NGY4YWJiZGIwOTQ1Mjg3Y2ZlM2QgPSAkKCc8ZGl2IGlkPSJodG1sXzIwYThlMDFhZWUzNTRmOGFiYmRiMDk0NTI4N2NmZTNkIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5BYmVyZmVsZGllIENsdXN0ZXIgNjwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfOGYwMTUzM2ZkOTZlNDkwNDhhN2EzOWU4MmZiZDZmNzUuc2V0Q29udGVudChodG1sXzIwYThlMDFhZWUzNTRmOGFiYmRiMDk0NTI4N2NmZTNkKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzA0NjQ0ODRkYzMxMjRmMjVhYmNjYmI4ZjZjZTNhYmUxLmJpbmRQb3B1cChwb3B1cF84ZjAxNTMzZmQ5NmU0OTA0OGE3YTM5ZTgyZmJkNmY3NSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl83NzIxYWNiYWE0ZTM0OTZiOWM2N2FkN2NjN2Y3MWZkZSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy44NDc3NzI1LDE0NC45NjIwMDc5NzE1NDA3NF0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmY0MTIxIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmNDEyMSIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF83NzZjYjIwYjI4ZTc0ZTVhYjUyZTU4MTQyMTVhY2RjNCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF80ZmVjZmNlMWQ4ZTk0MzhhOWM5M2Y4M2Y2NGVmYWRkZSA9ICQoJzxkaXYgaWQ9Imh0bWxfNGZlY2ZjZTFkOGU5NDM4YTljOTNmODNmNjRlZmFkZGUiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkFsYmVydCBQYXJrIChWaWMuKSBDbHVzdGVyIDEyPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF83NzZjYjIwYjI4ZTc0ZTVhYjUyZTU4MTQyMTVhY2RjNC5zZXRDb250ZW50KGh0bWxfNGZlY2ZjZTFkOGU5NDM4YTljOTNmODNmNjRlZmFkZGUpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfNzcyMWFjYmFhNGUzNDk2YjljNjdhZDdjYzdmNzFmZGUuYmluZFBvcHVwKHBvcHVwXzc3NmNiMjBiMjhlNzRlNWFiNTJlNTgxNDIxNWFjZGM0KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2Q5ZjVlMmE2MjE3NDQ1OGVhNWMzYzU2ZmUzNjczOTFiID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3Ljc3ODM5NTMsMTQ1LjAzMTI4MjNdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzAwYjVlYiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMwMGI1ZWIiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZDY1YWIzYTc1OWQzNDQzZmEzODI1MjNmMWViNzhmNGEgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZjY5MWUyMTU4NTc0NDk2MGJkZjc1Njk2ODZhM2E3ZjcgPSAkKCc8ZGl2IGlkPSJodG1sX2Y2OTFlMjE1ODU3NDQ5NjBiZGY3NTY5Njg2YTNhN2Y3IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5BbHBoaW5ndG9uIENsdXN0ZXIgNDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfZDY1YWIzYTc1OWQzNDQzZmEzODI1MjNmMWViNzhmNGEuc2V0Q29udGVudChodG1sX2Y2OTFlMjE1ODU3NDQ5NjBiZGY3NTY5Njg2YTNhN2Y3KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2Q5ZjVlMmE2MjE3NDQ1OGVhNWMzYzU2ZmUzNjczOTFiLmJpbmRQb3B1cChwb3B1cF9kNjVhYjNhNzU5ZDM0NDNmYTM4MjUyM2YxZWI3OGY0YSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9jNzk2ODkxZGIwYTA0NzBkOTE4MTQ5NjVlNGIxMDNjMCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy44NTY3NjE5LDE0NS4wMjA2OTA1XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiMyYWRkZGQiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMmFkZGRkIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzAzZDMxOTI0OGZiMjRhMTk5M2U1OWJiYWNmYjU4ZDQzID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzQ3NTI3N2FhNmE0ZjQ4YTlhNzQ5Y2Y4NDQyNTYwYjNhID0gJCgnPGRpdiBpZD0iaHRtbF80NzUyNzdhYTZhNGY0OGE5YTc0OWNmODQ0MjU2MGIzYSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QXJtYWRhbGUgKFZpYy4pIENsdXN0ZXIgNTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMDNkMzE5MjQ4ZmIyNGExOTkzZTU5YmJhY2ZiNThkNDMuc2V0Q29udGVudChodG1sXzQ3NTI3N2FhNmE0ZjQ4YTlhNzQ5Y2Y4NDQyNTYwYjNhKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2M3OTY4OTFkYjBhMDQ3MGQ5MTgxNDk2NWU0YjEwM2MwLmJpbmRQb3B1cChwb3B1cF8wM2QzMTkyNDhmYjI0YTE5OTNlNTliYmFjZmI1OGQ0Myk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8wN2EzNzcyZTY0ODc0MjdlOTI2MjFjM2M5MTMxNDViNSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy43NzUzMTYsMTQ0LjkyMTg0OV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjMDBiNWViIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzAwYjVlYiIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9hMmNkOGJkOThjMzE0MTdhOTM2MjE2YjlkZjE5NGU3OCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8zNzUyOTkyOGM3NjE0YzUyYjg5MGE3MTJmYmQyN2M1OCA9ICQoJzxkaXYgaWQ9Imh0bWxfMzc1Mjk5MjhjNzYxNGM1MmI4OTBhNzEyZmJkMjdjNTgiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkFzY290IFZhbGUgQ2x1c3RlciA0PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9hMmNkOGJkOThjMzE0MTdhOTM2MjE2YjlkZjE5NGU3OC5zZXRDb250ZW50KGh0bWxfMzc1Mjk5MjhjNzYxNGM1MmI4OTBhNzEyZmJkMjdjNTgpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMDdhMzc3MmU2NDg3NDI3ZTkyNjIxYzNjOTEzMTQ1YjUuYmluZFBvcHVwKHBvcHVwX2EyY2Q4YmQ5OGMzMTQxN2E5MzYyMTZiOWRmMTk0ZTc4KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzE1NjFhZTgxMWU3MzQ3Y2U4OWI5OWE1OTE3MzllZjZmID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3Ljg2MjA0NywxNDUuMDgxMjkwN10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjMDBiNWViIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzAwYjVlYiIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8zMjk5ZWY1YTZiZDk0MjBjOGRmODdhYWI0OTYyNjc2NyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF82MWM1OGJiYTNlZjk0MmU0YWU3YzBkODQyNmRmMTllMyA9ICQoJzxkaXYgaWQ9Imh0bWxfNjFjNThiYmEzZWY5NDJlNGFlN2MwZDg0MjZkZjE5ZTMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkFzaGJ1cnRvbiBDbHVzdGVyIDQ8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzMyOTllZjVhNmJkOTQyMGM4ZGY4N2FhYjQ5NjI2NzY3LnNldENvbnRlbnQoaHRtbF82MWM1OGJiYTNlZjk0MmU0YWU3YzBkODQyNmRmMTllMyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl8xNTYxYWU4MTFlNzM0N2NlODliOTlhNTkxNzM5ZWY2Zi5iaW5kUG9wdXAocG9wdXBfMzI5OWVmNWE2YmQ5NDIwYzhkZjg3YWFiNDk2MjY3NjcpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZDRmZDM1NjBmODdmNDc5MmI4NDlkOWVkNjFjMGRhNmIgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzguMDI3MjM2NSwxNDUuMTAyMTI2M10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjNTRmNmNiIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzU0ZjZjYiIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9jMzA5M2NiYjczNDk0ODcyYjE3Y2EzODdkMGQzZDk5YyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF81MTE4NGUxN2YwYzk0YmU0YjQwMzRmMjE5ODE3NGU3YyA9ICQoJzxkaXYgaWQ9Imh0bWxfNTExODRlMTdmMGM5NGJlNGI0MDM0ZjIxOTgxNzRlN2MiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkFzcGVuZGFsZSBDbHVzdGVyIDY8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2MzMDkzY2JiNzM0OTQ4NzJiMTdjYTM4N2QwZDNkOTljLnNldENvbnRlbnQoaHRtbF81MTE4NGUxN2YwYzk0YmU0YjQwMzRmMjE5ODE3NGU3Yyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9kNGZkMzU2MGY4N2Y0NzkyYjg0OWQ5ZWQ2MWMwZGE2Yi5iaW5kUG9wdXAocG9wdXBfYzMwOTNjYmI3MzQ5NDg3MmIxN2NhMzg3ZDBkM2Q5OWMpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMjNjNGNhZTRhZTVmNGMwMjhmMzg0ZDBjYjliNjk4NDIgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzguMDIyMTQzOCwxNDUuMTE5ODRdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzU0ZjZjYiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM1NGY2Y2IiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMWNiZTc4MjQwNGE5NDNhNmIxMmQxNGExZTAzNTAwMzggPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfY2EyOGVlMmFhZmYxNDI3MzhhNTk1YzdmMzUzMWQ1NDAgPSAkKCc8ZGl2IGlkPSJodG1sX2NhMjhlZTJhYWZmMTQyNzM4YTU5NWM3ZjM1MzFkNTQwIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Bc3BlbmRhbGUgR2FyZGVucyBDbHVzdGVyIDY8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzFjYmU3ODI0MDRhOTQzYTZiMTJkMTRhMWUwMzUwMDM4LnNldENvbnRlbnQoaHRtbF9jYTI4ZWUyYWFmZjE0MjczOGE1OTVjN2YzNTMxZDU0MCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl8yM2M0Y2FlNGFlNWY0YzAyOGYzODRkMGNiOWI2OTg0Mi5iaW5kUG9wdXAocG9wdXBfMWNiZTc4MjQwNGE5NDNhNmIxMmQxNGExZTAzNTAwMzgpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfYWVlMzkxNjFhNmRiNDdjNTk2YmE5ZGQwYTQzNjVhNDQgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuODY5OTIxNDUsMTQ0Ljk5MzQyNzg2NzM5MjQ4XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiNkNGRkODAiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjZDRkZDgwIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2U4MzVmMWRlMTFhZjQ0ZGU4NjJkNmU1NDNjOTIyOTg5ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2U0ZDdlMzA0NDdmMTRiYWE5NDA3ZjVhOWE3MTljODQyID0gJCgnPGRpdiBpZD0iaHRtbF9lNGQ3ZTMwNDQ3ZjE0YmFhOTQwN2Y1YTlhNzE5Yzg0MiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QmFsYWNsYXZhIChWaWMuKSBDbHVzdGVyIDk8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2U4MzVmMWRlMTFhZjQ0ZGU4NjJkNmU1NDNjOTIyOTg5LnNldENvbnRlbnQoaHRtbF9lNGQ3ZTMwNDQ3ZjE0YmFhOTQwN2Y1YTlhNzE5Yzg0Mik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9hZWUzOTE2MWE2ZGI0N2M1OTZiYTlkZDBhNDM2NWE0NC5iaW5kUG9wdXAocG9wdXBfZTgzNWYxZGUxMWFmNDRkZTg2MmQ2ZTU0M2M5MjI5ODkpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfN2MzOTAwZjIxMjllNDc3MThlODJhNTg1NTU2MDM4NzggPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuODA5MTczNywxNDUuMDgzMzY3OF0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmY0MTIxIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmNDEyMSIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9lMDJlNzU1ZTBmMzU0ZjIxOTM1Yzk4YjU1OGVhMWJjMSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8zNTY2MDRiZmViYTI0N2U5YmIwOTkxYTJlN2YxYjJhOCA9ICQoJzxkaXYgaWQ9Imh0bWxfMzU2NjA0YmZlYmEyNDdlOWJiMDk5MWEyZTdmMWIyYTgiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJhbHd5biBDbHVzdGVyIDEyPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9lMDJlNzU1ZTBmMzU0ZjIxOTM1Yzk4YjU1OGVhMWJjMS5zZXRDb250ZW50KGh0bWxfMzU2NjA0YmZlYmEyNDdlOWJiMDk5MWEyZTdmMWIyYTgpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfN2MzOTAwZjIxMjllNDc3MThlODJhNTg1NTU2MDM4NzguYmluZFBvcHVwKHBvcHVwX2UwMmU3NTVlMGYzNTRmMjE5MzVjOThiNTU4ZWExYmMxKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzdjMDhmN2ZkZWU1ODQ2YzI4NDk0ZGEyNGM3NTcxMWI4ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3Ljc5MTk1MTgsMTQ1LjA4NDIzNzJdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzU0ZjZjYiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM1NGY2Y2IiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZTZiNDNhOTQ0Y2M4NDRiYWJjNjQ2ZWI2NDE1ZjdlMzAgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfODc2ZGM1MzcwZmI0NDA4NTljM2MyN2RkYjUwYjgwNDcgPSAkKCc8ZGl2IGlkPSJodG1sXzg3NmRjNTM3MGZiNDQwODU5YzNjMjdkZGI1MGI4MDQ3IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5CYWx3eW4gTm9ydGggQ2x1c3RlciA2PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9lNmI0M2E5NDRjYzg0NGJhYmM2NDZlYjY0MTVmN2UzMC5zZXRDb250ZW50KGh0bWxfODc2ZGM1MzcwZmI0NDA4NTljM2MyN2RkYjUwYjgwNDcpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfN2MwOGY3ZmRlZTU4NDZjMjg0OTRkYTI0Yzc1NzExYjguYmluZFBvcHVwKHBvcHVwX2U2YjQzYTk0NGNjODQ0YmFiYzY0NmViNjQxNWY3ZTMwKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzg1ODJhY2FiOGU3YjQxZTViYWE5MzE5N2IxNmM5YzdhID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3Ljk4MjIxMjIsMTQ1LjAzODkwOThdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzAwYjVlYiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMwMGI1ZWIiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZDM2YmJmNjc1ZTViNDY2ZGEyYTdlYjg0ZWRlNmM5NjcgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZjIzNGY2ZmI3ZTI1NDkwN2E0MDQ0OGVlOWJiNTY4OWQgPSAkKCc8ZGl2IGlkPSJodG1sX2YyMzRmNmZiN2UyNTQ5MDdhNDA0NDhlZTliYjU2ODlkIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5CZWF1bWFyaXMgKFZpYy4pIENsdXN0ZXIgNDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfZDM2YmJmNjc1ZTViNDY2ZGEyYTdlYjg0ZWRlNmM5Njcuc2V0Q29udGVudChodG1sX2YyMzRmNmZiN2UyNTQ5MDdhNDA0NDhlZTliYjU2ODlkKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzg1ODJhY2FiOGU3YjQxZTViYWE5MzE5N2IxNmM5YzdhLmJpbmRQb3B1cChwb3B1cF9kMzZiYmY2NzVlNWI0NjZkYTJhN2ViODRlZGU2Yzk2Nyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9hZjk2YjU3ZDM5ZjA0YzZmYjdlM2Y0Yjg3ZjAzZTNkNyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy45MTc1MzU0LDE0NS4wMzY5MzQ2XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiMyYzdlZjciLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMmM3ZWY3IiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzI4YzY1YmE3ZjhkZjQ0NDQ5MDliZWJjYWQyY2U1OGJjID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzI0NmNiN2I5NWQyZTQ1MWE4OTczYjhmYTcwMTllZDRhID0gJCgnPGRpdiBpZD0iaHRtbF8yNDZjYjdiOTVkMmU0NTFhODk3M2I4ZmE3MDE5ZWQ0YSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QmVudGxlaWdoIENsdXN0ZXIgMzwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMjhjNjViYTdmOGRmNDQ0NDkwOWJlYmNhZDJjZTU4YmMuc2V0Q29udGVudChodG1sXzI0NmNiN2I5NWQyZTQ1MWE4OTczYjhmYTcwMTllZDRhKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2FmOTZiNTdkMzlmMDRjNmZiN2UzZjRiODdmMDNlM2Q3LmJpbmRQb3B1cChwb3B1cF8yOGM2NWJhN2Y4ZGY0NDQ0OTA5YmViY2FkMmNlNThiYyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9mMjYzMGJjMGM0ZGI0ZDVhOTc2Y2Q2YjM0MjBkODAwNiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy45MjIwMjAxLDE0NS4wNjY1NTI5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiM1NGY2Y2IiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjNTRmNmNiIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzhlZWMzYjIzMzQ5NTQ2YzdiY2M1ODAzMjRhOWYzYTI0ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2NiNGNhZGRmOTY0NjQ4YjI5ZThjMGM0NjI5ODk3Zjg0ID0gJCgnPGRpdiBpZD0iaHRtbF9jYjRjYWRkZjk2NDY0OGIyOWU4YzBjNDYyOTg5N2Y4NCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QmVudGxlaWdoIEVhc3QgQ2x1c3RlciA2PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF84ZWVjM2IyMzM0OTU0NmM3YmNjNTgwMzI0YTlmM2EyNC5zZXRDb250ZW50KGh0bWxfY2I0Y2FkZGY5NjQ2NDhiMjllOGMwYzQ2Mjk4OTdmODQpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfZjI2MzBiYzBjNGRiNGQ1YTk3NmNkNmIzNDIwZDgwMDYuYmluZFBvcHVwKHBvcHVwXzhlZWMzYjIzMzQ5NTQ2YzdiY2M1ODAzMjRhOWYzYTI0KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzlkNzk3ZGNmMTBiMTRlNGVhZTY3YzFmOGIyM2IwNWFkID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3LjgyMDA4MzU1LDE0NS4xNTAwMjExNzgzNTYyN10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjMmM3ZWY3IiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzJjN2VmNyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9kYzBhZGYzZDRkOWM0NjE5OTViZGYxY2VmMzQ3NDIxOCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9mNzE2YTMxODgxODk0MGIwYTgxMTJkYTc1YzRmYTkyYiA9ICQoJzxkaXYgaWQ9Imh0bWxfZjcxNmEzMTg4MTg5NDBiMGE4MTEyZGE3NWM0ZmE5MmIiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJsYWNrYnVybiBDbHVzdGVyIDM8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2RjMGFkZjNkNGQ5YzQ2MTk5NWJkZjFjZWYzNDc0MjE4LnNldENvbnRlbnQoaHRtbF9mNzE2YTMxODgxODk0MGIwYTgxMTJkYTc1YzRmYTkyYik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl85ZDc5N2RjZjEwYjE0ZTRlYWU2N2MxZjhiMjNiMDVhZC5iaW5kUG9wdXAocG9wdXBfZGMwYWRmM2Q0ZDljNDYxOTk1YmRmMWNlZjM0NzQyMTgpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZDYzMmRlNDg2MzIxNGJmMzg3NDQxMTdhNmE4ZGQ2MzYgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuODM4ODMyMiwxNDUuMTQ4Nzg0OF0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjNTRmNmNiIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzU0ZjZjYiIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9hNDVjN2Q5YThkYWQ0MWE2YjBlZTk1ODczNzY3Y2MwNiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF83N2ZhNDM1NmJmNjM0YTVlYWJjNjBkNDFlY2FkNDBiZSA9ICQoJzxkaXYgaWQ9Imh0bWxfNzdmYTQzNTZiZjYzNGE1ZWFiYzYwZDQxZWNhZDQwYmUiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJsYWNrYnVybiBTb3V0aCBDbHVzdGVyIDY8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2E0NWM3ZDlhOGRhZDQxYTZiMGVlOTU4NzM3NjdjYzA2LnNldENvbnRlbnQoaHRtbF83N2ZhNDM1NmJmNjM0YTVlYWJjNjBkNDFlY2FkNDBiZSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9kNjMyZGU0ODYzMjE0YmYzODc0NDExN2E2YThkZDYzNi5iaW5kUG9wdXAocG9wdXBfYTQ1YzdkOWE4ZGFkNDFhNmIwZWU5NTg3Mzc2N2NjMDYpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMGE2YzY2OTI1ZWI4NGY3MmIxY2Y4Mjc3MDM2Y2E0YTQgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzguMDYyOTM4MSwxNDUuMTE5NzQ1OV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjNTRmNmNiIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzU0ZjZjYiIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF82ZjQwYzliNzU3NWY0N2JhOGE2MzdmZjVmNWI1YjIzZiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8zNDk3MmYwMTllZTg0ZTlhYmY4MTA0M2VmOTI5MGY1MyA9ICQoJzxkaXYgaWQ9Imh0bWxfMzQ5NzJmMDE5ZWU4NGU5YWJmODEwNDNlZjkyOTBmNTMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJvbmJlYWNoIENsdXN0ZXIgNjwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNmY0MGM5Yjc1NzVmNDdiYThhNjM3ZmY1ZjViNWIyM2Yuc2V0Q29udGVudChodG1sXzM0OTcyZjAxOWVlODRlOWFiZjgxMDQzZWY5MjkwZjUzKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzBhNmM2NjkyNWViODRmNzJiMWNmODI3NzAzNmNhNGE0LmJpbmRQb3B1cChwb3B1cF82ZjQwYzliNzU3NWY0N2JhOGE2MzdmZjVmNWI1YjIzZik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8yNzUyYTUyMWIxNmI0ZWViOGQ5ODUzZDAwNzViNjQ5MyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy44MTM3MDMyLDE0NS4xMjM4MDUxMjVdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzAwYjVlYiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMwMGI1ZWIiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNjkwZTAyY2U0N2FmNDYyOWJjZjE2MTI2OTJlZTg0ZDAgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfN2MxMWMwOTJjZDI4NDJhMWFjZGIyZDMzMzgxOTY3NDUgPSAkKCc8ZGl2IGlkPSJodG1sXzdjMTFjMDkyY2QyODQyYTFhY2RiMmQzMzM4MTk2NzQ1IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Cb3ggSGlsbCAoVmljLikgQ2x1c3RlciA0PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF82OTBlMDJjZTQ3YWY0NjI5YmNmMTYxMjY5MmVlODRkMC5zZXRDb250ZW50KGh0bWxfN2MxMWMwOTJjZDI4NDJhMWFjZGIyZDMzMzgxOTY3NDUpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMjc1MmE1MjFiMTZiNGVlYjhkOTg1M2QwMDc1YjY0OTMuYmluZFBvcHVwKHBvcHVwXzY5MGUwMmNlNDdhZjQ2MjliY2YxNjEyNjkyZWU4NGQwKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzU4NzU1NmRjNGZlNjRlNTU5NjYwZmJhZTgwZGMyY2JmID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3LjgwNTY4MzEsMTQ1LjEyOTU3NDhdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2ZmMDAwMCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNmZjAwMDAiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfOWIwNjMwNzFmOTY4NGIwZTg0MmE5YzZjNjExYTVjMDYgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMzEwYmYyMmQ2OTk5NGY1NGEzZjBiOWU0NjRkYjJlYzggPSAkKCc8ZGl2IGlkPSJodG1sXzMxMGJmMjJkNjk5OTRmNTRhM2YwYjllNDY0ZGIyZWM4IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Cb3ggSGlsbCBOb3J0aCBDbHVzdGVyIDA8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzliMDYzMDcxZjk2ODRiMGU4NDJhOWM2YzYxMWE1YzA2LnNldENvbnRlbnQoaHRtbF8zMTBiZjIyZDY5OTk0ZjU0YTNmMGI5ZTQ2NGRiMmVjOCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl81ODc1NTZkYzRmZTY0ZTU1OTY2MGZiYWU4MGRjMmNiZi5iaW5kUG9wdXAocG9wdXBfOWIwNjMwNzFmOTY4NGIwZTg0MmE5YzZjNjExYTVjMDYpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNDZhNTYzMDNkOTBmNDJiMWJiYmJjZTlmYjY5NTE0NWMgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuODM2NjIzNiwxNDUuMTIzMzY0NF0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjNTRmNmNiIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzU0ZjZjYiIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF82MmQ1YjdlYTNiOTE0MWM1YmVlMmY0YzZiMjZhYmU0OCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9hNDcxMzJmMDJmZWU0NTUyYjc0Y2UwOGIwZWEzMjQ0YSA9ICQoJzxkaXYgaWQ9Imh0bWxfYTQ3MTMyZjAyZmVlNDU1MmI3NGNlMDhiMGVhMzI0NGEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJveCBIaWxsIFNvdXRoIENsdXN0ZXIgNjwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNjJkNWI3ZWEzYjkxNDFjNWJlZTJmNGM2YjI2YWJlNDguc2V0Q29udGVudChodG1sX2E0NzEzMmYwMmZlZTQ1NTJiNzRjZTA4YjBlYTMyNDRhKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzQ2YTU2MzAzZDkwZjQyYjFiYmJiY2U5ZmI2OTUxNDVjLmJpbmRQb3B1cChwb3B1cF82MmQ1YjdlYTNiOTE0MWM1YmVlMmY0YzZiMjZhYmU0OCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8wMmRmZTA1MzY2ODc0YmZhYTkzYmE3YjNiMWU1MzY2YyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy45MDgxOTYyLDE0NC45OTU3OTkxXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiNkNGRkODAiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjZDRkZDgwIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2M5Y2EwZDBjODNiNDRjYzk4MjAzNDNkYWUwYmVjZTdiID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzdlODJmODljZjk3NTRhMTE5YWExNzE1NTA1OWViOTIyID0gJCgnPGRpdiBpZD0iaHRtbF83ZTgyZjg5Y2Y5NzU0YTExOWFhMTcxNTUwNTllYjkyMiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QnJpZ2h0b24gKFZpYy4pIENsdXN0ZXIgOTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYzljYTBkMGM4M2I0NGNjOTgyMDM0M2RhZTBiZWNlN2Iuc2V0Q29udGVudChodG1sXzdlODJmODljZjk3NTRhMTE5YWExNzE1NTA1OWViOTIyKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzAyZGZlMDUzNjY4NzRiZmFhOTNiYTdiM2IxZTUzNjZjLmJpbmRQb3B1cChwb3B1cF9jOWNhMGQwYzgzYjQ0Y2M5ODIwMzQzZGFlMGJlY2U3Yik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl80MDgzOGVmOWM0M2E0MWIzYmYyMjM2NjcwN2Q4ZjNkMiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy45MTcxNzI4LDE0NS4wMTYzNjYzXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiM1NGY2Y2IiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjNTRmNmNiIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzFlMDc5YWNiYWRhMDQzYjA4N2Q5ZThlZWExODg5Mzg2ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzY2MGFjOWRjN2I1MTQwYmE4ZjJlOTE4ODhlNTE0NWM4ID0gJCgnPGRpdiBpZD0iaHRtbF82NjBhYzlkYzdiNTE0MGJhOGYyZTkxODg4ZTUxNDVjOCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QnJpZ2h0b24gRWFzdCBDbHVzdGVyIDY8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzFlMDc5YWNiYWRhMDQzYjA4N2Q5ZThlZWExODg5Mzg2LnNldENvbnRlbnQoaHRtbF82NjBhYzlkYzdiNTE0MGJhOGYyZTkxODg4ZTUxNDVjOCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl80MDgzOGVmOWM0M2E0MWIzYmYyMjM2NjcwN2Q4ZjNkMi5iaW5kUG9wdXAocG9wdXBfMWUwNzlhY2JhZGEwNDNiMDg3ZDllOGVlYTE4ODkzODYpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMGQ1YmVmNmUzNjMyNGZiMjk4YTQzNzFjODk0YzY0ZTEgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuNzY2NDcxNSwxNDQuOTYxMzEwM10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjMmFkZGRkIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzJhZGRkZCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF80NGIyYjZiZTdjZTY0OGIwYTE1YmVhZTFjMTIxNGQ4NyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8wMGUyNDM0Y2UwMjc0NWE5YjkyMDY0ZTdiMmYzNTZlNiA9ICQoJzxkaXYgaWQ9Imh0bWxfMDBlMjQzNGNlMDI3NDVhOWI5MjA2NGU3YjJmMzU2ZTYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJydW5zd2ljayAoVmljLikgQ2x1c3RlciA1PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF80NGIyYjZiZTdjZTY0OGIwYTE1YmVhZTFjMTIxNGQ4Ny5zZXRDb250ZW50KGh0bWxfMDBlMjQzNGNlMDI3NDVhOWI5MjA2NGU3YjJmMzU2ZTYpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMGQ1YmVmNmUzNjMyNGZiMjk4YTQzNzFjODk0YzY0ZTEuYmluZFBvcHVwKHBvcHVwXzQ0YjJiNmJlN2NlNjQ4YjBhMTViZWFlMWMxMjE0ZDg3KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzQ1YmJkZjA0NGNjYzRjNDY5N2Q0YTY5MGI1ZGMwYzBjID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3Ljc2ODg4MDIsMTQ0Ljk3NzY4MjVdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2ZmYjM2MCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNmZmIzNjAiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMDMzMWZiN2EzMjgyNGViN2IzMTZhNGViNTEwOTMyNWQgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMWMyZjFlMzA0NmUyNGJmNTljMGViN2MxYmRmNDgxYTQgPSAkKCc8ZGl2IGlkPSJodG1sXzFjMmYxZTMwNDZlMjRiZjU5YzBlYjdjMWJkZjQ4MWE0IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5CcnVuc3dpY2sgRWFzdCBDbHVzdGVyIDEwPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8wMzMxZmI3YTMyODI0ZWI3YjMxNmE0ZWI1MTA5MzI1ZC5zZXRDb250ZW50KGh0bWxfMWMyZjFlMzA0NmUyNGJmNTljMGViN2MxYmRmNDgxYTQpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfNDViYmRmMDQ0Y2NjNGM0Njk3ZDRhNjkwYjVkYzBjMGMuYmluZFBvcHVwKHBvcHVwXzAzMzFmYjdhMzI4MjRlYjdiMzE2YTRlYjUxMDkzMjVkKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzZjYTFkNTJlNzgzMDQ3ZWZhMzVhOGM1Mjk0M2RjMTQ0ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3Ljc2MzMzMzQsMTQ0Ljk0MjU1NjNdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzAwYjVlYiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMwMGI1ZWIiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYmIyMDc5ZDQ1MGVmNDJhOGE4ZWUwMzdiNWI4Zjc5MWYgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfYmEzN2E2MzY5MDAzNDQ0Yzg0ZWYyZDk4ZTIwNWQzMDggPSAkKCc8ZGl2IGlkPSJodG1sX2JhMzdhNjM2OTAwMzQ0NGM4NGVmMmQ5OGUyMDVkMzA4IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5CcnVuc3dpY2sgV2VzdCBDbHVzdGVyIDQ8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2JiMjA3OWQ0NTBlZjQyYThhOGVlMDM3YjViOGY3OTFmLnNldENvbnRlbnQoaHRtbF9iYTM3YTYzNjkwMDM0NDRjODRlZjJkOThlMjA1ZDMwOCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl82Y2ExZDUyZTc4MzA0N2VmYTM1YThjNTI5NDNkYzE0NC5iaW5kUG9wdXAocG9wdXBfYmIyMDc5ZDQ1MGVmNDJhOGE4ZWUwMzdiNWI4Zjc5MWYpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZGRmMmUzYzIxNWZkNDM4ZjkxNmJjMzhjYzQwMDQzZjIgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuNzY2MzEwOSwxNDUuMTIxMjgxMDE0NzU4NDRdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzU0ZjZjYiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM1NGY2Y2IiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYmY3ZGE3ZWQxNTRkNDQ2N2FjOGFmNjI1ZTRjYmI2ZWEgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZDIxZjUwMTdiOWYwNGNiYzg4NjMzNGU3ODg5Y2IyN2UgPSAkKCc8ZGl2IGlkPSJodG1sX2QyMWY1MDE3YjlmMDRjYmM4ODYzMzRlNzg4OWNiMjdlIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5CdWxsZWVuIENsdXN0ZXIgNjwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYmY3ZGE3ZWQxNTRkNDQ2N2FjOGFmNjI1ZTRjYmI2ZWEuc2V0Q29udGVudChodG1sX2QyMWY1MDE3YjlmMDRjYmM4ODYzMzRlNzg4OWNiMjdlKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2RkZjJlM2MyMTVmZDQzOGY5MTZiYzM4Y2M0MDA0M2YyLmJpbmRQb3B1cChwb3B1cF9iZjdkYTdlZDE1NGQ0NDY3YWM4YWY2MjVlNGNiYjZlYSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl85YjU1ODc5MDdjNGU0NzlkYmFhYTRhMmQyOGUzMTliNSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy44Mjc2MjIzNSwxNDUuMDA4MDkxMTk2MzQ2NF0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmZiMzYwIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmYjM2MCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF81NDkyYTg5ZDkzNDM0MjEzODE0MDIwYWMxZTQxMGQ1NCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8zM2RmYjgyNWQzYzI0ZGU4OTE1YTZkMjlhY2E0YmY1YiA9ICQoJzxkaXYgaWQ9Imh0bWxfMzNkZmI4MjVkM2MyNGRlODkxNWE2ZDI5YWNhNGJmNWIiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJ1cm5sZXkgQ2x1c3RlciAxMDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNTQ5MmE4OWQ5MzQzNDIxMzgxNDAyMGFjMWU0MTBkNTQuc2V0Q29udGVudChodG1sXzMzZGZiODI1ZDNjMjRkZTg5MTVhNmQyOWFjYTRiZjViKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzliNTU4NzkwN2M0ZTQ3OWRiYWFhNGEyZDI4ZTMxOWI1LmJpbmRQb3B1cChwb3B1cF81NDkyYTg5ZDkzNDM0MjEzODE0MDIwYWMxZTQxMGQ1NCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl81ZmZjNDBjNmVkOGQ0NTg4YjI3MjZmNGM0NmExMjlmOSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy44NTE2NzEsMTQ1LjA4MDU3NjddLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzU2NDFmZCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM1NjQxZmQiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZTY0MzZmNWUzMmM5NDg1Mjk3ZjcxODIzOTM0YTc1YTkgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNjliNzQzNDEzODg2NGYyNWI4ZTRlZjAzMjc1ZWUwYWIgPSAkKCc8ZGl2IGlkPSJodG1sXzY5Yjc0MzQxMzg4NjRmMjViOGU0ZWYwMzI3NWVlMGFiIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5CdXJ3b29kIChWaWMuKSBDbHVzdGVyIDI8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2U2NDM2ZjVlMzJjOTQ4NTI5N2Y3MTgyMzkzNGE3NWE5LnNldENvbnRlbnQoaHRtbF82OWI3NDM0MTM4ODY0ZjI1YjhlNGVmMDMyNzVlZTBhYik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl81ZmZjNDBjNmVkOGQ0NTg4YjI3MjZmNGM0NmExMjlmOS5iaW5kUG9wdXAocG9wdXBfZTY0MzZmNWUzMmM5NDg1Mjk3ZjcxODIzOTM0YTc1YTkpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfYzk4NzJiZTRlZTEwNGVmNDhiMTgzNjA2MjY4YzFlODQgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuODUzNTkzNSwxNDUuMTUwNDIzXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiMwMGI1ZWIiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMDBiNWViIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2U5NjYwYzQ0NzMwODQzOGZiYjk0MzFiYjgyNDVmOTVmID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzM4MGIzZjZjOTAxOTQxZDU4MjhiY2U3YmZmNTc3ZDY5ID0gJCgnPGRpdiBpZD0iaHRtbF8zODBiM2Y2YzkwMTk0MWQ1ODI4YmNlN2JmZjU3N2Q2OSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QnVyd29vZCBFYXN0IENsdXN0ZXIgNDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfZTk2NjBjNDQ3MzA4NDM4ZmJiOTQzMWJiODI0NWY5NWYuc2V0Q29udGVudChodG1sXzM4MGIzZjZjOTAxOTQxZDU4MjhiY2U3YmZmNTc3ZDY5KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2M5ODcyYmU0ZWUxMDRlZjQ4YjE4MzYwNjI2OGMxZTg0LmJpbmRQb3B1cChwb3B1cF9lOTY2MGM0NDczMDg0MzhmYmI5NDMxYmI4MjQ1Zjk1Zik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9hYWExYTRhNGYzMzc0YjRhOWYxYjVmYTFmY2MzYWUxNyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy44Mzg0NjIzLDE0NS4wNzQwNzY3XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiNmZjAwMDAiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzA4ZTZiYWI1N2I0ZTRhY2ViNzg3YWI5NDhiYzM1OGEzID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2RjYjMwZTE1ZWVkMDRjOTViZjE1YmY3OTEzMDA4MGUwID0gJCgnPGRpdiBpZD0iaHRtbF9kY2IzMGUxNWVlZDA0Yzk1YmYxNWJmNzkxMzAwODBlMCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Q2FtYmVyd2VsbCAoVmljLikgQ2x1c3RlciAwPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8wOGU2YmFiNTdiNGU0YWNlYjc4N2FiOTQ4YmMzNThhMy5zZXRDb250ZW50KGh0bWxfZGNiMzBlMTVlZWQwNGM5NWJmMTViZjc5MTMwMDgwZTApOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfYWFhMWE0YTRmMzM3NGI0YTlmMWI1ZmExZmNjM2FlMTcuYmluZFBvcHVwKHBvcHVwXzA4ZTZiYWI1N2I0ZTRhY2ViNzg3YWI5NDhiYzM1OGEzKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzA4OWQ3M2VkNTlmNzQ4YjE5MWQ0MTM5YWI4NWQ4NGVhID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3LjgyNDc0NjksMTQ1LjA4MDc2MzZdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2ZmNDEyMSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNmZjQxMjEiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNTFhZTU2Mzk0OTJmNDkzYWE2ZmI2OTkxYTJlZDM3NGQgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfM2FhOGY0MmMwNWMxNGM2OWFjZDYxMzc0MGQ0MmRmNWIgPSAkKCc8ZGl2IGlkPSJodG1sXzNhYThmNDJjMDVjMTRjNjlhY2Q2MTM3NDBkNDJkZjViIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5DYW50ZXJidXJ5IChWaWMuKSBDbHVzdGVyIDEyPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF81MWFlNTYzOTQ5MmY0OTNhYTZmYjY5OTFhMmVkMzc0ZC5zZXRDb250ZW50KGh0bWxfM2FhOGY0MmMwNWMxNGM2OWFjZDYxMzc0MGQ0MmRmNWIpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMDg5ZDczZWQ1OWY3NDhiMTkxZDQxMzlhYjg1ZDg0ZWEuYmluZFBvcHVwKHBvcHVwXzUxYWU1NjM5NDkyZjQ5M2FhNmZiNjk5MWEyZWQzNzRkKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzI4Zjk3ZjkxNjE1MDRlNjVhOTQ4YTNlZWFkOTgyNWI4ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3LjgwMDQyMjgsMTQ0Ljk2ODQzNDNdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzAwYjVlYiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMwMGI1ZWIiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNTJhYmMzNDg1MTM4NDkxYjljM2I5NDZiMjNkN2E4MzEgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMzFjZDBhYTM0MzRlNGMwZmEzYjFlMjQwYjE2OTY0MjAgPSAkKCc8ZGl2IGlkPSJodG1sXzMxY2QwYWEzNDM0ZTRjMGZhM2IxZTI0MGIxNjk2NDIwIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5DYXJsdG9uIChWaWMuKSBDbHVzdGVyIDQ8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzUyYWJjMzQ4NTEzODQ5MWI5YzNiOTQ2YjIzZDdhODMxLnNldENvbnRlbnQoaHRtbF8zMWNkMGFhMzQzNGU0YzBmYTNiMWUyNDBiMTY5NjQyMCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl8yOGY5N2Y5MTYxNTA0ZTY1YTk0OGEzZWVhZDk4MjViOC5iaW5kUG9wdXAocG9wdXBfNTJhYmMzNDg1MTM4NDkxYjljM2I5NDZiMjNkN2E4MzEpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMzNmM2FlMTk4NWQyNDdjZjg1MDk2NzA1MmViOGVlY2QgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuNzg0NTU4NSwxNDQuOTcyODU1M10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmY0MTIxIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmNDEyMSIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9lYWM3ZWQ4NmU1MDA0NDZlYTQxMjQ0YjYyNzhkMWVmNyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9mYjU2NTQyMjZlNTI0OGQzYjU4NmNjZTdiNTg1ZTFmOSA9ICQoJzxkaXYgaWQ9Imh0bWxfZmI1NjU0MjI2ZTUyNDhkM2I1ODZjY2U3YjU4NWUxZjkiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNhcmx0b24gTm9ydGggQ2x1c3RlciAxMjwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfZWFjN2VkODZlNTAwNDQ2ZWE0MTI0NGI2Mjc4ZDFlZjcuc2V0Q29udGVudChodG1sX2ZiNTY1NDIyNmU1MjQ4ZDNiNTg2Y2NlN2I1ODVlMWY5KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzMzZjNhZTE5ODVkMjQ3Y2Y4NTA5NjcwNTJlYjhlZWNkLmJpbmRQb3B1cChwb3B1cF9lYWM3ZWQ4NmU1MDA0NDZlYTQxMjQ0YjYyNzhkMWVmNyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl81MGYwMmQ5YjU2Yzc0ZmUxOTZhZDdhZDlkZWZhNTE5MSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy44ODYwMjksMTQ1LjA1ODEyNzFdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzJjN2VmNyIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMyYzdlZjciLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZWVjNzYxYmM1ZTU4NGQxMzhkNWFmMWMzNDlhZmFlMTEgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNjBjZTU3ZWYwMjVmNDBhYWI2YTc0NjRlNWMwNzc1ZTIgPSAkKCc8ZGl2IGlkPSJodG1sXzYwY2U1N2VmMDI1ZjQwYWFiNmE3NDY0ZTVjMDc3NWUyIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5DYXJuZWdpZSBDbHVzdGVyIDM8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2VlYzc2MWJjNWU1ODRkMTM4ZDVhZjFjMzQ5YWZhZTExLnNldENvbnRlbnQoaHRtbF82MGNlNTdlZjAyNWY0MGFhYjZhNzQ2NGU1YzA3NzVlMik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl81MGYwMmQ5YjU2Yzc0ZmUxOTZhZDdhZDlkZWZhNTE5MS5iaW5kUG9wdXAocG9wdXBfZWVjNzYxYmM1ZTU4NGQxMzhkNWFmMWMzNDlhZmFlMTEpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNDJjODQ2OGY3YzdjNDVkZTg0N2FiMGM2YmNjZWVhMzEgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzguMDc4Mzg5NSwxNDUuMTIzMjc0OF0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjMDBiNWViIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzAwYjVlYiIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF84NWE0OTIyZDcwMmM0N2NlODEzOGI3MzhkOWVkMmI4NCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF82ZDFlM2VkNzBmY2Y0YzQxYWQ2M2MyYWY4YTY3OTY2ZSA9ICQoJzxkaXYgaWQ9Imh0bWxfNmQxZTNlZDcwZmNmNGM0MWFkNjNjMmFmOGE2Nzk2NmUiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNhcnJ1bSBDbHVzdGVyIDQ8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzg1YTQ5MjJkNzAyYzQ3Y2U4MTM4YjczOGQ5ZWQyYjg0LnNldENvbnRlbnQoaHRtbF82ZDFlM2VkNzBmY2Y0YzQxYWQ2M2MyYWY4YTY3OTY2ZSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl80MmM4NDY4ZjdjN2M0NWRlODQ3YWIwYzZiY2NlZWEzMS5iaW5kUG9wdXAocG9wdXBfODVhNDkyMmQ3MDJjNDdjZTgxMzhiNzM4ZDllZDJiODQpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMmQ1MjM1NDcwOGE5NGRjN2E2NjgzOTc4ZGJkNzEzMzAgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuODgyMjY1MDAwMDAwMDA0LDE0NS4wMjI0NjI4Mzc0MDk0Ml0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmY0MTIxIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmNDEyMSIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8zZjVhNjA0N2RlZTY0ZDIwYmM1MGI0ZDgyY2ViYzA4YiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9kMzc1ZWZjZjUyZmE0NDA2YmE5ZDNmYzc1YTQ5YWVmNSA9ICQoJzxkaXYgaWQ9Imh0bWxfZDM3NWVmY2Y1MmZhNDQwNmJhOWQzZmM3NWE0OWFlZjUiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNhdWxmaWVsZCBDbHVzdGVyIDEyPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8zZjVhNjA0N2RlZTY0ZDIwYmM1MGI0ZDgyY2ViYzA4Yi5zZXRDb250ZW50KGh0bWxfZDM3NWVmY2Y1MmZhNDQwNmJhOWQzZmM3NWE0OWFlZjUpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMmQ1MjM1NDcwOGE5NGRjN2E2NjgzOTc4ZGJkNzEzMzAuYmluZFBvcHVwKHBvcHVwXzNmNWE2MDQ3ZGVlNjRkMjBiYzUwYjRkODJjZWJjMDhiKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzBlYWMwMGNmM2FmMTQ0ZWViYTA1MGZkNTA0MzhhMzZiID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3Ljg4MTI3NiwxNDUuMDQyMDg0N10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjNTY0MWZkIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzU2NDFmZCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF82YzVhYmQ3ZWM2MDI0MGNhOWU4Yzc1NTFjZDRjZTI2NCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF84ZTU0ZTI0ZTk5Y2E0N2Q4OTY4ZTI2NTA5NDcwZDRmZSA9ICQoJzxkaXYgaWQ9Imh0bWxfOGU1NGUyNGU5OWNhNDdkODk2OGUyNjUwOTQ3MGQ0ZmUiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNhdWxmaWVsZCBFYXN0IENsdXN0ZXIgMjwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNmM1YWJkN2VjNjAyNDBjYTllOGM3NTUxY2Q0Y2UyNjQuc2V0Q29udGVudChodG1sXzhlNTRlMjRlOTljYTQ3ZDg5NjhlMjY1MDk0NzBkNGZlKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzBlYWMwMGNmM2FmMTQ0ZWViYTA1MGZkNTA0MzhhMzZiLmJpbmRQb3B1cChwb3B1cF82YzVhYmQ3ZWM2MDI0MGNhOWU4Yzc1NTFjZDRjZTI2NCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl85MWVmMGJhYzFhNTk0MTZmYTVjMDdiOTUzMzhlYmUwNyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy44NzA4MjgsMTQ1LjAyMTgwMDVdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzAwYjVlYiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMwMGI1ZWIiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfOTAwNzUxYjdhMjNkNDk0NThiYTA2Zjk1MGFmZTBmMjkgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfYjJmNDI0NDgzMTgzNDRmNGFjNTg2MDQ3NjFkYzAzNTkgPSAkKCc8ZGl2IGlkPSJodG1sX2IyZjQyNDQ4MzE4MzQ0ZjRhYzU4NjA0NzYxZGMwMzU5IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5DYXVsZmllbGQgTm9ydGggQ2x1c3RlciA0PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF85MDA3NTFiN2EyM2Q0OTQ1OGJhMDZmOTUwYWZlMGYyOS5zZXRDb250ZW50KGh0bWxfYjJmNDI0NDgzMTgzNDRmNGFjNTg2MDQ3NjFkYzAzNTkpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfOTFlZjBiYWMxYTU5NDE2ZmE1YzA3Yjk1MzM4ZWJlMDcuYmluZFBvcHVwKHBvcHVwXzkwMDc1MWI3YTIzZDQ5NDU4YmEwNmY5NTBhZmUwZjI5KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzZmZGI2MGMxYTc2OTQwNTc4ODBhZjBhODhiZWUwZjYxID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3Ljg5NDY5OSwxNDUuMDI0OTMyM10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjNTRmNmNiIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzU0ZjZjYiIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF85MzA0MjEyYWQwYWY0OTM0YTNjYjU0MGVhMWE3OThmZCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9jZGJjMzhlYzA4MGI0ZjQ2YmYyOWQ1NWE4NTIyMzA5ZiA9ICQoJzxkaXYgaWQ9Imh0bWxfY2RiYzM4ZWMwODBiNGY0NmJmMjlkNTVhODUyMjMwOWYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNhdWxmaWVsZCBTb3V0aCBDbHVzdGVyIDY8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzkzMDQyMTJhZDBhZjQ5MzRhM2NiNTQwZWExYTc5OGZkLnNldENvbnRlbnQoaHRtbF9jZGJjMzhlYzA4MGI0ZjQ2YmYyOWQ1NWE4NTIyMzA5Zik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl82ZmRiNjBjMWE3Njk0MDU3ODgwYWYwYTg4YmVlMGY2MS5iaW5kUG9wdXAocG9wdXBfOTMwNDIxMmFkMGFmNDkzNGEzY2I1NDBlYTFhNzk4ZmQpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZjZmZTJkNjNlNmUzNDVjMjg5MzhhNDU4NzE3YWZmYzEgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzguMDUxODM0NCwxNDUuMTE1ODQ2Nl0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjMmM3ZWY3IiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzJjN2VmNyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9kMmVmMjFhYThhMjk0Nzk5YTY0ZGUzMmViZjkxOTc3YyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9jNjBmMDNmNDIzMDY0OGMzOWMwNzdhNDlkOGViYjZlNyA9ICQoJzxkaXYgaWQ9Imh0bWxfYzYwZjAzZjQyMzA2NDhjMzljMDc3YTQ5ZDhlYmI2ZTciIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNoZWxzZWEgQ2x1c3RlciAzPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9kMmVmMjFhYThhMjk0Nzk5YTY0ZGUzMmViZjkxOTc3Yy5zZXRDb250ZW50KGh0bWxfYzYwZjAzZjQyMzA2NDhjMzljMDc3YTQ5ZDhlYmI2ZTcpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfZjZmZTJkNjNlNmUzNDVjMjg5MzhhNDU4NzE3YWZmYzEuYmluZFBvcHVwKHBvcHVwX2QyZWYyMWFhOGEyOTQ3OTlhNjRkZTMyZWJmOTE5NzdjKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzkzMGY1ZjVhNmE3OTRiYTY5MjRkYTk3ZDg5ODc4YTg5ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM4LjA0MDg0MDMsMTQ1LjEzNDEzNTRdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzU0ZjZjYiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM1NGY2Y2IiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZDgyNzE4ZDZjNWY0NDc5ZGFlMWM4MTQxMTc1YTlhMTYgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMTdiNzAzMWM5YTFhNGY5ZjhmZDA4ZjFhMDMzZjFjNGUgPSAkKCc8ZGl2IGlkPSJodG1sXzE3YjcwMzFjOWExYTRmOWY4ZmQwOGYxYTAzM2YxYzRlIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5DaGVsc2VhIEhlaWdodHMgQ2x1c3RlciA2PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9kODI3MThkNmM1ZjQ0NzlkYWUxYzgxNDExNzVhOWExNi5zZXRDb250ZW50KGh0bWxfMTdiNzAzMWM5YTFhNGY5ZjhmZDA4ZjFhMDMzZjFjNGUpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfOTMwZjVmNWE2YTc5NGJhNjkyNGRhOTdkODk4NzhhODkuYmluZFBvcHVwKHBvcHVwX2Q4MjcxOGQ2YzVmNDQ3OWRhZTFjODE0MTE3NWE5YTE2KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2I1M2NkMTczOTE5NzRkZDdhZWQyODllM2MxYmIxODQyID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3Ljk2MzQxODEsMTQ1LjA2MTU2NjhdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzAwYjVlYiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMwMGI1ZWIiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfOGM3MjVhY2VmMjg0NDAwMGJlMzQ2ZTI2NjhjYmVlMjUgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMzM3MGE5N2ViMzA3NDJkZWJjOTY5ZjQ5NGNiMjFhNzAgPSAkKCc8ZGl2IGlkPSJodG1sXzMzNzBhOTdlYjMwNzQyZGViYzk2OWY0OTRjYjIxYTcwIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5DaGVsdGVuaGFtIChWaWMuKSBDbHVzdGVyIDQ8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzhjNzI1YWNlZjI4NDQwMDBiZTM0NmUyNjY4Y2JlZTI1LnNldENvbnRlbnQoaHRtbF8zMzcwYTk3ZWIzMDc0MmRlYmM5NjlmNDk0Y2IyMWE3MCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9iNTNjZDE3MzkxOTc0ZGQ3YWVkMjg5ZTNjMWJiMTg0Mi5iaW5kUG9wdXAocG9wdXBfOGM3MjVhY2VmMjg0NDAwMGJlMzQ2ZTI2NjhjYmVlMjUpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMTA1NTJiMWNiMzIxNGJjYTliYTQwYjYwMDA0NzY4ZjQgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuNzg4ODc3MzUsMTQ0Ljk5NTM2MjY1Mzc2OTA2XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiNmZmIzNjAiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjZmZiMzYwIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2NkOGM3ZjJjMDAwMjQzYWU4ZmViYzcyYWU2MTkzMmZhID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzg1OWM5OWUxYmJkNzQ2NjhhMTg3YzhhYTU2ZTc2Njc4ID0gJCgnPGRpdiBpZD0iaHRtbF84NTljOTllMWJiZDc0NjY4YTE4N2M4YWE1NmU3NjY3OCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Q2xpZnRvbiBIaWxsIENsdXN0ZXIgMTA8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2NkOGM3ZjJjMDAwMjQzYWU4ZmViYzcyYWU2MTkzMmZhLnNldENvbnRlbnQoaHRtbF84NTljOTllMWJiZDc0NjY4YTE4N2M4YWE1NmU3NjY3OCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl8xMDU1MmIxY2IzMjE0YmNhOWJhNDBiNjAwMDQ3NjhmNC5iaW5kUG9wdXAocG9wdXBfY2Q4YzdmMmMwMDAyNDNhZThmZWJjNzJhZTYxOTMyZmEpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMDc1YjEzMzZhMWRhNGU2ZGIxODU0YmIyMzUwOGE5MWYgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuNzQ0OTc1MiwxNDQuOTY0MzMxNF0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjODAwMGZmIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzgwMDBmZiIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8xMmZmNzMzZWI0OGQ0ZjhkODRhNTU1ZTUzZGI0OWQzNSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF84ZmEyOTk1NDNlMGQ0OGQ4OWU2YmM4ZWRjNjRjNjc3YiA9ICQoJzxkaXYgaWQ9Imh0bWxfOGZhMjk5NTQzZTBkNDhkODllNmJjOGVkYzY0YzY3N2IiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNvYnVyZyBDbHVzdGVyIDE8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzEyZmY3MzNlYjQ4ZDRmOGQ4NGE1NTVlNTNkYjQ5ZDM1LnNldENvbnRlbnQoaHRtbF84ZmEyOTk1NDNlMGQ0OGQ4OWU2YmM4ZWRjNjRjNjc3Yik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl8wNzViMTMzNmExZGE0ZTZkYjE4NTRiYjIzNTA4YTkxZi5iaW5kUG9wdXAocG9wdXBfMTJmZjczM2ViNDhkNGY4ZDg0YTU1NWU1M2RiNDlkMzUpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfYmRlYTcwZDI3NTk4NGE5Yzk5NWVmNDQyMTQxOTJlOGYgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuODAyMTA0LDE0NC45ODgxMzg3XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiNkNGRkODAiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjZDRkZDgwIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2Y4MWZmMTg5Y2E1NzQ2YjBiYWZjMzcwOTlmM2IzNWI3ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzZhNjdhMzQyNzMxZjRkYjRiMWViNjNmNGFlZWU0ZDEzID0gJCgnPGRpdiBpZD0iaHRtbF82YTY3YTM0MjczMWY0ZGI0YjFlYjYzZjRhZWVlNGQxMyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Q29sbGluZ3dvb2QgKFZpYy4pIENsdXN0ZXIgOTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfZjgxZmYxODljYTU3NDZiMGJhZmMzNzA5OWYzYjM1Yjcuc2V0Q29udGVudChodG1sXzZhNjdhMzQyNzMxZjRkYjRiMWViNjNmNGFlZWU0ZDEzKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2JkZWE3MGQyNzU5ODRhOWM5OTVlZjQ0MjE0MTkyZThmLmJpbmRQb3B1cChwb3B1cF9mODFmZjE4OWNhNTc0NmIwYmFmYzM3MDk5ZjNiMzViNyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9hYTY2ZTIwMDBjMGM0NzlhYWNkOTI3ODMxYzhiZmQwMyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy44MjkwMjI3LDE0NC45OTI3NzM1XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiMyYWRkZGQiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMmFkZGRkIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzgyMzJlOTM0YjM1NDQwOGI4Y2YwODFkZDkwMjA3MzNkID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzE5MDdkM2MxNjc5MDQ5ODg4MTg0NTNjODA2Zjk2MjZlID0gJCgnPGRpdiBpZD0iaHRtbF8xOTA3ZDNjMTY3OTA0OTg4ODE4NDUzYzgwNmY5NjI2ZSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Q3JlbW9ybmUgKFZpYy4pIENsdXN0ZXIgNTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfODIzMmU5MzRiMzU0NDA4YjhjZjA4MWRkOTAyMDczM2Quc2V0Q29udGVudChodG1sXzE5MDdkM2MxNjc5MDQ5ODg4MTg0NTNjODA2Zjk2MjZlKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2FhNjZlMjAwMGMwYzQ3OWFhY2Q5Mjc4MzFjOGJmZDAzLmJpbmRQb3B1cChwb3B1cF84MjMyZTkzNGIzNTQ0MDhiOGNmMDgxZGQ5MDIwNzMzZCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8yZWIzZThlMDk3NzE0ZWY5YWQwYjJlMjk5ZTA4MmI5ZCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy44MTIxMjM0LDE0NS4wNjY0Mzc3XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiMwMGI1ZWIiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMDBiNWViIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzg2ODkxNGE1ZTE0NTQwODM5NDkwYThmZmY3YTNkYjc2ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzU2NDhhODJiZWQ3OTQ1MDVhYjRiZjg0ZmVjMDBhYWUzID0gJCgnPGRpdiBpZD0iaHRtbF81NjQ4YTgyYmVkNzk0NTA1YWI0YmY4NGZlYzAwYWFlMyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RGVlcGRlbmUgKFZpYy4pIENsdXN0ZXIgNDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfODY4OTE0YTVlMTQ1NDA4Mzk0OTBhOGZmZjdhM2RiNzYuc2V0Q29udGVudChodG1sXzU2NDhhODJiZWQ3OTQ1MDVhYjRiZjg0ZmVjMDBhYWUzKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzJlYjNlOGUwOTc3MTRlZjlhZDBiMmUyOTllMDgyYjlkLmJpbmRQb3B1cChwb3B1cF84Njg5MTRhNWUxNDU0MDgzOTQ5MGE4ZmZmN2EzZGI3Nik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8xMmM0MjJlYTQ3MGE0MzEyOTczNzI1Zjg0YTQwZGYzNyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy45NzU1NzYyLDE0NS4xMjU2NDc4XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiM1NGY2Y2IiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjNTRmNmNiIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzU4MDg1YzM4ZGM3NzQwZmViMDJiOTRjZmE2MjBhZWUzID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzJiMzIwMjhkYmFkOTRjZjhiOTBlNWNmZGUxYzA0M2U0ID0gJCgnPGRpdiBpZD0iaHRtbF8yYjMyMDI4ZGJhZDk0Y2Y4YjkwZTVjZmRlMWMwNDNlNCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RGluZ2xleSBWaWxsYWdlIENsdXN0ZXIgNjwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNTgwODVjMzhkYzc3NDBmZWIwMmI5NGNmYTYyMGFlZTMuc2V0Q29udGVudChodG1sXzJiMzIwMjhkYmFkOTRjZjhiOTBlNWNmZGUxYzA0M2U0KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzEyYzQyMmVhNDcwYTQzMTI5NzM3MjVmODRhNDBkZjM3LmJpbmRQb3B1cChwb3B1cF81ODA4NWMzOGRjNzc0MGZlYjAyYjk0Y2ZhNjIwYWVlMyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl81NjYwOGZjMWE2MzA0Zjc1YTg5MjI3NWYzYWY3NDZmMyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy44MTc1NDIzLDE0NC45Mzk0OTIzXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiNkNGRkODAiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjZDRkZDgwIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2MwYWU3Mjg1ZTI5MTQyYzA4ZTVlZDk4NjFlOGVjNzdiID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzUzMWVmNGU2Y2RlNzRmOTA4MjE0OWUzNmRmMGFmODRjID0gJCgnPGRpdiBpZD0iaHRtbF81MzFlZjRlNmNkZTc0ZjkwODIxNDllMzZkZjBhZjg0YyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RG9ja2xhbmRzIENsdXN0ZXIgOTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYzBhZTcyODVlMjkxNDJjMDhlNWVkOTg2MWU4ZWM3N2Iuc2V0Q29udGVudChodG1sXzUzMWVmNGU2Y2RlNzRmOTA4MjE0OWUzNmRmMGFmODRjKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzU2NjA4ZmMxYTYzMDRmNzVhODkyMjc1ZjNhZjc0NmYzLmJpbmRQb3B1cChwb3B1cF9jMGFlNzI4NWUyOTE0MmMwOGU1ZWQ5ODYxZThlYzc3Yik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl83YTA5OTg2MmRkNzY0NzI3OTlkMGMzMWE2OTA5ZWRiNCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy43ODQ4Mjk5LDE0NS4xMjM4NDMxXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiM1NjQxZmQiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjNTY0MWZkIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2UxYzBmOWRhZGYzZjRkODVhOTU4YjRkZWQ5MGU0Yjk4ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzJlMjdjMzU2NjYyNjQwNzdiYWRkOWQyZDY3YjMzY2EyID0gJCgnPGRpdiBpZD0iaHRtbF8yZTI3YzM1NjY2MjY0MDc3YmFkZDlkMmQ2N2IzM2NhMiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RG9uY2FzdGVyIENsdXN0ZXIgMjwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfZTFjMGY5ZGFkZjNmNGQ4NWE5NThiNGRlZDkwZTRiOTguc2V0Q29udGVudChodG1sXzJlMjdjMzU2NjYyNjQwNzdiYWRkOWQyZDY3YjMzY2EyKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzdhMDk5ODYyZGQ3NjQ3Mjc5OWQwYzMxYTY5MDllZGI0LmJpbmRQb3B1cChwb3B1cF9lMWMwZjlkYWRmM2Y0ZDg1YTk1OGI0ZGVkOTBlNGI5OCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8zMGI0ZGM4MzhkYzA0YzI0OWUyMzU2ZGM3ZDI5Y2U0NyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy43Nzg4NzQ0LDE0NS4xNjM4ODE1XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiM1NGY2Y2IiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjNTRmNmNiIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzdiZDViMzFmN2Y3YzQ1Y2ViMzAwMmZmZTk0MTFkZDAyID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzczMGY1Y2RjZWJhZTRkYzZhYWYwYTA2ZWRmN2E4ODBjID0gJCgnPGRpdiBpZD0iaHRtbF83MzBmNWNkY2ViYWU0ZGM2YWFmMGEwNmVkZjdhODgwYyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RG9uY2FzdGVyIEVhc3QgQ2x1c3RlciA2PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF83YmQ1YjMxZjdmN2M0NWNlYjMwMDJmZmU5NDExZGQwMi5zZXRDb250ZW50KGh0bWxfNzMwZjVjZGNlYmFlNGRjNmFhZjBhMDZlZGY3YTg4MGMpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMzBiNGRjODM4ZGMwNGMyNDllMjM1NmRjN2QyOWNlNDcuYmluZFBvcHVwKHBvcHVwXzdiZDViMzFmN2Y3YzQ1Y2ViMzAwMmZmZTk0MTFkZDAyKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2NlMDI2Y2Q4YmI2ZTRmNmViZWE4NzA2NGUzNGNhNWI0ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3Ljc4OTgwOTQsMTQ1LjE5MTQ5MTVdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzU0ZjZjYiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM1NGY2Y2IiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYjEzNGRkZGIxNzZlNDkxM2I0NjQzZWIwYjhkOGJmNjcgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZTUxNGY3NjVmZmVkNDFhZmEzOWViMjRiMGE3NzM5NGEgPSAkKCc8ZGl2IGlkPSJodG1sX2U1MTRmNzY1ZmZlZDQxYWZhMzllYjI0YjBhNzczOTRhIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Eb252YWxlIENsdXN0ZXIgNjwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYjEzNGRkZGIxNzZlNDkxM2I0NjQzZWIwYjhkOGJmNjcuc2V0Q29udGVudChodG1sX2U1MTRmNzY1ZmZlZDQxYWZhMzllYjI0YjBhNzczOTRhKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2NlMDI2Y2Q4YmI2ZTRmNmViZWE4NzA2NGUzNGNhNWI0LmJpbmRQb3B1cChwb3B1cF9iMTM0ZGRkYjE3NmU0OTEzYjQ2NDNlYjBiOGQ4YmY2Nyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl81M2RiYzExYWI1YWI0ZDk3OTdlM2QzYzU4NjdiMGRkNiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy44MTI0OTgsMTQ0Ljk4NTg4NTFdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzgwMDBmZiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4MDAwZmYiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMGI0N2NlMjQ0NDAyNDE5YWE4NzM0YjVhM2VjYzFlYTggPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfYWJlNDUwOGMzMmJiNDQwYjg4YzZjMzk5ZTJlODE3NDUgPSAkKCc8ZGl2IGlkPSJodG1sX2FiZTQ1MDhjMzJiYjQ0MGI4OGM2YzM5OWUyZTgxNzQ1IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5FYXN0IE1lbGJvdXJuZSBDbHVzdGVyIDE8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzBiNDdjZTI0NDQwMjQxOWFhODczNGI1YTNlY2MxZWE4LnNldENvbnRlbnQoaHRtbF9hYmU0NTA4YzMyYmI0NDBiODhjNmMzOTllMmU4MTc0NSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl81M2RiYzExYWI1YWI0ZDk3OTdlM2QzYzU4NjdiMGRkNi5iaW5kUG9wdXAocG9wdXBfMGI0N2NlMjQ0NDAyNDE5YWE4NzM0YjVhM2VjYzFlYTgpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfOTRjNzhiODczYTczNDYxZmI5MjRkNTc2YjY1OTBlMTMgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzguMDM4MDczOCwxNDUuMTA4MzkyM10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjNTRmNmNiIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzU0ZjZjYiIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8xZjllMTNhMjZmNDA0NDJhYmI4YmM1ZWMyZDhjMThmOSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9jZjllYTlkM2MyYTM0NDIwODlhODVjMTUyNGIwZDAyYSA9ICQoJzxkaXYgaWQ9Imh0bWxfY2Y5ZWE5ZDNjMmEzNDQyMDg5YTg1YzE1MjRiMGQwMmEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkVkaXRodmFsZSBDbHVzdGVyIDY8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzFmOWUxM2EyNmY0MDQ0MmFiYjhiYzVlYzJkOGMxOGY5LnNldENvbnRlbnQoaHRtbF9jZjllYTlkM2MyYTM0NDIwODlhODVjMTUyNGIwZDAyYSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl85NGM3OGI4NzNhNzM0NjFmYjkyNGQ1NzZiNjU5MGUxMy5iaW5kUG9wdXAocG9wdXBfMWY5ZTEzYTI2ZjQwNDQyYWJiOGJjNWVjMmQ4YzE4ZjkpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZGZjNjMxMzgyODA4NGIzZWEzMzQ5MWRjNGFkZWRmNGEgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuODg1ODQyNSwxNDUuMDA3MDE1Ml0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmY0MTIxIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmNDEyMSIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF81YzRiNWFlNTgwZmM0MmRlYjczNzI4OTEwM2M4YmIxYiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF83MjhlYWM0MWJlZWE0Zjc1OWI4YWYyOTVhZjk0MzA5OSA9ICQoJzxkaXYgaWQ9Imh0bWxfNzI4ZWFjNDFiZWVhNGY3NTliOGFmMjk1YWY5NDMwOTkiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkVsc3Rlcm53aWNrIENsdXN0ZXIgMTI8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzVjNGI1YWU1ODBmYzQyZGViNzM3Mjg5MTAzYzhiYjFiLnNldENvbnRlbnQoaHRtbF83MjhlYWM0MWJlZWE0Zjc1OWI4YWYyOTVhZjk0MzA5OSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9kZmM2MzEzODI4MDg0YjNlYTMzNDkxZGM0YWRlZGY0YS5iaW5kUG9wdXAocG9wdXBfNWM0YjVhZTU4MGZjNDJkZWI3MzcyODkxMDNjOGJiMWIpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMzZkMGUzZjU4MWZlNDU0NGEwODhiZmZkZTVkMDliZmIgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuODc4ODU2OCwxNDQuOTg1NTQ4N10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmY0MTIxIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmNDEyMSIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9lYzIzOGQzOGU1ZWQ0ZDI0OWRiMGI2MTlkMzYyM2NjMCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9iZGI2MDlhOTcwMGY0OTc1OGZmOWFjMjNlZWJlZDJkOSA9ICQoJzxkaXYgaWQ9Imh0bWxfYmRiNjA5YTk3MDBmNDk3NThmZjlhYzIzZWViZWQyZDkiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkVsd29vZCBDbHVzdGVyIDEyPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9lYzIzOGQzOGU1ZWQ0ZDI0OWRiMGI2MTlkMzYyM2NjMC5zZXRDb250ZW50KGh0bWxfYmRiNjA5YTk3MDBmNDk3NThmZjlhYzIzZWViZWQyZDkpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMzZkMGUzZjU4MWZlNDU0NGEwODhiZmZkZTVkMDliZmIuYmluZFBvcHVwKHBvcHVwX2VjMjM4ZDM4ZTVlZDRkMjQ5ZGIwYjYxOWQzNjIzY2MwKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzI1NWFjODIwNjkzMDRmZTRiMTg1ZDRmZjE1ZDk2OTZjID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3Ljc1NTk2Mjk5OTk5OTk5NCwxNDQuOTE2MTc3OTA0MDExNDJdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzJjN2VmNyIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMyYzdlZjciLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNGM4MGY4MWI2NjhkNDFlNzhjNzQ3NDg1MDFlODY5MGQgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfODQ3YjQ0ZjBmYmY1NDNmNmIxZDI4OWQzZjAxYTFlNTIgPSAkKCc8ZGl2IGlkPSJodG1sXzg0N2I0NGYwZmJmNTQzZjZiMWQyODlkM2YwMWExZTUyIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Fc3NlbmRvbiBDbHVzdGVyIDM8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzRjODBmODFiNjY4ZDQxZTc4Yzc0NzQ4NTAxZTg2OTBkLnNldENvbnRlbnQoaHRtbF84NDdiNDRmMGZiZjU0M2Y2YjFkMjg5ZDNmMDFhMWU1Mik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl8yNTVhYzgyMDY5MzA0ZmU0YjE4NWQ0ZmYxNWQ5Njk2Yy5iaW5kUG9wdXAocG9wdXBfNGM4MGY4MWI2NjhkNDFlNzhjNzQ3NDg1MDFlODY5MGQpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfY2I5YmZhNGIzNzE2NDc3MzlmMTgxMDVjOGM0ZGFkZmEgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuNzM4NDc2MiwxNDQuOTAxMjAyN10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjMDBiNWViIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzAwYjVlYiIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF80MzgxZTc0OTFjYjM0NDliOTRjOTExYTU5OGEyZmQwNSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9kNzliNWYwNmFiYjQ0NTgyOTY5NTkwNzJlNDVlNzBjMiA9ICQoJzxkaXYgaWQ9Imh0bWxfZDc5YjVmMDZhYmI0NDU4Mjk2OTU5MDcyZTQ1ZTcwYzIiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkVzc2VuZG9uIE5vcnRoIENsdXN0ZXIgNDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNDM4MWU3NDkxY2IzNDQ5Yjk0YzkxMWE1OThhMmZkMDUuc2V0Q29udGVudChodG1sX2Q3OWI1ZjA2YWJiNDQ1ODI5Njk1OTA3MmU0NWU3MGMyKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2NiOWJmYTRiMzcxNjQ3NzM5ZjE4MTA1YzhjNGRhZGZhLmJpbmRQb3B1cChwb3B1cF80MzgxZTc0OTFjYjM0NDliOTRjOTExYTU5OGEyZmQwNSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8wOWY3ZTEyOTU5NDU0MmYxOTY2NDM2NDcxMThlNTUwNCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy43NzkxOTMzLDE0NS4wMTcwMDVdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzJjN2VmNyIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMyYzdlZjciLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZjQ4Y2JjZDNkYzBhNGY1NWJjYTNlZjkyN2U2ODc1NmQgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNjQ2OGE3NTUxYzcyNGE1NDkyYTU5Y2E3ZTg5NDI3MzMgPSAkKCc8ZGl2IGlkPSJodG1sXzY0NjhhNzU1MWM3MjRhNTQ5MmE1OWNhN2U4OTQyNzMzIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5GYWlyZmllbGQgKFZpYy4pIENsdXN0ZXIgMzwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfZjQ4Y2JjZDNkYzBhNGY1NWJjYTNlZjkyN2U2ODc1NmQuc2V0Q29udGVudChodG1sXzY0NjhhNzU1MWM3MjRhNTQ5MmE1OWNhN2U4OTQyNzMzKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzA5ZjdlMTI5NTk0NTQyZjE5NjY0MzY0NzExOGU1NTA0LmJpbmRQb3B1cChwb3B1cF9mNDhjYmNkM2RjMGE0ZjU1YmNhM2VmOTI3ZTY4NzU2ZCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9mNDRhOWJhNDk5MTk0NGQyYjY0YjlhMGZiNTFkNzJhZiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy44MDEwMzgyLDE0NC45NzkyNjExXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiMyYWRkZGQiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMmFkZGRkIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzQ0OGY4ZjRkNWI2MDQ3NWZhOTM2YjFjMTRjZTk5ZjUwID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzBkNTE1MzZmNTI5YjQzNmFhNGU3OTQzMmY2ZDdkMjQ3ID0gJCgnPGRpdiBpZD0iaHRtbF8wZDUxNTM2ZjUyOWI0MzZhYTRlNzk0MzJmNmQ3ZDI0NyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Rml0enJveSAoVmljLikgQ2x1c3RlciA1PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF80NDhmOGY0ZDViNjA0NzVmYTkzNmIxYzE0Y2U5OWY1MC5zZXRDb250ZW50KGh0bWxfMGQ1MTUzNmY1MjliNDM2YWE0ZTc5NDMyZjZkN2QyNDcpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfZjQ0YTliYTQ5OTE5NDRkMmI2NGI5YTBmYjUxZDcyYWYuYmluZFBvcHVwKHBvcHVwXzQ0OGY4ZjRkNWI2MDQ3NWZhOTM2YjFjMTRjZTk5ZjUwKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2M0Y2ViNGYyZTE1YzRhMTE4NjBiNGFkZTAzZTI3ZTFhID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3Ljc4MzMzMTYsMTQ0Ljk4MzcwNzRdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2ZmYjM2MCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNmZmIzNjAiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZDNiYzM3Yzc4YzM3NDVhZDgyMTkwYTQ3YmVhYjMwMTYgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNDRlOTVmOTQwMWU3NGI4MmJhMGUyYTA0ZDgxZjZiMTIgPSAkKCc8ZGl2IGlkPSJodG1sXzQ0ZTk1Zjk0MDFlNzRiODJiYTBlMmEwNGQ4MWY2YjEyIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5GaXR6cm95IE5vcnRoIENsdXN0ZXIgMTA8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2QzYmMzN2M3OGMzNzQ1YWQ4MjE5MGE0N2JlYWIzMDE2LnNldENvbnRlbnQoaHRtbF80NGU5NWY5NDAxZTc0YjgyYmEwZTJhMDRkODFmNmIxMik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9jNGNlYjRmMmUxNWM0YTExODYwYjRhZGUwM2UyN2UxYS5iaW5kUG9wdXAocG9wdXBfZDNiYzM3Yzc4YzM3NDVhZDgyMTkwYTQ3YmVhYjMwMTYpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfYjNkYzc1YTI0N2IwNGY5YTkyMDZjZWZjYmE0NmE0ZmUgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuNzg2NzU4NywxNDQuOTE5MzY2OF0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmYwMDAwIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9iZWJjMmZiYjMzY2Y0YzU5YjkyZDVlY2ZmZWNlMmVjZSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9mMjM1ZWVkN2JkZWI0YjIwYmJmZDA3NjU5YjI0YWU2MSA9ICQoJzxkaXYgaWQ9Imh0bWxfZjIzNWVlZDdiZGViNGIyMGJiZmQwNzY1OWIyNGFlNjEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkZsZW1pbmd0b24gQ2x1c3RlciAwPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9iZWJjMmZiYjMzY2Y0YzU5YjkyZDVlY2ZmZWNlMmVjZS5zZXRDb250ZW50KGh0bWxfZjIzNWVlZDdiZGViNGIyMGJiZmQwNzY1OWIyNGFlNjEpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfYjNkYzc1YTI0N2IwNGY5YTkyMDZjZWZjYmE0NmE0ZmUuYmluZFBvcHVwKHBvcHVwX2JlYmMyZmJiMzNjZjRjNTliOTJkNWVjZmZlY2UyZWNlKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzNlMWE2NDE2Yzc4YjQ1MTk4NTM0OTIyZmNlYzhkMTc2ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3Ljg5NjY0MjUsMTQ1LjAwNDE3NTldLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzU2NDFmZCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM1NjQxZmQiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNTUzY2FkM2U0ODU5NGIwNDg3YTk4ODNkNTNmYTVlZDAgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZTU0MGE3YmM3MWQ4NGU3NDhiYWIyZGRlYzVkODE5YTUgPSAkKCc8ZGl2IGlkPSJodG1sX2U1NDBhN2JjNzFkODRlNzQ4YmFiMmRkZWM1ZDgxOWE1IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5HYXJkZW52YWxlIENsdXN0ZXIgMjwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNTUzY2FkM2U0ODU5NGIwNDg3YTk4ODNkNTNmYTVlZDAuc2V0Q29udGVudChodG1sX2U1NDBhN2JjNzFkODRlNzQ4YmFiMmRkZWM1ZDgxOWE1KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzNlMWE2NDE2Yzc4YjQ1MTk4NTM0OTIyZmNlYzhkMTc2LmJpbmRQb3B1cChwb3B1cF81NTNjYWQzZTQ4NTk0YjA0ODdhOTg4M2Q1M2ZhNWVkMCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl81MmZlMTU0MTIyYWI0MjM1OTRmYjc0ZjAzNjdkZjYwZSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy44OTIzNzkxLDE0NS4wNDEzNjg5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiM1NjQxZmQiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjNTY0MWZkIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2M5ZDAwNDY1OWIzNjQxNmQ4NWU3YzJjODkxZWFkMDZkID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzEzNGVjYTZiMzUyMzRjN2I5ZDdjZjAxYTlhN2M3NmE5ID0gJCgnPGRpdiBpZD0iaHRtbF8xMzRlY2E2YjM1MjM0YzdiOWQ3Y2YwMWE5YTdjNzZhOSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+R2xlbiBIdW50bHkgQ2x1c3RlciAyPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9jOWQwMDQ2NTliMzY0MTZkODVlN2MyYzg5MWVhZDA2ZC5zZXRDb250ZW50KGh0bWxfMTM0ZWNhNmIzNTIzNGM3YjlkN2NmMDFhOWE3Yzc2YTkpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfNTJmZTE1NDEyMmFiNDIzNTk0ZmI3NGYwMzY3ZGY2MGUuYmluZFBvcHVwKHBvcHVwX2M5ZDAwNDY1OWIzNjQxNmQ4NWU3YzJjODkxZWFkMDZkKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzg2NWFhNzRhYjY0OTQ2MTU5YmU5OGI4Y2ZkMmZkYTAwID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3Ljg1NTgxNCwxNDUuMDY0NjExN10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmZiMzYwIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmYjM2MCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9lZGQ3Mzg2YjM5YWY0NjU0YTlhZjRmOTg5Y2UxZjg2MSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9jODZhZmRhZTkyMWU0MGJkOWY4YTkzODU0Y2IzMjRiMiA9ICQoJzxkaXYgaWQ9Imh0bWxfYzg2YWZkYWU5MjFlNDBiZDlmOGE5Mzg1NGNiMzI0YjIiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkdsZW4gSXJpcyAoVmljLikgQ2x1c3RlciAxMDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfZWRkNzM4NmIzOWFmNDY1NGE5YWY0Zjk4OWNlMWY4NjEuc2V0Q29udGVudChodG1sX2M4NmFmZGFlOTIxZTQwYmQ5ZjhhOTM4NTRjYjMyNGIyKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzg2NWFhNzRhYjY0OTQ2MTU5YmU5OGI4Y2ZkMmZkYTAwLmJpbmRQb3B1cChwb3B1cF9lZGQ3Mzg2YjM5YWY0NjU0YTlhZjRmOTg5Y2UxZjg2MSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9hYTBlNzQ5YjEzNzg0Y2UzOWYzNTE3NDM2ODM3NWU0MyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy45MzgxMTI3LDE0NS4wMDE0NzY0XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiMyYzdlZjciLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMmM3ZWY3IiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzg0M2Y2Zjc5YmVmODRiMTk5ZDQ5Y2E0YTIwZjZkM2ZhID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzc2ZDRkOTM2MjFmYjQ0YWRhODE3ZWQzYWZiODkxNjliID0gJCgnPGRpdiBpZD0iaHRtbF83NmQ0ZDkzNjIxZmI0NGFkYTgxN2VkM2FmYjg5MTY5YiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+SGFtcHRvbiAoVmljLikgQ2x1c3RlciAzPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF84NDNmNmY3OWJlZjg0YjE5OWQ0OWNhNGEyMGY2ZDNmYS5zZXRDb250ZW50KGh0bWxfNzZkNGQ5MzYyMWZiNDRhZGE4MTdlZDNhZmI4OTE2OWIpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfYWEwZTc0OWIxMzc4NGNlMzlmMzUxNzQzNjgzNzVlNDMuYmluZFBvcHVwKHBvcHVwXzg0M2Y2Zjc5YmVmODRiMTk5ZDQ5Y2E0YTIwZjZkM2ZhKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzAyOTJiYjNkNGJiNTRkZjFiNTZlYjA4ODhiM2U3MGQwID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3LjkzODU0MDksMTQ1LjAzMDY3MjZdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzU0ZjZjYiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM1NGY2Y2IiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYjNmYzRiZDYxY2Y1NDliYzg3NDdmZjRjNWM3ODk1NjQgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfODFkMWFhMzA1N2NlNDNiOTlkZDQ4NzFkMjIwNTE3ZWYgPSAkKCc8ZGl2IGlkPSJodG1sXzgxZDFhYTMwNTdjZTQzYjk5ZGQ0ODcxZDIyMDUxN2VmIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5IYW1wdG9uIEVhc3QgQ2x1c3RlciA2PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9iM2ZjNGJkNjFjZjU0OWJjODc0N2ZmNGM1Yzc4OTU2NC5zZXRDb250ZW50KGh0bWxfODFkMWFhMzA1N2NlNDNiOTlkZDQ4NzFkMjIwNTE3ZWYpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMDI5MmJiM2Q0YmI1NGRmMWI1NmViMDg4OGIzZTcwZDAuYmluZFBvcHVwKHBvcHVwX2IzZmM0YmQ2MWNmNTQ5YmM4NzQ3ZmY0YzVjNzg5NTY0KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2ZmNDRkN2FiZWM0MTRiMGFiZWI2NzAzYTFjNTM4YzRmID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3LjgyNDQyNDYsMTQ1LjAzMTcyMDddLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzJhZGRkZCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMyYWRkZGQiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYmQ1ZDM1N2JlMzJiNDNlNzhjMzkyMWFhZTg4Y2NjYzggPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfYjU1ZjFlNWZmMDlkNDZmYWJjNjRhYmFkZDY3YjQ2NGMgPSAkKCc8ZGl2IGlkPSJodG1sX2I1NWYxZTVmZjA5ZDQ2ZmFiYzY0YWJhZGQ2N2I0NjRjIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5IYXd0aG9ybiAoVmljLikgQ2x1c3RlciA1PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9iZDVkMzU3YmUzMmI0M2U3OGMzOTIxYWFlODhjY2NjOC5zZXRDb250ZW50KGh0bWxfYjU1ZjFlNWZmMDlkNDZmYWJjNjRhYmFkZDY3YjQ2NGMpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfZmY0NGQ3YWJlYzQxNGIwYWJlYjY3MDNhMWM1MzhjNGYuYmluZFBvcHVwKHBvcHVwX2JkNWQzNTdiZTMyYjQzZTc4YzM5MjFhYWU4OGNjY2M4KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2I2NjIxMjVkNDczYTRiNjk5ZmIwNDMxYWYxMDIwZWVmID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3LjgzMTM3ODUsMTQ1LjA0OTk3OTddLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzU2NDFmZCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM1NjQxZmQiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYjEwODk1MDkwNjMyNDY2NzgwYWYxMGUwMmFhNzhiYTYgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZWJjNGQ0ZTkzMTIyNDI3N2FmMmU4ZmFkZjljYjQxNDUgPSAkKCc8ZGl2IGlkPSJodG1sX2ViYzRkNGU5MzEyMjQyNzdhZjJlOGZhZGY5Y2I0MTQ1IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5IYXd0aG9ybiBFYXN0IENsdXN0ZXIgMjwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYjEwODk1MDkwNjMyNDY2NzgwYWYxMGUwMmFhNzhiYTYuc2V0Q29udGVudChodG1sX2ViYzRkNGU5MzEyMjQyNzdhZjJlOGZhZGY5Y2I0MTQ1KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2I2NjIxMjVkNDczYTRiNjk5ZmIwNDMxYWYxMDIwZWVmLmJpbmRQb3B1cChwb3B1cF9iMTA4OTUwOTA2MzI0NjY3ODBhZjEwZTAyYWE3OGJhNik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl85MmUxYmYwOGM3NTU0ZDRhYWE2MzQwOTE1MDU0ZDU1ZiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy45NDg0NDM4LDE0NS4wNDE4OF0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjNTY0MWZkIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzU2NDFmZCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF81YjcxOTFhZDgyMWI0M2I0OWM1M2QyMzcwZGY3OTU5MSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9jMzE0NWUxOGQ0OTE0MzkxOTI1M2QwMGY4MmI4NDNiNiA9ICQoJzxkaXYgaWQ9Imh0bWxfYzMxNDVlMThkNDkxNDM5MTkyNTNkMDBmODJiODQzYjYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkhpZ2hldHQgQ2x1c3RlciAyPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF81YjcxOTFhZDgyMWI0M2I0OWM1M2QyMzcwZGY3OTU5MS5zZXRDb250ZW50KGh0bWxfYzMxNDVlMThkNDkxNDM5MTkyNTNkMDBmODJiODQzYjYpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfOTJlMWJmMDhjNzU1NGQ0YWFhNjM0MDkxNTA1NGQ1NWYuYmluZFBvcHVwKHBvcHVwXzViNzE5MWFkODIxYjQzYjQ5YzUzZDIzNzBkZjc5NTkxKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzEzNWUwOWZjYTNiYjQ3MTNhZGUxNWUwNDE2NTkzMDA5ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3Ljg5NDI5MSwxNDUuMDc2NDMwOV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjMDBiNWViIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzAwYjVlYiIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8xMDVjODNlNTM5YmU0OWI3OGQ4OGI3YmM1YjI3YmZlMSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF81MWZlNzI2ODczZTU0YmE5OTg3ZGRkYmQyZWFlZGJkMSA9ICQoJzxkaXYgaWQ9Imh0bWxfNTFmZTcyNjg3M2U1NGJhOTk4N2RkZGJkMmVhZWRiZDEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkh1Z2hlc2RhbGUgQ2x1c3RlciA0PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8xMDVjODNlNTM5YmU0OWI3OGQ4OGI3YmM1YjI3YmZlMS5zZXRDb250ZW50KGh0bWxfNTFmZTcyNjg3M2U1NGJhOTk4N2RkZGJkMmVhZWRiZDEpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMTM1ZTA5ZmNhM2JiNDcxM2FkZTE1ZTA0MTY1OTMwMDkuYmluZFBvcHVwKHBvcHVwXzEwNWM4M2U1MzliZTQ5Yjc4ZDg4YjdiYzViMjdiZmUxKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzQzYTIxYWMzYWVmNDQzM2Q4MDQ1YTFmODg2MWRkNTE4ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3Ljc5MzkzNzgsMTQ0LjkzMDU2NDVdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzgwMDBmZiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4MDAwZmYiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNGQ2Y2EwNzc1MWUxNDE1MmE5OTI2OTlhYjQ0ZTM5ZGUgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNWU4ZTU5NGI4MzA0NDJlNzliM2JjYjg5MzRhZDE2ZTMgPSAkKCc8ZGl2IGlkPSJodG1sXzVlOGU1OTRiODMwNDQyZTc5YjNiY2I4OTM0YWQxNmUzIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5LZW5zaW5ndG9uIChWaWMuKSBDbHVzdGVyIDE8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzRkNmNhMDc3NTFlMTQxNTJhOTkyNjk5YWI0NGUzOWRlLnNldENvbnRlbnQoaHRtbF81ZThlNTk0YjgzMDQ0MmU3OWIzYmNiODkzNGFkMTZlMyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl80M2EyMWFjM2FlZjQ0MzNkODA0NWExZjg4NjFkZDUxOC5iaW5kUG9wdXAocG9wdXBfNGQ2Y2EwNzc1MWUxNDE1MmE5OTI2OTlhYjQ0ZTM5ZGUpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMTU2YjI5N2M4Y2I3NDcwMDliMTk5NzAwMDc3MWZkOTUgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuODAzMzU4NiwxNDUuMDMzMzI4NV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmY0MTIxIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmNDEyMSIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF83YmI2OGQ2MWZiZjI0ODM5ODk3MWFiNWZlZDY3YzhkZCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9hMDRkYjllY2YxNjU0MDM0OTA3MTkxZmY5NDk5ZjA2YiA9ICQoJzxkaXYgaWQ9Imh0bWxfYTA0ZGI5ZWNmMTY1NDAzNDkwNzE5MWZmOTQ5OWYwNmIiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPktldyAoVmljLikgQ2x1c3RlciAxMjwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfN2JiNjhkNjFmYmYyNDgzOTg5NzFhYjVmZWQ2N2M4ZGQuc2V0Q29udGVudChodG1sX2EwNGRiOWVjZjE2NTQwMzQ5MDcxOTFmZjk0OTlmMDZiKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzE1NmIyOTdjOGNiNzQ3MDA5YjE5OTcwMDA3NzFmZDk1LmJpbmRQb3B1cChwb3B1cF83YmI2OGQ2MWZiZjI0ODM5ODk3MWFiNWZlZDY3YzhkZCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8wYjI5Y2IzNmQ0ZTM0MmYyOTZiZDQ4YTY3ZDEzM2E0YSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy43OTA0MjE1LDE0NS4wNTI3OTA2XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiM1NGY2Y2IiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjNTRmNmNiIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzdmMzRlMzE2OGI2MzRhMGU4OWU4Y2Q1ZDE1N2VlOWIxID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzRmN2I4M2MzY2QzMTQzOWFiNjhhYWE5MWZjM2I3ZDc4ID0gJCgnPGRpdiBpZD0iaHRtbF80ZjdiODNjM2NkMzE0MzlhYjY4YWFhOTFmYzNiN2Q3OCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+S2V3IEVhc3QgQ2x1c3RlciA2PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF83ZjM0ZTMxNjhiNjM0YTBlODllOGNkNWQxNTdlZTliMS5zZXRDb250ZW50KGh0bWxfNGY3YjgzYzNjZDMxNDM5YWI2OGFhYTkxZmMzYjdkNzgpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMGIyOWNiMzZkNGUzNDJmMjk2YmQ0OGE2N2QxMzNhNGEuYmluZFBvcHVwKHBvcHVwXzdmMzRlMzE2OGI2MzRhMGU4OWU4Y2Q1ZDE1N2VlOWIxKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzk0YmJiZTdhYmY1OTQ0OGVhZWNjNTA2NDZlMjI4ODQ3ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3Ljg0MTM5NCwxNDUuMDM1ODQwMl0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmZiMzYwIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmYjM2MCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9mOWZmZDhlM2Y0NmE0YTRmYWUwY2UyYmE0MWRjOGE4ZSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9hZDFhNzI3ZmRlMGI0ZTllYTkzNmM4OWUyZjU0MTBiMSA9ICQoJzxkaXYgaWQ9Imh0bWxfYWQxYTcyN2ZkZTBiNGU5ZWE5MzZjODllMmY1NDEwYjEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPktvb3lvbmcgQ2x1c3RlciAxMDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfZjlmZmQ4ZTNmNDZhNGE0ZmFlMGNlMmJhNDFkYzhhOGUuc2V0Q29udGVudChodG1sX2FkMWE3MjdmZGUwYjRlOWVhOTM2Yzg5ZTJmNTQxMGIxKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzk0YmJiZTdhYmY1OTQ0OGVhZWNjNTA2NDZlMjI4ODQ3LmJpbmRQb3B1cChwb3B1cF9mOWZmZDhlM2Y0NmE0YTRmYWUwY2UyYmE0MWRjOGE4ZSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9hODYxMjkxMDc1YmU0MDdkYjQwODkyMWM2ZGNmYmM2NSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy44NTc2MDg4LDE0NS4wMzUwNjY2XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiNhYmY2OWIiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjYWJmNjliIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2Y1YmFmZTY2Nzk1NTQwOGNhZTBiYWFlYzRhZGM1MzRmID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2UyMTRmNGNmNzlkMzRjOWVhZjIxZDdjMjcwNWExZTZiID0gJCgnPGRpdiBpZD0iaHRtbF9lMjE0ZjRjZjc5ZDM0YzllYWYyMWQ3YzI3MDVhMWU2YiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+TWFsdmVybiAoVmljLikgQ2x1c3RlciA4PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9mNWJhZmU2Njc5NTU0MDhjYWUwYmFhZWM0YWRjNTM0Zi5zZXRDb250ZW50KGh0bWxfZTIxNGY0Y2Y3OWQzNGM5ZWFmMjFkN2MyNzA1YTFlNmIpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfYTg2MTI5MTA3NWJlNDA3ZGI0MDg5MjFjNmRjZmJjNjUuYmluZFBvcHVwKHBvcHVwX2Y1YmFmZTY2Nzk1NTQwOGNhZTBiYWFlYzRhZGM1MzRmKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2Y2NmY1NmQ0ODkwMTRiZmY5MjNiZjVhNmI0ZmNkNzJkID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3Ljg3NjgzNSwxNDUuMDY1ODczOV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmYwMDAwIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF85ODUyYjk2NjEwZDM0Y2M4YmY1ZmRmYTI2MDQ2MTQ0NiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8yMDg4N2Q3M2ZmZjk0M2RiYjYwNzYzMDhiNTZkZjJmZiA9ICQoJzxkaXYgaWQ9Imh0bWxfMjA4ODdkNzNmZmY5NDNkYmI2MDc2MzA4YjU2ZGYyZmYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPk1hbHZlcm4gRWFzdCBDbHVzdGVyIDA8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzk4NTJiOTY2MTBkMzRjYzhiZjVmZGZhMjYwNDYxNDQ2LnNldENvbnRlbnQoaHRtbF8yMDg4N2Q3M2ZmZjk0M2RiYjYwNzYzMDhiNTZkZjJmZik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9mNjZmNTZkNDg5MDE0YmZmOTIzYmY1YTZiNGZjZDcyZC5iaW5kUG9wdXAocG9wdXBfOTg1MmI5NjYxMGQzNGNjOGJmNWZkZmEyNjA0NjE0NDYpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNDk2M2QxYzBmMDZiNDlhN2I3NjgzZGYyMDMyMDlkN2QgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuOTEwOTc2OSwxNDUuMDM4MTQzN10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjMDBiNWViIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzAwYjVlYiIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF81ZGJjMmYyNjM3YmU0ZWViOGE3ZDIwN2EzZTJjZmQ2MyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF82OWM5OWYzNjFiMTU0MjM0ODQ5M2JkMzFjMzA0YWViNCA9ICQoJzxkaXYgaWQ9Imh0bWxfNjljOTlmMzYxYjE1NDIzNDg0OTNiZDMxYzMwNGFlYjQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPk1jS2lubm9uIENsdXN0ZXIgNDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNWRiYzJmMjYzN2JlNGVlYjhhN2QyMDdhM2UyY2ZkNjMuc2V0Q29udGVudChodG1sXzY5Yzk5ZjM2MWIxNTQyMzQ4NDkzYmQzMWMzMDRhZWI0KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzQ5NjNkMWMwZjA2YjQ5YTdiNzY4M2RmMjAzMjA5ZDdkLmJpbmRQb3B1cChwb3B1cF81ZGJjMmYyNjM3YmU0ZWViOGE3ZDIwN2EzZTJjZmQ2Myk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9kY2UwZWVjNGJlNjQ0NGJhYTc3ZTk0NjdiNmQzYmRiZiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy44MTQyMTc2LDE0NC45NjMxNjA4XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiNmZjdlNDEiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjZmY3ZTQxIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzAxZDAxMjFlY2M4NjQ5ZDk4MmI2ODBkMzUwNzQ4OWYyID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzk1MGQ3ZWIyYzgxNzQzNjJhMGViNTVhOWFiYjgwYTk0ID0gJCgnPGRpdiBpZD0iaHRtbF85NTBkN2ViMmM4MTc0MzYyYTBlYjU1YTlhYmI4MGE5NCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+TWVsYm91cm5lIENsdXN0ZXIgMTE8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzAxZDAxMjFlY2M4NjQ5ZDk4MmI2ODBkMzUwNzQ4OWYyLnNldENvbnRlbnQoaHRtbF85NTBkN2ViMmM4MTc0MzYyYTBlYjU1YTlhYmI4MGE5NCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9kY2UwZWVjNGJlNjQ0NGJhYTc3ZTk0NjdiNmQzYmRiZi5iaW5kUG9wdXAocG9wdXBfMDFkMDEyMWVjYzg2NDlkOTgyYjY4MGQzNTA3NDg5ZjIpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfOTk2YjUxZGM1M2M0NGI0MmI3YjllZWEwYTZiMTMxZTcgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuOTgxMzY2OSwxNDUuMDY4OTUxMV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmY0MTIxIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmNDEyMSIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9iMzZiZGJjMmQzODE0YjFlOTRlOWUyNzEwOGIyMDk0NiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8wNjlkZGNlY2ExNzc0ZjAwYjVmYWEwMGZiNzM3YzkxZCA9ICQoJzxkaXYgaWQ9Imh0bWxfMDY5ZGRjZWNhMTc3NGYwMGI1ZmFhMDBmYjczN2M5MWQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPk1lbnRvbmUgQ2x1c3RlciAxMjwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYjM2YmRiYzJkMzgxNGIxZTk0ZTllMjcxMDhiMjA5NDYuc2V0Q29udGVudChodG1sXzA2OWRkY2VjYTE3NzRmMDBiNWZhYTAwZmI3MzdjOTFkKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzk5NmI1MWRjNTNjNDRiNDJiN2I5ZWVhMGE2YjEzMWU3LmJpbmRQb3B1cChwb3B1cF9iMzZiZGJjMmQzODE0YjFlOTRlOWUyNzEwOGIyMDk0Nik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9kODlmMWFiMThlYjQ0MWIwYjk0NmRhNDczNzU1MTBkMCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy44NTExNTExLDE0NC45NjIwMzk5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiM1NjQxZmQiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjNTY0MWZkIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzg4ZmY4MGEzZjZlYzRjMWVhZWIyYTE3ZTA4YzYyNDIxID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzY1NjliNTA5NTNlZDQ5ZWFhMmMzNDczNDIwNzY5YmM1ID0gJCgnPGRpdiBpZD0iaHRtbF82NTY5YjUwOTUzZWQ0OWVhYTJjMzQ3MzQyMDc2OWJjNSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+TWlkZGxlIFBhcmsgKFZpYy4pIENsdXN0ZXIgMjwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfODhmZjgwYTNmNmVjNGMxZWFlYjJhMTdlMDhjNjI0MjEuc2V0Q29udGVudChodG1sXzY1NjliNTA5NTNlZDQ5ZWFhMmMzNDczNDIwNzY5YmM1KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2Q4OWYxYWIxOGViNDQxYjBiOTQ2ZGE0NzM3NTUxMGQwLmJpbmRQb3B1cChwb3B1cF84OGZmODBhM2Y2ZWM0YzFlYWViMmExN2UwOGM2MjQyMSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl83ZmU2YWZiZTQzZjc0Mjg2YjMzYTJjZTU4ZDYzMGYwYiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy44MTk1MzUsMTQ1LjEwNTUzMzRdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzAwYjVlYiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMwMGI1ZWIiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYmE0ODI1NmE3MjM5NGZjYTllZWEzMjcxMzMxYmQ3NGIgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMjFiNDEzNWU0YjMyNDc3MGJjYWNjMWUzMGIxN2M2YzMgPSAkKCc8ZGl2IGlkPSJodG1sXzIxYjQxMzVlNGIzMjQ3NzBiY2FjYzFlMzBiMTdjNmMzIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Nb250IEFsYmVydCBDbHVzdGVyIDQ8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2JhNDgyNTZhNzIzOTRmY2E5ZWVhMzI3MTMzMWJkNzRiLnNldENvbnRlbnQoaHRtbF8yMWI0MTM1ZTRiMzI0NzcwYmNhY2MxZTMwYjE3YzZjMyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl83ZmU2YWZiZTQzZjc0Mjg2YjMzYTJjZTU4ZDYzMGYwYi5iaW5kUG9wdXAocG9wdXBfYmE0ODI1NmE3MjM5NGZjYTllZWEzMjcxMzMxYmQ3NGIpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZDgxYjQ3ZTY4N2ZkNGQ5OTg0NDVlN2Q5YzZkMWYzNGMgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuODAzMzE1MiwxNDUuMTEwNzIyNl0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjNTRmNmNiIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzU0ZjZjYiIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF85YzcwYmIwODdhMDA0YjFiYTE5M2EyYWZmMGM5MjQ3NSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9hMTI0MzRjZGQ2YTg0ZDA0OWNlOGU5NGFkYjk0MDNlZiA9ICQoJzxkaXYgaWQ9Imh0bWxfYTEyNDM0Y2RkNmE4NGQwNDljZThlOTRhZGI5NDAzZWYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPk1vbnQgQWxiZXJ0IE5vcnRoIENsdXN0ZXIgNjwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfOWM3MGJiMDg3YTAwNGIxYmExOTNhMmFmZjBjOTI0NzUuc2V0Q29udGVudChodG1sX2ExMjQzNGNkZDZhODRkMDQ5Y2U4ZTk0YWRiOTQwM2VmKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2Q4MWI0N2U2ODdmZDRkOTk4NDQ1ZTdkOWM2ZDFmMzRjLmJpbmRQb3B1cChwb3B1cF85YzcwYmIwODdhMDA0YjFiYTE5M2EyYWZmMGM5MjQ3NSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl83YjIzNWYyNGU3NDA0NGIxYTg3YTIzZWU4ZDE4NDUxOCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy43NjU1NzUsMTQ0LjkyMTI4NjRdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzJjN2VmNyIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMyYzdlZjciLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfOTg0YWIwYmJiOTE2NDg3MGFkYzkzNmI4ZmUyMjRhYjYgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZjgzZjkyODYwMWYzNGYyODhmNDdjOGViZGQzMWUyY2QgPSAkKCc8ZGl2IGlkPSJodG1sX2Y4M2Y5Mjg2MDFmMzRmMjg4ZjQ3YzhlYmRkMzFlMmNkIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Nb29uZWUgUG9uZHMgQ2x1c3RlciAzPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF85ODRhYjBiYmI5MTY0ODcwYWRjOTM2YjhmZTIyNGFiNi5zZXRDb250ZW50KGh0bWxfZjgzZjkyODYwMWYzNGYyODhmNDdjOGViZGQzMWUyY2QpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfN2IyMzVmMjRlNzQwNDRiMWE4N2EyM2VlOGQxODQ1MTguYmluZFBvcHVwKHBvcHVwXzk4NGFiMGJiYjkxNjQ4NzBhZGM5MzZiOGZlMjI0YWI2KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzgwOWI3MTM1YWE0MjQ2YTdhODQ0MjhmZDBhYWYwMGZiID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3LjkzNDM5MTksMTQ1LjAzNjg5NTJdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzgwMDBmZiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4MDAwZmYiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMWFkNzkwMDAxNGVmNDQwNGFjMGZmZjFlYjc3ZDJmNjQgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNTM4NjcxMzgzOWEyNDg1OTkyNjk3ZTAzNmUwNmFjNmYgPSAkKCc8ZGl2IGlkPSJodG1sXzUzODY3MTM4MzlhMjQ4NTk5MjY5N2UwMzZlMDZhYzZmIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Nb29yYWJiaW4gQ2x1c3RlciAxPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8xYWQ3OTAwMDE0ZWY0NDA0YWMwZmZmMWViNzdkMmY2NC5zZXRDb250ZW50KGh0bWxfNTM4NjcxMzgzOWEyNDg1OTkyNjk3ZTAzNmUwNmFjNmYpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfODA5YjcxMzVhYTQyNDZhN2E4NDQyOGZkMGFhZjAwZmIuYmluZFBvcHVwKHBvcHVwXzFhZDc5MDAwMTRlZjQ0MDRhYzBmZmYxZWI3N2QyZjY0KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzkzMjYyNjIwNDgwYTQ4ZTM5NGNhMDcyZmNhMzRiYzY5ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM4LjAwNjcxNzQsMTQ1LjA4NzU2NThdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzAwYjVlYiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMwMGI1ZWIiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfM2UzMzg0OTI1MzNmNDFmNjkyYTc1NzZlNTQwYjIyYmUgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfYmVlZWRkMWQ2YThmNGYzZGI4NmFkN2MzMjRlYjhkMzAgPSAkKCc8ZGl2IGlkPSJodG1sX2JlZWVkZDFkNmE4ZjRmM2RiODZhZDdjMzI0ZWI4ZDMwIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Nb3JkaWFsbG9jIENsdXN0ZXIgNDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfM2UzMzg0OTI1MzNmNDFmNjkyYTc1NzZlNTQwYjIyYmUuc2V0Q29udGVudChodG1sX2JlZWVkZDFkNmE4ZjRmM2RiODZhZDdjMzI0ZWI4ZDMwKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzkzMjYyNjIwNDgwYTQ4ZTM5NGNhMDcyZmNhMzRiYzY5LmJpbmRQb3B1cChwb3B1cF8zZTMzODQ5MjUzM2Y0MWY2OTJhNzU3NmU1NDBiMjJiZSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9jNjBkODM5MDc2Yjc0ZmQ5YmU0MzAxNmRhZDFmMDM2NiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy44OTAxODQ1LDE0NS4wNjczMzVdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzU2NDFmZCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM1NjQxZmQiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZGFkY2JhNzcwYjMwNGRlYTkwOWJmZDQwMjRhYjU1YmUgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfYzk0NTUzMzU1ZjVkNDMyNWJhYjUxNzg5NzEwNWI2NmMgPSAkKCc8ZGl2IGlkPSJodG1sX2M5NDU1MzM1NWY1ZDQzMjViYWI1MTc4OTcxMDViNjZjIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5NdXJydW1iZWVuYSBDbHVzdGVyIDI8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2RhZGNiYTc3MGIzMDRkZWE5MDliZmQ0MDI0YWI1NWJlLnNldENvbnRlbnQoaHRtbF9jOTQ1NTMzNTVmNWQ0MzI1YmFiNTE3ODk3MTA1YjY2Yyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9jNjBkODM5MDc2Yjc0ZmQ5YmU0MzAxNmRhZDFmMDM2Ni5iaW5kUG9wdXAocG9wdXBfZGFkY2JhNzcwYjMwNGRlYTkwOWJmZDQwMjRhYjU1YmUpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNmVlZmFiNmVkN2U1NDU4YmFkNDgyZGM1NDFjZTg3YTUgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuODA3NjA5MiwxNDQuOTQyMzUxNF0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjMmFkZGRkIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzJhZGRkZCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9lZmU2NzljMjUyYTc0NDllYTA2ZTQ3ZjY2NTBhNzM5NCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9jYTAzMTE2NjkyZDM0NmJkYjNkODQwZmYzMmE2MzMxNCA9ICQoJzxkaXYgaWQ9Imh0bWxfY2EwMzExNjY5MmQzNDZiZGIzZDg0MGZmMzJhNjMzMTQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPk5vcnRoIE1lbGJvdXJuZSBDbHVzdGVyIDU8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2VmZTY3OWMyNTJhNzQ0OWVhMDZlNDdmNjY1MGE3Mzk0LnNldENvbnRlbnQoaHRtbF9jYTAzMTE2NjkyZDM0NmJkYjNkODQwZmYzMmE2MzMxNCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl82ZWVmYWI2ZWQ3ZTU0NThiYWQ0ODJkYzU0MWNlODdhNS5iaW5kUG9wdXAocG9wdXBfZWZlNjc5YzI1MmE3NDQ5ZWEwNmU0N2Y2NjUwYTczOTQpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMTY5Nzc3NjAxN2QzNDdmMmEzNDMzYmZkY2Y3MTQwYTkgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuNzczMDgsMTQ1LjAxMDMyNDU0Nzc1Nzg1XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiM1NGY2Y2IiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjNTRmNmNiIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2Y5YjMyMTY4NjUxNDQwZmZhN2NhMTMzZmNmNzRlNzAzID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzUzMWMxMjgxM2RkNTRlNGY4ZjA0MzkxM2YzMDI2NTAxID0gJCgnPGRpdiBpZD0iaHRtbF81MzFjMTI4MTNkZDU0ZTRmOGYwNDM5MTNmMzAyNjUwMSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Tm9ydGhjb3RlIENsdXN0ZXIgNjwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfZjliMzIxNjg2NTE0NDBmZmE3Y2ExMzNmY2Y3NGU3MDMuc2V0Q29udGVudChodG1sXzUzMWMxMjgxM2RkNTRlNGY4ZjA0MzkxM2YzMDI2NTAxKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzE2OTc3NzYwMTdkMzQ3ZjJhMzQzM2JmZGNmNzE0MGE5LmJpbmRQb3B1cChwb3B1cF9mOWIzMjE2ODY1MTQ0MGZmYTdjYTEzM2ZjZjc0ZTcwMyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8yNjFmNzI0Y2MzMGM0ZmYwOGM2MWRjZTFmNjVmM2JhMSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy45MDM1NjUzLDE0NS4wMzk1MDNdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzU0ZjZjYiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM1NGY2Y2IiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMzE4YzBiZTUzNDkyNDQ4NDk4Y2Q5NzM4YjBkMjM4MTcgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfODVlOTMwMDg5ODBkNGE2ZjgyNWZmYTkzMDY2NmU1YjQgPSAkKCc8ZGl2IGlkPSJodG1sXzg1ZTkzMDA4OTgwZDRhNmY4MjVmZmE5MzA2NjZlNWI0IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Pcm1vbmQgQ2x1c3RlciA2PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8zMThjMGJlNTM0OTI0NDg0OThjZDk3MzhiMGQyMzgxNy5zZXRDb250ZW50KGh0bWxfODVlOTMwMDg5ODBkNGE2ZjgyNWZmYTkzMDY2NmU1YjQpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMjYxZjcyNGNjMzBjNGZmMDhjNjFkY2UxZjY1ZjNiYTEuYmluZFBvcHVwKHBvcHVwXzMxOGMwYmU1MzQ5MjQ0ODQ5OGNkOTczOGIwZDIzODE3KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzRlMzQyMDdiYzVlMTQzNDZiZTJlZDZjYWNhZDM1ZmRlID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3Ljk5MzE1NjMsMTQ1LjA3NjM3MDNdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzgwMDBmZiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4MDAwZmYiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMWRjOTBjMDQyOWQ4NDgwNjlkMDk3ZjYwZDc0YjQ2ZTQgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfM2I1MTdjYzc5ZWI4NGJkNjg0YTZkZGI1MmE2NTE5ODkgPSAkKCc8ZGl2IGlkPSJodG1sXzNiNTE3Y2M3OWViODRiZDY4NGE2ZGRiNTJhNjUxOTg5IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5QYXJrZGFsZSBDbHVzdGVyIDE8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzFkYzkwYzA0MjlkODQ4MDY5ZDA5N2Y2MGQ3NGI0NmU0LnNldENvbnRlbnQoaHRtbF8zYjUxN2NjNzllYjg0YmQ2ODRhNmRkYjUyYTY1MTk4OSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl80ZTM0MjA3YmM1ZTE0MzQ2YmUyZWQ2Y2FjYWQzNWZkZS5iaW5kUG9wdXAocG9wdXBfMWRjOTBjMDQyOWQ4NDgwNjlkMDk3ZjYwZDc0YjQ2ZTQpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfYzNmZmUzYmVmYjU1NGM3OTkwOTZlZWJhOTY5ZmExNmIgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuNzg3MTE0OCwxNDQuOTUxNTUzM10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjODBmZmI0IiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzgwZmZiNCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF84MzRjNDEzOGMyMDk0MjM2YWQ3YWNiYTQ5OTIxNDRhZiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF83Njg4OWI2ZDYzYmQ0N2FhYWM0YWE1NGMwMDdkZjRhZiA9ICQoJzxkaXYgaWQ9Imh0bWxfNzY4ODliNmQ2M2JkNDdhYWFjNGFhNTRjMDA3ZGY0YWYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlBhcmt2aWxsZSAoVmljLikgQ2x1c3RlciA3PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF84MzRjNDEzOGMyMDk0MjM2YWQ3YWNiYTQ5OTIxNDRhZi5zZXRDb250ZW50KGh0bWxfNzY4ODliNmQ2M2JkNDdhYWFjNGFhNTRjMDA3ZGY0YWYpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfYzNmZmUzYmVmYjU1NGM3OTkwOTZlZWJhOTY5ZmExNmIuYmluZFBvcHVwKHBvcHVwXzgzNGM0MTM4YzIwOTQyMzZhZDdhY2JhNDk5MjE0NGFmKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzU2MzRkOWVmZGViMzQ4MDQ4YmMxZTExMGFkNDJmMWE2ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3LjczMDgwOTksMTQ0LjkyODIxODldLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2ZmMDAwMCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNmZjAwMDAiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZGU5ZjlmMDEyNmQ1NGQxNzlhYTBlNjMyMjllZDI5MWEgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZjBhM2VlMjM1NmYwNGU2YTgyMDYxNDVlM2M4OThkZTEgPSAkKCc8ZGl2IGlkPSJodG1sX2YwYTNlZTIzNTZmMDRlNmE4MjA2MTQ1ZTNjODk4ZGUxIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5QYXNjb2UgVmFsZSBDbHVzdGVyIDA8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2RlOWY5ZjAxMjZkNTRkMTc5YWEwZTYzMjI5ZWQyOTFhLnNldENvbnRlbnQoaHRtbF9mMGEzZWUyMzU2ZjA0ZTZhODIwNjE0NWUzYzg5OGRlMSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl81NjM0ZDllZmRlYjM0ODA0OGJjMWUxMTBhZDQyZjFhNi5iaW5kUG9wdXAocG9wdXBfZGU5ZjlmMDEyNmQ1NGQxNzlhYTBlNjMyMjllZDI5MWEpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMzliMDQwY2VkNDI3NDQxYWFiMDE1NDJiYzZiYmM1M2YgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuNzQzNDI2NSwxNDQuOTM4NDc2N10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjNTRmNmNiIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzU0ZjZjYiIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9lODVhY2FjMjMwZjk0MGI5OGE4MWE2MDY2Njk2NmQ0ZiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9iNmJhNTkyMDNiOWY0MDJkOTRhZDgzOWViZTVmZDY1MCA9ICQoJzxkaXYgaWQ9Imh0bWxfYjZiYTU5MjAzYjlmNDAyZDk0YWQ4MzllYmU1ZmQ2NTAiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlBhc2NvZSBWYWxlIFNvdXRoIENsdXN0ZXIgNjwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfZTg1YWNhYzIzMGY5NDBiOThhODFhNjA2NjY5NjZkNGYuc2V0Q29udGVudChodG1sX2I2YmE1OTIwM2I5ZjQwMmQ5NGFkODM5ZWJlNWZkNjUwKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzM5YjA0MGNlZDQyNzQ0MWFhYjAxNTQyYmM2YmJjNTNmLmJpbmRQb3B1cChwb3B1cF9lODVhY2FjMjMwZjk0MGI5OGE4MWE2MDY2Njk2NmQ0Zik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl82ZmYyMmQ3MWI4MTg0ZjY3OTJmNmU3MDc2MWExYTI1MiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zOC4wNjk0NDkzLDE0NS4xNDMyMzI5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiMwMGI1ZWIiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMDBiNWViIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzQ2OGYxMjZmNGJhNDRhYjQ5MTZlMGEzOGJiOWYzNGEwID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzE3MzY0MzEwNTk2OTRiZGI5YWEyYThjNDc2YWJiOWI3ID0gJCgnPGRpdiBpZD0iaHRtbF8xNzM2NDMxMDU5Njk0YmRiOWFhMmE4YzQ3NmFiYjliNyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+UGF0dGVyc29uIExha2VzIENsdXN0ZXIgNDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNDY4ZjEyNmY0YmE0NGFiNDkxNmUwYTM4YmI5ZjM0YTAuc2V0Q29udGVudChodG1sXzE3MzY0MzEwNTk2OTRiZGI5YWEyYThjNDc2YWJiOWI3KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzZmZjIyZDcxYjgxODRmNjc5MmY2ZTcwNzYxYTFhMjUyLmJpbmRQb3B1cChwb3B1cF80NjhmMTI2ZjRiYTQ0YWI0OTE2ZTBhMzhiYjlmMzRhMCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9hYzQ5MDdkYzc0MTM0MGQxYWEwMzM2YjdkZWFhMTk2YiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy44MzMzNjEzLDE0NC45MjE5MjAzXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiM1NGY2Y2IiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjNTRmNmNiIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzAxYjg1YWUyMWIxZTQxMzI5YThmYjdkZDQ5YWFkMWE3ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2VhZTkyMGY0NDQ1OTQ2ZGNhNjAyMWIxNjk5NWZjOWY2ID0gJCgnPGRpdiBpZD0iaHRtbF9lYWU5MjBmNDQ0NTk0NmRjYTYwMjFiMTY5OTVmYzlmNiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+UG9ydCBNZWxib3VybmUgQ2x1c3RlciA2PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8wMWI4NWFlMjFiMWU0MTMyOWE4ZmI3ZGQ0OWFhZDFhNy5zZXRDb250ZW50KGh0bWxfZWFlOTIwZjQ0NDU5NDZkY2E2MDIxYjE2OTk1ZmM5ZjYpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfYWM0OTA3ZGM3NDEzNDBkMWFhMDMzNmI3ZGVhYTE5NmIuYmluZFBvcHVwKHBvcHVwXzAxYjg1YWUyMWIxZTQxMzI5YThmYjdkZDQ5YWFkMWE3KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzU2NjI3ZWE2NWE1YjQ3NTJhMDhhZWU0OTQwMTMwZDliID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3Ljg1MTkxMzUsMTQ1LjAwMDU5OTNdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzgwMDBmZiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4MDAwZmYiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYjNlYjFlOThiNDJkNDRkN2E2MmFkODY2MTRhNzhhOGEgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMjM2ZTQ5NGIxMWJhNDc2ZmI2ZTI5MDlhMWEzNWZkNGEgPSAkKCc8ZGl2IGlkPSJodG1sXzIzNmU0OTRiMTFiYTQ3NmZiNmUyOTA5YTFhMzVmZDRhIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5QcmFocmFuIENsdXN0ZXIgMTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYjNlYjFlOThiNDJkNDRkN2E2MmFkODY2MTRhNzhhOGEuc2V0Q29udGVudChodG1sXzIzNmU0OTRiMTFiYTQ3NmZiNmUyOTA5YTFhMzVmZDRhKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzU2NjI3ZWE2NWE1YjQ3NTJhMDhhZWU0OTQwMTMwZDliLmJpbmRQb3B1cChwb3B1cF9iM2ViMWU5OGI0MmQ0NGQ3YTYyYWQ4NjYxNGE3OGE4YSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9iZGUxY2JmZDY3YWY0OTU5OWQ1MTExZGU1MmE5MmI1ZCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy43ODE4MDQxLDE0NC45NjY1Njc1XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiNmZjAwMDAiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzc0OTc1ZGQwYWI4MjQ0Y2ZhMGNhYjVlYzJmNTVkY2JiID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2Y1Yjk1MTkzN2JiOTQxYmY4Y2FhYTFjNzcwMmNmMzgyID0gJCgnPGRpdiBpZD0iaHRtbF9mNWI5NTE5MzdiYjk0MWJmOGNhYWExYzc3MDJjZjM4MiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+UHJpbmNlcyBIaWxsIENsdXN0ZXIgMDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNzQ5NzVkZDBhYjgyNDRjZmEwY2FiNWVjMmY1NWRjYmIuc2V0Q29udGVudChodG1sX2Y1Yjk1MTkzN2JiOTQxYmY4Y2FhYTFjNzcwMmNmMzgyKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2JkZTFjYmZkNjdhZjQ5NTk5ZDUxMTFkZTUyYTkyYjVkLmJpbmRQb3B1cChwb3B1cF83NDk3NWRkMGFiODI0NGNmYTBjYWI1ZWMyZjU1ZGNiYik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8zZTJhY2IxZmIxOTg0YWEzYTc4NGRlY2VlZTdkOTg5OCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy44MDc0NSwxNDQuOTkwNzE3NTc5NzIwODddLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzJhZGRkZCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMyYWRkZGQiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZTM4N2M5MDBiOTg4NDI0N2IzY2JiYjhiZDZiMTYxMDcgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZjNkOGFiYTZkYTFlNDc1YjhkYWY5NTEzMjZlMzQ0YmQgPSAkKCc8ZGl2IGlkPSJodG1sX2YzZDhhYmE2ZGExZTQ3NWI4ZGFmOTUxMzI2ZTM0NGJkIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5SaWNobW9uZCAoVmljLikgQ2x1c3RlciA1PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9lMzg3YzkwMGI5ODg0MjQ3YjNjYmJiOGJkNmIxNjEwNy5zZXRDb250ZW50KGh0bWxfZjNkOGFiYTZkYTFlNDc1YjhkYWY5NTEzMjZlMzQ0YmQpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfM2UyYWNiMWZiMTk4NGFhM2E3ODRkZWNlZWU3ZDk4OTguYmluZFBvcHVwKHBvcHVwX2UzODdjOTAwYjk4ODQyNDdiM2NiYmI4YmQ2YjE2MTA3KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzNiZDE2NmYzOTVkMzRmODRhOWQ0YWU5YjYxZThjOWVhID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3Ljg3Nzc5NzgsMTQ0Ljk5NTI3NzhdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2Q0ZGQ4MCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNkNGRkODAiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNmNkY2NiNTg4ODJmNDIwY2IyZjEzZWQ0Y2U5MWVkNjIgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZWVhZjgwYzg5ODA2NGJmOWI3OTlkNWY4ZjU3NGFiNDIgPSAkKCc8ZGl2IGlkPSJodG1sX2VlYWY4MGM4OTgwNjRiZjliNzk5ZDVmOGY1NzRhYjQyIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5SaXBwb25sZWEgQ2x1c3RlciA5PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF82Y2RjY2I1ODg4MmY0MjBjYjJmMTNlZDRjZTkxZWQ2Mi5zZXRDb250ZW50KGh0bWxfZWVhZjgwYzg5ODA2NGJmOWI3OTlkNWY4ZjU3NGFiNDIpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfM2JkMTY2ZjM5NWQzNGY4NGE5ZDRhZTliNjFlOGM5ZWEuYmluZFBvcHVwKHBvcHVwXzZjZGNjYjU4ODgyZjQyMGNiMmYxM2VkNGNlOTFlZDYyKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzJjYTgzMmU3ZGU1ZTQ2YTJhZWRhODAxZjRhODJjNmM3ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3Ljk1MDMwMSwxNDUuMDA0Mzg3NV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmY0MTIxIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmNDEyMSIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF81NGJkMWEzNTcxYjU0MjI3YmU5ZTM0MTYyOTlkNDdiOCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF83MmE2YTcyODczMTE0MzgxODI0MzQ0OTJkOTJlYjg2NiA9ICQoJzxkaXYgaWQ9Imh0bWxfNzJhNmE3Mjg3MzExNDM4MTgyNDM0NDkyZDkyZWI4NjYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlNhbmRyaW5naGFtIChWaWMuKSBDbHVzdGVyIDEyPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF81NGJkMWEzNTcxYjU0MjI3YmU5ZTM0MTYyOTlkNDdiOC5zZXRDb250ZW50KGh0bWxfNzJhNmE3Mjg3MzExNDM4MTgyNDM0NDkyZDkyZWI4NjYpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMmNhODMyZTdkZTVlNDZhMmFlZGE4MDFmNGE4MmM2YzcuYmluZFBvcHVwKHBvcHVwXzU0YmQxYTM1NzFiNTQyMjdiZTllMzQxNjI5OWQ0N2I4KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2Q0YmY4YjQyOGIzZDQ2YWE4ODIxYWY3YTU1YjlkNTU2ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3LjgzMzQ2MDMsMTQ0Ljk1NzAxODhdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzJhZGRkZCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMyYWRkZGQiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNDM3MDY0ZDMzYTRhNDRjZmI3MGE0N2VhOTdmNmZmYTUgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMzlmZWZmYzBiYjRlNGM5MzlkOTgzNDI3Nzg1OTY0NTUgPSAkKCc8ZGl2IGlkPSJodG1sXzM5ZmVmZmMwYmI0ZTRjOTM5ZDk4MzQyNzc4NTk2NDU1IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Tb3V0aCBNZWxib3VybmUgQ2x1c3RlciA1PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF80MzcwNjRkMzNhNGE0NGNmYjcwYTQ3ZWE5N2Y2ZmZhNS5zZXRDb250ZW50KGh0bWxfMzlmZWZmYzBiYjRlNGM5MzlkOTgzNDI3Nzg1OTY0NTUpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfZDRiZjhiNDI4YjNkNDZhYTg4MjFhZjdhNTViOWQ1NTYuYmluZFBvcHVwKHBvcHVwXzQzNzA2NGQzM2E0YTQ0Y2ZiNzBhNDdlYTk3ZjZmZmE1KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2M3N2ZhYjQzYTk3ODRiZDU5NzU5NzA1MWFiMTE4ZTMwID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3LjgyNTMyNzksMTQ0Ljk0OTM0MzJdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2ZmNDEyMSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNmZjQxMjEiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYzk1NzlkYzlmZDQ5NDAyMmE0MThmNzFhNzdjYzQxOTggPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfN2Y3MWMxOGJjZWVmNDM5MjgzZDYwYTU5OGYwZDM3MjUgPSAkKCc8ZGl2IGlkPSJodG1sXzdmNzFjMThiY2VlZjQzOTI4M2Q2MGE1OThmMGQzNzI1IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Tb3V0aCBXaGFyZiBDbHVzdGVyIDEyPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9jOTU3OWRjOWZkNDk0MDIyYTQxOGY3MWE3N2NjNDE5OC5zZXRDb250ZW50KGh0bWxfN2Y3MWMxOGJjZWVmNDM5MjgzZDYwYTU5OGYwZDM3MjUpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfYzc3ZmFiNDNhOTc4NGJkNTk3NTk3MDUxYWIxMThlMzAuYmluZFBvcHVwKHBvcHVwX2M5NTc5ZGM5ZmQ0OTQwMjJhNDE4ZjcxYTc3Y2M0MTk4KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzk4Nzg4N2IwOTRmNjQ1NzY5ZDI4MGRhNjkxYTNiNjU1ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3LjgzNzc2OTUsMTQ0Ljk5MTg1MzddLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2ZmNDEyMSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNmZjQxMjEiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMGFkMzZiZDY1ZjA2NDdlY2I2MzZhZDAxZmM5YjJlZTAgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMDU3YjFlMTI3YjE3NGE2NmJjODgzNThhNjE5YjNkM2QgPSAkKCc8ZGl2IGlkPSJodG1sXzA1N2IxZTEyN2IxNzRhNjZiYzg4MzU4YTYxOWIzZDNkIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Tb3V0aCBZYXJyYSBDbHVzdGVyIDEyPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8wYWQzNmJkNjVmMDY0N2VjYjYzNmFkMDFmYzliMmVlMC5zZXRDb250ZW50KGh0bWxfMDU3YjFlMTI3YjE3NGE2NmJjODgzNThhNjE5YjNkM2QpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfOTg3ODg3YjA5NGY2NDU3NjlkMjgwZGE2OTFhM2I2NTUuYmluZFBvcHVwKHBvcHVwXzBhZDM2YmQ2NWYwNjQ3ZWNiNjM2YWQwMWZjOWIyZWUwKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzEwZWExNDA4NTM1YzRhM2FiYzg3MmY3NDdhYzNmNTRiID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3LjgyNTM2MTgsMTQ0Ljk2NDAyMDNdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2Q0ZGQ4MCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNkNGRkODAiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNDIxMGY2NzhkMGJlNGM3ZDg1YzBiMTNjMjExMzE1MGUgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfYzc5Mzk5MGRlNTY1NDk4OWIyYjBmM2M2NmQxNGI5OWUgPSAkKCc8ZGl2IGlkPSJodG1sX2M3OTM5OTBkZTU2NTQ5ODliMmIwZjNjNjZkMTRiOTllIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Tb3V0aGJhbmsgQ2x1c3RlciA5PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF80MjEwZjY3OGQwYmU0YzdkODVjMGIxM2MyMTEzMTUwZS5zZXRDb250ZW50KGh0bWxfYzc5Mzk5MGRlNTY1NDk4OWIyYjBmM2M2NmQxNGI5OWUpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMTBlYTE0MDg1MzVjNGEzYWJjODcyZjc0N2FjM2Y1NGIuYmluZFBvcHVwKHBvcHVwXzQyMTBmNjc4ZDBiZTRjN2Q4NWMwYjEzYzIxMTMxNTBlKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2Q5MDk5ZmZlYjkxODRjYjlhY2UzOTZlZDU1YWNlNTJlID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3Ljg2MzgyNjEsMTQ0Ljk4MTYzN10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjODAwMGZmIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzgwMDBmZiIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9iZjhhMTQ3MGRkNjA0MTA4OWZhYTMwYzVkMGQ3ZWIyYSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF83N2NmYTdiY2UyOGE0OTRkOWUxNmYxYjYxOTAyYzk5YyA9ICQoJzxkaXYgaWQ9Imh0bWxfNzdjZmE3YmNlMjhhNDk0ZDllMTZmMWI2MTkwMmM5OWMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlN0IEtpbGRhIChWaWMuKSBDbHVzdGVyIDE8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2JmOGExNDcwZGQ2MDQxMDg5ZmFhMzBjNWQwZDdlYjJhLnNldENvbnRlbnQoaHRtbF83N2NmYTdiY2UyOGE0OTRkOWUxNmYxYjYxOTAyYzk5Yyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9kOTA5OWZmZWI5MTg0Y2I5YWNlMzk2ZWQ1NWFjZTUyZS5iaW5kUG9wdXAocG9wdXBfYmY4YTE0NzBkZDYwNDEwODlmYWEzMGM1ZDBkN2ViMmEpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfYmEyZTJmMWM3MDYxNGMxNzgwZWI4MjI2YjQ1NDYxN2UgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuODY2ODIxOCwxNDUuMDAyMTY1N10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmYwMDAwIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF83NWNkYmNhNGE1NWY0ZTA2YTE3ZmQ5Y2U4ZmZjMzAyOSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9lN2M1ZjhjODNiOGU0ZGMyYTc1OWRmYzFlYmUxNjFjNiA9ICQoJzxkaXYgaWQ9Imh0bWxfZTdjNWY4YzgzYjhlNGRjMmE3NTlkZmMxZWJlMTYxYzYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlN0IEtpbGRhIEVhc3QgQ2x1c3RlciAwPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF83NWNkYmNhNGE1NWY0ZTA2YTE3ZmQ5Y2U4ZmZjMzAyOS5zZXRDb250ZW50KGh0bWxfZTdjNWY4YzgzYjhlNGRjMmE3NTlkZmMxZWJlMTYxYzYpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfYmEyZTJmMWM3MDYxNGMxNzgwZWI4MjI2YjQ1NDYxN2UuYmluZFBvcHVwKHBvcHVwXzc1Y2RiY2E0YTU1ZjRlMDZhMTdmZDljZThmZmMzMDI5KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzEwYmNkZTczMDZjMjQyMTU4YjA3MDhlNjhmMDFmZDg4ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3Ljg1Nzk5NTgsMTQ0Ljk3MTI0NTJdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzAwYjVlYiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMwMGI1ZWIiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfOGY3YzM2YzBjZDNjNDJmOWJkZDQ0ZDg0NGRlZWY0NzcgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNTQ0MGZmODQyYWUxNDQ2N2EwZjE1YzllZjFlZjViY2EgPSAkKCc8ZGl2IGlkPSJodG1sXzU0NDBmZjg0MmFlMTQ0NjdhMGYxNWM5ZWYxZWY1YmNhIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5TdCBLaWxkYSBXZXN0IENsdXN0ZXIgNDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfOGY3YzM2YzBjZDNjNDJmOWJkZDQ0ZDg0NGRlZWY0Nzcuc2V0Q29udGVudChodG1sXzU0NDBmZjg0MmFlMTQ0NjdhMGYxNWM5ZWYxZWY1YmNhKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzEwYmNkZTczMDZjMjQyMTU4YjA3MDhlNjhmMDFmZDg4LmJpbmRQb3B1cChwb3B1cF84ZjdjMzZjMGNkM2M0MmY5YmRkNDRkODQ0ZGVlZjQ3Nyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl81MDRhMzEzNmQ5Nzk0ZTg4YmEzZDczZTBhM2MyMDk2YyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy44MjQxMTgzLDE0NS4wOTg2MjEyXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiM4MDAwZmYiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjODAwMGZmIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzM1NWViMmE5M2EyNzQ3MDQ4YTVlNDZiNDVhMjNmYjllID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzJjNGQ0MDNlNGE1ZjRlMjZiNWFiZTA3YzQzYzk0MTc2ID0gJCgnPGRpdiBpZD0iaHRtbF8yYzRkNDAzZTRhNWY0ZTI2YjVhYmUwN2M0M2M5NDE3NiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+U3VycmV5IEhpbGxzIENsdXN0ZXIgMTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMzU1ZWIyYTkzYTI3NDcwNDhhNWU0NmI0NWEyM2ZiOWUuc2V0Q29udGVudChodG1sXzJjNGQ0MDNlNGE1ZjRlMjZiNWFiZTA3YzQzYzk0MTc2KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzUwNGEzMTM2ZDk3OTRlODhiYTNkNzNlMGEzYzIwOTZjLmJpbmRQb3B1cChwb3B1cF8zNTVlYjJhOTNhMjc0NzA0OGE1ZTQ2YjQ1YTIzZmI5ZSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9iN2MxYTcwZjc1ZWE0ODkwYmM2ZDEzMDA1MTA3NWU2YyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy43NTQwNTg4LDE0NS4xNDg2MTUyXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiM1NGY2Y2IiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjNTRmNmNiIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzA2OGMzNzczYzJiNTRkZjc5Y2RkNzFjZTI2YzM2ODJkID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2U5ZjNlMTVmMDNmMTRmMzA4ODNiMjViYWNlODhhYjU3ID0gJCgnPGRpdiBpZD0iaHRtbF9lOWYzZTE1ZjAzZjE0ZjMwODgzYjI1YmFjZTg4YWI1NyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+VGVtcGxlc3Rvd2UgQ2x1c3RlciA2PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8wNjhjMzc3M2MyYjU0ZGY3OWNkZDcxY2UyNmMzNjgyZC5zZXRDb250ZW50KGh0bWxfZTlmM2UxNWYwM2YxNGYzMDg4M2IyNWJhY2U4OGFiNTcpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfYjdjMWE3MGY3NWVhNDg5MGJjNmQxMzAwNTEwNzVlNmMuYmluZFBvcHVwKHBvcHVwXzA2OGMzNzczYzJiNTRkZjc5Y2RkNzFjZTI2YzM2ODJkKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2RjNmVmOWE3N2Q3NDQ1YWNhOGFjOGVmMDZmNDI4NGU3ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3Ljc2Mzg3NDksMTQ1LjExMjI3NjJdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzU0ZjZjYiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM1NGY2Y2IiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZDIzMjc4ZTVmNDZmNGYxMjlkNjY0NWNjZGIxMDVjMDEgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNGVmNGJiMmJjZTQzNGM1MWJlNDVkMTFiMjAxNzdiN2IgPSAkKCc8ZGl2IGlkPSJodG1sXzRlZjRiYjJiY2U0MzRjNTFiZTQ1ZDExYjIwMTc3YjdiIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5UZW1wbGVzdG93ZSBMb3dlciBDbHVzdGVyIDY8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2QyMzI3OGU1ZjQ2ZjRmMTI5ZDY2NDVjY2RiMTA1YzAxLnNldENvbnRlbnQoaHRtbF80ZWY0YmIyYmNlNDM0YzUxYmU0NWQxMWIyMDE3N2I3Yik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9kYzZlZjlhNzdkNzQ0NWFjYThhYzhlZjA2ZjQyODRlNy5iaW5kUG9wdXAocG9wdXBfZDIzMjc4ZTVmNDZmNGYxMjlkNjY0NWNjZGIxMDVjMDEpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNTgxNDM2OTM2YmNiNDJhOTg3MTkwMWZhZjJkMjc5YjcgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuNzU1MDI3NiwxNDQuOTk4NjEzOV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjODAwMGZmIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzgwMDBmZiIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9hMWExYzE4YzA1Yjg0MDk4YWM5YmMzNDQ5ZGEwNTExMiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8zODI0MmY1YzE0NmI0NzI4ODllNDE5YWIxNWMxMDk2MiA9ICQoJzxkaXYgaWQ9Imh0bWxfMzgyNDJmNWMxNDZiNDcyODg5ZTQxOWFiMTVjMTA5NjIiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlRob3JuYnVyeSBDbHVzdGVyIDE8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2ExYTFjMThjMDViODQwOThhYzliYzM0NDlkYTA1MTEyLnNldENvbnRlbnQoaHRtbF8zODI0MmY1YzE0NmI0NzI4ODllNDE5YWIxNWMxMDk2Mik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl81ODE0MzY5MzZiY2I0MmE5ODcxOTAxZmFmMmQyNzliNy5iaW5kUG9wdXAocG9wdXBfYTFhMWMxOGMwNWI4NDA5OGFjOWJjMzQ0OWRhMDUxMTIpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZjg4N2JjYjJmZmY3NDNlNzg2MDZlMGNlNDgxOThkM2UgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuODQxNjE0MSwxNDUuMDE3NjIxNV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjNTRmNmNiIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzU0ZjZjYiIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8yMjE0MzIxMzQxMjQ0N2ZhYWJkMTY3NzljZGNjMTkxNiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9lZDAzYWY0YTEzY2U0ZWRkOGRjYjQzNjEyNGE3ZmY3ZiA9ICQoJzxkaXYgaWQ9Imh0bWxfZWQwM2FmNGExM2NlNGVkZDhkY2I0MzYxMjRhN2ZmN2YiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlRvb3JhayBDbHVzdGVyIDY8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzIyMTQzMjEzNDEyNDQ3ZmFhYmQxNjc3OWNkY2MxOTE2LnNldENvbnRlbnQoaHRtbF9lZDAzYWY0YTEzY2U0ZWRkOGRjYjQzNjEyNGE3ZmY3Zik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9mODg3YmNiMmZmZjc0M2U3ODYwNmUwY2U0ODE5OGQzZS5iaW5kUG9wdXAocG9wdXBfMjIxNDMyMTM0MTI0NDdmYWFiZDE2Nzc5Y2RjYzE5MTYpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZTI1MzE5OWEyZTI4NGQ0YmI5NGM2NjdjZTZlOTVlMDMgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuNzgwNzU1MywxNDQuOTM1NTAzXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiMwMGI1ZWIiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMDBiNWViIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzI2YzQ2YzIyNjg1YTQyYzBhMDc0NjZlMzIxNGNhMzQ3ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzdjNzU0NTFhM2YyODQwMjVhOTMxNjhkMDkxZTE2NGVjID0gJCgnPGRpdiBpZD0iaHRtbF83Yzc1NDUxYTNmMjg0MDI1YTkzMTY4ZDA5MWUxNjRlYyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+VHJhdmFuY29yZSBDbHVzdGVyIDQ8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzI2YzQ2YzIyNjg1YTQyYzBhMDc0NjZlMzIxNGNhMzQ3LnNldENvbnRlbnQoaHRtbF83Yzc1NDUxYTNmMjg0MDI1YTkzMTY4ZDA5MWUxNjRlYyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9lMjUzMTk5YTJlMjg0ZDRiYjk0YzY2N2NlNmU5NWUwMy5iaW5kUG9wdXAocG9wdXBfMjZjNDZjMjI2ODVhNDJjMGEwNzQ2NmUzMjE0Y2EzNDcpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfODdkNWVjMWNjOTk5NDgyYTljYzYzN2U5YjcwYjBiYzggPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzguMDE0Nzc4MywxNDUuMTMwNTE2Nl0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjNTRmNmNiIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzU0ZjZjYiIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMywKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9hODQxNWE5ZDBiNDQ0NjhjOWRlYmMyY2VmYzA2OWM0Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF80ZTYzMDQ5MmExYWQ0ZWYyYjg3MTZjNzcyYTQ1OWQwYyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF80Y2U0ZDE1NjdkYWE0OGVmYWY4OGQwZDQzMTMxN2ExOSA9ICQoJzxkaXYgaWQ9Imh0bWxfNGNlNGQxNTY3ZGFhNDhlZmFmODhkMGQ0MzEzMTdhMTkiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPldhdGVyd2F5cyBDbHVzdGVyIDY8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzRlNjMwNDkyYTFhZDRlZjJiODcxNmM3NzJhNDU5ZDBjLnNldENvbnRlbnQoaHRtbF80Y2U0ZDE1NjdkYWE0OGVmYWY4OGQwZDQzMTMxN2ExOSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl84N2Q1ZWMxY2M5OTk0ODJhOWNjNjM3ZTliNzBiMGJjOC5iaW5kUG9wdXAocG9wdXBfNGU2MzA0OTJhMWFkNGVmMmI4NzE2Yzc3MmE0NTlkMGMpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNzZlYWY0YjM3MjNhNDI0M2E0ZmZhMTUzOTdkNGRiYWEgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzcuODEwNDQ3NTUsMTQ0LjkyMDQyOTc5MzUwMTM0XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiM1NGY2Y2IiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjNTRmNmNiIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAzLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2E4NDE1YTlkMGI0NDQ2OGM5ZGViYzJjZWZjMDY5YzQzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2QwMjQ4OTU5M2I1MTQyOGVhM2QwYTI2ZjE2ZWJjNDU2ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzkxZDUyNWU3MjFlOTQzY2NiZmZjZTQ4MGNmMzU1NDk2ID0gJCgnPGRpdiBpZD0iaHRtbF85MWQ1MjVlNzIxZTk0M2NjYmZmY2U0ODBjZjM1NTQ5NiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+V2VzdCBNZWxib3VybmUgQ2x1c3RlciA2PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9kMDI0ODk1OTNiNTE0MjhlYTNkMGEyNmYxNmViYzQ1Ni5zZXRDb250ZW50KGh0bWxfOTFkNTI1ZTcyMWU5NDNjY2JmZmNlNDgwY2YzNTU0OTYpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfNzZlYWY0YjM3MjNhNDI0M2E0ZmZhMTUzOTdkNGRiYWEuYmluZFBvcHVwKHBvcHVwX2QwMjQ4OTU5M2I1MTQyOGVhM2QwYTI2ZjE2ZWJjNDU2KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzcyM2IzNjAzN2NkODQ0ZTliZDY2MzUyMmZhZDczNTU3ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3Ljg1NDUyMjcsMTQ0Ljk5MjE2NjVdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzgwMDBmZiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4MDAwZmYiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDMsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTg0MTVhOWQwYjQ0NDY4YzlkZWJjMmNlZmMwNjljNDMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMGFiYTE2NmZjNTVjNDFhNTljZjliMzg5YzBhZjhkZGIgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMGJlNzdiODYwYWVjNGIzNWIwNmFlYzQxNThhZGJhNzQgPSAkKCc8ZGl2IGlkPSJodG1sXzBiZTc3Yjg2MGFlYzRiMzViMDZhZWM0MTU4YWRiYTc0IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5XaW5kc29yIChWaWMuKSBDbHVzdGVyIDE8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzBhYmExNjZmYzU1YzQxYTU5Y2Y5YjM4OWMwYWY4ZGRiLnNldENvbnRlbnQoaHRtbF8wYmU3N2I4NjBhZWM0YjM1YjA2YWVjNDE1OGFkYmE3NCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl83MjNiMzYwMzdjZDg0NGU5YmQ2NjM1MjJmYWQ3MzU1Ny5iaW5kUG9wdXAocG9wdXBfMGFiYTE2NmZjNTVjNDFhNTljZjliMzg5YzBhZjhkZGIpOwoKICAgICAgICAgICAgCiAgICAgICAgCjwvc2NyaXB0Pg== onload="this.contentDocument.open();this.contentDocument.write(atob(this.getAttribute('data-html')));this.contentDocument.close();" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



Let's take a closer look at the data of each cluster.

The following piece of code will iterate the clusters and show:

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

    Cluster 0:



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


    Cluster 1:



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


    Cluster 2:



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


    Cluster 3:



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


    Cluster 4:



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


    Cluster 5:



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


    Cluster 6:



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


    Cluster 7:



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


    Cluster 8:



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


    Cluster 9:



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


    Cluster 10:



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


    Cluster 11:



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


    Cluster 12:



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


## Results and Discussion <a name="results"></a>

Clusters with suburbs showing a mean of 0.3 or more tell us that 30% or more of the families there have children; these are prospective candidates to work with. If the suburbs don't have enough venues or an irregular distribution among the types of venues, they are definitely candidates for the work of the local government.

It's important to note the limitations of the analysis in this notebook:
> We only get venues in a radius of 500 meters from the centre of the suburb and limit the search to only 30 venues. A substantial improvement would be not to limit the number of venues we can gather. Also, to request the venues in a radius big enough to go out of each suburb, then filter out those venues not located in the suburb by matching the address.

> The elbow method to select the number of clusters is not the most appropriate way to select groups of clusters in this study. There is no evident change in the curve of the graph that indicates an optimal number. This may be due to the high diversity of data among the suburbs in Melbourne. A more realistic approach would be for the local government to define how many groups they want to work with, depending on budget, resources and time to tackle the field studies and projects. It can be 13, as shown here or a different number. Having said that, it is also possible to play with KElbowVisualizer – silhouette and calinski_harabasz. For more information on those algorithms check this link https://www.scikit-yb.org/en/latest/api/cluster/elbow.html.

> Another point to be aware of is that this study was done only for the SA4 area 'Melbourne - Inner' which is the most populated of the SA4 areas conforming Melbourne. However, the local government will be interested in replicating this study to the rest of the suburbs of the city: Melbourne - North West, Melbourne - West, Melbourne - North East, Melbourne - Inner East, Melbourne - South East, Melbourne - Inner South and Melbourne - Outer East.

One approach is to select three different number of clusters suitable for the work and build three reports for the local government to choose the most feasible for the task.

## Conclusion <a name="conclusion"></a>

The study presented here is a good start point for the local government of Melbourne to solve the problem of suitable venues for families with children. With a full licence of Foursquare to request data about venues, plus gathering information from government archives and other online services like Google Places, the dataset of venues would be much more valuable for a better outcome.

It is critical to have discussions with stakeholders and contractors in charge of the project and logistics to define what number of clusters is the most appropriate.

After having a plan base on this study, further discussion with local councils can be of great feedback to make it more accurate and find the best possible distribution of the local resources in the area.

### Thank you for reviewing this work!

This notebook was created by [Israel Tiomno](https://www.linkedin.com/in/tiomno/) following the fantastic work of [Alex Aklson](https://www.linkedin.com/in/aklson/) and [Polong Lin](https://www.linkedin.com/in/polonglin/) in the notebook "Segmenting and Clustering Neighborhoods in New York City" as a guide.

<hr>

Copyright &copy; 2020 [Israel Tiomno](https://github.com/tiomno). This notebook and its source code are released under the terms of the MIT License.
