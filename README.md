# redisinsight-demo


# Getting Started

## Install Redis Stack on your Mac

Two different ways

- Either download manually
- Using brew

```
brew tap redis-stack/redis-stack
brew install --cask redis-stack
```

## Start Redis Stack server

```
 redis-stack-server
 ```
 
 ## Test the connectivity
 
 ```
 redis-cli
 ```
 
 ## Open up RedisInsight from the application 
 
 ## Conect by providing 
 <img width="1217" alt="Screen Shot 2022-04-19 at 8 35 29 PM" src="https://user-images.githubusercontent.com/313480/164035282-c52f3dec-15bc-4031-a773-a2cc24094aa5.png">

 

## Demo -1 : Bike Shop using RedisJSON

In this tutorial, we will go through an example of a bike shop. We will show the different capabilities of Redis Stack.

## Storing and managing JSON

Let's examine the query for creating a single bike:

- Create a single bike info in JSON format

```
JSON.SET bikes:1 $ '{
    "model": "Hyperion", 
    "brand": "Velorim", 
    "price": 844, 
    "type": "Enduro bikes", 
    "specs": {
        "material": "full-carbon", 
        "weight": 8.7
    }, 
    "description": "This is a mid-travel trail slayer that is a fantastic daily driver or one bike quiver. At this price point, you get a Shimano 105 hydraulic groupset with a RS510 crank set. The wheels have had a slight upgrade for 2022, so you\'re now getting DT Swiss R470 rims with the Formula hubs. Put it all together and you get a bike that helps redefine what can be done for this price."
    }' 
```



You can use JSONPath to access any part of the stored JSON document:

Getting the specific field:

```
JSON.GET bikes:1 $.model
JSON.GET bikes:1 $.specs.material
```

```
JSON.GET bikes:1 $.specs.material

"[\"full-carbon\"]"


JSON.GET bikes:1 $.model

"[\"Vanth\"]"
```

## Update specific fields

```
JSON.SET bikes:1 $.model '"Hyperion"'
```

Now let's load a lot more bikes so we can play around with the queries:

```
JSON.SET bikes:0 $ '{"model": "Deimos", "brand": "Ergonom", "price": 4972, "type": "Enduro bikes", "specs": {"material": "alloy", "weight": 14.0}, "description": "Redesigned for the 2020 model year, this bike impressed our testers and is the best all-around trail bike we\'ve ever tested. It has a lightweight frame and all-carbon fork, with cables routed internally. It\u2019s for the rider who wants both efficiency and capability."}' 
JSON.SET bikes:1 $ '{"model": "Vanth", "brand": "Tots", "price": 4971, "type": "eBikes", "specs": {"material": "full-carbon", "weight": 13.0}, "description": "If you\'re looking for the best commuter eBike for your trip to and from the office, that will keep you rolling from home to work. The hydraulic disc brakes provide powerful and modulated braking even in wet conditions, whilst the 3x8 drivetrain offers a huge choice of gears. If you\'re after a budget option, this is one of the best bikes you could get."}' 
JSON.SET bikes:2 $ '{"model": "Kirk", "brand": "Bold bicycles", "price": 2508, "type": "Kids bikes", "specs": {"material": "alloy", "weight": 7.8}, "description": "These bikes pretty easy to ride while also being lightweight enough and quite durable. At this price point, you get a Shimano 105 hydraulic groupset with a RS510 crank set. The wheels have had a slight upgrade for 2022, so you\'re now getting DT Swiss R470 rims with the Formula hubs. Put it all together and you get a bike that helps redefine what can be done for this price."}' 
```

## Indexing and Query

Conceptually, Redis is based on the key-value database paradigm. Every piece of data is associated with a key, either directly or indirectly. If you want to retrieve data based on anything besides the key, you'll need to implement an index that leverages one of the many data types available in Redis.


In addition to storing and managing JSON, Redis Stack provides the capability to index and query it. After we specify what we want to be indexed, the indexing happens automatically (and synchronously) when new data is saved.

```
FT.CREATE idx:bikes
 ON JSON 
    PREFIX 1 "bikes:"
 SCHEMA 
   $.model AS model TEXT NOSTEM SORTABLE 
   $.brand AS brand TEXT NOSTEM SORTABLE
   $.price AS price NUMERIC SORTABLE 
   $.type AS type TAG 
   $.specs.material AS material TAG
   $.specs.weight AS weight NUMERIC SORTABLE
   $.description as description TEXT
 ```
 
 Now that we have our data indexed, it's really easy to run full-text searches on it and filter by number, tag and even geo positio
 
 
 ```
 FT.SEARCH idx:bikes "@type:{eBikes}"
 ```
 <img width="839" alt="Screen Shot 2022-04-19 at 9 01 25 PM" src="https://user-images.githubusercontent.com/313480/164040647-559bff3e-68de-425b-9e10-b65d7348945e.png">

 












## Demo -2 
