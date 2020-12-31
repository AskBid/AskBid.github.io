---
layout: post
title:      "Front-end JS Project"
date:       2020-12-31 04:57:02 -0500
permalink:  front-end_js_project
---

In this project I tried to improve on the [Sinatra Project](https://askbid.github.io/engaging_with_activerecord) topic with the new knowledge that I gained from the front-end Javascript section.
<br><br>
This time was a pleasure to fit the interface I had in mind with the tools at my disposal, while in the previous projects I was looking to achieve an interactivity without the appropriate frontend tools like Javascript.
<br><br>
The app is quite simple on the surface, the User chooses between two models: Stake addresses and Pools to follow. Once he has selected a few instances of those models he can shuffle through the epochs and see how those Stakes and Pools have performed along each Epoch.
For a better understanding of the App context (the Cardano Blockchain environment) you can have a little read of [this section in the `README.md` file](https://github.com/AskBid/delegation-explorer#about-cardano-delegations).

<br><br><br><br>
## Backend
<br><br>
A `User` can have many `Stake` addresses, a `Stake` address can be owned from only one owner, but in our case we decided for a `many_to_many` association as a `User` who doesn't own a `Stake` address may still be willing to track it.
Because each `Stake` address can be delegated to a different `Pool` for each epoch, and because for each epoch it will receive certain `:rewards`, I have decided to separate the concerncs of ownership `User` => `Stake` address from that of the `Stake` address historical condition in each epoch. Therefore each `Stake` address will have a `has_many` association with `ActiveStake` that will indicate the `:rewards` and `:amount` staked related to each `Stake` address for each epoch (`:epochno`). Every `ActiveStake` will `belong_to` a `Pool` so to indicate the delegation for that particular `:epochno` of the `Stake` address associated. 
<br><br>
Once the User has selected the `Stake` addresses he wants to track, he can then select to follow several `Pools`. The association of the followed `Pools` is that of `many_to_many`. 
<br><br>
Here is a flow chart of the schema of the database that should summarise the architecture I thought for this project's DB:
![](https://raw.githubusercontent.com/AskBid/delegation-explorer/main/fow-chart.jpg)
<br><br>
Once the `User` enters a `Stake` a `fetch` POST request is sent to the backend where the `Stake` is assigned to the `User`. The `render()` is triggered once the POST request *promise* has been fulfilled. 
The `render()` function sends two GET requests to the beackend, one checks for `ActiveStake`s that have an `:epochno` as selected in the frontend interface, and the other function checks for all the `Pool`s associated with the current `User`.
At first the user won't have any `Pool`s followed, therefore only the the `Pool` to wich his `Stake` is asociated with will appear automatically in the main Tab, together with the delegated `:amount` and `:rewards` received in the selected Epoch (`:epochno`).
Acting on another form the user can then send a POST request to add followed `Pools` after which a new `render()` will be triggered.
Notice that also the GET request returning the followed `Pool`s requires a parameter to indicate what `:epochno` is selected. Infact in the backend for each `Pool` a ratio of `pool.amount / pool.rewards` for the Epoch will be calculated and returned in the rendered JSON as `:reward_ratio` which will be used in the frontend to calculate what the `:rewards` would have been if the delegation `active_stake.amount` was delegated to each `Pool` (`active_stake.amount * pool.reward_ratio`).
Since GET cannot have a request body the `:epochno` is passed to the backend as  a parameter in the action/URL path`?epochno=${epoch.current}`.
To add the `:reward_ratio` to each `Pool` rendered without adding another table with all the PoolEpochs I had to add `:methods` to the `render json:` function as follow:
```
        @pools = @user.followed_pools
        Pool.current_epoch = params[:epochno]
        render json: @pools, only: [:ticker, :id, :poolid], methods: :reward_ratio
```
I used a Class variable `Pool.current_epoch` as it wasn't possible to assign an argument to a method in `render json:`.
<br><br>
here is what the user will see after having performed the actions above:
![](https://raw.githubusercontent.com/AskBid/delegation-explorer/main/screengrab_demo.png)
<br><br>
The User shuffling through epoch will see the values changing accordingly to each Epoch's delegation amount, actual rewards received, and the potential rewards for each Pool followed calculated for each Pool's rewards_ratio calculated in that Epoch.
<br>
<br>
<br>
<br>
## Frontend
<br><br>
### `AppStorage` and Global Variables
In the frontend I have used a Class `AppStorage` to store all the variables that I would have commonly made Global, as the backend URL, the current User details and the Epochs min, max and current values.
Making a class with Class variables makes it a more secure way to share variables at Global level. I tried to replicate what is in ruby a Class variable of the type `@@var` by the use of Javascript `static function()` and by adding attributes to the Class itself in the shape of `MyClass.newAttribute = 'newValue'`.

For the backend URL I used a static method since there isn't a sesible place where that could ahve passed in as Class.attribute. So I used the following code:
```
class AppStorage {
	static BACKEND_URL() {
    return "http://127.0.0.1:3000"
  };
}
```
This way the URL is accessible anywhere in the code by `AppStorage.BACKEND_URL()`.
<br><br>
In other cases it made sense to assign new attributes to the Class at particular points, like when the User logs in `AppStorage.session = new Session(obj.username, obj.id, obj.token)` or when a successfule GET requests for the epoch values has been received `AppStorage.epoch = new Epoch(obj.min, obj.max, obj.max);`.

<br><br>
### JWT token and .User, .Session
Other classes have been used to organize the function and variables related to the User login/signup and the Session itself which uses a JWT token to authenticate the user in the backend. Session has methods that store the JWT token in `localStorage.token` so that when the user comes back to the page in the future, a `restoreSession()`  checks if a token is present in `localStorage` then it will login the associated User by querying the backend.
<br><br>
### .Tabs Classes and Hineritance
The frontend requires the use of a collection of nested `<div>`s all with different values. It would have been hard work to reuse all of this logic as functions, so I was glad I could organize it in classes with related methods to assign values into the `<div>`s structure with ease.
Each of the Tabs type required a method to create the first skeleton of the HTML tab. Therefore all of those `<div>` classes `extend` from a `.String_to_html` Class that creates a Javascript Element out of a string of HTML that each Tab and RowValue uses to create its initial skeleton. Other methods then enrich this skeleton with whatever is necessary.



