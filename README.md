# Demonstrating RedisInsight Features

- 1. User database
- 2. Bike Shop
- 3. A Real-time Sensor Analytics


## 1. User Database


Let us import a user database (6k keys). This dataset contains users stored as Redis Hashes.


### 
**Users**

The user hashes contain the following fields:



*   `user:id` : The key of the hash.
*   `first_name` : First Name.
*   `last_name` : Last name.
*   `email` : email address.
*   `gender` : Gender (male/female).
*   `ip_address` : IP address.
*   `country` : Country Name.
*   `country_code` : Country Code.
*   `city` : City of the user.
*   `longitude` : Longitude of the user.
*   `latitude` : Latitude of the user.
*   `last_login` : Epoch time of the last login.

### Step 1: Cloning the repository

Open up the CLI terminal and run the following command:

 ```bash
  git clone https://github.com/redis-developer/redis-datasets
  cd redis-datasets/user-database
 ```


### Step 2. Importing the user database:


 ```bash
  redis-cli -h localhost -p 6379 < ./import_users.redis
 ```


Refresh the keys view by clicking as shown below:

![image](https://user-images.githubusercontent.com/313480/164141331-5c990773-c18b-4c6c-ae58-60f99d812791.png)




You can get a real-time view of the data in your Redis database as shown below:



Select any key in the keys view and the key's value gets displayed in the right hand side that includes Fields and values.




### Step 3. Modifying a key

![image](https://user-images.githubusercontent.com/313480/164141367-c57b5df1-7949-4ea1-be99-d9a8677dfa64.png)




Enter key name, field and value.





### Step 4: Using CLI

RedisInsight CLI lets you run commands against a Redis server. You don’t need to remember the syntax - the integrated help shows you all the arguments and validates your command as you type.


![image](https://user-images.githubusercontent.com/313480/164141411-7cdcdc4e-2a6a-4a61-8de7-eb43d6d228d0.png)



 

## Demonstrating Bike Shop using RedisJSON

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
 
 ## Search by Type
 
 ```
 FT.SEARCH idx:bikes "@type:{eBikes}"
 ```
 <img width="839" alt="Screen Shot 2022-04-19 at 9 01 25 PM" src="https://user-images.githubusercontent.com/313480/164040647-559bff3e-68de-425b-9e10-b65d7348945e.png">

 

## Search by Category and Price


```
FT.SEARCH idx:bikes "@type:{eBikes} @price:[200 1200]" 
    RETURN 9 $.model AS model $.type AS type $.price AS price 
```

<img width="844" alt="Screen Shot 2022-04-19 at 9 04 02 PM" src="https://user-images.githubusercontent.com/313480/164041208-af7f0731-0fb8-41b6-a1a4-0f401036d62c.png">

# Even more search parameters

```
FT.SEARCH idx:bikes "mudguards @type:{Mountain bikes} @price:[1000 3000]" 
    RETURN 12 $.model AS model $.type AS type $.price AS price $.description AS description
```
<img width="847" alt="Screen Shot 2022-04-19 at 9 04 58 PM" src="https://user-images.githubusercontent.com/313480/164041385-17eb08ab-eacd-4b17-8782-2c491622501e.png">


 
 ## Agregate Search
 
 
 ## Bike Count Per Category
 
 ```
 FT.AGGREGATE idx:bikes "@price:[1000 3000]" 
    GROUPBY 1 @type 
        REDUCE COUNT 0 AS count 
    SORTBY 2 @count "DESC"
 ```
 
 
<img width="836" alt="Screen Shot 2022-04-19 at 9 06 23 PM" src="https://user-images.githubusercontent.com/313480/164041672-c4b0dae1-98ab-4f51-96a9-b0acad4148ea.png">


## Average Bike Weigth per Category

```
T.AGGREGATE idx:bikes "*" 
    GROUPBY 1 @type 
        REDUCE AVG 1 @weight AS average_bike_weight 
    SORTBY 2 @average_bike_weight "DESC"
 ```
 <img width="837" alt="Screen Shot 2022-04-19 at 9 07 17 PM" src="https://user-images.githubusercontent.com/313480/164041839-be157191-0f2d-4ec9-b654-7f14734adaa1.png">


## Bike Per Weight Range


```
FT.AGGREGATE idx:bikes "*"
    APPLY "floor(@weight)" AS weight
    GROUPBY 1 @weight
      REDUCE count 0 AS count
    SORTBY 1 @weight
```
<img width="829" alt="Screen Shot 2022-04-19 at 9 08 16 PM" src="https://user-images.githubusercontent.com/313480/164042028-789595f2-67f4-47d1-b2f9-98268bd17bb0.png">




 
 
 











## Demo- II


## VECTOR SIMILARITY SEARCH

Redis Stack supports Vector Similarity Search - a new feature that is going to help to search unstructured data, like text and images. Currently, it's only supported for hashes, but JSON support is coming soon.

To be able to run these queries you need to have a vector representation of your text, or image, also called vector embeddings. They are done by a machine learning model and stored in Redis. Let's look at an example query:

```
HSET bikes:10000  
    model 'Varuna' 
    brand 'Eva' 
    price 1398 
    type 'Road bikes' 
    material 'aluminium' 
    weight 7.5 
    description 'This bike delivers a lot of bike for the money. The Shimano Claris 8-speed groupset gives plenty of gear range to tackle hills and there’s room for mudguards and a rack too.  That said, we feel this bike is a fantastic option for the rider seeking the versatility that this highly adjustable bike provides.' 
    description_embeddings "\xe8\x86\r<\xd6\x98\xcb<\xe8\x7f\x02\xbc\x13\xbf$=!\xa6\xd093\xab\x95<l\x7f\x01\xbd{\x7f\xd0\xbcE\x02\x84=\x9fD\xc1\xbcD\x9c#\xbc\xe8\x08\xe4\xbc\xa6g7=fg:<j\xfdJ=\xe5\xc5x\xbd=I9<\xf9\x0b\xe4\xbc\x01\xd6\x1c\xbc\xed\x15\x95\xbdZT\x1a\xbcY\xd8*\xbd\x1e)\x18=\xf0a\xea\xbc%=\x9a=\xa3\xb5E<I\x99(;\xc5\xee\x1b\xbci\xcb/\xbd\x99\xbf\x8f\xbc\x16\xb11\xbd\x98\xe0X\xbcW\xb3\x03\xbd\xf72\x93;?\x93&=\xd8\xf5\xc3;\xcf\x9fv\xbc\xe1\x87\x84\xbc\xf8\x07+\xbd\xaa.\x98\xbb\x80\n\x0f=\x19/\x82\xbc\x8d\xf5\x10<\x9ec\xb0<\xec8l\xba\xac\xb3O\xbcl\xb9\x95=?C\xb2<\xd3u\xa8\xbc\xe6\xf6\xe6<\xad\x8bj\xbce\xb7\xee\xbc`\x88\x00\xbd\xc2\xb8g<@\xcah<\xea\\-\xbc\xc1\xae\xd9;\xe0\xca\xc1\xbd\x1c\xa5\x11\xbdSm9=\xb5\x0f\xca<\xef\xa9\x01;K\xda\"<\xd8\x82\x86=z\xdc\xc6<nl\x04=l\xfdq\xbc\xcf:\xb1<:zL<\x99\x8b\t;\x80\xcc\xb9\xbc\"\xc3\x16\xbd\xe7\xad\xb8\xbc\xe6\xda\x15<|\xb3\xa0\xbc\x1f\xed\xda<\x8f(\xef\xbb>\xb1\x86\xbc4\x9f\x9a\xbd\x0c\xdd\x96\xbcl\x18\x97<N7\xa7\xbc6{\x05=\xd3\x8c\xa2\xbd\xa8H\x1a<\xa7\xe4\xc8<fh\x16<\xe6{?\xbc\"\x88\xcb<\xdf\".<\xcc\xb7\xd7\xbbL|\xc8\xbbT\x03\x98\xbb\xae\xc7\xac\xbaS\x99%\xbb\x91)\xf2\xba\x9d*\x93\xbc4\xfe\x1d\xbd\xb4\x85=<\xee(\xa7\xbc\x90\xdc\xbd<]\xbdS=\x7f\x10L\xbcw#Y\xbd\xa8\xbb\xfe\xbc\x84\x9d\x06>\x91\x1e\x86;\xe8t\x84<\x84u\x86<\xf1`\x8d<\xc2\xf4A\xbcBk\x86=\xd2i\x9e<\x9b\x9b\xa7\xbc:\xb8\xc1\xbc\xec\xb6\x83\xbc\x91xH\xbd\xf6\x94\x9e\xbc%[\xcd<\xe3\xcd\xc7<\xdd\xad!;E\xe6\x9b<\xc9[6=\xd2\xdaz\xbd\x10\xe1\x7f=\xee\x95F\xbc\x97\x8a\x8f\xbb\x0c\xb7c\xbd\xf3;\xe8\xbc\xa7um\xbd5\x9c}\xbb\xcf\xeb\x97\xbb\x05\xac\xdf<\x8d\x14\x19\xbd\xd2L\xd0<\xad\\\xff<\xda\x1e#\xbb\x8a\xec\xc7\xbcF!&=\x90\xe9\x95\xbc\x9d\xbc,\xbdD,0\xbd\xb1\x81\x01=\xd5^\x86<\xdf\xd0\x19\xbd \xa0\x02\xbdX\xc2\x12=-qh<B\xdc\x10\xbd\xbeL\x19\xbd\xa3\xef\x89<\xa8\x1eY=1\xd0\xa2\xbc\x85\x166=\xb0\\\xfb\xbc\xe6Z\xfa\xbb\x91\xf5I\xbd18\xb1\xbc?\x10\xd4\xbc\xafK}\xbc\x03\xc8\x92=\x9b\xfd\xa3\xbb\xe8\xb9\xfe9k\x13x\xbd\x0e/\xe7<\xc2_\x9f<\x90P\x83<\xed\xbf\x98=r7\x1f=k\xf1\xac;\xa6\x82\xca;\xfc\xa9\x1f=N\xc1\x96\xbb\xda\x9e/\xbd\xec\xda\x81:ab.\xbc\x81\xa0\x81\xbd\x88\xea==\xcc\x82\x10<\x9c\x06\x1c\xbd\x9e\xf3Y=\x08\xd5\xd9<\xf3\xfe\"\xbdF]D=iF\xc0:_KS\xbd\xe1\xd6J=\x96\x18\xf1\xbc\x14\xa4\xe0<\x08\x15a\xbdQ\xf5\x0c\xbd/\xb5\xa1\xbc\xe1\xf4\x02\xbd*\xdf\xdb\xbc\xce\x02~\xbc\xd5Vf=\x90\xd1p;\xee\x8c[\xbdx\xa6\x96<\xa8\xee\xa3=\x9f.\x02\xbd\x87tF\xbd\xaa5\xfd\xbd >\xde\xba\xf77\xce\xbc\xba\x05\x06=\x14_\xa2\xbb0\xa6\x12\xbcpUq=\xe0\xdf\x1a\xbd\x8e\x15\xa2\xbcoK+\xbc<\xc6\xdc<\xfc\x8d\xe5\xbc\xdbr\xbe:M\xba\x0c\xbdY\x97\x04=\x05\x94\xfb;\xaa\xe0\x1f=\xb6%\x06\xbc\xa5i\xa9<\xc2u(:\x87\x7f-\xbd\xa975<\xd9\x17v\xbd1\xef\xd8\xbc\xdb\xe7\x16<\xec\x97/;\xac\xcb\x03=\xfdl\x9d\xbd\x16\xcb\x0f\xbd\xa37a\xbd\nU\xd2<]\x92\x83=\xa2\xc7\x96:\xf4WD=\xf8\xcd0=\xb4\x81(=\xc2\n\xf7\xbc\xb1zC<\x8bN\xcd\xbbg\x05\t\xbb\xc2\x05\x1b=~\x12\xd3\xba^\xbe\xcd\xbbS!t=\x17\xe6\x03=\xc7-1\xbci\x9b\x84\xbc\xd4bD<\xf7*\x13=\x1bV\x9b\xbd\xfdlP\xb7\x1d\x88\x94=$\xd8\x0b\xbd\xae \x98=\x18\x161\xbc\xf1\xab\xd2\xbcAmz<\xd8\xfa\x7f=*$%<m\xf4L\xbc\xee\xd8\\\xbd\xae\xd0\x14\xbd!\x0b\x90\xbc(\xb7\x07>\xba\x8b6\xbdD\xa2\x1c\xbd\xa3%\x85=\xf9\x86\xbe\xbcH\xec\xf1;\x96\xb5\xaa<\xdccI\xbd\x16\xfa@=\xac;\xed\xbc\xf9L\xa6\xbb*\xc8K\xbd\xeb5\xe9\xbc\xa8\xe87=\xb6\x02y\xbd\xe3\x0f\xae\xbc\x01\x03\r=\x14pO\xbd\x94\x1e\xbb<\x19\x02L=\xdaQ\x05;\\\xa9D\xbb\x0e\xca\x9d<O\xc7{\xbd\xdcJ\xe3\xbc?f-\xbc[\xd3)\xbd\xb8$\xcf=\x98d\xe5<4\x96R<\xc7\xcb\xe3\xbcboA<cC\xc1<\xf2#I=<\x0e\x16\xbd\xe6KR=\x10\x93>\xbcF\x19X=\xc1\xc8\xec:4\xd9\xb4<\xa5\x0e\xe3<&^\x98\xbb\xd4\x97D<\xf2\xfd\x83=\x0c\x10\x13\xbc\xf5[[\xbd\xbex\xe7;\x9e#4\xbd.\xe7\x80\xbc\x87\xca\x91<\xaed\xf8\xbc\xb1\x0e\xda\xbc\x89\xc6\xb2=W\x82p<\xcb\xaf\x9d\xbc\xbd\x058\xbd;\xa4g\xbdY(l<cb\x18\xbd\xfb\x10\x06<D\x8a,\xbd\x89t\x00\xbd\xa6\xef\xda\xbd\x84\xf5v=9\xae\xdc\xbc\xccP\xa8=\xe0C\xb8\xbc\xe9\x16[\xbd\x93\xcd\x7f;\xb9!\xa1;\x95\x01\xd5\xbcksK\xbdp-\xdd<;\xa1\xbf<\xdbb\x0b\xbc\x9a\x07\x0c\xbc\xb8\x1c\x1e\xbd\xe8o\xc2=<\x0f\x8d\xbb\x98\xe1J;6\xfe\x9b<\xdf\xcf\xc4<0\x0e\xfc<\xd9Z$=b\xc9<\xbdmU]\xbd\xbe\xd7]\xbci\xd6\r=\xa5S\xfa\xbc!\xf1\xd8\xbc\x95\xf1C\xbd?\xaa\x8e\xbbD*\x14\xba\x04%>\xbd?Q\xb1\xbdM\xd6#\xbd\xcc\xd2T\xbb\xbc\x136\xbd\xb9\xa3A=\x04\xf3\xe0<\xc8K%=\x88\xf1\x90;L\x15\x8b=\x80\xd62\xbd\x08\xb3J\xbd\x18\xd8a<@\xd0\xca:Y8\x7f<3\x14\x90\xbd\x9d\x03\xd6<\xd4U}<\xf8Br\xbc\x9cd\x95\xbc\xbdO\"\xbd\xff\r\xb1<I\x16\"\xbd\xf6\x02b\xbd(\xabD\xbb \x0c\x9a\xbc\xcdb\xa5\xbd\xc7\"n\xbc\x1c\xf1\xd1\xbc@\xc0Q\xbc\x86\xac\x10=\xaa\xa6\xa9\xbc%8\x87\xbd\xaf\x1a$=S\x92\x9f=\xc2\x19\xcd\xbc\xa8iU;4\x85\xc2\xbb\x01e\xc7\xbd\xa3\xeb\xad<\xad\x99b=`\x14\xad\xbd\x97\xfa\xb5<4\x82\xac\xbc\"\x7f\x91<\xbfn[=*e\x02<6\x82\x8c\xbc\xba\xbf#=\xb8\x8d\xce;\x03\xd4D\xbc| \x9f<V\xf8.=\\\x19\x84\xba\x829,=M\x99\xaa;\x08\xc4\xe3<b\xc4\x1b=\xe2\xb9\x07\xbd\x19%\x1e\xbd\x9aw#=\xfb\x89Y<E\xb0\xf7\xba\xc3\xab\x15\xbb\x0e\xcff\xbc\r:U=\xaal\x9a\xbc\x86\xf9\x82;\x12\xd1\xc2;\xd2\"\x85=\x05VN\xbc{(\x05\xbd\xa0\xdeY\xba\xb5\xf1\x9b\xbc\xe2|x<\xf3\xaf\xb7:#)\xdd\xbc\"\xa1I=\xb2SE=\xb5\xc0+=)\xc6\x8e\xbc\xd7\xef\x1c=\xa6\x8d\x01<ni\x19\xbcBl\xd5;xi\x03\xbb\xb2\xbc\x05\xbc\xcc\xac\xc8<\xda4\x9d\xbd\x81\xe7\xe6\xbc\xa9$\xe4<\xda\xedl<\xe2\n\x1a\xbdF\x00\x97\xbd\xec\xcbG\xbb\xed<\xfa\xbc\xee\xe0\xa2;e\xd1\xac\xbc\xe9T\x1d<Yb6:\xa8\xfa\x1e\xbd/=\x92=\x01\x12\x94=\xfd\xab\xda\xbb\x84\xbf4=\x8e/\xef\xba\xaaA\xe7;VU#=.Z\xf0\xbb\xce\xb3\x02\xbd{\x06n\xbc\xcc\x1e&\xbb\xb6Mf;a\xf5\xdb<1W\xe5\xbc\xb0\x84\x1a\xbcMoK<\xff*J=7a\x0c\xbd\x13\xbc\xfe\xbb U\xb5<%\x00\xfe;( X<]\xb5\xba<V\x1c\xda\xbc@d\n=F\xed\xb8<\x1e\xc2\xe4\xbd\xd7\xf6\x07:?\xc5)\xbcr\x13\xd6\xbcE\xa5\x12=\x0c\xdd\x87<\xbf*\x91:\xa3\xf8\x86=\xe6w\x83<\x9c\nV\tWH\x869\xd4f\xdb\xbcY\x17\x13=\xecT\x8f<d\xb7\xa79QX\xaf<\xa7\t\x06\xbdP\xcf\xd7\xbcR]9;\x13\xb1\x88\xbc\x1a\x95\xf4;\x8e\xa7\xba;\xc7\x1bV<\\\x02b=\xb9,\x1a\xbdY\x15\xf7<\xeb\xda\x0c\xbc\x16.\x81<%`_\xbcnQ+=\xa6\r\xc5\xb9\x98\x0b\x9a\xbc\xc5i]<\x05\xafL\xbd/\x93j<\xab\x9a\xe6:\x9f\xce\x8d<mp\x99<\x89[\xde\xbc\xe3\xc5Y<\x98\xb3\xfb;\xee\xaba\xbd\x1c\xd8J\xbd):!<\"\xbb\x05\xbd\"\xbf\xf6;3]];_B\xe8<\x9eC/=\x95_\x13=X\xb8H<\xf5\"\x01=4{\x81\xbc`Bc<\xbc\xeb.=\x18\xaf\x12;\"X\xd5<\xb2&;\xbc\x05l\xb0;$\x96b\xbc\x90\x08^\xbd\x13\xf3\xc2\xbc{\xe2\x9e<=0q<Y\xff\xe7<\xe8\xc9b<KV\xac<|K\xe6<17\x169@\xf9\xd1<\x13\xfe\xff<\xf0\xd1Y=\x9cQ\x9b<\xcf\x9eC=y\x9dx\xbd\xe0\x89\xe5\xbc\xed\xacA\xbd\xaf\xbe\xf8:<Vh=\xef\t\x889Gun<\x89\xb7\xa8\xbcu\xe5g<\xe7\xd1\xe8<\xfc\x94\xc7<\xa0\x05\x08=6Ul\xba\xd7\xf7\x98=\x83 6\xbd\xc8\xd0&=\xc4\xfd{<\xf7!\xcd<\x0f\x94\x90\xbbA\xe4\r=\xd4k\xe8\xbc\xc7\xa53<yA==1\xff\x99=tc4\xbb\x1d\xb0+<\x8d\x96\xa5\xbc\x85C\x7f\xbcC\x8e\xd0<\x96L\xbd8I\xa5\xc4<\x14\xa7\xc6<\x11\x95B\xbd\x17\xd7\x16<\x03\x05\x06<\xad+\xf9<G\xac\x1b\xbd\xb4T>\xbdLL\xa5<\xec\xc2+\xbd:\xda\x1f=\x95\x00\xef8\x0c\x9aH=;D2;\x02\x06\x1d\xbdw\x83\xb8<\xbf\xd7Q<\xe3\x81\x07\xbb?}\xac\xbc\xd4F\xbb=\xdfY\xbb<\xa0\x02\xd8;\x97\x95d=\xb5\xc3\x12=A\xdd\xb8\xbc\xbc\x99\x97\xbd\x0c\xb0\x81<\xb0\xe9f\xbc\xbd\xdcR=Q\xc5l<\xa5\xf7\xc3\xbcn\xe8C\xbc\xdf&]\xbd\xb6\x1b5<\xc4\xad\xc5=\"\xe4\xb9\xbd\xe9\xbdt9VS\x92\xbc\xf90\xa4=uG\x81\xbc\xf4\x83h=>M\x16<eb\xd0<\xb1r2\xbd\xc1q\xce<\xe7\xe0D\xbb\x12B\xb9\xbcH\xcd\x85\xbd\xef\x96\x86\xbd\"\xbf\x12;\x8f\xc7\x06\xbc\x05\xf1\xf5:MR-\xbdZB\x86::u\x0e\xbb\xe6\xaf\x99\xbc\x96\xac\xef:[\"*;Gs+\xbc\xb6\x8aC<|\x98!=\xab\xb5\t=n\xcd\x1c\xbd\x0c\x9b6=\xa79\xb4<\xc8\xd4\x06\xbd\xa8u\x80\xbd\xe7\xe6r=\x13\xb1e<4-\x02<&\xc6S\xbc\xf3E\xac\xbc\x028O\xbdU\x17\xb7\xba\xec\xd7\x96=e\x1a\xad<\xc8\x88%=\xed\xb9 \xbc]\x8f\x0e=\xc6\xd5\x89\xbc\xa7\xa7e\xbcFPi=\xa6R\\\xbc\xf0|\xad<\x93n\xbf<\xab3\x02\xbdVIP<\xca\xa3\x9e\xbd\xebd0=G\xb8\xb8\xbc\xae<\xc2<\xd5aH\xbd\x1c\xc45\xba\xc1\xea\xd4\xbc\x0b\xf3\xb6=\x84\x9c\x84\xbc\x18\xdf$<\x9d\xb8\xac<\x1d\xa5\xd7\xbc\x00\x18\xb5\xbc?\xd6B\xbc\x1em\x12;\xb6\x0b!\xbd \x1ef\xbb\xcc\xf3\x07<\xca\x81\xe0;v\x88\xa7\xbc\x8a\x1c\x19=<\x0c>\xbc\xf0\xc0j\xbd\xc2\xdfH\xbd\xf4\x08}\xbd3lC=\x05(\x1f\xbdb\xfc\xbd;=O\x1c\xbd1[\xad<\xcf\xdc\x84\xbc\x10\"\xfe<\xf9c\xde\xbcZ\x8f\x0b\xbdb\xc8:\xbdN\xb1\"=\xd8\x84\xd0\xbc\xac\x0e5\xbd\xa7\xacL\xbdRr}<*8*\xbdx9Y=\xd0\x87\x9b=\x97\x87\xa1=\xf2D\xd6\xbc U1<\\\x9cf;7\x91\x89\xbb\x9b\xd7D=c\x08\xa3<\x9d\x16\x01\xba\\M\xa1\xbc\xcf\x05\xae\xbc\xd6UK\xbd\xb2\xb8\x95;\x8a]\xa2;\x9c-\x0b\xbd\xf1|8=7\xb5\xb9<M\x14\x8a<\x12\xaa#=a\xae\xb3<7Tq\xbb\xa6\xf5\x0b\xbdh@\x85<\\h\xda\xbc{(/<\x82Bw\xbd4\xb5(\xbc\xde\xdf\xd8\xbc\xdc\xeeE\xbd\xb2\xa0\xe1\xbb\xfe\xc2\xf4<]\xa2\xfb\xbc\xc2\xbe$\xb7\x12\x1a\xc4\xbd(\xf2\xd3<B^V\xbd\xd4,}\xbdD\x8d\x8b;i\x15F<\\\x18^<\xae#\xa5<\x89v]=_\x8b7\xbbxB6<,W\r=\x0f\x93\x82\xbd\xf4}\x13;\xe3\xc1\x0c\xbdYD\xc2<t\x8d1\xbc"
```
    
 # Demo 3 - Sensor Analytics
 
Loading BME680 Sensor data directly into RedisTimeSeries(over Redis Enterprise Cloud)


### Pre-requisite

- Jetson Nano 4GB/2GB Model/Raspberry Pi/Arduino(76.9.187.74, picoXXX)
- Redis Enterprise Cloud Account configured with Subscription and Account
   - Endpoint Required
   - Password
   - Port
- Install Python3 on your Jetson board system
   
   
### Install Required Modules

Pleaee follow the below steps:

1. Install bme680 python module


```
$ pip3 install bme680
Password:
Collecting bme680
  Downloading bme680-1.0.5-py3-none-any.whl (11 kB)
Installing collected packages: bme680
Successfully installed bme680-1.0.5
```

2. Install smbus python module 

```
 pip3 install smbus
```

## Setting up Jetson Nano with BME680


## Cloning the Repository

```
 git clone https://github.com/redis-developer/redis-datasets
 cd redis-datasets/redistimeseries/realtime-sensor-jetson
```


## Running the script

- Change the entries as per your Redis Enterprise Cloud account 

```
 python3 sensorloader.py --host <endpoint> --port <port>  --password <password> 
```

- For Example

```
ajeetraina@Ajeets-MacBook-Pro ~ % sudo python3 sensorloader.py --host redis-10415.c212.ap-south-1-1.ec2.cloud.redislabs.com --port 10415 --password XXXX
redis-12929.c212.ap-south-1-1.ec2.cloud.redislabs.com:12929> auth <password>
OK
redis-12929.c212.ap-south-1-1.ec2.cloud.redislabs.com:12929> monitor
OK
1611046300.446452 [0 122.179.79.106:53715] "info" "server"
1611046300.450452 [0 122.179.79.106:53717] "info" "stats"
1611046300.450452 [0 122.179.79.106:53716] "info" "clients"
1611046300.486452 [0 122.179.79.106:53714] "info" "memory"
1611046300.486452 [0 122.179.79.106:53713] "info" "server"
1611046300.494452 [0 122.179.79.106:53715] "info" "memory"
1611046300.498452 [0 122.179.79.106:53717] "info" "commandstats"
1611046300.522452 [0 122.179.79.106:53716] "dbsize"
1611046301.498452 [0 122.179.79.106:53714] "info" "memory"
1611046301.498452 [0 122.179.79.106:53713] "info" "server"
1611046301.498452 [0 122.179.79.106:53715] "info" "server"
1611046301.498452 [0 122.179.79.106:53716] "info" "clients"
1611046301.498452 [0 122.179.79.106:53717] "info" "stats"
1611046301.554452 [0 122.179.79.106:53714] "info" "memory"
1611046301.562452 [0 122.179.79.106:53717] "info" "commandstats"
1611046301.566452 [0 122.179.79.106:53716] "dbsize"
1611046301.598452 [0 70.167.220.160:43390] "MULTI"
1611046301.598452 [0 70.167.220.160:43390] "ts.add" "ts:temperature" "1611046301456" "29.26"
1611046301.598452 [0 70.167.220.160:43390] "ts.add" "ts:pressure" "1611046301456" "952.03"
1611046301.598452 [0 70.167.220.160:43390] "ts.add" "ts:humidity" "1611046301456" "13.804"
1611046301.598452 [0 70.167.220.160:43390] "EXEC"
1611046302.458452 [0 122.179.79.106:53715] "info" "server"
1611046302.462452 [0 122.179.79.106:53714] "info" "memory"
1611046302.462452 [0 122.179.79.106:53716] "info" "server"
1611046302.462452 [0 122.179.79.106:53713] "info" "stats"
1611046302.462452 [0 122.179.79.106:53717] "info" "clients"
```

## Plotting it over Grafana


It will be exciting to plot the sensor data over Grafana. To implement this, run the below command either on your laptop or on your preferable IoT device:=


```
$ docker run -d -e  "GF_INSTALL_PLUGINS=redis-datasource" -p 3000:3000 grafana/grafana
```


Ensure that Grafana container is up and running as shown below:

```
$ docker ps
CONTAINER ID   IMAGE             COMMAND     CREATED         STATUS         PORTS                    NAMES
38148d69f114   grafana/grafana   "/run.sh"   5 seconds ago   Up 4 seconds   0.0.0.0:3000->3000/tcp   reverent_feynman
```


