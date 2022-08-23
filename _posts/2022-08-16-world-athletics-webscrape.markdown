---
layout: "post"
subtitle: ""
background: '/img/posts/world-athletics-webscrape/alessandro-venturi-XW_2N0Z9jmo-unsplash.jpg'
---

### Motivation
Athletics is a sport where only the numbers matter; however, as a fan I've noticed that any analysis of athletes is shallow. The most I have seen is who has run the fastest time, which doesn't require any analysis only observation. I wanted to see what more could be said about the world's greatest distance runners. Therefore, I ventured to [World Athletics][1] to collect the data necessary for my future analysis.

[1]: https://worldathletics.org/ "World Athletics Home Page"

#### Managing dependencies
Previously, in my experience with virtual environments, I had conflicts between the versions of my dependencies, so I spared myself the frustration and used miniconda to manage things.
![Img of miniconda shell]({{site.url}}\img\posts\world-athletics-webscrape\miniconda-pt.png){:.img-fluid}
<span class="caption text-muted">Miniconda's Conda forge channel manages installations</span>

#### Website
I collected data on the season's best over each event they ran of each athlete. This data was only readily accessible from each athlete's individual page. Originally, I was [loading each page with selenium and parsing the html using css selectors][2], but parsing with selenium is slow. I learned that Beautiful soup could parse faster, so it cut the runtime down to around an hour. However, I learned the best way to go about getting this data in a consistent and robust manner was to access the hidden api.

[2]: https://github.com/treydur/world-athletics-scrape "World Athletics Scrape v1"

![World Athletics Ranking Page]({{site.url}}\img\posts\world-athletics-webscrape\website-pt.png){:.img-fluid}
<span class="caption text-muted">Top 1000 1500m men as of 2022-08-16</span>

### Hidden API
From the google developer tab I found their graphql and imported the cURL into insomnia to see how I could request the data from the server.<br>
![Insomnia GET request]({{site.url}}\img\posts\world-athletics-webscrape\insomnia-pt.png){:.img-fluid}
<span class="caption text-muted">Insomnia GUI</span>

### Extracting Data
#### Pulling from HTML
Unfortunately, the request returned an html file, but thankfully the json data I wanted was located in a script tag with a class. After pulling the data from the script tag I was finally able to access all the data about the athletes I wanted.
```python
def main():    
    for url in get_all_athlete_links(NUM_PAGES):
        response = requests.request("GET", url, data=payload, headers=headers)
        htmlStr = response.text
        try:
            jsondata = html_to_json(htmlStr=htmlStr)
        except:
            continue
        queryData(jsondata=jsondata)     

main()
```
#### Pulling from json
Although the method to get the json data was more complex than simply loading the page and parsing, it is much faster and makes accessing the data easier. Of course, extracting from json is much easier because it is like a python dictionary object.
<pre>
discipline = jsondata['props']['pageProps']['competitor']['progressionOfSeasonsBests'][0]['discipline']
date = jsondata['props']['pageProps']['competitor']['progressionOfSeasonsBests'][0]['results'][0]['date']
numericResult = jsondata['props']['pageProps']['competitor']['progressionOfSeasonsBests'][0]['results'][0]['numericResult']
</pre>


*If you would like to see the full code, please check the Google collab link on my resume.*

### Output
<!-- ![Img of miniconda shell]({{site.url}}\img\posts\world-athletics-webscrape\csv_results-pt.png){:.img-fluid}
<span class="caption text-muted">CSV file of data</span> -->
<pre>
firstName,lastName,countryName,birthDate,isIndoor,disciplineCode,discipline,date,numericResult,mark,venue,resultScore,aaId
Ylies,MIHOUBI,France,24 OCT 2003,False,800,800 Metres,04 JUN 2022,109.09,1:49.09,Troyes (FRA),1052,14902264
Ylies,MIHOUBI,France,24 OCT 2003,False,1000,1000 Metres,22 AUG 2020,144.66,2:24.66,"Stade de Vernonnet, Vernon (FRA)",967,14902264
</pre>

### Remarks
I collected 21,630 races from 1000 different athletes in 18 minutes. I am satisfied with the method of collection and the amount of data obtained. However, I am concerned that I had to request 1000 different links to obtain this data. I didn't use selenium to load all the athlete pages as I did before (slow!), but I am mindful and would rather not send that many requests to a server regularly. In the future I hope to learn how to rotate proxies and make requests asynchronously.