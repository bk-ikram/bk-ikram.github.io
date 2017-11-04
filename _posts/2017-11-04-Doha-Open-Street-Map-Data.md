---
layout: post
title:  "Wrangling and Auditing Doha Open Street Map Data"
subtitle: "Using Python and MongoDB to clean, process, audit, and explore Doha through its user-contributed OSM data."
date:   2017-11-04 22:30:13 +0200
author: "Ikram Boukhelif"
background: '/img/doha-qatar.jpg'
---
# Data Wrangling with Open Street Map of Doha, Qatar

### Map Area

For this project, I have decided to use data from the city of Doha in Qatar because that is where I have been living for a few years. The data was an OSM XML file downloaded from a metro extract on Mapzen. The data can found [here](https://mapzen.com/data/metro-extracts/metro/doha_qatar/).
The file used has a size of 60.191 Mb, which is manageable. The entire dataset was used for exploration and auditing, rather than a smaller sample.
 
**Note:** This data was downloaded on 10-September-2017 and may change in content and/or availability.

### Initial Data exploration

The file used has a size of 60.191 Mb, which is manageable. The entire dataset was used for exploration and auditing, rather than a smaller sample.

#### Tag types


```python
from get_data import get_tag_types
get_tag_types("doha_qatar.osm")
```

    There is a total of 830157 tags in the Doha osm file
    The breakdown of those tags is:-
    




    [('nd', 319832),
     ('node', 249108),
     ('tag', 209647),
     ('way', 46654),
     ('member', 3936),
     ('relation', 978),
     ('bounds', 1),
     ('osm', 1)]



Next, we look at the breakdown of all parent and child elements in the file:


```python
from get_data import iter_get_tag_types
iter_get_tag_types("doha_qatar.osm")
```

    There is a total of 830157 tags (parent and child) in the Doha osm file
    The breakdown of those tags is:
    




    [('nd', 319832),
     ('node', 249108),
     ('tag', 209647),
     ('way', 46654),
     ('member', 3936),
     ('relation', 978),
     ('bounds', 1),
     ('osm', 1)]



Then, we look at the most occuring key values used in the tags used to describe the *nodes* in the dataset.

#### Node types


```python
from get_data import elem_tag_types
Results = elem_tag_types("doha_qatar.osm","node"); Results[0:20]
```




    [('name', 5078),
     ('source', 4549),
     ('addr:street', 3545),
     ('shop', 3255),
     ('power', 1806),
     ('amenity', 1658),
     ('highway', 1037),
     ('barrier', 533),
     ('created_by', 499),
     ('name:en', 435),
     ('tourism', 287),
     ('addr:housenumber', 277),
     ('name:ar', 245),
     ('addr:city', 245),
     ('office', 242),
     ('cuisine', 206),
     ('addr:country', 141),
     ('traffic_signals', 140),
     ('man_made', 131),
     ('operator', 127)]



The complete list can be found in Appendix A.

### Problems with the map

#### Confusing tag key

After a quick inspection of those tags, it seems odd that there is a tag key named "created_by", when there is a dedicated attribute "user" attribute for each element, indicating its contributor. Let's look at the first 3 nodes containing this key in its tags:


```python
from get_data import inspect_by_tag,disp_complete_elem
results=inspect_by_tag("doha_qatar.osm","node","created_by")
disp_complete_elem(results,3)
```

    Inspecting Element 0:
    [('changeset', '551724'), ('uid', '4772'), ('timestamp', '2007-11-18T13:46:21Z'), ('lon', '51.5514899'), ('version', '1'), ('user', 'dmgroom_coastlines'), ('lat', '25.6274158'), ('id', '121831156')]
    Tags in this element:
    [('k', 'created_by'), ('v', 'JOSM')]
    Inspecting Element 1:
    [('changeset', '551724'), ('uid', '4772'), ('timestamp', '2007-11-18T13:46:21Z'), ('lon', '51.5520577'), ('version', '1'), ('user', 'dmgroom_coastlines'), ('lat', '25.6274136'), ('id', '121831157')]
    Tags in this element:
    [('k', 'created_by'), ('v', 'JOSM')]
    Inspecting Element 2:
    [('changeset', '551724'), ('uid', '4772'), ('timestamp', '2007-11-18T13:46:21Z'), ('lon', '51.5506383'), ('version', '1'), ('user', 'dmgroom_coastlines'), ('lat', '25.6274191'), ('id', '121831162')]
    Tags in this element:
    [('k', 'created_by'), ('v', 'JOSM')]
    

It seems like the same contributor "dmgroom_coastlines" might have created all of the nodes containing this key. If this was the case, then it would be expected to have a small number of elements containing this tag.


```python
len(results)
```




    661



This large number warrants a closer inspection. Let's look at all of the values for the key "created_by" and their corresponding contributors.


```python
from get_data import created_by_users
created_by_users(results)
```




    [('Skywave', 'Potlatch 0.8c'),
     ('OJW', 'Potlatch 0.10b'),
     ('MR_Claude', 'Potlatch 0.10b'),
     ('dmgroom', 'JOSM'),
     ('dmgroom_coastlines', 'JOSM')]



It looks like two users were responsible for these entries. The nature of their "created_by" values, and the number of entries they have contributed to suggests that they might be bots.

#### Multiple languages

Looking further into the list of tag keys, there are "name:en" and "name:ar", for the English and equivalent Arabic name of some venues, as is expected. Oher languages include French and Turkish.

#### Breast

There is a surprising key "breast" which was used 37 times. This odd find needs some looking into:


```python
results=inspect_by_tag("doha_qatar.osm","node","breast")
disp_complete_elem(results)
```

    Inspecting Element 0:
    [('changeset', '50431357'), ('uid', '6063818'), ('timestamp', '2017-07-20T12:01:27Z'), ('lon', '51.4884776'), ('version', '1'), ('user', 'Qatar University'), ('lat', '25.3735734'), ('id', '4982617771')]
    Tags in this element:
    [('k', 'breast'), ('v', '0')]
    [('k', 'height'), ('v', '10')]
    [('k', 'width'), ('v', '3')]
    [('k', 'window'), ('v', 'yes/fixed')]
    Inspecting Element 1:
    [('changeset', '50431357'), ('uid', '6063818'), ('timestamp', '2017-07-20T12:01:27Z'), ('lon', '51.4887331'), ('version', '1'), ('user', 'Qatar University'), ('lat', '25.3736765'), ('id', '4982617772')]
    Tags in this element:
    [('k', 'breast'), ('v', '0')]
    [('k', 'height'), ('v', '10')]
    [('k', 'width'), ('v', '2.85')]
    [('k', 'window'), ('v', 'yes/fixed')]
    Inspecting Element 2:
    [('changeset', '50431357'), ('uid', '6063818'), ('timestamp', '2017-07-20T12:01:28Z'), ('lon', '51.4887135'), ('version', '1'), ('user', 'Qatar University'), ('lat', '25.3736845'), ('id', '4982617773')]
    Tags in this element:
    [('k', 'breast'), ('v', '0')]
    [('k', 'height'), ('v', '10')]
    [('k', 'width'), ('v', '1.88')]
    [('k', 'window'), ('v', 'yes/fixed')]
    

All other instances look similar to this one, and "breast" always takes on a value of 0. I assume from this that they all describe windows at Qatar University, and that "breast" is just a misspelled measurement dimension.

#### False Streets

A quick look at the street names revealed that there are many entries with the tag "way",which didn't describe any streets at all. In order to get an estimate of the number of false streets, I have compiled a short list of common words which occured in them.


```python
from audit import problem_words
print problem_words
```

    ['school', 'district', 'compound', 'office', 'schule', 'mall', 'mart', 'center', 'parking', 'clinic', 'station', 'kindergarten', 'centre', 'complex', 'lounge', 'restaurant', 'grocery', 'supermarket', 'cafeteria', 'group', 'village', 'villa', 'villas', 'hotel', 'nursery']
    

This list was in no way complete, but it did reveal that there was a large number of places which was mistakenly categrorized as a "way" rather than a "node". Some "relations" such as roundabouts and interchanges are also entered as "nodes".


```python
from audit import get_street_names,check_problem_streets
names=get_street_names("doha_qatar.osm")
false_streets=check_problem_streets(names)
print "There are {} such entries. Here are some examples:".format(len(false_streets))
print false_streets[0:5]
```

    There are 724 such entries. Here are some examples:
    ['Merweb Hotel', 'The Center', 'Al Nahda School', 'Souq Parking Lot South', 'Nissan Petrol Station']
    

This number is worrying, and there needs to be a better and more comprehensive method of identifying false streets before they can be correctly reclassified as nodes.

One thing to note from my experience living here, is that most street names in Doha end in "Street" or "Road" or abbreviations thereof. I wanted to check if some of the other street types, "Avenue", "Boulevard", and "Way" are used here.


```python
from audit import check_alt_streets
alt_streets=check_alt_streets(names)
print "There are {} streets with these alternative street types. Here are some examples:".format(len(alt_streets))
print alt_streets[0:20]
```

    There are 68 streets with these alternative street types. Here are some examples:
    ['Marina Way 1', 'Marina 8 Way', 'Marina Way 2', 'Via Marsa Arabia Way', 'Via Marsa Arabia Way', 'Marina Way 30', 'Via Marsa Arabia Way', 'Via Marsa Arabia Way', 'Via Marsa Arabia Way', 'Via Marsa Arabia Way', 'Pearl Boulevard', 'Pearl Boulevard', 'Pearl Boulevard', 'Pearl Boulevard', 'Pearl Boulevard', 'Pearl Boulevard', 'Pearl Boulevard', 'Pearl Boulevard', 'Pearl Boulevard', 'Pearl Boulevard']
    

Looking at the entire list, it looks like these streets occur almost exclusively at The Pearl and QIPCO compound. This is interesting because both of these places are very popular among western expats.

#### City Address

Lets inspect the 'addr:city' and 'addr:coutry' fields in the nodes, to see what kind of values they contain.


```python
from audit import audit_nodes
print "The existing values for the field 'addr:city' are:"
print audit_nodes("doha_qatar.osm","addr:city")
print "And the existing values for the field 'addr:country' are:"
print audit_nodes("doha_qatar.osm","addr:country")
```

    The existing values for the field 'addr:city' are:
    [('Doha', 200), ('Ain Khalid Gate', 41), ('doha', 2), ('Doha City', 1), ('Qatar', 1)]
    And the existing values for the field 'addr:country' are:
    [('QA', 141)]
    

The values for "addr:country" are consistent, however, that is not the case with the "addr:city" field. Only a few nodes have this key present, and there are a few existing variations of its value. Also Ain Khalid Gate lies within Doha. These values should all be changed to "Doha".

### Export to MongoDb

In this process, the data is cleaned, reshaped, and saved as JSON documents for export to MongoDb

#### Data Cleaning:

* The "addr:city" fields are fixed as discussed earlier
* All "ways" which were identified as "false streets" are ignored. The list used to identify them is updated. This step is done several time until the street values are sufficiently clean.
* The node key "breast" is replaced with "breadth".

#### Reshaping:

* Only entities with tags "way" and "node" are exported.
* The tag of each entity is added to a field named "tag_type".
* Every second-level tag containing a colon (:) such as "addr:city" and "addr:country" will be reshaped into a subdocuments within a document; i.e. {"address":{["country","city"]}}
* latitude and longitude values are added as float numbers to an array named "position"
* Fields pertaining to the creation of the entity are added as subdocuments under a document "created".
* For ways, all reference nodes "nd ref" are added to an array named "node_ref".
* If an entitity lacks a "name" key in its tags, but has its name in different languages stored under keys "name:en","name:ar",etc, The english name "name:en" is used as its name, and the other name fields are stored in a list under "other_names".

#### Exporting:

The entities are saved as JSON documents in a file, and saved as a collection in a MongoDB database.


```python
#from audit import process_map
#process_map("doha_qatar.osm")
```

#### File Sizes

Downloaded XML file - 60 MB - doha_qatar.osm
Exported JSON file - 68 MB - doha_qatar.osm.json

#### Number of entries/documents

Downloaded XML file - 296,741 entries - doha_qatar.osm
Exported JSON file - 295,156 - doha_qatar.osm.json

## Data Exploration

The JSON file doha_qatar.osm.json is imported to a MongoDb collection called doha_qatar, under a data base called osm.


```python
#from process_data import import_json
#import_json(""doha_qatar.osm.json"")

```

Now that our database is created, a connection can be made using pymongo in order to query it.


```python
from pymongo import MongoClient
client = MongoClient()
doha = client.osm.doha_qatar
```

#### Number of Documents


```python
doha.count()
```




    295156



#### Number of Nodes and Ways


```python
nodes=doha.find({"tag_type":"node"}).count()
ways=doha.find({"tag_type":"way"}).count()
print "There are {} nodes and {} ways.".format(nodes,ways)
```

    There are 249108 nodes and 46048 ways.
    

#### Number of unique contributors


```python
results=doha.distinct("created.user")
len(results)
```




    696



#### Top contributors


```python
from pprint import pprint
pipeline=[{"$group":{"_id":"$created.user","count":{"$sum":1}}},
          {"$sort":{"count":-1}},
          {"$limit":10}]
results=doha.aggregate(pipeline)
pprint(list(results))
```

    [{u'_id': u'airlifter', u'count': 29529},
     {u'_id': u'Seandebasti', u'count': 24728},
     {u'_id': u'endo', u'count': 19842},
     {u'_id': u'Baconcrisp', u'count': 17578},
     {u'_id': u'gjall1973', u'count': 14422},
     {u'_id': u'rovingmedic', u'count': 13572},
     {u'_id': u'eXmajor', u'count': 13181},
     {u'_id': u'robgeb', u'count': 12464},
     {u'_id': u'garylancelot', u'count': 12105},
     {u'_id': u'lyx', u'count': 8354}]
    

#### Amenities

Let's look at the most occuring amenities.


```python
pipeline=[{"$match":{"amenity":{"$exists":1}}},
          {"$group":{"_id":"$amenity","count":{"$sum":1}}},
          {"$sort":{"count":-1}},
          {"$limit":5}]
pprint(list(doha.aggregate(pipeline)))
```

    [{u'_id': u'restaurant', u'count': 597},
     {u'_id': u'parking', u'count': 490},
     {u'_id': u'place_of_worship', u'count': 470},
     {u'_id': u'school', u'count': 197},
     {u'_id': u'cafe', u'count': 148}]
    

#### Restaurants


```python
pipeline=[{"$match":{"name":{"$exists":1}}},
          {"$match":{"amenity":"restaurant"}},
          {"$group":{"_id":"$name","count":{"$sum":1}}},
          {"$sort":{"count":-1}},
          {"$limit":10}]
pprint(list(doha.aggregate(pipeline)))
```

    [{u'_id': u'Pizza Hut', u'count': 10},
     {u'_id': u'Afghan Brothers Restaurant', u'count': 3},
     {u'_id': u'Hot Chicken', u'count': 3},
     {u'_id': u'Marhaba Istambul', u'count': 3},
     {u'_id': u'Khaiber Restaurant', u'count': 2},
     {u'_id': u'Sky Restaurant', u'count': 2},
     {u'_id': u'Choice Restaurant', u'count': 2},
     {u'_id': u'Jawahar Restaurant', u'count': 2},
     {u'_id': u'Own Pizza', u'count': 2},
     {u'_id': u'Al-Kababji', u'count': 2}]
    

I was surprised to see that Mcdonald's was not on this list, so I looked at the other amenities and found that other than "restaurant", there is also "fast_food" which might cover Mcdonald's. I decided to look at both of these, and cafes to see which "food" amenities had the most branches in Doha.


```python
pipeline=[{"$match":{"name":{"$exists":1}}},
          {"$match":{"amenity":{"$in":["restaurant","cafe","fast_food"]}}},
          {"$group":{"_id":"$name","count":{"$sum":1}}},
          {"$sort":{"count":-1}},
          {"$limit":10}]
pprint(list(doha.aggregate(pipeline)))
```

    [{u'_id': u'KFC', u'count': 16},
     {u'_id': u'Pizza Hut', u'count': 11},
     {u'_id': u"McDonald's", u'count': 10},
     {u'_id': u"Papa John's", u'count': 10},
     {u'_id': u'Subway', u'count': 9},
     {u'_id': u'Baskin Robbins', u'count': 9},
     {u'_id': u'Burger King', u'count': 7},
     {u'_id': u"Hardee's", u'count': 7},
     {u'_id': u'Caribou Coffee', u'count': 6},
     {u'_id': u'Costa', u'count': 5}]
    

These results are much more realistic, but not accurate. A cross-check with www.zomato.com reveals that there is closer to 23 McDonald's in Doha rather than 10, and 35 KFCs as opposed to 16. This suggests that this data is incomplete.

#### Malls

Because it seems like there is a handful of few malls being completed every year in Doha, I wanted to see how many there were.


```python
pipeline1=[{"$match":{"shop":"mall"}}]
pipeline2=[{"$match":{"shop":"mall"}},
          {"$project":{"_id":0,"name":1}},
          {"$limit":5}]
num_malls=len(list(doha.aggregate(pipeline1)))
en_malls=list(doha.aggregate(pipeline2))
print "There are {} malls in qatar. Here are some of them:-".format(num_malls)
pprint(en_malls)
```

    There are 41 malls in qatar. Here are some of them:-
    [{u'name': u'The Boulevard Mall'},
     {u'name': u'Centrepoint'},
     {u'name': {u'ar': u'\u0633\u0648\u0642 \u0648\u0627\u0642\u0641'}},
     {u'name': u'Alhazm Mall'},
     {u'name': u'The Center'}]
    

### Map Accuracy

In order to look at the accuracy of the geopositions of the node data, I plotted them onto a map of doha. The location of Doha's center is obtained from wikipedia.


```python
import folium
m=folium.Map(location=[25.286667, 51.533333],zoom_start=13)
```


```python
#Note: To display the map and bypass ipython's limit, start the notebook from anaconda using 
# "jupyter notebook --NotebookApp.iopub_data_rate_limit=1.0e10"
#The geographic coordinates of each node, if present, is extracted.
import numpy as np
results=doha.find({"pos":{"$exists":1},"addr.city":"Doha"})
positions = [doc['pos'] for doc in results]
from folium import plugins
plugins.HeatMap(positions).add_to(m);m
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><iframe src="data:text/html;charset=utf-8;base64,PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgPHNjcmlwdD5MX1BSRUZFUl9DQU5WQVMgPSBmYWxzZTsgTF9OT19UT1VDSCA9IGZhbHNlOyBMX0RJU0FCTEVfM0QgPSBmYWxzZTs8L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2FqYXguZ29vZ2xlYXBpcy5jb20vYWpheC9saWJzL2pxdWVyeS8xLjExLjEvanF1ZXJ5Lm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvanMvYm9vdHN0cmFwLm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9jZG5qcy5jbG91ZGZsYXJlLmNvbS9hamF4L2xpYnMvTGVhZmxldC5hd2Vzb21lLW1hcmtlcnMvMi4wLjIvbGVhZmxldC5hd2Vzb21lLW1hcmtlcnMuanMiPjwvc2NyaXB0PgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmNzcyIgLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvY3NzL2Jvb3RzdHJhcC5taW4uY3NzIiAvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiIC8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vZm9udC1hd2Vzb21lLzQuNi4zL2Nzcy9mb250LWF3ZXNvbWUubWluLmNzcyIgLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9jZG5qcy5jbG91ZGZsYXJlLmNvbS9hamF4L2xpYnMvTGVhZmxldC5hd2Vzb21lLW1hcmtlcnMvMi4wLjIvbGVhZmxldC5hd2Vzb21lLW1hcmtlcnMuY3NzIiAvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL3Jhd2dpdC5jb20vcHl0aG9uLXZpc3VhbGl6YXRpb24vZm9saXVtL21hc3Rlci9mb2xpdW0vdGVtcGxhdGVzL2xlYWZsZXQuYXdlc29tZS5yb3RhdGUuY3NzIiAvPgogICAgPHN0eWxlPmh0bWwsIGJvZHkge3dpZHRoOiAxMDAlO2hlaWdodDogMTAwJTttYXJnaW46IDA7cGFkZGluZzogMDt9PC9zdHlsZT4KICAgIDxzdHlsZT4jbWFwIHtwb3NpdGlvbjphYnNvbHV0ZTt0b3A6MDtib3R0b206MDtyaWdodDowO2xlZnQ6MDt9PC9zdHlsZT4KICAgIAogICAgICAgICAgICA8c3R5bGU+ICNtYXBfODg1ZTM3NDlmYmFkNGQzNmE5NmJkMTM1ODg2ZjViYjEgewogICAgICAgICAgICAgICAgcG9zaXRpb24gOiByZWxhdGl2ZTsKICAgICAgICAgICAgICAgIHdpZHRoIDogMTAwLjAlOwogICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICBsZWZ0OiAwLjAlOwogICAgICAgICAgICAgICAgdG9wOiAwLjAlOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICA8L3N0eWxlPgogICAgICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vbGVhZmxldC5naXRodWIuaW8vTGVhZmxldC5oZWF0L2Rpc3QvbGVhZmxldC1oZWF0LmpzIj48L3NjcmlwdD4KPC9oZWFkPgo8Ym9keT4gICAgCiAgICAKICAgICAgICAgICAgPGRpdiBjbGFzcz0iZm9saXVtLW1hcCIgaWQ9Im1hcF84ODVlMzc0OWZiYWQ0ZDM2YTk2YmQxMzU4ODZmNWJiMSIgPjwvZGl2PgogICAgICAgIAo8L2JvZHk+CjxzY3JpcHQ+ICAgIAogICAgCgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBib3VuZHMgPSBudWxsOwogICAgICAgICAgICAKCiAgICAgICAgICAgIHZhciBtYXBfODg1ZTM3NDlmYmFkNGQzNmE5NmJkMTM1ODg2ZjViYjEgPSBMLm1hcCgKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICdtYXBfODg1ZTM3NDlmYmFkNGQzNmE5NmJkMTM1ODg2ZjViYjEnLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAge2NlbnRlcjogWzI1LjI4NjY2Nyw1MS41MzMzMzNdLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgem9vbTogMTMsCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBtYXhCb3VuZHM6IGJvdW5kcywKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIGxheWVyczogW10sCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB3b3JsZENvcHlKdW1wOiBmYWxzZSwKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIGNyczogTC5DUlMuRVBTRzM4NTcKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgfSk7CiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciB0aWxlX2xheWVyXzdjMzhkNTQ1MjhjODQwMWQ5NGJjZjdmZWRlMWU0MDNiID0gTC50aWxlTGF5ZXIoCiAgICAgICAgICAgICAgICAnaHR0cHM6Ly97c30udGlsZS5vcGVuc3RyZWV0bWFwLm9yZy97en0ve3h9L3t5fS5wbmcnLAogICAgICAgICAgICAgICAgewogICJhdHRyaWJ1dGlvbiI6IG51bGwsIAogICJkZXRlY3RSZXRpbmEiOiBmYWxzZSwgCiAgIm1heFpvb20iOiAxOCwgCiAgIm1pblpvb20iOiAxLCAKICAibm9XcmFwIjogZmFsc2UsIAogICJzdWJkb21haW5zIjogImFiYyIKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfODg1ZTM3NDlmYmFkNGQzNmE5NmJkMTM1ODg2ZjViYjEpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBoZWF0X21hcF9hNjllNjEyNDFkYjM0ZjE2OWMxYzkzYjhjOTdmN2JiMiA9IEwuaGVhdExheWVyKAogICAgICAgICAgICAgICAgW1syNS4yODQ4NzY2LCA1MS41MDUyNTgzXSwgWzI1LjMxNjM3NSwgNTEuNTEzNDkyNl0sIFsyNS4yODQ0ODM1LCA1MS41MDUyNzk0XSwgWzI1LjI0NjY0NjQsIDUxLjQ3NjAxNl0sIFsyNS4zMTY2MTEzLCA1MS41MTM5NzUzXSwgWzI1LjI2Njc1NCwgNTEuNDg2NjU3XSwgWzI1LjQwMjM5MSwgNTEuNDMzMzI1M10sIFsyNS4zMTE5MjkxLCA1MS40OTg5MTg3XSwgWzI1LjI3NjQyMDMsIDUxLjUxMzU3MjJdLCBbMjUuMjc3MDU1OCwgNTEuNTEzNTU2OV0sIFsyNS4yNzM1MzQ5LCA1MS41MTUzODk4XSwgWzI1LjI3Njk3OCwgNTEuNTEyNDZdLCBbMjUuMjc2OTA5LCA1MS41MTM1Njg3XSwgWzI1LjI3NjYwNCwgNTEuNTEzNTMxNV0sIFsyNS4zNzI0MTg2LCA1MS41NTU4NzEzXSwgWzI1LjI3ODI1ODYsIDUxLjUyMDM2MzZdLCBbMjUuMzM2Nzc0NiwgNTEuNTA2NDk0Ml0sIFsyNS4zMzYzNzA3LCA1MS41MDY0MzMzXSwgWzI1LjMxNzUyMTgsIDUxLjQ5NzgxMDNdLCBbMjUuMzE1MTkzMiwgNTEuNDk1MDI5Ml0sIFsyNS4zNDAwNDY0LCA1MS41MTE4NDQzXSwgWzI1LjMwODY0MDIsIDUxLjQ5NzA3ODRdLCBbMjUuMzE1MzUxMywgNTEuNDk0OTQ2NF0sIFsyNS4zMTc0MTA3LCA1MS40OTgwNjgzXSwgWzI1LjI4MTQyLCA1MS41MDcxNDk5XSwgWzI1LjM3NDkwODEsIDUxLjU0MTQ2MjVdLCBbMjUuMjg4NTU4MSwgNTEuNTA1NzgyMV0sIFsyNS4yNzcyNDU0LCA1MS41MDg1NzA5XSwgWzI1LjM2OTkyNTMsIDUxLjU0MDU2OTFdLCBbMjUuMzczNTIxNywgNTEuNTQ1NDc4OF0sIFsyNS4zNDgzMzMsIDUxLjUyOTU3ODhdLCBbMjUuMjc3MTA3MywgNTEuNTEzMzI4NF0sIFsyNS4yNjk1MTcsIDUxLjU0Njc3NTZdLCBbMjUuMzAzNDg0NCwgNTEuNDk0Njg3NF1dLAogICAgICAgICAgICAgICAgewogICAgICAgICAgICAgICAgICAgIG1pbk9wYWNpdHk6IDAuNSwKICAgICAgICAgICAgICAgICAgICBtYXhab29tOiAxOCwKICAgICAgICAgICAgICAgICAgICBtYXg6IDEuMCwKICAgICAgICAgICAgICAgICAgICByYWRpdXM6IDI1LAogICAgICAgICAgICAgICAgICAgIGJsdXI6IDE1LAogICAgICAgICAgICAgICAgICAgIGdyYWRpZW50OiBudWxsCiAgICAgICAgICAgICAgICAgICAgfSkKICAgICAgICAgICAgICAgIC5hZGRUbyhtYXBfODg1ZTM3NDlmYmFkNGQzNmE5NmJkMTM1ODg2ZjViYjEpOwogICAgICAgIAo8L3NjcmlwdD4=" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



All of those points lie within the boundary of Doha.

#### Schools

Next, let's look at the distribution of schools. I decided to also include universities, colleges, and language schools.


```python
m=folium.Map(location=[25.286667, 51.533333],zoom_start=13)
results=doha.find({"pos":{"$exists":1},"amenity":{"$in":["school","college","university","language_school"]}})
positions = [doc['pos'] for doc in results]
plugins.HeatMap(positions).add_to(m);m
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><iframe src="data:text/html;charset=utf-8;base64,PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgPHNjcmlwdD5MX1BSRUZFUl9DQU5WQVMgPSBmYWxzZTsgTF9OT19UT1VDSCA9IGZhbHNlOyBMX0RJU0FCTEVfM0QgPSBmYWxzZTs8L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2FqYXguZ29vZ2xlYXBpcy5jb20vYWpheC9saWJzL2pxdWVyeS8xLjExLjEvanF1ZXJ5Lm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvanMvYm9vdHN0cmFwLm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9jZG5qcy5jbG91ZGZsYXJlLmNvbS9hamF4L2xpYnMvTGVhZmxldC5hd2Vzb21lLW1hcmtlcnMvMi4wLjIvbGVhZmxldC5hd2Vzb21lLW1hcmtlcnMuanMiPjwvc2NyaXB0PgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmNzcyIgLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvY3NzL2Jvb3RzdHJhcC5taW4uY3NzIiAvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiIC8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vZm9udC1hd2Vzb21lLzQuNi4zL2Nzcy9mb250LWF3ZXNvbWUubWluLmNzcyIgLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9jZG5qcy5jbG91ZGZsYXJlLmNvbS9hamF4L2xpYnMvTGVhZmxldC5hd2Vzb21lLW1hcmtlcnMvMi4wLjIvbGVhZmxldC5hd2Vzb21lLW1hcmtlcnMuY3NzIiAvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL3Jhd2dpdC5jb20vcHl0aG9uLXZpc3VhbGl6YXRpb24vZm9saXVtL21hc3Rlci9mb2xpdW0vdGVtcGxhdGVzL2xlYWZsZXQuYXdlc29tZS5yb3RhdGUuY3NzIiAvPgogICAgPHN0eWxlPmh0bWwsIGJvZHkge3dpZHRoOiAxMDAlO2hlaWdodDogMTAwJTttYXJnaW46IDA7cGFkZGluZzogMDt9PC9zdHlsZT4KICAgIDxzdHlsZT4jbWFwIHtwb3NpdGlvbjphYnNvbHV0ZTt0b3A6MDtib3R0b206MDtyaWdodDowO2xlZnQ6MDt9PC9zdHlsZT4KICAgIAogICAgICAgICAgICA8c3R5bGU+ICNtYXBfOTE0MWE5OTA0MmQxNGViOWI0ZTQ5MTBjNjQ0NTU3OTEgewogICAgICAgICAgICAgICAgcG9zaXRpb24gOiByZWxhdGl2ZTsKICAgICAgICAgICAgICAgIHdpZHRoIDogMTAwLjAlOwogICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICBsZWZ0OiAwLjAlOwogICAgICAgICAgICAgICAgdG9wOiAwLjAlOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICA8L3N0eWxlPgogICAgICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vbGVhZmxldC5naXRodWIuaW8vTGVhZmxldC5oZWF0L2Rpc3QvbGVhZmxldC1oZWF0LmpzIj48L3NjcmlwdD4KPC9oZWFkPgo8Ym9keT4gICAgCiAgICAKICAgICAgICAgICAgPGRpdiBjbGFzcz0iZm9saXVtLW1hcCIgaWQ9Im1hcF85MTQxYTk5MDQyZDE0ZWI5YjRlNDkxMGM2NDQ1NTc5MSIgPjwvZGl2PgogICAgICAgIAo8L2JvZHk+CjxzY3JpcHQ+ICAgIAogICAgCgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBib3VuZHMgPSBudWxsOwogICAgICAgICAgICAKCiAgICAgICAgICAgIHZhciBtYXBfOTE0MWE5OTA0MmQxNGViOWI0ZTQ5MTBjNjQ0NTU3OTEgPSBMLm1hcCgKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICdtYXBfOTE0MWE5OTA0MmQxNGViOWI0ZTQ5MTBjNjQ0NTU3OTEnLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAge2NlbnRlcjogWzI1LjI4NjY2Nyw1MS41MzMzMzNdLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgem9vbTogMTMsCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBtYXhCb3VuZHM6IGJvdW5kcywKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIGxheWVyczogW10sCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB3b3JsZENvcHlKdW1wOiBmYWxzZSwKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIGNyczogTC5DUlMuRVBTRzM4NTcKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgfSk7CiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciB0aWxlX2xheWVyX2YzYjk2Mzk5OTU3MDQyODdhY2M1MGY5NGY2MzNjNTVlID0gTC50aWxlTGF5ZXIoCiAgICAgICAgICAgICAgICAnaHR0cHM6Ly97c30udGlsZS5vcGVuc3RyZWV0bWFwLm9yZy97en0ve3h9L3t5fS5wbmcnLAogICAgICAgICAgICAgICAgewogICJhdHRyaWJ1dGlvbiI6IG51bGwsIAogICJkZXRlY3RSZXRpbmEiOiBmYWxzZSwgCiAgIm1heFpvb20iOiAxOCwgCiAgIm1pblpvb20iOiAxLCAKICAibm9XcmFwIjogZmFsc2UsIAogICJzdWJkb21haW5zIjogImFiYyIKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfOTE0MWE5OTA0MmQxNGViOWI0ZTQ5MTBjNjQ0NTU3OTEpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBoZWF0X21hcF9jNDM2ZDg2OTYxYjk0NmI4YWVkOTdjMGY2YWY3OWYxMSA9IEwuaGVhdExheWVyKAogICAgICAgICAgICAgICAgW1syNS4yMjk5MDY3LCA1MS41MDExOTM5XSwgWzI1LjMyMzk4NDcsIDUxLjQzNjE2NjddLCBbMjUuMjUyNDM2LCA1MS40NDk4NTcxXSwgWzI1LjI1OTUyNjUsIDUxLjQ4MzA4MjVdLCBbMjUuMjYzMzU5MywgNTEuNDgxNTI4N10sIFsyNS4zNTkyODIzLCA1MS40Nzk2NDA3XSwgWzI1LjI0MTk4MzYsIDUxLjQ2NjQzNTldLCBbMjUuMzE0MDkyLCA1MS40MzkwNTQ0XSwgWzI1LjMxNDc3MTIsIDUxLjQzMzA2NDZdLCBbMjUuMzE0MzUxOSwgNTEuNDQxMjQ5MV0sIFsyNS4yMTM3ODgyLCA1MS41NzgwNzhdLCBbMjUuMzM2MjAxLCA1MS41MTAxNzE1XSwgWzI1LjM3MTIwMTUsIDUxLjQ5OTEzMjRdLCBbMjUuMDA2MDMwOSwgNTEuNTM4ODMzXSwgWzI1LjMyMjcsIDUxLjQ0Mzk4MjhdLCBbMjUuNDYxNTA1OCwgNTEuNTQwNDYxNl0sIFsyNS4zMjA4NjU1LCA1MS40MjQzMjE4XSwgWzI1LjMwMjU1ODcsIDUxLjQ4NTg1MDldLCBbMjUuMjY1NDE2OCwgNTEuNDgxNzUwNl0sIFsyNS4yMzY3ODQ4LCA1MS41NTA0ODgxXSwgWzI1LjIzMDcwNTYsIDUxLjQ5MTU3NzVdLCBbMjUuMzI4OTE1LCA1MS4zODYxNTE2XSwgWzI1LjIzMTE0MjQsIDUxLjQ5NDU1NjVdLCBbMjUuMjMyMjg4NiwgNTEuNDk0Mzc2Ml0sIFsyNS4zMjg4ODU1LCA1MS4zODcyNTMxXSwgWzI1LjMwMjU0ODMsIDUxLjQ4NTgzNjZdLCBbMjUuMjc5NzUzLCA1MS40OTE0NjVdLCBbMjUuMTk1Mzg3NywgNTEuNDc4NjU2M10sIFsyNS40NDkwODc4LCA1MS40Mzc4OTI4XSwgWzI1LjMwMTc4NjIsIDUxLjQ5MzU1NjhdLCBbMjUuMzg4MDk0OSwgNTEuNDgzOTAyNl0sIFsyNS4zODc5NywgNTEuNDgzOTA1XSwgWzI1LjIwNDYxNTksIDUxLjQ0NzU3MDhdLCBbMjUuMzQ5ODA1NCwgNTEuNDQ1ODM1M10sIFsyNS4yNjMyMDY3LCA1MS41Mjc5NDc2XSwgWzI1LjIxMTUxNzQsIDUxLjUyMjQ5MDZdLCBbMjUuMjY0NDM3NSwgNTEuNDgzNTc4M10sIFsyNS4xOTA0ODQzLCA1MS41MDM2OTY4XSwgWzI1LjQwNDk1MSwgNTEuNDU5NDE5M10sIFsyNS4yMjcwNjc3LCA1MS41NDQ4MzldXSwKICAgICAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgICAgICBtaW5PcGFjaXR5OiAwLjUsCiAgICAgICAgICAgICAgICAgICAgbWF4Wm9vbTogMTgsCiAgICAgICAgICAgICAgICAgICAgbWF4OiAxLjAsCiAgICAgICAgICAgICAgICAgICAgcmFkaXVzOiAyNSwKICAgICAgICAgICAgICAgICAgICBibHVyOiAxNSwKICAgICAgICAgICAgICAgICAgICBncmFkaWVudDogbnVsbAogICAgICAgICAgICAgICAgICAgIH0pCiAgICAgICAgICAgICAgICAuYWRkVG8obWFwXzkxNDFhOTkwNDJkMTRlYjliNGU0OTEwYzY0NDU1NzkxKTsKICAgICAgICAKPC9zY3JpcHQ+" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>




```python
results=doha.find({"pos":{"$exists":1},"amenity":{"$in":["school","university","college"]},"name":{"$exists":1}})
names,positions=[],[]
for doc in results:
    if type(doc["name"]) != dict:
        names.append(doc["name"])
    else:
        languages=["en","ar","fr"]
        for lan in languages:
            if (lan in doc["name"]):
                names.append(doc["name"][lan])
    positions.append(doc["pos"])
```

As I have expected, there are a lot of schools in Old Al Rayyan. This is because they all belong to a large organisation, Qatar Foundation. However, there seems to be a school on water off the coast from Lusail. I don't know any such schools, and will assume it is a mistake.
I wanted to get the names of the schools on the map, so I decided to plot them as markers on the map. Unfortunately, I could only plot 32 of them on the map. Otherwise, the map would not show up at all, this might be a limitation in jupyter or in the settings I am using.


```python
from folium import FeatureGroup, Marker
m=folium.Map(location=[25.286667, 51.533333],zoom_start=12)
feature_group = folium.FeatureGroup("Locations")
c=0
for pos, name in zip(positions, names):
    if c>32:
        break
    feature_group.add_child(folium.Marker(location=pos,popup=name))
    c+=1
m.add_child(feature_group)
print "Click on a marker for the school's name"
m
```

    Click on a marker for the school's name
    




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><iframe src="data:text/html;charset=utf-8;base64,PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgPHNjcmlwdD5MX1BSRUZFUl9DQU5WQVMgPSBmYWxzZTsgTF9OT19UT1VDSCA9IGZhbHNlOyBMX0RJU0FCTEVfM0QgPSBmYWxzZTs8L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2FqYXguZ29vZ2xlYXBpcy5jb20vYWpheC9saWJzL2pxdWVyeS8xLjExLjEvanF1ZXJ5Lm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvanMvYm9vdHN0cmFwLm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9jZG5qcy5jbG91ZGZsYXJlLmNvbS9hamF4L2xpYnMvTGVhZmxldC5hd2Vzb21lLW1hcmtlcnMvMi4wLjIvbGVhZmxldC5hd2Vzb21lLW1hcmtlcnMuanMiPjwvc2NyaXB0PgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmNzcyIgLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvY3NzL2Jvb3RzdHJhcC5taW4uY3NzIiAvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiIC8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vZm9udC1hd2Vzb21lLzQuNi4zL2Nzcy9mb250LWF3ZXNvbWUubWluLmNzcyIgLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9jZG5qcy5jbG91ZGZsYXJlLmNvbS9hamF4L2xpYnMvTGVhZmxldC5hd2Vzb21lLW1hcmtlcnMvMi4wLjIvbGVhZmxldC5hd2Vzb21lLW1hcmtlcnMuY3NzIiAvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL3Jhd2dpdC5jb20vcHl0aG9uLXZpc3VhbGl6YXRpb24vZm9saXVtL21hc3Rlci9mb2xpdW0vdGVtcGxhdGVzL2xlYWZsZXQuYXdlc29tZS5yb3RhdGUuY3NzIiAvPgogICAgPHN0eWxlPmh0bWwsIGJvZHkge3dpZHRoOiAxMDAlO2hlaWdodDogMTAwJTttYXJnaW46IDA7cGFkZGluZzogMDt9PC9zdHlsZT4KICAgIDxzdHlsZT4jbWFwIHtwb3NpdGlvbjphYnNvbHV0ZTt0b3A6MDtib3R0b206MDtyaWdodDowO2xlZnQ6MDt9PC9zdHlsZT4KICAgIAogICAgICAgICAgICA8c3R5bGU+ICNtYXBfNDhhY2M5YTY5ZjQ3NDQxYmIwNWI0NDgxMzI3YTM3MjAgewogICAgICAgICAgICAgICAgcG9zaXRpb24gOiByZWxhdGl2ZTsKICAgICAgICAgICAgICAgIHdpZHRoIDogMTAwLjAlOwogICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICBsZWZ0OiAwLjAlOwogICAgICAgICAgICAgICAgdG9wOiAwLjAlOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICA8L3N0eWxlPgogICAgICAgIAo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwXzQ4YWNjOWE2OWY0NzQ0MWJiMDViNDQ4MTMyN2EzNzIwIiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGJvdW5kcyA9IG51bGw7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgdmFyIG1hcF80OGFjYzlhNjlmNDc0NDFiYjA1YjQ0ODEzMjdhMzcyMCA9IEwubWFwKAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgJ21hcF80OGFjYzlhNjlmNDc0NDFiYjA1YjQ0ODEzMjdhMzcyMCcsCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB7Y2VudGVyOiBbMjUuMjg2NjY3LDUxLjUzMzMzM10sCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB6b29tOiAxMiwKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIG1heEJvdW5kczogYm91bmRzLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgbGF5ZXJzOiBbXSwKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIHdvcmxkQ29weUp1bXA6IGZhbHNlLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgY3JzOiBMLkNSUy5FUFNHMzg1NwogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB9KTsKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHRpbGVfbGF5ZXJfM2E4M2EzYmYzZjZmNDg5Y2IzNjBiMDc4NzZlMjNmMWIgPSBMLnRpbGVMYXllcigKICAgICAgICAgICAgICAgICdodHRwczovL3tzfS50aWxlLm9wZW5zdHJlZXRtYXAub3JnL3t6fS97eH0ve3l9LnBuZycsCiAgICAgICAgICAgICAgICB7CiAgImF0dHJpYnV0aW9uIjogbnVsbCwgCiAgImRldGVjdFJldGluYSI6IGZhbHNlLCAKICAibWF4Wm9vbSI6IDE4LCAKICAibWluWm9vbSI6IDEsIAogICJub1dyYXAiOiBmYWxzZSwgCiAgInN1YmRvbWFpbnMiOiAiYWJjIgp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF80OGFjYzlhNjlmNDc0NDFiYjA1YjQ0ODEzMjdhMzcyMCk7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGZlYXR1cmVfZ3JvdXBfYWU2MzVlZDczNTkzNGZmYWFjYmQ2YWQyMGRhN2MwMWYgPSBMLmZlYXR1cmVHcm91cCgKICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzQ4YWNjOWE2OWY0NzQ0MWJiMDViNDQ4MTMyN2EzNzIwKTsKICAgICAgICAKICAgIAoKICAgICAgICAgICAgdmFyIG1hcmtlcl9mN2IyYWNmOTgwZGI0ODIwODJhOTVhZjBmODEzZTFmNCA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzI1LjIyOTkwNjcsNTEuNTAxMTkzOV0sCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgaWNvbjogbmV3IEwuSWNvbi5EZWZhdWx0KCkKICAgICAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICAgICApCiAgICAgICAgICAgICAgICAuYWRkVG8oZmVhdHVyZV9ncm91cF9hZTYzNWVkNzM1OTM0ZmZhYWNiZDZhZDIwZGE3YzAxZik7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF83ZTdhZTU3YmE2YTU0NGMwYTkyNTliNDA0Nzc1NzNhZiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9jMzVlZjJlY2NiOWU0M2M0OTA5ODNhNzE5ZmU2NTM2NSA9ICQoJzxkaXYgaWQ9Imh0bWxfYzM1ZWYyZWNjYjllNDNjNDkwOTgzYTcxOWZlNjUzNjUiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlBhcmsgSG91c2UgRW5nbGlzaCBTY2hvb2w8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzdlN2FlNTdiYTZhNTQ0YzBhOTI1OWI0MDQ3NzU3M2FmLnNldENvbnRlbnQoaHRtbF9jMzVlZjJlY2NiOWU0M2M0OTA5ODNhNzE5ZmU2NTM2NSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgbWFya2VyX2Y3YjJhY2Y5ODBkYjQ4MjA4MmE5NWFmMGY4MTNlMWY0LmJpbmRQb3B1cChwb3B1cF83ZTdhZTU3YmE2YTU0NGMwYTkyNTliNDA0Nzc1NzNhZik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAoKICAgICAgICAgICAgdmFyIG1hcmtlcl85ZmQxYjdlZTUwYzY0OTUwOWRkY2ZiOTk1MDA4YTdmMiA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzI1LjMyMzk4NDcsNTEuNDM2MTY2N10sCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgaWNvbjogbmV3IEwuSWNvbi5EZWZhdWx0KCkKICAgICAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICAgICApCiAgICAgICAgICAgICAgICAuYWRkVG8oZmVhdHVyZV9ncm91cF9hZTYzNWVkNzM1OTM0ZmZhYWNiZDZhZDIwZGE3YzAxZik7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8xZTMyNThhMzk3NTM0Yjc2OWM5NDNhYWVlYTM4NDk3MCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8wNmU0NWY1NGQ2ODI0ODFmYTUxNTY0NDJlMWE4ZGVjZSA9ICQoJzxkaXYgaWQ9Imh0bWxfMDZlNDVmNTRkNjgyNDgxZmE1MTU2NDQyZTFhOGRlY2UiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlFhdGFyIFVuaXZlcnNpdHkgLSBXaXJlbGVzcyBJbm5vdmF0aW9ucyBDZW50ZXIgKFFVV0lDKTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMWUzMjU4YTM5NzUzNGI3NjljOTQzYWFlZWEzODQ5NzAuc2V0Q29udGVudChodG1sXzA2ZTQ1ZjU0ZDY4MjQ4MWZhNTE1NjQ0MmUxYThkZWNlKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBtYXJrZXJfOWZkMWI3ZWU1MGM2NDk1MDlkZGNmYjk5NTAwOGE3ZjIuYmluZFBvcHVwKHBvcHVwXzFlMzI1OGEzOTc1MzRiNzY5Yzk0M2FhZWVhMzg0OTcwKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCgogICAgICAgICAgICB2YXIgbWFya2VyX2ExOGZhNGFmNTA4YjQ1NjE4YjliNmQyYTVhYTI0NTk2ID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMjUuMjU5NTI2NSw1MS40ODMwODI1XSwKICAgICAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgICAgICBpY29uOiBuZXcgTC5JY29uLkRlZmF1bHQoKQogICAgICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgICAgICkKICAgICAgICAgICAgICAgIC5hZGRUbyhmZWF0dXJlX2dyb3VwX2FlNjM1ZWQ3MzU5MzRmZmFhY2JkNmFkMjBkYTdjMDFmKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzRjMWIxZGI5NTBkYTQ0ZWRhNWM3YTQzYjE1MTc5NzU0ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2Y1ZTE0MTA2Yjg0ZDQ5Zjc5NDM1NDY2ZDRiZTMyN2Y4ID0gJCgnPGRpdiBpZD0iaHRtbF9mNWUxNDEwNmI4NGQ0OWY3OTQzNTQ2NmQ0YmUzMjdmOCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RG9oYSBDb2xsZWdlPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF80YzFiMWRiOTUwZGE0NGVkYTVjN2E0M2IxNTE3OTc1NC5zZXRDb250ZW50KGh0bWxfZjVlMTQxMDZiODRkNDlmNzk0MzU0NjZkNGJlMzI3ZjgpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl9hMThmYTRhZjUwOGI0NTYxOGI5YjZkMmE1YWEyNDU5Ni5iaW5kUG9wdXAocG9wdXBfNGMxYjFkYjk1MGRhNDRlZGE1YzdhNDNiMTUxNzk3NTQpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKCiAgICAgICAgICAgIHZhciBtYXJrZXJfN2NjYjA3ZDU2MmM1NDc1YTkzN2ZkODZkMzU4Zjg1OTUgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFsyNS4yNjMzNTkzLDUxLjQ4MTUyODddLAogICAgICAgICAgICAgICAgewogICAgICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICAgICAgfQogICAgICAgICAgICAgICAgKQogICAgICAgICAgICAgICAgLmFkZFRvKGZlYXR1cmVfZ3JvdXBfYWU2MzVlZDczNTkzNGZmYWFjYmQ2YWQyMGRhN2MwMWYpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYTBkOTgwYWU3NmM5NGVhMzkxNmExOWIyNGQxY2EyM2IgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMzA5MWQyYWVlMzgzNGZkZGEyMGFhOTcwMmRhMjNmZjggPSAkKCc8ZGl2IGlkPSJodG1sXzMwOTFkMmFlZTM4MzRmZGRhMjBhYTk3MDJkYTIzZmY4IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5BbWVyaWNhbiBTY2hvb2wgb2YgRG9oYSAoQVNEKTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYTBkOTgwYWU3NmM5NGVhMzkxNmExOWIyNGQxY2EyM2Iuc2V0Q29udGVudChodG1sXzMwOTFkMmFlZTM4MzRmZGRhMjBhYTk3MDJkYTIzZmY4KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBtYXJrZXJfN2NjYjA3ZDU2MmM1NDc1YTkzN2ZkODZkMzU4Zjg1OTUuYmluZFBvcHVwKHBvcHVwX2EwZDk4MGFlNzZjOTRlYTM5MTZhMTliMjRkMWNhMjNiKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCgogICAgICAgICAgICB2YXIgbWFya2VyX2VlODNiYmE3ZDg4ZjQ0MmZiYmRjNzlmZjQ1NjQ5ZDIzID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMjUuMzU5MjgyMyw1MS40Nzk2NDA3XSwKICAgICAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgICAgICBpY29uOiBuZXcgTC5JY29uLkRlZmF1bHQoKQogICAgICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgICAgICkKICAgICAgICAgICAgICAgIC5hZGRUbyhmZWF0dXJlX2dyb3VwX2FlNjM1ZWQ3MzU5MzRmZmFhY2JkNmFkMjBkYTdjMDFmKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2RmYjk5M2U1ZmQ2NTRhZDE4OTk3NzY3Y2EwYzM5ZjAzID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzE1YzZiZjk0YTA2MDQ1ZmRiNjVmMzc0M2NlMTFlNTFiID0gJCgnPGRpdiBpZD0iaHRtbF8xNWM2YmY5NGEwNjA0NWZkYjY1ZjM3NDNjZTExZTUxYiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Q29sbGVnZSBvZiB0aGUgTm9ydGggQXRsYW50aWMgUWF0YXIgKENOQVEpPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9kZmI5OTNlNWZkNjU0YWQxODk5Nzc2N2NhMGMzOWYwMy5zZXRDb250ZW50KGh0bWxfMTVjNmJmOTRhMDYwNDVmZGI2NWYzNzQzY2UxMWU1MWIpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl9lZTgzYmJhN2Q4OGY0NDJmYmJkYzc5ZmY0NTY0OWQyMy5iaW5kUG9wdXAocG9wdXBfZGZiOTkzZTVmZDY1NGFkMTg5OTc3NjdjYTBjMzlmMDMpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKCiAgICAgICAgICAgIHZhciBtYXJrZXJfNTU5OTVhNzJkYTIxNDU4MGJlMWIzMTk5ZjUwZGU4NGQgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFsyNS4yNDE5ODM2LDUxLjQ2NjQzNTldLAogICAgICAgICAgICAgICAgewogICAgICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICAgICAgfQogICAgICAgICAgICAgICAgKQogICAgICAgICAgICAgICAgLmFkZFRvKGZlYXR1cmVfZ3JvdXBfYWU2MzVlZDczNTkzNGZmYWFjYmQ2YWQyMGRhN2MwMWYpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNDc0YjE1NDljNjk2NDVmZTkyZTg5N2Y4YTA0OGVjZmQgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfOTI5NDgyYWRjMzNkNGRlMjhhMDU4MzI3N2NlNTdlZTYgPSAkKCc8ZGl2IGlkPSJodG1sXzkyOTQ4MmFkYzMzZDRkZTI4YTA1ODMyNzdjZTU3ZWU2IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Eb2hhIEJyaXRpc2ggU2Nob29sPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF80NzRiMTU0OWM2OTY0NWZlOTJlODk3ZjhhMDQ4ZWNmZC5zZXRDb250ZW50KGh0bWxfOTI5NDgyYWRjMzNkNGRlMjhhMDU4MzI3N2NlNTdlZTYpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl81NTk5NWE3MmRhMjE0NTgwYmUxYjMxOTlmNTBkZTg0ZC5iaW5kUG9wdXAocG9wdXBfNDc0YjE1NDljNjk2NDVmZTkyZTg5N2Y4YTA0OGVjZmQpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKCiAgICAgICAgICAgIHZhciBtYXJrZXJfMWIwN2RlODFmMjNkNDZhNGI4ODIwYzM4NmRmMTdhOTYgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFsyNS4zMTQwOTIsNTEuNDM5MDU0NF0sCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgaWNvbjogbmV3IEwuSWNvbi5EZWZhdWx0KCkKICAgICAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICAgICApCiAgICAgICAgICAgICAgICAuYWRkVG8oZmVhdHVyZV9ncm91cF9hZTYzNWVkNzM1OTM0ZmZhYWNiZDZhZDIwZGE3YzAxZik7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9jYzExOGY5NDg4M2E0ZTU0OGE1NjE0YTBjYzBjMGYxNiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8yZDI4NDlmMzVjMDA0NTY2YTc5NTBjN2RkNjZkN2QxYSA9ICQoJzxkaXYgaWQ9Imh0bWxfMmQyODQ5ZjM1YzAwNDU2NmE3OTUwYzdkZDY2ZDdkMWEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlRleGFzIEEmTSBVbml2ZXJzaXR5IC0gUWF0YXI8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2NjMTE4Zjk0ODgzYTRlNTQ4YTU2MTRhMGNjMGMwZjE2LnNldENvbnRlbnQoaHRtbF8yZDI4NDlmMzVjMDA0NTY2YTc5NTBjN2RkNjZkN2QxYSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgbWFya2VyXzFiMDdkZTgxZjIzZDQ2YTRiODgyMGMzODZkZjE3YTk2LmJpbmRQb3B1cChwb3B1cF9jYzExOGY5NDg4M2E0ZTU0OGE1NjE0YTBjYzBjMGYxNik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAoKICAgICAgICAgICAgdmFyIG1hcmtlcl82YjUzY2I0MWQyOGE0YTE3YmVjNzJkMzhhNWE1OWIyNyA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzI1LjMxNDc3MTIsNTEuNDMzMDY0Nl0sCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgaWNvbjogbmV3IEwuSWNvbi5EZWZhdWx0KCkKICAgICAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICAgICApCiAgICAgICAgICAgICAgICAuYWRkVG8oZmVhdHVyZV9ncm91cF9hZTYzNWVkNzM1OTM0ZmZhYWNiZDZhZDIwZGE3YzAxZik7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF81YmNmNDVmYmNhMWE0ZDRjODQwZmY4Y2Y3YWQyZjdhZiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9hOTdlZWJmYjQ3NDY0NmEwODc2NTFiODhjY2Y5NDQyYSA9ICQoJzxkaXYgaWQ9Imh0bWxfYTk3ZWViZmI0NzQ2NDZhMDg3NjUxYjg4Y2NmOTQ0MmEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlZpcmdpbmlhIENvbW1vbndlYWx0aCBVbml2ZXJzaXR5IC0gUWF0YXI8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzViY2Y0NWZiY2ExYTRkNGM4NDBmZjhjZjdhZDJmN2FmLnNldENvbnRlbnQoaHRtbF9hOTdlZWJmYjQ3NDY0NmEwODc2NTFiODhjY2Y5NDQyYSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgbWFya2VyXzZiNTNjYjQxZDI4YTRhMTdiZWM3MmQzOGE1YTU5YjI3LmJpbmRQb3B1cChwb3B1cF81YmNmNDVmYmNhMWE0ZDRjODQwZmY4Y2Y3YWQyZjdhZik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAoKICAgICAgICAgICAgdmFyIG1hcmtlcl85MDhkNjhhZjJjOTM0MjI1YWFjMTk1ODBjMmU4ZGU0YSA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzI1LjMxNDM1MTksNTEuNDQxMjQ5MV0sCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgaWNvbjogbmV3IEwuSWNvbi5EZWZhdWx0KCkKICAgICAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICAgICApCiAgICAgICAgICAgICAgICAuYWRkVG8oZmVhdHVyZV9ncm91cF9hZTYzNWVkNzM1OTM0ZmZhYWNiZDZhZDIwZGE3YzAxZik7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF81OWIwNmQ1YmIyNTU0ZjkwYThlMTFlMmI0OTc2OTQ3NyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF84YWYyMTNiZDFmZWQ0YWM5YjhlYzc2NWU4YjE4YjNlZiA9ICQoJzxkaXYgaWQ9Imh0bWxfOGFmMjEzYmQxZmVkNGFjOWI4ZWM3NjVlOGIxOGIzZWYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlN0dWRlbnQgUmVjcmVhdGlvbiBDZW50ZXI8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzU5YjA2ZDViYjI1NTRmOTBhOGUxMWUyYjQ5NzY5NDc3LnNldENvbnRlbnQoaHRtbF84YWYyMTNiZDFmZWQ0YWM5YjhlYzc2NWU4YjE4YjNlZik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgbWFya2VyXzkwOGQ2OGFmMmM5MzQyMjVhYWMxOTU4MGMyZThkZTRhLmJpbmRQb3B1cChwb3B1cF81OWIwNmQ1YmIyNTU0ZjkwYThlMTFlMmI0OTc2OTQ3Nyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAoKICAgICAgICAgICAgdmFyIG1hcmtlcl9lZTI3YzRmZjkyYmI0YjNiOTMwMDQ2NjBlNTNlM2ZkNiA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzI1LjIxMzc4ODIsNTEuNTc4MDc4XSwKICAgICAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgICAgICBpY29uOiBuZXcgTC5JY29uLkRlZmF1bHQoKQogICAgICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgICAgICkKICAgICAgICAgICAgICAgIC5hZGRUbyhmZWF0dXJlX2dyb3VwX2FlNjM1ZWQ3MzU5MzRmZmFhY2JkNmFkMjBkYTdjMDFmKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2MwZjc3ODVjYTY1ODQ5NTJiZWIwNjFkNmNkYjk2NTRkID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzZmNWU3OWFiNzNjYTQ3N2Q4MTEwYzMxYWRiZmRjZDJlID0gJCgnPGRpdiBpZD0iaHRtbF82ZjVlNzlhYjczY2E0NzdkODExMGMzMWFkYmZkY2QyZSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+U2hhbnRpbmlrZXRhbiBJbmRpYW4gU2Nob29sPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9jMGY3Nzg1Y2E2NTg0OTUyYmViMDYxZDZjZGI5NjU0ZC5zZXRDb250ZW50KGh0bWxfNmY1ZTc5YWI3M2NhNDc3ZDgxMTBjMzFhZGJmZGNkMmUpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl9lZTI3YzRmZjkyYmI0YjNiOTMwMDQ2NjBlNTNlM2ZkNi5iaW5kUG9wdXAocG9wdXBfYzBmNzc4NWNhNjU4NDk1MmJlYjA2MWQ2Y2RiOTY1NGQpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKCiAgICAgICAgICAgIHZhciBtYXJrZXJfNzFjNzIxZmJjMzAwNDA0YmFhNGJhZTJmMjQ4OWMwNTkgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFsyNS4zMzYyMDEsNTEuNTEwMTcxNV0sCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgaWNvbjogbmV3IEwuSWNvbi5EZWZhdWx0KCkKICAgICAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICAgICApCiAgICAgICAgICAgICAgICAuYWRkVG8oZmVhdHVyZV9ncm91cF9hZTYzNWVkNzM1OTM0ZmZhYWNiZDZhZDIwZGE3YzAxZik7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8xNWY1ODc0NmY0MTk0YjI1OGZhNjY1MmRiMTA2NWIwYSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9iZGE0MjZkMGZmZTA0OTc5YmY1OTIzZTY1OTk4MjNiMyA9ICQoJzxkaXYgaWQ9Imh0bWxfYmRhNDI2ZDBmZmUwNDk3OWJmNTkyM2U2NTk5ODIzYjMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkRvaGEgQ29sbGVnZSBXZXN0IEJheTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMTVmNTg3NDZmNDE5NGIyNThmYTY2NTJkYjEwNjViMGEuc2V0Q29udGVudChodG1sX2JkYTQyNmQwZmZlMDQ5NzliZjU5MjNlNjU5OTgyM2IzKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBtYXJrZXJfNzFjNzIxZmJjMzAwNDA0YmFhNGJhZTJmMjQ4OWMwNTkuYmluZFBvcHVwKHBvcHVwXzE1ZjU4NzQ2ZjQxOTRiMjU4ZmE2NjUyZGIxMDY1YjBhKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCgogICAgICAgICAgICB2YXIgbWFya2VyXzMyNTUwNjJkYWVlMjRhNTJhNDg4MjA3ODlmZTY3NGNiID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMjUuMzcxMjAxNSw1MS40OTkxMzI0XSwKICAgICAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgICAgICBpY29uOiBuZXcgTC5JY29uLkRlZmF1bHQoKQogICAgICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgICAgICkKICAgICAgICAgICAgICAgIC5hZGRUbyhmZWF0dXJlX2dyb3VwX2FlNjM1ZWQ3MzU5MzRmZmFhY2JkNmFkMjBkYTdjMDFmKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2IxYjg2NzU2Y2E1ZDQyMjdhM2MyNjQ3YmE2YTY3Y2I1ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2M4ZjdmMzM5ZTU1NDQwMDU4ZmEyYWY1OTZhNTI5MjVkID0gJCgnPGRpdiBpZD0iaHRtbF9jOGY3ZjMzOWU1NTQ0MDA1OGZhMmFmNTk2YTUyOTI1ZCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QWwgV2F0YW5peWEgSW50ZXJuYXRpb25hbCBTY2hvb2wgKEFXSVMpPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9iMWI4Njc1NmNhNWQ0MjI3YTNjMjY0N2JhNmE2N2NiNS5zZXRDb250ZW50KGh0bWxfYzhmN2YzMzllNTU0NDAwNThmYTJhZjU5NmE1MjkyNWQpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl8zMjU1MDYyZGFlZTI0YTUyYTQ4ODIwNzg5ZmU2NzRjYi5iaW5kUG9wdXAocG9wdXBfYjFiODY3NTZjYTVkNDIyN2EzYzI2NDdiYTZhNjdjYjUpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKCiAgICAgICAgICAgIHZhciBtYXJrZXJfODY5ZWQzNTRmZGNlNDE3NThlNjk3NWY2OTRmNTU0ZGYgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFsyNS4wMDYwMzA5LDUxLjUzODgzM10sCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgaWNvbjogbmV3IEwuSWNvbi5EZWZhdWx0KCkKICAgICAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICAgICApCiAgICAgICAgICAgICAgICAuYWRkVG8oZmVhdHVyZV9ncm91cF9hZTYzNWVkNzM1OTM0ZmZhYWNiZDZhZDIwZGE3YzAxZik7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF82OTBjNTc3MmUwOGI0ZTQzOTM5ZjE2ODg3ZjFiZTczZCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF85MjcxOWZkZDg0N2M0NDdmODc0NGM3MDQxNTcwOGIxNSA9ICQoJzxkaXYgaWQ9Imh0bWxfOTI3MTlmZGQ4NDdjNDQ3Zjg3NDRjNzA0MTU3MDhiMTUiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPk1JQyBTZW5pb3IgU0Nob29sPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF82OTBjNTc3MmUwOGI0ZTQzOTM5ZjE2ODg3ZjFiZTczZC5zZXRDb250ZW50KGh0bWxfOTI3MTlmZGQ4NDdjNDQ3Zjg3NDRjNzA0MTU3MDhiMTUpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl84NjllZDM1NGZkY2U0MTc1OGU2OTc1ZjY5NGY1NTRkZi5iaW5kUG9wdXAocG9wdXBfNjkwYzU3NzJlMDhiNGU0MzkzOWYxNjg4N2YxYmU3M2QpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKCiAgICAgICAgICAgIHZhciBtYXJrZXJfZmVkZGE0MmFmOWVkNDZhOWFhYTc3ZWViOWFjN2I2OTEgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFsyNS4zMjI3LDUxLjQ0Mzk4MjhdLAogICAgICAgICAgICAgICAgewogICAgICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICAgICAgfQogICAgICAgICAgICAgICAgKQogICAgICAgICAgICAgICAgLmFkZFRvKGZlYXR1cmVfZ3JvdXBfYWU2MzVlZDczNTkzNGZmYWFjYmQ2YWQyMGRhN2MwMWYpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNzUxNTA2YTRjZWE0NDVmMmFhNjAxNzZjZDQ2Yjk1ZDAgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMWQ0MTU0NzIyYTM1NGFhM2JjZjI0MzFiZTUzNDBiMDIgPSAkKCc8ZGl2IGlkPSJodG1sXzFkNDE1NDcyMmEzNTRhYTNiY2YyNDMxYmU1MzQwYjAyIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5RYXRhciBGb3VuZGF0aW9uPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF83NTE1MDZhNGNlYTQ0NWYyYWE2MDE3NmNkNDZiOTVkMC5zZXRDb250ZW50KGh0bWxfMWQ0MTU0NzIyYTM1NGFhM2JjZjI0MzFiZTUzNDBiMDIpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl9mZWRkYTQyYWY5ZWQ0NmE5YWFhNzdlZWI5YWM3YjY5MS5iaW5kUG9wdXAocG9wdXBfNzUxNTA2YTRjZWE0NDVmMmFhNjAxNzZjZDQ2Yjk1ZDApOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKCiAgICAgICAgICAgIHZhciBtYXJrZXJfOGFmNTllZjIxMDJjNDI4N2FjY2VmODM2YmU5MTg2NWUgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFsyNS4zMjA4NjU1LDUxLjQyNDMyMThdLAogICAgICAgICAgICAgICAgewogICAgICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICAgICAgfQogICAgICAgICAgICAgICAgKQogICAgICAgICAgICAgICAgLmFkZFRvKGZlYXR1cmVfZ3JvdXBfYWU2MzVlZDczNTkzNGZmYWFjYmQ2YWQyMGRhN2MwMWYpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZTNlODBkOGQ3ZjVlNDllZDg4ODUzYTFmMDJiNTBmY2MgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfOTlmNTc0MzcyNWZlNDZjOGJlYTc1ZTk1YjZhMjBiYjYgPSAkKCc8ZGl2IGlkPSJodG1sXzk5ZjU3NDM3MjVmZTQ2YzhiZWE3NWU5NWI2YTIwYmI2IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5IQktVIFJlc2VhcmNoIENvbXBsZXg8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2UzZTgwZDhkN2Y1ZTQ5ZWQ4ODg1M2ExZjAyYjUwZmNjLnNldENvbnRlbnQoaHRtbF85OWY1NzQzNzI1ZmU0NmM4YmVhNzVlOTViNmEyMGJiNik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgbWFya2VyXzhhZjU5ZWYyMTAyYzQyODdhY2NlZjgzNmJlOTE4NjVlLmJpbmRQb3B1cChwb3B1cF9lM2U4MGQ4ZDdmNWU0OWVkODg4NTNhMWYwMmI1MGZjYyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAoKICAgICAgICAgICAgdmFyIG1hcmtlcl9lM2UwOTkxOTI4ZjU0OWY0OGFlZjE4M2ZhYmY3NTgxNiA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzI1LjMwMjU1ODcsNTEuNDg1ODUwOV0sCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgaWNvbjogbmV3IEwuSWNvbi5EZWZhdWx0KCkKICAgICAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICAgICApCiAgICAgICAgICAgICAgICAuYWRkVG8oZmVhdHVyZV9ncm91cF9hZTYzNWVkNzM1OTM0ZmZhYWNiZDZhZDIwZGE3YzAxZik7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9mM2Y1M2Q0NDA4OTg0ZDkzODdjYTJhNzA2Y2NiMzAxOSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8zZWE1ODRiMTVlM2M0MWRjODg5MGM1Yjg1ZjhlNzhjZSA9ICQoJzxkaXYgaWQ9Imh0bWxfM2VhNTg0YjE1ZTNjNDFkYzg4OTBjNWI4NWY4ZTc4Y2UiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkFsIFJheWFoIERyaXZpbmcgU2Nob29sPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9mM2Y1M2Q0NDA4OTg0ZDkzODdjYTJhNzA2Y2NiMzAxOS5zZXRDb250ZW50KGh0bWxfM2VhNTg0YjE1ZTNjNDFkYzg4OTBjNWI4NWY4ZTc4Y2UpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl9lM2UwOTkxOTI4ZjU0OWY0OGFlZjE4M2ZhYmY3NTgxNi5iaW5kUG9wdXAocG9wdXBfZjNmNTNkNDQwODk4NGQ5Mzg3Y2EyYTcwNmNjYjMwMTkpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKCiAgICAgICAgICAgIHZhciBtYXJrZXJfOWQyMDZkNDEyNGYzNGY2MDgxZmQwNmE0YTZlYjQ5MjEgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFsyNS4yMzY3ODQ4LDUxLjU1MDQ4ODFdLAogICAgICAgICAgICAgICAgewogICAgICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICAgICAgfQogICAgICAgICAgICAgICAgKQogICAgICAgICAgICAgICAgLmFkZFRvKGZlYXR1cmVfZ3JvdXBfYWU2MzVlZDczNTkzNGZmYWFjYmQ2YWQyMGRhN2MwMWYpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYzhlM2I5ZTQxOTM4NDg2ZmJjY2JhMjQ2MzE1ZGQ0NGMgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZTAxZjYwMGYzNzllNDdkOTliZTg0ZGFkNjIwOTg2ZGQgPSAkKCc8ZGl2IGlkPSJodG1sX2UwMWY2MDBmMzc5ZTQ3ZDk5YmU4NGRhZDYyMDk4NmRkIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5QZWFybCBTY2hvb2w8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2M4ZTNiOWU0MTkzODQ4NmZiY2NiYTI0NjMxNWRkNDRjLnNldENvbnRlbnQoaHRtbF9lMDFmNjAwZjM3OWU0N2Q5OWJlODRkYWQ2MjA5ODZkZCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgbWFya2VyXzlkMjA2ZDQxMjRmMzRmNjA4MWZkMDZhNGE2ZWI0OTIxLmJpbmRQb3B1cChwb3B1cF9jOGUzYjllNDE5Mzg0ODZmYmNjYmEyNDYzMTVkZDQ0Yyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAoKICAgICAgICAgICAgdmFyIG1hcmtlcl8wMDg0NDgzOGZjYmE0YzMyODQzZGU1MTAzMGJhMWI2OSA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzI1LjIzMDcwNTYsNTEuNDkxNTc3NV0sCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgaWNvbjogbmV3IEwuSWNvbi5EZWZhdWx0KCkKICAgICAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICAgICApCiAgICAgICAgICAgICAgICAuYWRkVG8oZmVhdHVyZV9ncm91cF9hZTYzNWVkNzM1OTM0ZmZhYWNiZDZhZDIwZGE3YzAxZik7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9iMjI3NzRmNzQxZGY0NDQ2ODUyNmYxOTIzN2VmMGM3YiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8yNTVkOGMyODAzMDk0MmQ1OGQ0YTIxNzJmYTZlMzU2YyA9ICQoJzxkaXYgaWQ9Imh0bWxfMjU1ZDhjMjgwMzA5NDJkNThkNGEyMTcyZmE2ZTM1NmMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlR1bmlzaWFuIHNjaG9vbCBhYnUgaGFtb3VyPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9iMjI3NzRmNzQxZGY0NDQ2ODUyNmYxOTIzN2VmMGM3Yi5zZXRDb250ZW50KGh0bWxfMjU1ZDhjMjgwMzA5NDJkNThkNGEyMTcyZmE2ZTM1NmMpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl8wMDg0NDgzOGZjYmE0YzMyODQzZGU1MTAzMGJhMWI2OS5iaW5kUG9wdXAocG9wdXBfYjIyNzc0Zjc0MWRmNDQ0Njg1MjZmMTkyMzdlZjBjN2IpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKCiAgICAgICAgICAgIHZhciBtYXJrZXJfOTFhODFjNGU1OGRhNDYyOTg0NTc4ZDkyNzk5NDYxOTUgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFsyNS4zMjg5MTUsNTEuMzg2MTUxNl0sCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgaWNvbjogbmV3IEwuSWNvbi5EZWZhdWx0KCkKICAgICAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICAgICApCiAgICAgICAgICAgICAgICAuYWRkVG8oZmVhdHVyZV9ncm91cF9hZTYzNWVkNzM1OTM0ZmZhYWNiZDZhZDIwZGE3YzAxZik7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8zMjIzMDEzMjNkY2I0MzNhOTI5MWYxOGM2ZTFhMTgyYyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9kMmRkODMwOTI0NTM0YTViODM3ODY3YTQ5OGE1NDYwNiA9ICQoJzxkaXYgaWQ9Imh0bWxfZDJkZDgzMDkyNDUzNGE1YjgzNzg2N2E0OThhNTQ2MDYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPldvcms8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzMyMjMwMTMyM2RjYjQzM2E5MjkxZjE4YzZlMWExODJjLnNldENvbnRlbnQoaHRtbF9kMmRkODMwOTI0NTM0YTViODM3ODY3YTQ5OGE1NDYwNik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgbWFya2VyXzkxYTgxYzRlNThkYTQ2Mjk4NDU3OGQ5Mjc5OTQ2MTk1LmJpbmRQb3B1cChwb3B1cF8zMjIzMDEzMjNkY2I0MzNhOTI5MWYxOGM2ZTFhMTgyYyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAoKICAgICAgICAgICAgdmFyIG1hcmtlcl8yYzE0YjI2MzhkNDQ0MWQyODk5ZWVmYmUzMGI2ZWMzOCA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzI1LjIzMTE0MjQsNTEuNDk0NTU2NV0sCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgaWNvbjogbmV3IEwuSWNvbi5EZWZhdWx0KCkKICAgICAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICAgICApCiAgICAgICAgICAgICAgICAuYWRkVG8oZmVhdHVyZV9ncm91cF9hZTYzNWVkNzM1OTM0ZmZhYWNiZDZhZDIwZGE3YzAxZik7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9jZmY0NzQ1ZWMzMWY0NzRjOTZhOTYxNjQyZGE0MDNlZCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF82ZDE2ZjRiYTYwNjg0Zjc4OWIyZTFkMDJlYmY5NWFkNSA9ICQoJzxkaXYgaWQ9Imh0bWxfNmQxNmY0YmE2MDY4NGY3ODliMmUxZDAyZWJmOTVhZDUiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPm5vYmxlIGludGVybmF0aW9uYWwgc2Nob29sPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9jZmY0NzQ1ZWMzMWY0NzRjOTZhOTYxNjQyZGE0MDNlZC5zZXRDb250ZW50KGh0bWxfNmQxNmY0YmE2MDY4NGY3ODliMmUxZDAyZWJmOTVhZDUpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl8yYzE0YjI2MzhkNDQ0MWQyODk5ZWVmYmUzMGI2ZWMzOC5iaW5kUG9wdXAocG9wdXBfY2ZmNDc0NWVjMzFmNDc0Yzk2YTk2MTY0MmRhNDAzZWQpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKCiAgICAgICAgICAgIHZhciBtYXJrZXJfYmQ5NDVhMjk4NDM3NDk3MWFmZGVkOTA5MGQ0NTNjNjggPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFsyNS4yMzIyODg2LDUxLjQ5NDM3NjJdLAogICAgICAgICAgICAgICAgewogICAgICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICAgICAgfQogICAgICAgICAgICAgICAgKQogICAgICAgICAgICAgICAgLmFkZFRvKGZlYXR1cmVfZ3JvdXBfYWU2MzVlZDczNTkzNGZmYWFjYmQ2YWQyMGRhN2MwMWYpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMjVlMzllNWJhYjc1NGE4NThlNDA2MmM0Yjk1ZGI2NzUgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZmVmMjE4Nzg1YzljNDQxZDk4ZTUwNGJmODZkNzA4YmEgPSAkKCc8ZGl2IGlkPSJodG1sX2ZlZjIxODc4NWM5YzQ0MWQ5OGU1MDRiZjg2ZDcwOGJhIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5wYXJrPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8yNWUzOWU1YmFiNzU0YTg1OGU0MDYyYzRiOTVkYjY3NS5zZXRDb250ZW50KGh0bWxfZmVmMjE4Nzg1YzljNDQxZDk4ZTUwNGJmODZkNzA4YmEpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl9iZDk0NWEyOTg0Mzc0OTcxYWZkZWQ5MDkwZDQ1M2M2OC5iaW5kUG9wdXAocG9wdXBfMjVlMzllNWJhYjc1NGE4NThlNDA2MmM0Yjk1ZGI2NzUpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKCiAgICAgICAgICAgIHZhciBtYXJrZXJfNTJkNjk1ZDZhNzY2NDZkNjk4ZGY0ZGIwOTI5ZmU0M2QgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFsyNS4zMjg4ODU1LDUxLjM4NzI1MzFdLAogICAgICAgICAgICAgICAgewogICAgICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICAgICAgfQogICAgICAgICAgICAgICAgKQogICAgICAgICAgICAgICAgLmFkZFRvKGZlYXR1cmVfZ3JvdXBfYWU2MzVlZDczNTkzNGZmYWFjYmQ2YWQyMGRhN2MwMWYpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYzAyZjk4YjFjZjcxNDU4ZGE2MWE1ZWI0OTJlNmViMjAgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfYzAxZTk1MjA5N2E1NGQ3OGFjMjZhMmJiNGMwZWNhOTEgPSAkKCc8ZGl2IGlkPSJodG1sX2MwMWU5NTIwOTdhNTRkNzhhYzI2YTJiYjRjMGVjYTkxIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5hbGkgYmluIGphc3NpbSBzZWNvbmRhcnkgc2Nob29sIGZvciBib3lzPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9jMDJmOThiMWNmNzE0NThkYTYxYTVlYjQ5MmU2ZWIyMC5zZXRDb250ZW50KGh0bWxfYzAxZTk1MjA5N2E1NGQ3OGFjMjZhMmJiNGMwZWNhOTEpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl81MmQ2OTVkNmE3NjY0NmQ2OThkZjRkYjA5MjlmZTQzZC5iaW5kUG9wdXAocG9wdXBfYzAyZjk4YjFjZjcxNDU4ZGE2MWE1ZWI0OTJlNmViMjApOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKCiAgICAgICAgICAgIHZhciBtYXJrZXJfMzNjZDA3YTJiMDY0NDNmM2E4ZjgxMGU4MWE1Y2ZmNzcgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFsyNS4zMDI1NDgzLDUxLjQ4NTgzNjZdLAogICAgICAgICAgICAgICAgewogICAgICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICAgICAgfQogICAgICAgICAgICAgICAgKQogICAgICAgICAgICAgICAgLmFkZFRvKGZlYXR1cmVfZ3JvdXBfYWU2MzVlZDczNTkzNGZmYWFjYmQ2YWQyMGRhN2MwMWYpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMGMyZDcxMzEwZGQ3NDE0Mzk1NWVlYTkxN2Y4NTU3NTggPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZTVkNzdmMDhlNjgyNDA0YmFlNTI0NWZmYjY4N2JlMWIgPSAkKCc8ZGl2IGlkPSJodG1sX2U1ZDc3ZjA4ZTY4MjQwNGJhZTUyNDVmZmI2ODdiZTFiIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5BbC1SYXlhaCBEcml2aW5nIHNjaG9vbDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMGMyZDcxMzEwZGQ3NDE0Mzk1NWVlYTkxN2Y4NTU3NTguc2V0Q29udGVudChodG1sX2U1ZDc3ZjA4ZTY4MjQwNGJhZTUyNDVmZmI2ODdiZTFiKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBtYXJrZXJfMzNjZDA3YTJiMDY0NDNmM2E4ZjgxMGU4MWE1Y2ZmNzcuYmluZFBvcHVwKHBvcHVwXzBjMmQ3MTMxMGRkNzQxNDM5NTVlZWE5MTdmODU1NzU4KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCgogICAgICAgICAgICB2YXIgbWFya2VyXzRkMjljMDM2YTlkNTQwZDI4MjE4Yjg1NDhkNWFiMmMxID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMjUuMjc5NzUzLDUxLjQ5MTQ2NV0sCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgaWNvbjogbmV3IEwuSWNvbi5EZWZhdWx0KCkKICAgICAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICAgICApCiAgICAgICAgICAgICAgICAuYWRkVG8oZmVhdHVyZV9ncm91cF9hZTYzNWVkNzM1OTM0ZmZhYWNiZDZhZDIwZGE3YzAxZik7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9jNTMwMGMxZjNlNzU0Yjc0YWVlOTg1NGJmZGI3NjQ3NiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8wNzUxYzlhNDUxNmU0NDE2YTg2ODNjZTNiZDhlZWRhNiA9ICQoJzxkaXYgaWQ9Imh0bWxfMDc1MWM5YTQ1MTZlNDQxNmE4NjgzY2UzYmQ4ZWVkYTYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJyaXRpc2ggQ291bmNpbCBRYXRhcjwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYzUzMDBjMWYzZTc1NGI3NGFlZTk4NTRiZmRiNzY0NzYuc2V0Q29udGVudChodG1sXzA3NTFjOWE0NTE2ZTQ0MTZhODY4M2NlM2JkOGVlZGE2KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBtYXJrZXJfNGQyOWMwMzZhOWQ1NDBkMjgyMThiODU0OGQ1YWIyYzEuYmluZFBvcHVwKHBvcHVwX2M1MzAwYzFmM2U3NTRiNzRhZWU5ODU0YmZkYjc2NDc2KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCgogICAgICAgICAgICB2YXIgbWFya2VyXzJhZDhmYTEwY2M1MjQwOTZiNWQyOWM5NTllYWU0MDEzID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMjUuMTk1Mzg3Nyw1MS40Nzg2NTYzXSwKICAgICAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgICAgICBpY29uOiBuZXcgTC5JY29uLkRlZmF1bHQoKQogICAgICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgICAgICkKICAgICAgICAgICAgICAgIC5hZGRUbyhmZWF0dXJlX2dyb3VwX2FlNjM1ZWQ3MzU5MzRmZmFhY2JkNmFkMjBkYTdjMDFmKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2YzNWE5NmNmYmY1MDQzMmZhNTYzZDkwNGNiYjNlYTExID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2FkZDQyNzJjNjYyYzQ5NjlhNjMyZTYyNDhhZWM5M2Y2ID0gJCgnPGRpdiBpZD0iaHRtbF9hZGQ0MjcyYzY2MmM0OTY5YTYzMmU2MjQ4YWVjOTNmNiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+REFMTEEgRFJJVklORyBBQ0FERU1ZPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9mMzVhOTZjZmJmNTA0MzJmYTU2M2Q5MDRjYmIzZWExMS5zZXRDb250ZW50KGh0bWxfYWRkNDI3MmM2NjJjNDk2OWE2MzJlNjI0OGFlYzkzZjYpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl8yYWQ4ZmExMGNjNTI0MDk2YjVkMjljOTU5ZWFlNDAxMy5iaW5kUG9wdXAocG9wdXBfZjM1YTk2Y2ZiZjUwNDMyZmE1NjNkOTA0Y2JiM2VhMTEpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKCiAgICAgICAgICAgIHZhciBtYXJrZXJfZDMzY2NjNzcwNzIwNDY3MmJhZWRiOWZmNWEzMTJjOTQgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFsyNS40NDkwODc4LDUxLjQzNzg5MjhdLAogICAgICAgICAgICAgICAgewogICAgICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICAgICAgfQogICAgICAgICAgICAgICAgKQogICAgICAgICAgICAgICAgLmFkZFRvKGZlYXR1cmVfZ3JvdXBfYWU2MzVlZDczNTkzNGZmYWFjYmQ2YWQyMGRhN2MwMWYpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfODgxNTA1YjIwMjRiNDJlM2E4OTU2ODA2NDQ1MzJmYzggPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMjY3ZDUxMWNmZTBkNGY4NjgyZTE2MTRmYTYyZTY0MzIgPSAkKCc8ZGl2IGlkPSJodG1sXzI2N2Q1MTFjZmUwZDRmODY4MmUxNjE0ZmE2MmU2NDMyIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5NYXJpYSBBbCBRaWJ0ZXlhIFByZXAuPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF84ODE1MDViMjAyNGI0MmUzYTg5NTY4MDY0NDUzMmZjOC5zZXRDb250ZW50KGh0bWxfMjY3ZDUxMWNmZTBkNGY4NjgyZTE2MTRmYTYyZTY0MzIpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl9kMzNjY2M3NzA3MjA0NjcyYmFlZGI5ZmY1YTMxMmM5NC5iaW5kUG9wdXAocG9wdXBfODgxNTA1YjIwMjRiNDJlM2E4OTU2ODA2NDQ1MzJmYzgpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKCiAgICAgICAgICAgIHZhciBtYXJrZXJfMmFkOTFlMzgxYjY0NDc3MmE0NTJlYjhjNGZlODEwYmYgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFsyNS4zODgwOTQ5LDUxLjQ4MzkwMjZdLAogICAgICAgICAgICAgICAgewogICAgICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICAgICAgfQogICAgICAgICAgICAgICAgKQogICAgICAgICAgICAgICAgLmFkZFRvKGZlYXR1cmVfZ3JvdXBfYWU2MzVlZDczNTkzNGZmYWFjYmQ2YWQyMGRhN2MwMWYpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMWViZDRmNTczMDczNGIyMTk5YTM3NWFiZjI5MDQyMjcgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNTAzODExMThiYjVkNGIwNTlkNjRjMzJjYTU0YTcxNGYgPSAkKCc8ZGl2IGlkPSJodG1sXzUwMzgxMTE4YmI1ZDRiMDU5ZDY0YzMyY2E1NGE3MTRmIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Eb2hhIGluc3RpdHV0ZSBmb3IgZ3JhZHVhdGUgc3R1ZGllczwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMWViZDRmNTczMDczNGIyMTk5YTM3NWFiZjI5MDQyMjcuc2V0Q29udGVudChodG1sXzUwMzgxMTE4YmI1ZDRiMDU5ZDY0YzMyY2E1NGE3MTRmKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBtYXJrZXJfMmFkOTFlMzgxYjY0NDc3MmE0NTJlYjhjNGZlODEwYmYuYmluZFBvcHVwKHBvcHVwXzFlYmQ0ZjU3MzA3MzRiMjE5OWEzNzVhYmYyOTA0MjI3KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCgogICAgICAgICAgICB2YXIgbWFya2VyX2IzZWZiMzRmNGVhNzRhZjZiYjMxODVjNmUwNTQ4M2M1ID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMjUuMzg3OTcsNTEuNDgzOTA1XSwKICAgICAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgICAgICBpY29uOiBuZXcgTC5JY29uLkRlZmF1bHQoKQogICAgICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgICAgICkKICAgICAgICAgICAgICAgIC5hZGRUbyhmZWF0dXJlX2dyb3VwX2FlNjM1ZWQ3MzU5MzRmZmFhY2JkNmFkMjBkYTdjMDFmKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2U3ZmMzNjFhMGY2NjQ3ZGRhMWQyZDYzNTEyYTdkNjEwID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzYxOTBlZTcxZGRjODQ5ZTZiZGNiNDVhZDNhODBjNTFjID0gJCgnPGRpdiBpZD0iaHRtbF82MTkwZWU3MWRkYzg0OWU2YmRjYjQ1YWQzYTgwYzUxYyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RG9oYSBpbnN0aXR1dGUgZm9yIGdyYWR1YXRlIHN0dWRpZXM8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2U3ZmMzNjFhMGY2NjQ3ZGRhMWQyZDYzNTEyYTdkNjEwLnNldENvbnRlbnQoaHRtbF82MTkwZWU3MWRkYzg0OWU2YmRjYjQ1YWQzYTgwYzUxYyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgbWFya2VyX2IzZWZiMzRmNGVhNzRhZjZiYjMxODVjNmUwNTQ4M2M1LmJpbmRQb3B1cChwb3B1cF9lN2ZjMzYxYTBmNjY0N2RkYTFkMmQ2MzUxMmE3ZDYxMCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAoKICAgICAgICAgICAgdmFyIG1hcmtlcl9kYjUyOTAyZWIwMzU0YzAyOWJkYWYyYjBhNzJjNmM3YyA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzI1LjIwNDYxNTksNTEuNDQ3NTcwOF0sCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgaWNvbjogbmV3IEwuSWNvbi5EZWZhdWx0KCkKICAgICAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICAgICApCiAgICAgICAgICAgICAgICAuYWRkVG8oZmVhdHVyZV9ncm91cF9hZTYzNWVkNzM1OTM0ZmZhYWNiZDZhZDIwZGE3YzAxZik7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF83MzVkYzYyZjc1NmE0MWYxYmFlNzVjYTI3ZGE5NDEwMSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9jY2Y5N2U2MTU1MWI0MzBhOWY5ZGQ0ZGUxYzNkMGVlZSA9ICQoJzxkaXYgaWQ9Imh0bWxfY2NmOTdlNjE1NTFiNDMwYTlmOWRkNGRlMWMzZDBlZWUiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJhcndhIGNvbW1lcmNpYWwgYXZlbnVlPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF83MzVkYzYyZjc1NmE0MWYxYmFlNzVjYTI3ZGE5NDEwMS5zZXRDb250ZW50KGh0bWxfY2NmOTdlNjE1NTFiNDMwYTlmOWRkNGRlMWMzZDBlZWUpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl9kYjUyOTAyZWIwMzU0YzAyOWJkYWYyYjBhNzJjNmM3Yy5iaW5kUG9wdXAocG9wdXBfNzM1ZGM2MmY3NTZhNDFmMWJhZTc1Y2EyN2RhOTQxMDEpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKCiAgICAgICAgICAgIHZhciBtYXJrZXJfY2M1YWI0OThhYzZmNGI1NzlmMjQ2YjczYmJmNDhkN2IgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFsyNS4zNDk4MDU0LDUxLjQ0NTgzNTNdLAogICAgICAgICAgICAgICAgewogICAgICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICAgICAgfQogICAgICAgICAgICAgICAgKQogICAgICAgICAgICAgICAgLmFkZFRvKGZlYXR1cmVfZ3JvdXBfYWU2MzVlZDczNTkzNGZmYWFjYmQ2YWQyMGRhN2MwMWYpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZDhjZGQyY2Q5ZWYzNDVkZDg4ZDExNTFlY2Q5ZTU4NmIgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfYzI2YmUyZjc1ODBiNDhlMzhjNTlhOTRiM2JiY2ViOWMgPSAkKCc8ZGl2IGlkPSJodG1sX2MyNmJlMmY3NTgwYjQ4ZTM4YzU5YTk0YjNiYmNlYjljIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5HSEVSQVMgSU5URVJOQVRJT05BTCBTQ0hPT0w8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2Q4Y2RkMmNkOWVmMzQ1ZGQ4OGQxMTUxZWNkOWU1ODZiLnNldENvbnRlbnQoaHRtbF9jMjZiZTJmNzU4MGI0OGUzOGM1OWE5NGIzYmJjZWI5Yyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgbWFya2VyX2NjNWFiNDk4YWM2ZjRiNTc5ZjI0NmI3M2JiZjQ4ZDdiLmJpbmRQb3B1cChwb3B1cF9kOGNkZDJjZDllZjM0NWRkODhkMTE1MWVjZDllNTg2Yik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAoKICAgICAgICAgICAgdmFyIG1hcmtlcl9lN2I1ZWU2ZGJlYWM0ZDQ0OWJlOTRmNTQ3NzNkODk0MiA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzI1LjI2MzIwNjcsNTEuNTI3OTQ3Nl0sCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgaWNvbjogbmV3IEwuSWNvbi5EZWZhdWx0KCkKICAgICAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICAgICApCiAgICAgICAgICAgICAgICAuYWRkVG8oZmVhdHVyZV9ncm91cF9hZTYzNWVkNzM1OTM0ZmZhYWNiZDZhZDIwZGE3YzAxZik7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF84NDMyNDdiMDUwYzc0YWMzYWQyN2JiYjkwYjBhZmRkNyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8zMDBhNGFlOTZmZDQ0NTdlYjFjNzIxMzc5NGJkMTRjMCA9ICQoJzxkaXYgaWQ9Imh0bWxfMzAwYTRhZTk2ZmQ0NDU3ZWIxYzcyMTM3OTRiZDE0YzAiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkxhIFZpbGxhIEhvc3BpdGFsaXR5PC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF84NDMyNDdiMDUwYzc0YWMzYWQyN2JiYjkwYjBhZmRkNy5zZXRDb250ZW50KGh0bWxfMzAwYTRhZTk2ZmQ0NDU3ZWIxYzcyMTM3OTRiZDE0YzApOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl9lN2I1ZWU2ZGJlYWM0ZDQ0OWJlOTRmNTQ3NzNkODk0Mi5iaW5kUG9wdXAocG9wdXBfODQzMjQ3YjA1MGM3NGFjM2FkMjdiYmI5MGIwYWZkZDcpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKCiAgICAgICAgICAgIHZhciBtYXJrZXJfYjZkZDVlMTgwZmM5NDBkNjg2NTQ4Nzk3ZTIzOTBiNzkgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFsyNS4yMTE1MTc0LDUxLjUyMjQ5MDZdLAogICAgICAgICAgICAgICAgewogICAgICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICAgICAgfQogICAgICAgICAgICAgICAgKQogICAgICAgICAgICAgICAgLmFkZFRvKGZlYXR1cmVfZ3JvdXBfYWU2MzVlZDczNTkzNGZmYWFjYmQ2YWQyMGRhN2MwMWYpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZGZlNGFlMGQwOWNmNGRmMGE5OTQ2MzIyZGUwNDk4YmUgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZGVmM2M2ZWVkMzAyNDQ2ZGJiMzc0MmYzYjJmYTdiMmEgPSAkKCc8ZGl2IGlkPSJodG1sX2RlZjNjNmVlZDMwMjQ0NmRiYjM3NDJmM2IyZmE3YjJhIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5CaXJsYSBzY2hvb2w8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2RmZTRhZTBkMDljZjRkZjBhOTk0NjMyMmRlMDQ5OGJlLnNldENvbnRlbnQoaHRtbF9kZWYzYzZlZWQzMDI0NDZkYmIzNzQyZjNiMmZhN2IyYSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgbWFya2VyX2I2ZGQ1ZTE4MGZjOTQwZDY4NjU0ODc5N2UyMzkwYjc5LmJpbmRQb3B1cChwb3B1cF9kZmU0YWUwZDA5Y2Y0ZGYwYTk5NDYzMjJkZTA0OThiZSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAoKICAgICAgICAgICAgdmFyIG1hcmtlcl80NzExOGU0MTgxYjM0NTYzYTE2NmM2NTE5OGY4YThkMiA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzI1LjI2NDQzNzUsNTEuNDgzNTc4M10sCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgaWNvbjogbmV3IEwuSWNvbi5EZWZhdWx0KCkKICAgICAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICAgICApCiAgICAgICAgICAgICAgICAuYWRkVG8oZmVhdHVyZV9ncm91cF9hZTYzNWVkNzM1OTM0ZmZhYWNiZDZhZDIwZGE3YzAxZik7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF81OGM4YjUwMDNiNTY0ODVjYTI1MzBhMTZlMjc4YjQyNSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF84ZDY0ZDhhODUzMDg0NjEyYWMyOTMyZDMyOGM3NWQ1ZCA9ICQoJzxkaXYgaWQ9Imh0bWxfOGQ2NGQ4YTg1MzA4NDYxMmFjMjkzMmQzMjhjNzVkNWQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkFtZXJpY2FuIEx5Y2V0dWZmIFByZSAtIFNjaG9vbDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNThjOGI1MDAzYjU2NDg1Y2EyNTMwYTE2ZTI3OGI0MjUuc2V0Q29udGVudChodG1sXzhkNjRkOGE4NTMwODQ2MTJhYzI5MzJkMzI4Yzc1ZDVkKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBtYXJrZXJfNDcxMThlNDE4MWIzNDU2M2ExNjZjNjUxOThmOGE4ZDIuYmluZFBvcHVwKHBvcHVwXzU4YzhiNTAwM2I1NjQ4NWNhMjUzMGExNmUyNzhiNDI1KTsKCiAgICAgICAgICAgIAogICAgICAgIAo8L3NjcmlwdD4=" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



#### Sports

Finally, I wanted to get an idea on how popular sports are, and which ones have the most locations dedicated to them.


```python
import matplotlib.pyplot as plt
import seaborn as sns
from collections import defaultdict
%matplotlib inline
sports=doha.find({"sport":{"$exists":"true"}},{"sport":1,"_id":0})
num_sports=sports.count()
sportsdict=defaultdict(int)
for doc in sports:
    sportsdict[doc.values()[0].encode("ascii")]+=1
other_sports=[]
sum_others=0
threshold=5
for k,v in sportsdict.items():
    if (v<threshold):
        other_sports.append(k)
        sum_others+=v
        del sportsdict[k]
sportsdict["others"]=sum_others
colours = ['yellowgreen','indianred', 'gold', 'lightskyblue', 'lightcoral','slategrey','steelblue','orange']
plt.axis("equal")
p=plt.pie(sportsdict.values(),
        labels=sportsdict.keys(),
        startangle=90,
        radius=2.0,
        labeldistance=1.2,
        colors=colours,
        shadow=True)
plt.title('Sports practiced in Doha', bbox={'facecolor':'0.8', 'pad':5},position=(0.5,1.6))
print "There are {} locations dedicated to sports".format(num_sports)
print "Other sports include";pprint(other_sports)
```

    There are 293 locations dedicated to sports
    Other sports include
    ['9pin',
     'billiards',
     'gymnastics',
     'motor',
     'handball',
     'shooting',
     'taekwondo',
     'rugby_league',
     'cricket',
     '10pin',
     'squash',
     'baseball',
     'gaelic_games',
     'dog_racing',
     'fitness',
     'athletics']
    


![sports]( bk-ikram.github.io/img/sports-pie.png )


As expected, football('soccer') and equestrian sports are quite popular in Qatar. I was surprised to see that there were more tennis courts than football fields.

## Conclusion

Working with this dataset showed that it was incomplete, and a lot of entries made to it had inconsistencies and inaccuracies, amounting to a lot of dirty data. A large number of venues were listed as "ways" rather than "nodes". Problems such as this could have been easily avoided if first-time contributors to OpenStreetMap had to go through basic mandatory "training". Additionally, there was data that was entered in Arabic. This adds to the complexity in processing, auditing and analysing the entire dataset, and was ignored in this endeavor. Fortunately, this dataset has helped in providing some insights into Doha, even though I only explored a few fields.
