---
layout: post
title:  "SQLAlchemy with PostgreSQL earthdistance extension"
description: "Postgresql has a handy extension for calculating distances between coordinates on earth, we are looking at how it all fits into SQLAlchemy ORM."
date:   2015-03-19 20:30:16+01:00
size: 12
sizeDesktop: 4
textColor: grey-50
metaColor: grey-600
author: sander
---

The fact that Earth is round, but not a sphere makes calculating distances difficult. Fortunately PostgreSQL has an extension for just doing that - earthdistance. Doing the calculations in the database is superior as extensions are compiled and it enables us to build an index to reduce the amount of calculations needed to make at query time. I came across [this][postgres-radius] article to get me started, however considering I spent more time on making earthdistance work with SQLAlchemy, I decided to share my solution. The example below assumes that you already have earthdistance set up and working with a ORM model class 'Company' that has 'lat' and 'lng' fields present.


```python
from sqlalchemy import func
from example.model import Company
from example.core import db

def get_companies_from_amsterdam():
    #Calculating each company's distance from Amsterdam.
    loc_amsterdam = func.ll_to_earth(52.3667, 4.9000)
    loc_company = func.ll_to_earth(Company.lat, Company.lng)
    distance_func = func.earth_distance(loc_amsterdam, loc_company)
    query = db.session.query(Company, distance_func).order_by(distance_func) 
    
    #Resultset is no longer list of Company, but a list of tuples.
    result = query.all()
    mapped = []                                                                                                                  
    for row in result:
        company = row[0] 
        company.distance = row[1]
        mapped.append(company)
    return mapped
```

To speed up things, we can pre-compute <em>ll_to_earth(Company.lat, Company.lng)</em> using an index:


```python
from sqlalchemy import func, Index
from example.model import Company

Index('company_distance', func.ll_to_earth(Company.lat, Company.lng), postgresql_using='gist')
```

[postgres-radius]: http://johanndutoit.net/searching-in-a-radius-using-postgres/
