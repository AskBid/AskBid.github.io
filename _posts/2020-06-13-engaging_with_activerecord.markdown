---
layout: post
title:      "Engaging with ActiveRecord"
date:       2020-06-13 04:37:13 -0400
permalink:  engaging_with_activerecord
---

After all the labs I figured that I wasn't still truly getting ActiveRecord, in particular *associations*.

I thought that if things are too simple I can pass labs by rule-of-thumb, like you can solve some algebraic math formula  following a few tricks seen on the whiteboard, switching things around but without truly understanding what those actions mean.

To make the path more worthy I went for something  more  specific to model rather than a given clear drill that unfolds to the target smoothly.
Now I was truly engaged in understanding ActiveRecord.

## The Subject: an Intro to Cardano
Cardano is a blockchain in which the ledger that records transactions is maintained from competing Pools.

In Cardano time is divided by Epochs, each Epoch has something like >20k slots. 
For each slot one Pool can produce a block. 
For each block produced the Pool gets a reward in the form of native coins.
Pools have to win a lottery for each slot that they get to produce.
Pools have more chances to win the lottery the more coins they have.

Now the fun thing, each coin holder in the Cardano ecosystem (the User) can delegate their coins to a certain Pool so to give that Pool more chances to win in each lottery.
By doing so the User will get a reward proportional to his delegation in case his pool gets to produce blocks.

The User has then a decision to make, which Pool will he give his money? 
The more performing his pool will be, the more performing his delegation will be as well.

With Doubtful-Delegator (my project app) a User can select a group of pools he has delegated to and a group of Pools he wants to track. 
During the course of an epoch he can now compare the performance of the actual delegations and the performance of "would be" delegations.

## Drafting the Schema

![](https://github.com/AskBid/doubtful-delegator/blob/master/db/concerns/doubtful_delegator_2.jpg?raw=true)

I started with this *schema* that made a lot of sense back then, but now just shows me how confused I was.
The most difficult thing to overcome was that I had to record the performance for each pool in each epoch. And each user could delegate to each epoch. I was struggling to visualise how to make a time series be seen more like an item, a row in a table associated to other tables.
In the diagram above I was trying to associate the Pool to the User, because ultimately the User delegates to a Pool, not to an Epoch. As you can see though the path for the User to access the performance information of each Pool's Epoch was quite complex.

After all each Pool_Epoch has only one Pool, so I didn't need a *many_to_many* *association* therefore neither I needed a *join table* between them:
![](https://raw.githubusercontent.com/AskBid/doubtful-delegator/master/db/concerns/doubtful_delegator%2007.34.48.jpg/)

At this point I started to see that it was not necessary to abstract time (Epoch) in its own entity/table associated with each Pool_Epoch, after all a Pool_Epoch was in itself an indication of that time period. I further simplified the schema:
![](https://github.com/AskBid/doubtful-delegator/blob/master/db/concerns/doubtful_delegator_1tab_deep_1delegation-tab_only.jpg?raw=true)

I was always ending up with a *join table* of three elements which I soon realize creates problems in Active record because if I do:
```
user.pools << pool
user.pool_epochs << pool_epoch

user.delegations

=> [#<Delegation id: nil, user_id: 1 , pool_id: 1, pool_epoch_id: nil>, #<Delegation  id: nil, user_id: 1 , pool_id: 0, pool_epoch_id: 1>]
```
I was struggling to see how to create associate User, Pool and Pool_Epoch with each other in the same Delegation.
I now know I could probably do something like:
```
user.pools << pool
user.delegations.last.pool_epoch_id = pool_epoch.id

user.delegations

=> [#<Delegation id: nil, user_id: 1 , pool_id: 1, pool_epoch_id: 1>]
```
But I didn't know back then, and even now it feels not a good use of the abstractions given from ActiveRecord, it felt not that clean to have 3 *foreign keys*  in one *join table* and it still doesn't to be fair.

So I came to this final version of my DataBase:
![](https://github.com/AskBid/doubtful-delegator/blob/master/db/concerns/doubtful_delegator_1tab_deep_Pool2epoch.jpg?raw=true)
The User in real life chooses a Pool, but for the database is still correct to abstract the User to be connected to a Pool_Epoch which can be correlated to a Pool in case we want information about the Pool down the line.

The app is not using the historical properties of this schema, the app only considers the latest epoch, nevertheless this design could accommodate historical exploration of the records. 

## Doubts about the Design

This choice of database design gave me a few complexities I am not sure were necessary. 
In case I need to select certain User's Pool_Epochs that are connected only to certain Pools, I am forced to use #joins and #where which perhaps is not the best way in order to use ActiveRecord's abstractions strengths.
Not sure if the need of #joins and #where comes because of the subject's complexity or because I set up things wrong?

Another issue on this note is the two types of delegations (the real actual delegation and the desirable delegation).
This requires adding an extra step when using ActiveRecord to assign an object in the *many_to_many* relationship.
Because in this design the *join table* is not just a mechanism to connect two models, but a model in itself, with its own attributes complicates things.
```
user.pool_epochs << pool_epoch

user.delegations
=> [#<Delegation user_id: 1, pool_epoch_id: 1; kind: nil; amount: nil>]

#extra step needed
user.delegations.last.kind = 'desirable'
user.delegations.last.amount = 1.0
```
is there a better way that would fit the ActiveRecord way better?

## Routing

My main issue with routing is that I didn't have a use case that would fit a *RESTful* routing smoothly.

This is because for the `GET users/:id` what I am really showing are delegations belonging to the user.
The editing version of such a view would be to edit the delegations belonging to the user, but that is what the route `GET /delegations/:id/edit` should do.
With the help of http://www.restular.com/ I created the routing `GET user/:id/delegations/edit` and I couldn't use a `GET user/:id/delegations/:id/edit` because we are not editing only one Delegation rather all the delegations belonging to the user.

But here is another contradiction, `GET user/:id/delegations/edit` in reality will offer the ability to create new delegations (should have been  `GET user/:id/delegations/:ids/new` then?) and also to delete existing one and therefore uses a `delegations/new.erb` view.
I decided for `GET user/:id/delegations/edit` to direct to a `POST user/:id/delegations/edit` as new delegations are created (but also deleted :/) which then renders `delegations/edit.erb`  which requests `PATCH user/:id/delegations/edit` as in the view is possible to edit the .amount attribute of existing delegations.

## Views
Views wasn't a neat process either. The fact that I had many elements to be modified which were dependent on more than one *Model* attributes, gave me a few complications on how to transfer that information between *Controllers* and *Views*.

## Conclusion
The subject has not allowed me to have the cleanest architecture, but the process of trying to realise it helped me to understand the problems behind it.
By taking a subject as in the Fwitter lab I would have a cleaner result but much less understanding of the issues.
I am now more open to receive any concept aimed to solve some of the complications that I have faced along this project.
