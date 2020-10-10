---
layout: post
title:      "Rails Project - Cafes Rota"
date:       2020-10-10 17:52:36 -0400
permalink:  rails_project_-_cafes_rota
---

The app is about users choosing from a directory of Cafes their favourites and then keep track of their visits. 
The app can then propose to the User the Cafes that haven't been visited for longer. [Cafe Rota](https://github.com/AskBid/cafes-rota)

I started this project from drafting the database schema on draw.io

![draw.io](https://i.postimg.cc/tJJsZ3CL/cafes.jpg)

Here I came to the realisation that for an Object to have an array as an attribute it is actually required to have an all new table. Now after the project is almost done it seems obvious, but it wasn't that intuitive at first.

That came handy because I realised that a Form to create a new cafe was the perfect candidate for using `accept_nested_attributes_for` as the attributes that need to be arrays (Images, Links and Timings) have attributes for themselves and that needs therefore to be nested in the Cafe form_to.
A limitation of the current Rails project is that there isn't an exact number of Images or Links that a Cafe must have, yet the User could be willing to enter any amount of those. It would be needed a dynamic page where a new field could be entered. But currently in `cafes#new` controller I run an instance method `#populate_new_cafe` that supply a reasonable amount of empty nested objects that will appear in the `form_for` `fields_for` when `cafes/new.html.erb` is displayed.
If the user doesn't fill in some of those fields then this line `reject_if: proc { |attributes| attributes[:url].blank? }, allow_destroy: true` added to `accepts_nested_attributes_for :links, :images, :openings` will get rid of all the Image and Link fields that haven't been provided a `:url`.

Drafting the concept for this app was hard because I had difficulties to visualise where the need for a nested route (a requirement for the Rails Project) was necessary.
I needed a model to connect User and Cafe which also became the perfect candidate to store the date of each user visiting a certain cafe. That became the Visits table (initially called Goings).
I wasn't sure Visit would have been a good nested route as I thought each visit would have gone in `/users/:id` so I created a table for Notes that each user could have made for each Cafe. 
Url, Images and Timings made sense to be displayed in `cafes/:id` while for Notes could have made sense to have a `cafes/:id/notes`, `cafes/:id/notes/new`, `cafes/:id/notes/edit`.

At the time of writing I given up on Notes as I ended up with a nested routes for Visits:

```
resources :users, param: :slug do
		resources :visits, only: [:index, :new, :create]
end
```

I then realised that having the Visits nested to the User may not be that much necessary as with a logged in user it would be enough for the controller to find only that User's Visitsin the route `/visits` using `session[:user_id]`.
The same is true for `visits/new` and `:create`, a route with the `users/:id...` isn't really necessary because the user_id would be provided from `#current_user`.

I then decided to keep the nested route for Visit as it is interesting for a user to go and see the Visits of another User through the routes `users/:other_user_id/visits`.
The logged in User can currently go in `cafes/:id` and see links to all the `users/:other_user_id/visits` that have that same cafe in their Visits.

If I were to use a model for Notes that joins Users and Cafes, I would need to create a new Note in a`cafes/:id` page  rather with a `form_for(@cafe, @note)` that loads a `@note = @cafe.notes.build` from `cafes#show` or by linking to `cafes/:id/notes/new` with the form_for appearing in a new page.
`:user_id` for `@note` would be obtained in the controller from `#current_user`.

# Scopes
```
	scope :visiting_cafe, -> (cafe_id) {joins(:visits).where('cafe_id = ?', cafe_id)}
	scope :visiting_today, -> (todays_date) {joins(:visits).where('last_visited = ?', todays_date)}
```
those two scopes came handy in  showing which users are visiting a given Cafe in the `cafes/:id` page (`User.visiting_cafe)`.
For the section showing the user which are visiting that cafe today a nice chained scopes worked pretty nice: `User.visiting_cafe.visiting_today`.








