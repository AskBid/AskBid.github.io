---
layout: post
title:      "React Final Project"
date:       2021-04-14 22:12:19 +0000
permalink:  react_final_project
---

here is the link to the [project video walkthrough](https://youtu.be/OvHbIPD1YdQ)

### Things learned in the Backend

I learned how to deal with two databases. I needed to fetch data from a [`cardano-db-sync`](https://github.com/input-output-hk/cardano-db-sync) that I was running on a separate server. This populates a postgres database that is read-only. So I used that database to fetch data and then created anotehr local database to save the models related to my app.
I then reated two classes inheriting from `ActiveRecord::Base` the standard 
```ruby
class ApplicationRecord < ActiveRecord::Base
  self.abstract_class = true
end
```
that connects to the database I use with the models relative to my app and
```ruby
class DbSyncRecord < ActiveRecord::Base
  self.abstract_class = true
  establish_connection DB_SYNC_DB
  # DB_SYNC_DB is defined in `config/initializers/db_sync_database.rb`
end
```
which instead connects to the remote database populated from `cardano-db-sync`.

Then I had to understand well the structure of the `cardano-db-sync` so I could recreate all its models (only the one I needed really) and associations to use this database in the rails framework as activeRecord models. All those models had to inherit from `DbSyncRecord` rather than `ApplicationRecord`.

I was surprised on how this process worked rather smoothly, this models did not even need to be migrated as the database is already managed from the Cardano node. It was though quite hard to understand how to connect the remote database and make it work with the default one.

I had few issues also down the line when in queries that needed a JOIN table amongst tables on `DbSyncRecord`  and `ApplicationRecord` did not work, so I had to create manual methods that handled this sort of associations.

As for the rest has been a challenge to understand mainly the structure of Cardano and do all the backend calcualtion on that system to serve my frontend, I felt quite comfortable in using Ruby and Rails to accomodate all of that machinery and all I learned in previous project really sufficed to achieve what I wanted.

<br>

### Things learned in the Frontend

Looking back at when I started working on the front-end I realize how much I learned, this project was a very good exercise. 
#### Class/Functional - Containers/Components React Redux Thunk Hooks
Initially I was a bit confused on all the various techniques, as the way taught in the lessons through class components that feeds dumber functional component with props was not always matching all the online material that i was consulting to widen my understanding. 
In most cases online material shows functional components that uses hooks to get fed action creators and state attributes. So I have been perhaps a bit inconsistent with my style because I was trying to taste all the flavours: the more traditional flatiron way and the hooks way all by functional components.
This has caused me not to have the most clean structure to my code, but the handlign and understanding of both methods really helped me to settle in all the concepts and the react - redux - thunk flow.
I often started with the intention to have a main container component that was feeding all the logic to dumber functional component down the tree, but that always got somehow corrupted and the supposedly dumber component woth time got more complicated and required more and more hooks to get fed all the necessary tools to work properly.
Now that all my structure has been layed out to work, would be a good time to go back and rethink the design to try and make it cleaner, but it would take too much time and I have to conclude the project for now, nevertheless I definitely learned a lot and have a better overview on things.
There was an autocomplete code that I wanted to use in my components, but didn't work properly with my existing logic, so I had to disect it completely and really understand its workings to embed it the right way with my struccture, and that wa a great exercise that taught me a lot more hooks.

#### Router, Switch, Routes, History, Match
I started thinking that Router was just in the first `app.js` component to list and direct all the various pages, but then thanks to the a few study groups I understood that was much more than that and how integral it is in the good use of React. First of all it allows to switch components based on the URL you are in allowing for a more RESTful design. It also helps a lot to know how the history is passed down the components and when and how you can access it, the same for the parameters of the routes that often makes life much easier into passing all the required data in queries to the backend nodes.

#### Fetch API
I finally got forced into really understand the Fetch API, what really happens with the promises, why we need a function inside those `.then()`s and how to `.catch()` things. In particular trying to get what I wanted from error messages was a great way to force some learning on this topic. before this project I always favoured using `async` and `await` because it looked less enigmatic to me, but I now fear no more the `fetch()` API and actually favor it.

#### Bootstrap as Tailwind
I tend not to consider it so important, but it definitely gave me a lot of satisfaction to finally understand bootstrap and achieve my best performance in terms of front-end aesthetic. When I saw Tailwind in action and understood that bootstrap could do the same by simply giving classes to tags to control the flow in an already tested graphic style everythign became so much easier and consisten than when in the previous projects I tried to do all myself from CSS.
But I am definitely happy I put in the effort to make my hands dirty with CSS and flexbox methods in the past, because it has for sure helped me understand better certain behaviour and necessesities with the bootstrap/tailwind way.

### Final Thoughts
This was a very good primer that summed up for me all I gathered during this course. I feel I could go on for ages in perfectioning this project, and I probably I will, but all the mechanics are much clearer in my head after this exercise and moving forward my style can only improve and get cleaner the more clarity comes the more I exercise what I learned.

