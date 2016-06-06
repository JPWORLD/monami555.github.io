---
layout: post
title: Data Wrangling with MongoDB - Udacity course notes
date: '2014-12-14T18:09:00.004+01:00'
author: Monik
tags:
- Programming
- MongoDB
- Python
- Data Mining
- NoSQL
modified_time: '2016-01-13T20:58:34.006+01:00'
blogger_id: tag:blogger.com,1999:blog-5940427300271272994.post-1884713552860826320
blogger_orig_url: http://learningmonik.blogspot.com/2014/12/data-wrangling-with-mongodb.html
commentIssueId: 23
type: course
---

<div class="bg-info panel-body" markdown="1">
These are the notes I took while taking the "Data Wrangling with MongoDB" course at <a href="https://www.udacity.com/">Udacity</a>. It tells how to use Python to process CSV, XML, Excel, and how to work with MongoBD. Also some examples for page scraping in Python.
</div>

<h3>Table of contents</h3>
- TOC
{:toc max_level=2}


### 1. Data extraction Fundamentals

<ul>
    <li>data wrangling = gathering, extracting, cleaning and storing data</li>
    <li>you have to assure your data is correct before doing anything else with it, especially if a human was involved in writing the data</li>
</ul>

#### 1.1 Tabular data and CSV

<ul>
        <li>items (row) have fields (columns), and first row contains label</li>
        <li>parsing CSV in python - manually<br/><br/><span
                style="font-family: Courier New, Courier, monospace; font-size: xx-small;">with open(datafile, "r") as f:<br/>&nbsp; titles = string.split(f.readline(), ",") &nbsp; &nbsp; &nbsp; <br/>&nbsp; &nbsp; &nbsp;for line in f:<br/>&nbsp; &nbsp; &nbsp; &nbsp; data.append(create_data(titles,string.split(line, ",")))<br/>&nbsp; &nbsp; &nbsp; &nbsp; i=i+1<br/>&nbsp; &nbsp; &nbsp; &nbsp; if (i&gt;10):<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; break<br/>return data</span>
        </li>
        <li>parsing XLS in python - with XLRD module<br/><br/><span
                style="font-family: Courier New, Courier, monospace; font-size: xx-small;">import xlrd<br/><br/>workbook = xlrd.open_workbook(datafile)<br/>sheet = workbook.sheet_by_index(0)<br/>data = [[sheet.cell_value(r, col) for col in range(2)] for r in range(sheet.nrows)]<br/>minVal = data[1][1]<br/>minIndex = 1</span>
        </li>
        <li>parsing CSV with python - with CSV module<br/><br/><span
                style="font-family: Courier New, Courier, monospace; font-size: xx-small;">import csv<br/><br/>with open(datafile,'rb') as f:<br/>&nbsp; &nbsp; reader = csv.reader(f)#, delimiter=',', quotechar='|'<br/>&nbsp; &nbsp; name = reader.next()[1]<br/>&nbsp; &nbsp; print name<br/>&nbsp; &nbsp; reader.next()<br/>&nbsp; &nbsp; while True:<br/>&nbsp; &nbsp; &nbsp; try:<br/>&nbsp; &nbsp; &nbsp; &nbsp; data.append(reader.next())<br/>&nbsp; &nbsp; &nbsp; except StopIteration:<br/>&nbsp; &nbsp; &nbsp; &nbsp; break</span>
        </li>
        <li><span style="font-family: inherit;">writing CSV with python</span><br/><span
                style="font-family: Courier New, Courier, monospace; font-size: xx-small;">with open(filename, 'wb') as csvfile:</span><br/><span
                style="font-family: Courier New, Courier, monospace; font-size: xx-small;">&nbsp; writer = csv.writer(csvfile, delimiter='|')</span><br/><span
                style="font-family: Courier New, Courier, monospace; font-size: xx-small;">&nbsp; for row in data:</span><br/><span
                style="font-family: 'Courier New', Courier, monospace; font-size: xx-small;">&nbsp; &nbsp; writer.writerow(row)</span>
        </li>
    </ul>


#### 1.2. JSON format

<ul>
    <li>nested objects and arrays</li>
    <li>different items can have different fields</li>
</ul>

### 2. Data in More Complex Formats

#### 2.1. XML

<ul>
    <li>designed for platform independent data transfer</li>
    <li>supports validation</li>
    <li>the domain: citation analysis, analysing citations in XML versions of scientific publications</li>
    <li>XML parsing with python<br/><br/><span
            style="font-family: Courier New, Courier, monospace; font-size: xx-small;">import xml.etree.ElementTree as ET<br/><br/>tree = ET.parse(fname)<br/>root = tree.getroot()<br/><br/>for author in root.findall('./fm/bibl/aug/au'):<br/>&nbsp; &nbsp; data = {<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "fnm": author.find('fnm').text,<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "snm": author.find('snm').text,<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "email": author.find('email').text,<br/>&nbsp; &nbsp; }<br/>&nbsp; &nbsp; authors.append(data)</span>
    </li>
    <li>handling attributes<br/><br/><span
            style="font-family: 'Courier New', Courier, monospace; font-size: xx-small;">for author in root.findall('./fm/bibl/aug/au'):</span><br/><span
            style="font-family: Courier New, Courier, monospace; font-size: xx-small;">&nbsp; &nbsp;insrs = author.findall('./insr')<br/>&nbsp; &nbsp; &nbsp; &nbsp; for insr in insrs:<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; data["insr"].append(insr.attrib["iid"])<br/>&nbsp; &nbsp; &nbsp; &nbsp; print data</span>
    </li>
</ul>

#### 2.2. Data scraping

<div>The example of website about arrivals and departures and various airports. There are two combo boxes, so we would have to click a lot to get all the data. We want to rather write a script for us.</div>
<div>
        <ul>
            <li>first we view page source to find all the codes of the combobox lists</li>
            <li>then we open firebug to see what calls exactly are made, and which parts are depending on combobox clicks</li>
            <li>we try to identify the other parameters (e.g. sth like "_viewstate", etc)</li>
            <li>we write a script which makes call by call and generates a set of static HTML pages</li>
            <li>! important - first store the data, then analyse it ! - best practice</li>
            <li>Beautiful Soup - <span
                    style="font-family: Courier New, Courier, monospace; font-size: xx-small;">from bs4 import BeautifulSoup </span>- the module for finding the values of a field in HTML, expecially for this kind of tasks<br/><br/><span
                    style="font-family: Courier New, Courier, monospace; font-size: xx-small;">with open(page, "r") as html:<br/>&nbsp; &nbsp; &nbsp; &nbsp; soup = BeautifulSoup(html)<br/>&nbsp; &nbsp; &nbsp; &nbsp; data["eventvalidation"] = soup.find(id="__EVENTVALIDATION")['value']<br/>&nbsp; &nbsp; &nbsp; &nbsp; data["viewstate"] = soup.find(id="__VIEWSTATE")['value']</span>
            </li>
            <li>we also have to maintain session it turned out<br/><br/><span
                    style="font-family: 'Courier New', Courier, monospace; font-size: xx-small;"><strike>requests.post("http://..", data={...})</strike></span><br/><span
                    style="font-family: Courier New, Courier, monospace; font-size: xx-small;">s = requests.Session()<br/>s.post("http://..", data={...})</span>
            </li>
            <li>then write another script(s) that iterate(s) through the downloaded files, clean(s) up the data (=audits it), and do(es) what we need to do &nbsp;- this is actually covered in several practical exercises</li>
        </ul>
    </div>

### 3. Data Quality

<ul>
            <li>data cleaning is an iterative process</li>
            <li>measures of data quality:</li>
            <ul>
                <li>validity - how it conforms to a schema (official or inofficial)</li>
                <li>accuracy - "do all the street addresses exist?" - compare with some gold standard data</li>
                <li>completness - are some record missing?</li>
                <li>consistency - does some data contradict other parts of data</li>
                <li>uniformity&nbsp;- e.g. same units</li>
            </ul>
            <li>process:</li>
            <ul>
                <li>audit data (statistical analysis - how many types of which value exist)</li>
                <li>plan how to to correct</li>
                <li>test if it worked</li>
                <li>manual correction step</li>
                <li>repeat, until you have confidence in your data</li>
            </ul>
            <li>you can also do data enhancement besides the cleaning</li>
        </ul>

#### 3.1. Various small examples

<ul>
            <li>open street map data example - we change St. to Street and Av. to Avenue, etc., after seeing the statistics (validity)</li>
            <li>dbpedia data set about cities - we find the error where density is in persons/square km, and area is in square meters, we also see arrays instead of single values, multiple timezones&nbsp;(uniformity)</li>
            <li>dbpedia data set about countries - we find some names which are not a country, or are arrays, column shift, by comparing the data to ISO country codes (accuracy)</li>
            <li>example with video and screen capture of an exam - less likelihood that both are missing (completness)</li>
            <li>example with "who do i trust the most" - e.g. there are 2 different addresses for same person - have to decide which one is more reliable, e.g. how was the data collected, which one is more accurate, etc. (consistency)</li>
            <li>example of counting numbers of data types (including null) (uniformity)</li>
        </ul>

#### 3.2. Exercise

<div>The exercise is about analysing and cleaning cities data set. We count number of data types, deciding about which digit of areaLand we are more likely to use, and choosing the more accurate one, changing string array of city names to python array, checking the lat and lon locations.</div>


### 4. Working with MongoDB

<ul>
    <li>flexible schema, easy to handle hierarchical data</li>
    <li>JSON documents - convenient for programmers</li>
    <li>flexible deployment (local or cloud)</li>
</ul>

#### 4.1. Pymongo

<div>
            <ul>
                <li>is python driver for mongo - keeps connection to database</li>
                <li>minimal example<br/><br/><span
                        style="font-family: Courier New, Courier, monospace; font-size: xx-small;">from pymongo import MongoClient<br/><br/>client = MongoClient('localhost:27017')<br/>db = client[db_name]<br/>query = {"manufacturer" : "Porsche"}<br/>return db.autos.find(query)<br/><br/>return db.autos.find_one(query)</span>
                </li>
                <li>projection<br/><br/><span
                        style="font-family: Courier New, Courier, monospace; font-size: xx-small;">return db.autos.find(query, {"_id":0, "name":1})</span><br/><span
                        style="font-family: inherit;">* normally _id is included by default, unless specifies like above explicitly</span>
                </li>
                <li>insert<br/><br/><span
                        style="font-family: Courier New, Courier, monospace; font-size: xx-small;">db.autos.insert(auto)</span>
                </li>
                <li>mongoimport cmd line utility - imports whole JSON files to DB<br/><br/><span
                        style="font-family: Courier New, Courier, monospace; font-size: xx-small;">&gt;mongoimport -d examplesdb -c autos --file autos.json</span>
                </li>
                <li>operators - start from "$"</li>
                <li>$gt, $lt, $gte, $lte, $ne<br/><br/><span
                        style="font-family: Courier New, Courier, monospace; font-size: xx-small;">query = {"foundingDate" : {"$gte":datetime(2001,1,1), "$lte" : datetime(2099,12,31)}}</span>
                </li>
                <li>$exists<br/><br/><span
                        style="font-family: Courier New, Courier, monospace; font-size: xx-small;">query = {"governmentType":{"$exists" : 1}}</span>
                </li>
                <li>$regex</li>
                <li>queries work inside arrays</li>
                <li>also can work against other arrays, $in and $all</li>
                <li>$in<br/><br/><span
                        style="font-family: Courier New, Courier, monospace; font-size: xx-small;">query = {"assembly":{"$in":["Germany","United Kingdom","Japan"]}, "manufacturer":"Ford Motor Company"}</span>
                </li>
                <li>$all<br/><br/><span
                        style="font-family: 'Courier New', Courier, monospace; font-size: xx-small;">query = {"modelYears":{"$all":[1965, 1966, 1967]}}</span>
                </li>
                <li>can also access hierarchy<br/><br/><span
                        style="font-family: Courier New, Courier, monospace; font-size: xx-small;">query = {"dimensions.width":{"$gt":2.5}}</span>
                </li>
                <li>save(obj) method - insert or update (if the _id exists and such object is in db)</li>
                <li>update(obj) - for (bulk) partial updates</li>
                <li>update + $set<br/><br/><span
                        style="font-family: Courier New, Courier, monospace; font-size: xx-small;">city = db.cities.update({"name":"Munich",<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"country":"Germany"},<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; {"$set":{"isoCountryCode":"DEU"}})</span>
                </li>
                <li>update + $unset<br/><br/><span
                        style="font-family: 'Courier New', Courier, monospace; font-size: xx-small;">city = db.cities.update({"name":"Munich",</span><br
                        style="font-family: 'Courier New', Courier, monospace; font-size: x-small;"/><span
                        style="font-family: 'Courier New', Courier, monospace; font-size: xx-small;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"country":"Germany"},</span><br
                        style="font-family: 'Courier New', Courier, monospace; font-size: x-small;"/><span
                        style="font-family: 'Courier New', Courier, monospace; font-size: xx-small;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; {"$unset":{"isoCountryCode":<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"blahblah_this is ignored"}})</span>
                </li>
                <li>bulk update<br/><br/><span
                        style="font-family: 'Courier New', Courier, monospace; font-size: xx-small;">city = db.cities.update({"name":"Munich",</span><br
                        style="font-family: 'Courier New', Courier, monospace; font-size: x-small;"/><span
                        style="font-family: 'Courier New', Courier, monospace; font-size: xx-small;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"country":"Germany"},</span><br
                        style="font-family: 'Courier New', Courier, monospace; font-size: x-small;"/><span
                        style="font-family: 'Courier New', Courier, monospace; font-size: xx-small;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; {"$set":{"isoCountryCode":"DEU"}},<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; multi=True)</span>
                </li>
                <li>remove()<br/><br/><span
                        style="font-family: 'Courier New', Courier, monospace; font-size: xx-small;">city = db.cities.remove() </span><span
                        style="font-family: inherit;">- removes all</span></li>
                <li>remove(query)</li>
            </ul></div>

#### 4.2. Exercise

A lot of specific data cleaning and modifying, while copying it from CVS to mongo, row by row. It's about&nbsp;arachnid (spiders) data set.


### 5. Analysing Data


<ul>
        <li><span style="font-weight: normal;">twitter data set</span></li>
        <li><span style="font-weight: normal;">followers, followees, tweets, tweet contents</span></li>
        <li><span style="font-weight: normal;">user id is called "screen_name"</span></li>
        <li><span style="font-weight: normal;">tasks like "find who tweeted the most"</span></li>
    </ul>

#### 5.1. Aggregation framework in MongoDB

<ul>
        <li>$group and $sort<br/><br/><span
                style="font-family: Courier New, Courier, monospace; font-size: xx-small;">db = get_db('twitter')<br/>pipeline = [<br/>&nbsp; &nbsp;{"$group": {"_id":"$source", <br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"count": {"$sum": 1}}<br/>&nbsp; &nbsp;},<br/>&nbsp; &nbsp;{"$sort": {'count': -1}}]<br/><br/>result = db.tweets.aggregate(pipeline)</span>
        </li>
        <li>$skip and $limit - skip few first documents, or limit the output to number of documents only</li>
        <li>$match - for filtering, and $project<br/><br/><span
                style="font-family: Courier New, Courier, monospace; font-size: xx-small;">pipeline = [{"$match":{"user.time_zone":"Brasilia"}},<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; {"$match":{"user.statuses_count":{"$gte":100}}},<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; {"$project":{"followers":"$user.followers_count",<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"screen_name":"$user.screen_name",<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"tweets":"$user.statuses_count"}},<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; {"$sort":{"followers":-1}},<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; {"$limit":1}]</span>
        </li>
        <li>$unwind - will multiply the record for each value in an array<br/><br/><span
                style="font-family: Courier New, Courier, monospace; font-size: xx-small;">pipeline = [{"$match":{"country":"India"}},<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; {"$unwind":"$isPartOf"},<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; {"$group":{"_id":"$isPartOf", "count":{"$sum":1}}},<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; {"$sort":{"count":-1}},<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; {"$limit":1}]</span>
        </li>
        <li>$sum, $first, $last, $max, $min, $avg<br/><br/><span
                style="font-family: Courier New, Courier, monospace; font-size: xx-small;">pipeline = [{"$match":{"country":"India"}},<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; {"$unwind":"$isPartOf"},<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; {"$group":{"_id":"$isPartOf", <br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"avg":{"$avg":"$population"}}<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; },<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; {"$group":{"_id":"totalAvg",<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"avg":{"$avg":"$avg"}}<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;]</span>
        </li>
        <li>$push and $addToSet<br/><br/><span
                style="font-family: Courier New, Courier, monospace; font-size: xx-small;">pipeline = [{"$group":<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; {"_id":"$user.screen_name",<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"tweet_texts":{"$push":"$text"},<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"count":{"$sum":1}}}, <br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; {"$sort":{"count":-1}},<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; {"$limit":5}<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;]</span>
        </li>
        <li>indices - they are hierarchical; the hierarchy determines in which order an item can be searched - think about it<br/><br/><span
                style="font-family: Courier New, Courier, monospace; font-size: xx-small;">db.autos.ensureIndex({"name": 1})</span>
        </li>
        <li>geospatial indices - don't search by exact point, but "near"&nbsp;to&nbsp;points ($near)<br/><br/><span
                style="font-family: 'Courier New', Courier, monospace; font-size: xx-small;">db.autos.ensureIndex(loc_field, direction)</span>
        </li>
    </ul>

#### 5.2. Exercises

Just building different pipelines.

### 6. Case study

It's about OpenStreetMap data set, which can be downloaded in XML from their site. You can download part which you are looking at, or download data of major cities. They also have a very nice wiki.<br/><br/>The data is XML with "node"s and "way"s (way = street, road, etc). The data is human edited, so it contains errors.<br/>

#### 6.1. SAX XML parsing

<span
            style="font-family: Courier New, Courier, monospace;">import xml.etree.ElementTree as ET</span><br/><span
            style="font-family: Courier New, Courier, monospace;"><br/></span><span
            style="font-family: Courier New, Courier, monospace;">for event, item in ET.<b>iterparse</b>(xml_filename, events=("start",))</span><br/><span
            style="font-family: Courier New, Courier, monospace;">&nbsp; handle_node(elem) </span>
<br/>The non iterative parsing (reading all to memory at once) could go like:<br/><br/><span
        style="font-family: Courier New, Courier, monospace;">tree = ET.<b>parse</b>(</span><span
        style="font-family: 'Courier New', Courier, monospace;">xml_filename</span><span
        style="font-family: Courier New, Courier, monospace;">)</span><br/><span
        style="font-family: Courier New, Courier, monospace;">root = tree.getroot()</span><br/><span
        style="font-family: Courier New, Courier, monospace;">for child in root:</span><br/><span
        style="font-family: Courier New, Courier, monospace;">&nbsp; handle_node(child)</span><br/>
<ul>
    <li>node.tag - name of XML node</li>
    <li>node.attrib - dictionary of node attributes</li>
    <li>node.attrib.keys() - array of attribute names</li>
</ul>

#### 6.2 Regular expressions

<div><span style="font-family: Courier New, Courier, monospace;">import re</span></div>
<div><span style="font-family: Courier New, Courier, monospace;"><br/></span></div>
<div><span style="font-family: Courier New, Courier, monospace;">lower = re.compile(r'^([a-z]|_)*$')</span></div>
<div><span style="font-family: Courier New, Courier, monospace;">re.findall(lower, string)</span></div>
<div><span style="font-family: Courier New, Courier, monospace;"><br/></span></div>
<div><span style="font-family: Courier New, Courier, monospace;">m = lower.search(string)</span></div>
<div><span style="font-family: Courier New, Courier, monospace;">if m:</span></div>
<div><span style="font-family: Courier New, Courier, monospace;">&nbsp; substring = m.group()</span></div>

#### 6.3 Exercise

It's about parsing a XML document, and iterating over XML nodes and creating proper python dictionaries (specified in the task description).<br/><br/>Conclusions<br/>
<ul>
    <li>Reading the actual data helps a lot, makes you see misconceptions, and understand what is actually going on</li>
    <li>Read the task carefully and do not switch off thinking ;)</li>
    <li>Latitude comes before longitude</li>
</ul>
