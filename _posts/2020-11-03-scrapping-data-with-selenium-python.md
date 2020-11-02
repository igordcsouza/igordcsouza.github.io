---
layout: post
title: Scraping data with Selenium + Python
image: /img/selenium_python.png
---

"Why I need to scrap data?" 

This is probably the first thing you had asked yourself when you saw that title, right?

So I can give you at least two good motives: 
- Collect data that is not available in a JSON/CSV format.
- Test our application.

In this post, I'll only show you how to collect data from different sites, but the test approach is similar. 

I've decided to serve the League of Legends data in this project. Let's call it API.lol. The name is available, but it cost 100$, so I won't buy it yet. When we have a production-ready version, I will revisit this decision. :) 

Even though the [Riot Games](https://www.riotgames.com/en) have a [public API](https://developer.riotgames.com/apis) with some data, they didn't provide data from the championships. So if we want to provide this data, we need to grab it from public sites.
The best place to do that is [Lol Gamepedia](https://lol.gamepedia.com/). They didn't have all data from the matches, but they have one of the most important for us. Link's to [match history](https://lol.gamepedia.com/LEC/2020_Season/Summer_Playoffs/Match_History). 

Match history is a riot games site where you can find data about a specific [match](https://matchhistory.na.leagueoflegends.com/en/#match-details/ESPORTSTMNT06/1320904?gameHash=c24d87fe7e79e9f6). 
Ok, now we have a place to find the statistics of the matches. We need to create a script to parse the Gamepedia, looking for matches histories and for each of the links to parse the Riot site data. Don't forget the login step. It will be important in the future.

First things first, let's start to collect all the matches URLs.

# GamepediaScrapper

<code> 
class GamepediaScrapper:
    
    def __init__(self):
        driver = webdriver.Firefox()
        driver.get('https://lol.gamepedia.com/LEC/2020_Season/Summer_Playoffs/Match_History')
        i = 0
        linhas = driver.find_elements_by_xpath(
            "//div[@class='wide-content-scroll']//table[@class='wikitable hoverable-multirows mhgame sortable plainlinks jquery-tablesorter']//tbody//tr")
        matches = {}
        
        for linha in linhas:
            matches[i] = {}
            coluna = linha.find_elements_by_tag_name("td")
            matches[i]["matchHistory"] = coluna[12].find_element_by_tag_name("a").get_attribute('href')
            i = i + 1
        
        driver.close()
        self.matches = matches

    def getMatches(self):
        return self.matches
</code>

I'll ask sorry in advance for this and all other codes that I'll present here :) 
Ignoring my perfect code, let's talk about some code snippets.

`driver = webdriver.Firefox()` = Here we are instantiating the webdriver using the Firefox. 
`driver.get(URL)` = This command will open Firefox on the URL given. 

Selenium has a few functions that we can use to [locate a specific element](https://selenium-python.readthedocs.io/locating-elements.html) of the site.

We are using the `find_elements_by_xpath` that gives us the ability to find a sequence of class/ids/tags.

Almost all the functions have the `elements` and `element` version. If you use the `element`, the function will return only the first match.

To avoid that, you end up with tons of browser windows, do a `driver.close()` on the function's end. 

The hardest part of Selenium is to figure out how to create the best expression to find what you need. 


# MatchHistoryScrapper

Now that we already have all the link's it's time to collect the data. To not spend 1000 lines explaining each line, I'll focus on getting each match's duration. 

<code>
class MatchHistoryScrapper:
    
    def __init__(self,url=""):
        self.driver = webdriver.Firefox()
        self.driver.get(url)
        time.sleep(5)
        self.__login()
        time.sleep(10)
        self.__set_time_of_match()
        self.driver.close()

    def __login(self):
        elem = self.driver.find_element_by_name("username")
        elem.clear()
        elem.send_keys("")
        elem_password = self.driver.find_element_by_name("password")
        elem_password.clear()
        elem_password.send_keys("")
        elem_password.send_keys(Keys.RETURN)

    def __set_time_of_match(self):
        timeOfGame = self.driver.find_element_by_xpath("//span[@class='map-header-duration']//div").text
        self.time = timeOfGame

    def get_time(self):
        return self.time
</code>

I've had split into four functions. 
  * \__init__
  * __login
  * __set_time_of_match
  * get_time

When we use `mh = MatchHistoryScrapper()`, we need to pass the URL as a parameter, so the init function will call others functions in the right order using the parameter. 

The first step is to open the browser using the URL passed, but since we didn't have a valid session, the site will redirect us to the login page. 
With that in mind, we create the __login function that will `find_element_by_name` called username, clean whatever that is present on the field, and add the value that we pass on the `elem.send_keys("")`. In that case, I had given my username on the riot account. 
The same thing will follow for the password field, followed by the "return"/"submit" command.

Now we have a valid session, and we can get the data that we want.
To do so, we will use the `find_element_by_xpath` again. 

I won't go through the whole process of getting all the data from the page. Otherwise, we will spend months only on that part. But the result of this will be a folder with all the data in a JSON format. 


If you want to print all the times on console you could do something like this:

```
gs = GamepediaScrapper()
matches = gs.getMatches()
for key in matches:
    time.sleep(1)
    print("-----")
    match = MatchHistoryScrapper(matches[key]["matchHistory"])
    print(match.get_time())
```



---
Can i improve something?
Maybe explain with more details?
Please, let me know! :) 


---
Links:
  * https://selenium-python.readthedocs.io/
  * https://docs.python.org/3/tutorial/classes.html




  ---