---
title: "Restful APIs for CRUD of Restaurant"
layout: post
date: 2018-05-15
tag:

- python
- flask-restful
- sqlalchemy
- mysql

image:
headerImage: false
projects: true
hidden: true # don't count this post in blog pagination
description: "restful apis for CRUD of restanrants"
jemoji: '<img class="emoji" title=":python:" alt=":python:" src="/assets/images/language_icon/python.png" height="20" width="20" align="absmiddle">'
author: Jiaqi Xu
externalLink: false
---
# RESTful Server:  http://localhost:8080/v1


## Github
[Restful APIs for CRUD of Restaurant](https://github.com/jiaqi-xu/Restful_apis)

## Tags
Flask, Flask-RESTful, SQLAlchemy, Mysql

## How to start
1. run `database_setting.py` to create tables
2. run `sample_data.py` to insert data
3. run `runner.py` to run the app

## Introduction

`/restaurants`  *[Get all restaurants, Create new restaurant]*

`/restaurants/<restaurant_id>`  *[Delete, Update, Read restaurant]*

## Detailed Description

### /restaurants <span id="restaurants">Get all restaurants, Create new restaurant</span>

#### GET: obtains a list of restaurants (contains restaurant_id and name)
##### response(get /restaurants)
```
{
  "restaurants": [{
      "restaurant_id": 1,
      "name": "Full House"
    },
    {
      "restaurant_id": 2,
      "name": "OpenRice"
    },
    {
      "restaurant_id": 3,
      "name": "Gyu-Kaku Japanese BBQ"
    },
    {
      "restaurant_id": 4,
      "name": "Liuyishou Hotpot"
    },
    {
      "restaurant_id": 5,
      "name": "Moral Village Hotpot"
    },
    {
      "restaurant_id": 6,
      "name": "Little Sheep Mongolian HotPot"
    },
    {
      "restaurant_id": 7,
      "name": "Captain's Boil"
    }
  ]
}
```

#### POST: create new restaurant
##### form-data
```
name="Full House",
description=(
        "Modern, informal restaurant for classic specialties from "
        "Jiangxi, Sichuan & other Chinese regions."
),
category_id=1,
phone="(613)798-5697",
address="1766 Carling Ave, Ottawa, ON K2A 1E1",
city="Ottawa",
country="Canada"
```

##### response(post /restaurants)
```
{
   'restaurant_id': 1
   'success': 1,
   'message': 'create successfully.'
}
```

### /restaurants/<restaurant_id>:int <span id="restaurant-1">Read, Update, Delete</span>

#### GET: obtain details of the restaurant
##### response(get /restaurants/3)
```
{
  "restaurant_id": 3,
  "name": "Gyu-Kaku Japanese BBQ",
  "description": "Gyu-Kaku is a chain of Japanese yakiniku restaurants specializing in Japanese barbecue.",
  "category": {
    "category_id": 3,
    "name": "Barbecue"
  },
  "phone": "(514)866-8808",
  "address": "1255 Crescent St, Montreal, QC H3G 2B1",
  "city": "Montreal",
  "country": "Canada",
  "addtime": "Mon, 21 May 2018 15:23:27 -0000"
}
```

#### PUT: update the info of the restaurant
##### form-data
```
name="new name"
description="new desc"
category_id=<new_category:int>,
phone="new phone",
```
##### response(put /restaurants/3)
```
{
   'restaurant_id': 1
   'success': 1,
   'message': 'update successfully.'
}
```

#### DELETE: remove a restaurant
##### response(delete /restaurants/3)
```
{
   'restaurant_id': 1
   'success': 1,
   'message': 'delete successfully.'
}
```
