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






## Demo -2 
