# Food Truck App

```css
{ }
```

Add some pages

```
commit @debug
  [#page target: "Home"]
  [#page target: "Checkout"]
  [#page target: "Stripe"]
  [#page target: "UserQueue"]
  [#page target: "Owner"]
  [#page target: "EditItem"]
  [#page target: "OrderQueue"]
  [#page target: "Cashier"]
  [#page target: "TruckSettings"]
  [#page target: "SocialMedia"]
```

The main app window, and accompanying buttons

```
search @debug
  [#page target]

bind @browser
  [#row style: [margin-bottom: 20] children:
            [#a class: "button inset pill" href: "#/{{target}}" style: [padding: 0] children: [#nav-button target text: target style: [padding: "0.5em 1em"]]]]
  [#window class: "app-window" #default #app-window]
```

## User View

### Home page

Draw the hero image, map location, and menu listing

```
search @browser
  window = [#app-window target: "Home"]

search
   item = [#menu not(#disabled) name image cost menu-sort]
   [#app settings]
   

bind @browser
  window <- [children:
    // Hero
    [#div style: [
      height: "320px"
      background-size: "cover"
  background-image: "url({{settings.hero-image}})"]]

    // Location
    [#div class: "flex-row" style: [align-items: "center" height: "5em" overflow: "hidden"  background: "#EEE"]
    children:
    [#h3 text: "Location Now:" style: [flex: "0 0 auto" padding: "0 20" margin: 0 font-size: "2em" font-weight: 200]]
    [#img #mapz src: "https://goo.gl/euvdoF"
    style: [flex: 1 background-size: "cover"]]]

    // Menu
    [#div class: "flex-row" style: [position: "relative" justify-content: "center" align-items: "center"] children:
    [#h3 text: "Menu" class: "menu-title"]
      [#button #cart-btn target: "Checkout" icon: "ios-cart"]
    ]
    [#div #menu-pane style:[flex:"0 0 auto" flex-direction:"column"]
    children:
    [sort:menu-sort #menu-item #flags #buyable item
    ]]]
```

Display a quantity badge on the shopping cart button.

```eve
search @browser
  wrapper = [#cart-btn]

search
  [#app order]
  [#order-item order item count]
  item-count = sum[value: count per:order given:item]
  item-count > 0

bind @browser
  wrapper.children += [#div class: "qty-badge" text: "({{item-count}})"]

```


```css
.app-window {
  align-self: center;
  background: white;
  width: 432;
  height: 768;
  overflow-y: auto;
}

.hero-image {
  height: 320px;
  background-size: cover]
}

.menu-title {
  margin: 10 0;
  font-size: 2em;
  font-weight: 200;
  text-decoration: underline;
}
```

### Checkout Page

display the nav button, which is unconditional

```eve
search @browser
  window = [#app-window target: "Checkout"]

search
   [#app order]
bind @browser
   window.children += [#button target: "Home" icon: "ios-arrow-back" text:"Back to menu!"]
```

display the current order

- Draw a user queue banner at the very top if an order is pending for them

```eve
search @browser @session
  window = [#app-window target: "Checkout"]
   [#app order]
   [#order-item order item count]
   not (count = 0)
   total = sum[value: count * item.cost per:order given:item]

bind @browser
   window.children += [#div #checkout-container
     style:[flex-direction:"column" display:"flex"]
     children:
       [#div item children:
       [#div class: "checkout-image" style: [background-image: "url({{item.image}})"]]
       [#div text:"{{item.name}} {{count}} x ${{item.cost}} = ${{count * item.cost}}" sort:1]]
       [#div text:"Total: ${{total}}" sort:2]
       [#button target: "Stripe" text:"place order!" style:[border:"2px solid black" sort:3]]]
```

move from the order page to the menu page

```
search @browser @event @session
   a = [#app page:"checkout" order]
   [#click element:[#from-checkout-to-home]]
commit
   a.page := "homepage"
```

Move from the checkout page to the Stripe page:

```
search @browser @event @session
   a = [#app page:"checkout" order]
   [#click element:[#order-button]]
commit
   a.page := "stripe"

```

```css
.checkout-image { 
  align-self: stretch;
  flex: 0 0 90px;
  height: 90px;
  width: 90px;
  margin: 0;
  margin-right: 10;
  background-size: cover;
  background-repeat: no-repeat;
  background-position: middle center;
  border-radius: 4px;
}
```

### Stripe

This block draws the Stripe page.

```
search @browser
  window = [#app-window target: "Stripe"]

search
   [#app order]
   [#order-item order item count]
   total = sum[value: count * item.cost per:order given:item]


bind @browser
  window.children += [#div class:"stripe" children:
    [#div class:"stripe-title" children:
      [#img src:"http://i.imgur.com/QI1vh2o.jpg"]
      [#div text:"Mama Rob's"]]
    [#input class:"stripe-email" placeholder:"Email"]
    [#div class:"payment-info" children:
      [#input class:"stripe-card" placeholder:"Card Number"]
      [#input class:"stripe-exp" placeholder:"MM/YY"]
      [#input class:"stripe-cvc" placeholder:"CVC"]]
    [#button target: "UserQueue" class:"stripe-pay-btn" text:"Pay ${{total}}"]
  ]
```

```
search @browser @event @session
  a = [#app page:"stripe" order]
  [#click element:[class:"stripe-pay-btn"]]

commit
  a.page := "user-queue"
```

```css
.stripe {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.stripe-title {
  width: 100%;
  background-color: #ecf0f2;
  padding: 20px 20px;
  font-size: 1.5rem;
  display: flex;
  align-items: center;
}

.stripe-title img {
  box-shadow: 0px 0px 10px 2px #15148d;
  border-radius: 10px;
  height: 80px;
  float: left;
  margin-right: 15px;
}

.stripe .stripe-email {
  border: 1px solid #555555;
  border-radius: 4px;
  width: 60%;
  margin-top: 40px;
  font-size: 16px;
  padding: 5px;
}

.stripe .payment-info {
  margin: 20px 0px;
  border: 1px solid #555555;
  border-radius: 4px;
  width: 60%;
  display: flex;
  flex-wrap: wrap;
}

.stripe .stripe-card {
  border: 0px;
  border-top-left-radius: 4px;
  border-top-right-radius: 4px;
  border-bottom: 1px solid #a9a9a9;
  flex: 1 1 60%;
  font-size: 16px;
  padding: 5px;
}

.stripe .stripe-exp {
  border: 0px;
  border-bottom-left-radius: 4px;
  border-right: 1px solid #a9a9a9;
  flex: 0 0 30%;
  font-size: 16px;
  max-width: 50%;
  padding: 5px;
}

.stripe .stripe-cvc {
  border: 0px;
  border-bottom-right-radius: 4px;
  flex: 0 0 30%;
  font-size: 16px;
  max-width: 50%;
  padding: 5px;
}

.stripe .stripe-pay-btn {
  border: 1px solid #307ac5;
  border-radius: 4px;
  padding: 10px 50px;
  background: linear-gradient(#46b1e5, #3099db);
  color: white;
}
```

This block commits the current cart to a new order record, which then appears in the Orders Pending Queue, and empties the cart to "reset" it.

```
search @browser @event @session
  [#app order]
  cart = [#order-item order item count]
  count > 0
  next-order = if c = count[given: [#order]] then c + 1
               else 1
  a = [#app page:"stripe" order]
  [#click element:[class:"stripe-pay-btn"]]

commit
  [#order #my-order number:next-order status:"pending" items:
    [#order-item item count]]
  cart := none

```

### User Queue Page

- Draw “Order Confirmed!” text
- Draw queue status
- If order is not ready yet, display number of pending orders ahead of user
- If order is ready, display “Order #XX is ready!”
- Draw order recap
- Draw back button to go back to home page

```
search @browser
  window = [#app-window target: "UserQueue"]

search
  order = [#order #my-order number items status]
  (ahead, message, my-status) = if status:"ready" then ("Your order is ready!","","ready")
    else if count[given: [#order status:"pending" number < order.number]] = 1 then ("1","order ahead of you!","ahead")
    else if ahead = count[given: [#order status:"pending" number < order.number]] then (ahead,"orders ahead of you!","ahead")
    else ("0","orders ahead of you!","ahead")

bind @browser
  window.children += [#div class:"my-order" children:
    [#div class:"order-confirmed" text:"Order confirmed!"]
    [#div class:my-status text:ahead]
    [#div class:"msg" text:message]
    [#div class:"my-order-number" text:"Order #{{number}}"]
    [#div class:"my-food" text:"{{items.count}}x {{items.item.name}}"]
    [#div class:"spacer"]
  [#button target: "Home" text:"❮ Back to Mama Rob's"]]

```

This block redirects the user back to the home page if they've haven't ordered yet.

```
search @browser
  wrapper = [#page-wrapper page: "user-queue"]

search
  not ([#order #my-order])

bind @browser
  wrapper.children += [#div class:"my-order" children:
    [#div class:"order-confirmed" text:"Oops! Looks like you haven't ordered yet."]
    [#div class:"spacer"]
  [#button target: "Home" text:"❮ Back to Mama Rob's"]]
```

```css
.my-order {
  flex: 0 0 600px;
  margin-bottom: 20px;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
}

.my-order .order-confirmed {
  text-transform: uppercase;
  font-weight: bold;
  font-size: 14px;
  line-height: 14px;
  text-align: left;
  flex: 0 0 50px;
  padding-left: 20px;
  padding-top: 20px;
}

.my-order .ahead {
  flex: 1;
  font-weight: bold;
  font-size: 60px;
  line-height: 60px;
  text-align: center;
  color: #b4b4b4;
  flex: 0 0 100px;
  padding-top: 40px;
}

.my-order .ready {
  flex: 1;
  font-weight: bold;
  font-size: 36px;
  line-height: 36px;
  text-align: center;
  color: green;
  flex: 0 0 100px;
  padding-top: 56px;
}

.my-order .msg {
  flex: 1;
  text-align: center;
  flex: 0 0 40px;
}

.my-order .my-order-number {
  text-decoration: underline;
  text-transform: uppercase;
  font-weight: bold;
  font-size: 14px;
  line-height: 14px;
  text-align: left;
  height: 50px;
  padding-left: 20px;
  padding-top: 20px;
}

.my-order .my-food {
  padding-left: 20px;
}

.my-order .spacer {
  flex-grow: 10;
}

.my-order .back-btn {
  text-transform: uppercase;
  font-weight: bold;
  font-size: 14px;
  line-height: 14px;
  text-align: left;
  flex: 0 0 50px;
  padding-left: 20px;
  padding-top: 20px;
}

.my-order .back-btn:hover {
  color: #0076ce;
}

```

## Owner View

### Home Screen

- Draw top banner
- Draw truck name
- Draw settings button
- Clicking button takes you to truck settings page
- Draw Social Media button
- Clicking button takes you to Social Media page
- Draw address search bar
- Google Maps integration
- Phone location integration
- Draw Menu header
- Draw Menu text
- Draw left and arrows to show swipe direction
- Draw “+” button to add item
- Clicking brings up item listing overlay
- Draw menu items
- Draw picture
- Draw item name
- All items are on by default, and remember their last state
- Swipe left to turn item off
- Swipe right to turn item on
- Clicking brings up item listing overlay

```
search @browser
  window = [#app-window target: "Owner"]

search
   item = [#menu name image cost menu-sort]

bind @browser
  window <- [children:
  [#div style: [padding: 10] children:
    [#button target: "SocialMedia" text: "Social Media"]
    [#editable class: "ion-android-search btn bubbly" default: "Enter your address"]]

  [#div class: "flex-row" style: [padding: "10 20" background: "#EEE"] children:
    //[#div class: "ion-arrow-left-c btn-icon-start" text: "off"]
    [#div class: "flex-spacer" style: [text-align: "center"] text: "Menu"]
    //[#div class: "ion-arrow-right-c btn-icon-end" text: "on"] style: [padding-right: 10]
  ]

      [#div #menu-pane style:[flex:"0 0 auto" flex-direction:"column"]
     children:
       [sort:menu-sort class:"owner-page-items" #menu-item #toggleable #modifiable #flags #sortable item]]]
```

Open the social flow when the owner clicks the social button.

```
search @event @browser
  [#click element: [#social-btn]]

search
  app = [#app]

commit
  app.page := "social-flow"
```

### Edit Item

- Draw item photo
- Blank by default
- Clicking lets you choose photo from phone storage or live photo
- Draw item description text form
- Blank by default
- Draw item price text form
- Blank by default
- Draw item health icons
- Vegetarian
- Draw icon
- Draw “V” text
- Gluten-free
- Draw icon
- Draw “GF” text
- Spicy
- Draw icon
- Draw “Spicy” text
- All off by default
- Clicking toggles on/off
- Draw trash can icon
- Clicking brings up an “Are you sure?” yes/no prompt
- Clicking no reverts to item listing
- Clicking yes permanently deletes the item listing and returns to the home screen
- Draw accept button
- If name and price forms both have information, commit contents of all 3 forms plus 3 health icons to item listing
- If either name or price forms do not have information, pop up an error message saying “You’re missing a name/price!” with an OK button to revert to item listing

```
search @browser
  window = [#app-window target: "EditItem"]

search
  [#app current-item: item]
  name = if item.name then item.name else ""
  image = if item.image then item.image else ""
  cost = if item.cost then item.cost else 0
  description = if item.description then item.description else ""
  state = if wrapper = [#owner-view] then "unlocked"
          else "locked"

bind @browser
  window.class += "edit-item"
  window <- [children:
    [#div class: "item-top-bar" children:
      [#img src: image style: [height: "100%" background: "#DDD"]]
      [#editable tag: state form: "edit-item" field: "name" class: "item-name" default: "name" value: name]
      [#button #submit-form form: "edit-item" item icon: "checkmark"]]

    [#div class: "item-middle-bar" children:
      [#div style: [flex: 1] children:
        [#editable tag: state form: "edit-item" field: "description" class: "btn bubbly item-description" default: "description" value: description]
        [#editable tag: state form: "edit-item" field: "cost" class: "btn bubbly item-price" default: "price" value: cost]]

      [#food-flags #toggleable item]]
    [#div class: "flex-spacer"]
    [#div class: "item-bottom-bar" children:
      [#button #delete-item-btn item icon: "trash-a"]]]
```

When there isn't a current-item, redirect to the owner page.

```
search
  app = [#app page: "edit-item" not(current-item)]

commit
  app.page := "owner"
```

When the submit button is clicked, save the current form state to the item, then clear it.

```
search @event @browser
  [#click element: [#submit-form form item]]
  form-elem = [#editable form field value]

search
  app = [#app]

commit
  lookup[record: item attribute: field] := none
  lookup[record: item attribute: field value]
  app.page := "owner"
  app.current-item := none
```

When the delete button is clicked, remove the current item from the menu and renumber any items below it for sorting purposes.

```
search @event @browser @session
  [#click element: [#delete-item-btn item:clicked-item]]
  items-below = [#menu menu-sort > clicked-item.menu-sort]

search
  app = [#app]

commit
  clicked-item := none
  items-below.menu-sort := items-below.menu-sort - 1
  app.page := "owner"
```

```css
.edit-item { display: flex; flex-direction: column; }

.edit-item .item-top-bar {
  position: relative;
  display: flex;
  flex-direction: row;
  align-items: flex-start;
  height: 140;
  padding: 20;
}

.edit-item .item-top-bar > img { border-radius: 6px; }

.edit-item .item-name { flex: 1; padding: 0 20; font-size: 1.5em; }

.edit-item .submit-btn { position: absolute; right: 0; top: 0; padding-top: 30; padding-right: 20; }

.edit-item .item-middle-bar {
  display: flex;
  flex-direction: row;
  align-items: flex-start;
  padding: 0 20;
}

.edit-item .item-description { height: 5em; justify-content: flex-start !important; }
.edit-item .item-price { width: 5em; justify-content: flex-start !important; padding-left: 20; }
.edit-item .item-price:before { display: block; content: "$" }

.edit-item .item-bottom-bar {
  display: flex;
  flex-direction: row;
  justify-content: flex-end;
  margin-top: 10;
}

.edit-item .item-delete-btn { display: flex; flex: 0 0 auto; padding-right: 20; padding-bottom: 30;}

```

### Order Queue

The pending orders queue is a portion of the food truck app that is used onboard the food truck itself to help whoever is working the pass to see what orders are currently being made and which orders are completed and ready for pickup. This portion of the app resembles TodoMVC in a way, as it needs to show a list of pending orders (todos) which can be progressed along according to their states of completion.

Orders are placed by the customer on their mobile device or by the cashier in the truck, but both end up in the @`orders` database, which assigns them the #`order` tag and an order `number` attribute. The contents of the order itself are kept in the [body? Don’t know which attribute would be here], and are visible in the order queue to help keep track of the order, but are not adjustable here. Finally, each order has a `status` flag which affects how the order is displayed in the queue. When they enter the @`orders` database, their default value is `pending`, and can be progressed to `ready` once the order is prepared and finally to `complete` once the customer has picked up the order.

This block draws orders in the browser that have an order number, items in the order, and a status that isn't `done`. It excludes orders that are `done` so that once an order is complete, it is simply no longer drawn on the screen. In the future if we want to add a fancier disappearing or completion animation, etc, this block might have to be changed but for now the simple solution suits our needs. There is also some included `if` logic that draws the right icon with each order and keeps them sorted such that orders ready for pickup are always at the top, and all remain sorted by order number.

```eve
search @browser
  window = [#app-window target: "OrderQueue"]

search
  order = [#order number items status]
  status != "done"
  icon = if status = "ready" then "ion-android-checkmark-circle"
             else if status = "pending" then "ion-android-time"
  order-style = if status = "pending" then "style-pending"
                            else if status = "ready" then "style-ready"
  index = sort[value: (status, number), direction: ("down", "up")]

bind @browser
  window.children += [#div sort:index class:("pending-order" order-style) #pending-order order children:
    [#div class:"order-number" text:number]
    [#div class:"order-items" children:
        [#div text:"{{items.count}}x {{items.item.name}}"]]
    [#div class:(icon "order-status")]]
```

```
search @browser @event @session
  [#click element:[#pending-order order]]
  newstatus = if order.status = "pending" then "ready"
              else if order.status = "ready" then "done"
commit
    order.status := newstatus
```

This block is some mocked up order data and will assuredly be replaced by the actual orders database once all the different parts of the app are integrated.

### Cashier

- Draw top banner
- Draw truck name
- Draw open/closed icon
- Clicking brings up open/closed overlay
- Clicking open sets truck status to open for business and reverts back to order screen
- Clicking closed sets truck status to closed and reverts back to order screen
- Draw menu items
- Draw photo
- Draw item name
- Swiping right adds one to the cart
- Swiping left removes one from the cart
- Draw pay button
- Clicking pay button takes you to the Stripe payment page/overlay

```
search @browser
  window = [#app-window target: "Cashier"]

search
  [#app settings]
  item = [#menu not(#disabled) name image cost]

bind @browser
  window.class += "cashier-order"
  window <- [children:
    [#header class: "flex-row" style: [flex: "0 0 auto"] children:
      [#h1 text: settings.name]
      [#div class: "flex-spacer"]
      [#employee-menu]]

    [#div class: "menu-items" children:
      [#menu-item #buyable item]]

    [#div class: "flex-spacer"]
    [#div class: "cashier-bottom-bar" children:
      [#button target: "Checkout" #finalize-order-btn text: "Finalize Order"]]]
```

Move from the cashier page to the Stripe page:
```
search @browser @event @session
  [#app order]
  [#order-item order item count]
  not (count = 0)
  a = [#app page:"cashier-order" order]
  [#click element:[#finalize-order-btn]]
commit
   a.page := "stripe"
```

```css

.cashier-order { display: flex; flex-direction: column; }

.cashier-order header { border-bottom: 1px solid #DDD; box-shadow: 0 6px 6px -3px white; z-index: 1; }

.cashier-order h1 { margin: 20; margin-bottom: 0; }

.cashier-order .menu-btn { align-self: flex-start; padding: 20;  font-size: 2em; }

.cashier-order .menu-items { position: relative; overflow-y: auto; }

.cashier-order .cashier-bottom-bar { padding: 10 80; border-top: 1px solid #DDD; box-shadow: 0 -6px 6px -3px white; z-index: 1; }

```

### Social Media

Draw the social media page. There are two cases we want to handle. If the truck isn't set up or there are no #enabled social media #integrations, then we should display a message prompting the owner to add those in.

```
search @browser @session
  window = [#app-window target: "SocialMedia"]
  not([#app settings: [truck-name]]
  [#integration #enabled])
  
bind @browser
  window.children := [#div children: 
    [#div text: "Set up your truck to use social features"]
    [#button target: "TruckSettings" text: "Truck Settings"]]
```

When the truck has a name and at least one integration, draw the complete social interface

```
search @browser
  window = [#app-window target: "SocialMedia"]
  
search
  app = [#app settings: [truck-name]]

  // Integrations
  integration = [#integration #enabled]
  selected? = if integration = [#selected] then ""
              else "-outline"
  
bind @browser
  window.children := [#div children: 
    [#div text: truck-name]
    [#button #to-home text: "home"]
    [#div children:
      [#div text: "Add a message:"]
      [#editable #social-message default: "message..."]]    
    [#div #integrations children:
      [#div text: "Add a photo:"]
      [#image-container #social-image prompt: "photo..."]]
    [#div children: 
      [#div text: "Select social networks"]
      [#span #integration integration class: "ion-social-{{integration.name}}{{selected?}}"]]
    [#button #post-social text: "Post"]]
```

Clicking an integration toggles its selection status for posting

```
search
  [#app page: "social-flow"]

search @browser @event @session
  [#click element]
  element = [#integration integration]
  (add, remove) = if integration = [#selected] then ("","selected")
                  else ("selected", "")
  
  
commit
  integration.tag += add
  integration.tag -= remove  
```

The "post" button is enabled when at least one social integration is selected, and a message or an image is uploaded

```
search @browser @session
  x = if [#social-message value] then ""
      else if [#social-image image] then  ""
  [#integration #selected]
  post-button = [#post-social]
  
bind @browser
  post-button += #enabled
  post-button.class += "enabled"
```

When an enabled "post" button i clicked, submit the message and photo to the relevent social networks

```
search @browser @event @session
  [#click element: [#button #post-social #enabled]]
  message = if [#social-message value] then value
            else ""
  image = if [#social-image image] then image
          else ""
  [#integration #selected name]
  [#time timestamp]
  
commit
  [#social-message message image, time: timestamp, network: name]
```


Delayed Posting:

- If credentials are missing, bring up credential screen
- Draw time box
- Says “When?” if no choice has been entered yet
- Clicking brings up time overlay
- Draw “Now” toggle
- Draw time and date selection
- Draw date icon
- Clicking opens a calendar
- Clicking a date chooses that date and closes the calendar
- Draw time selector
- Scrolling up counts forward in time
- Scrolling down counts backwards in time
- Draw check box button
- When clicked, apply time choice to post
- If “Now” is toggled on, choose now and revert to social media screen
- If “Now” is not toggled on, and both a date and a time have been selected, choose that time and revert to social media screen
- If “Now” is not toggled on, and either a time or a date is missing, flash box red and do nothing

Draw social media queue:

- Draw thumbnail photo if there’s a photo on the post
- Draw preview text
- Draw scheduled time to post
- Swipe left to remove from the queue
- Click to parse post details into the social media form and remove message from the queue
- Replace any contents that may already be present with contents from queued post

Draw “Post!” button
- If the message form contains text, at least one social media icon is toggled on, and a time is selected, make button clickable
- When clicked, if time selected is “Now”, commit the message to social media
- When clicked, if time selected is a later date, add message to social media queue to be posted at that time


### Truck Settings

Add some integrations

```
commit
  [#integration name: "twitter"]
  [#integration name: "instagram"]
  [#integration name: "facebook"]
```

Structure the page

```
search @browser @session
  window = [#app-window target: "TruckSettings"]
  integration = [#integration]
  enabled? = if integration = [#enabled] then ""
             else "-outline"

bind @browser
  window.children := [#div children:
  	[#div #page-header children:
  		[#editable #truck-name default: "Tap to name your truck"]
      [#button #edit-truck-name text: "edit"]
      [#button target: "Home" text: "home"]]
    [#div #page-body children:
      [#image-container #hero-pic prompt: "Tap to set your cover photo"]
      [#editable #truck-description default: "Tap to add a description"]
      [#div text: "Integrations"]
      [#div #integrations children:
  [#span #integration integration class: "ion-social-{{integration.name}}{{enabled?}}"]
      ]]]
```

Save the name of the truck when the user sets it

```
search @browser
  [#truck-name value]

search
  [#app #owner settings]

commit
  settings.truck-name := value
```

Put editable into editing mode when the button is clicked

```
search @event @browser
  truck-name-editable = [#truck-name]
  [#click element: [#button #edit-truck-name]]

commit @browser
  truck-name-editable += #editing
```

Save truck description to settings when it changes

```
search @browser
  [#truck-description value]

search
  [#app #owner settings]

bind
  settings.truck-description := value

```

Clicking on an integration opens up a page to enter credentials.

```
search @event @browser @session
  [#click element: [#integration integration]]
  app = [#app]

commit
  app.page := "integration setup"
  app.integration := integration
```

Draw credential forms

```
search @browser @session
  window = [#app-window page: "IntegrationSetup"]
  [#app integration]
bind @browser
  window.children := [#div children:
    [#div class: "ion-social-{{integration.name}}"]
    [#div text: "Sign in to {{integration.name}}"]
    [#input #username placeholder: "Username"]
    [#br]
    [#input #password type: "password" placeholder: "Password"]
    [#br]
    [#button #submit-credentials text: "Submit"]
    [#button target: "TruckSettings" text: "Cancel"]
  ]
```

Clicking submit applies the credentials to the integration

```
search @event @browser @session
  [#click element: [#button #submit-credentials]]
  [#password value: password]
  [#username value: username]
  [#app integration]

commit
  integration.credentials.username := username
  integration.credentials.password := password
```

## Menu Item

Menu items are present on most pages of the site. On each, they're roughly the same. Different styles and behaviors can be enabled by adding optional tags to the component.

```
search @browser
  menu-item = [#menu-item item]
  mode = if menu-item = [#description] then "description"
         if menu-item = [#instructions] then "instructions"
         if menu-item = [#buyable] then "buyable"
         if menu-item = [#flags] then "flags"
         if menu-item = [#normal] then "normal"

search
  item = [#menu name image cost]
  disabled? = if item = [#disabled] then true else false

bind @browser
  menu-item <- [#div class: "menu-item" class: [disabled: disabled?] children:
    [#div class: "item-image" style: [background-image: "url({{image}})"]]
    [#div #menu-item-text item class: "item-text" | mode children:
      [#div class: "item-name" text: name]]
    [#div class: "item-cost" text: cost]
  [#div #sort-buttons]]
```

### Dietary Flags

Adds the item's dietary flags beneath its name, if available.

```
search @browser
  item-text = [#menu-item-text item mode: "flags"]

search
  dietary-flag = item.dietary

bind @browser
  item-text.children += [#div class:"item-flags" children:[#div #flagged sort: 2 dietary-flag]]

```

```
search @browser
  flags = [#div #flagged dietary-flag]
  new-class = if dietary-flag = "v" then "ion-leaf"
              if dietary-flag = "spicy" then "ion-flame"
              if dietary-flag = "gf" then "ion-ios-rose"

bind @browser
  flags.class += new-class


```

### Description
Adds the item's description beneath its name, if available.

```
search @browser
  item-text = [#menu-item-text item mode: "description"]

search
  description = item.description

bind @browser
  item-text.children += [#div sort: 2 class: "item-description" text: description]
```

### Instructions
Adds a "special instructions" blurb.

```
search @browser
  item-text = [#menu-item-text item mode: "instructions"]

bind @browser
  item-text.children += [#div sort: 2 class: "item-instructions" text: "special instructions?"]
```


### Buyable
Adds event hooks to add/remove item from the current order.

If the item is included in the current order, give it a badge indicating how many are being purchased.
```
search @browser
  menu-item = [#menu-item #buyable item]

search
  [#app order]
  [#order-item order item count]
  count > 0

bind @browser
  menu-item.children += [#div class: "qty-badge" text: count]
```

Add an item to the current order.
@TODO: Support touch gestures.
```
search @browser @event @session
  [#app order]
  [#click element:[#add-item-btn item]]
  [#menu-item #buyable item]
  not([#click element:[#remove-item-btn]])
  count = if [#order-item order item count:c] then c + 1 else 1
  
commit
  order-item = [#order-item order item]
  order-item.count := count
```

Remove an item from the current order.
@TODO: Support touch gestures.
```
search @browser @event @session
  [#app order]
  [#click element:[#remove-item-btn item]]
  [#menu-item #buyable item]

search
  order-item = [#order-item order item count > 0]

commit
  order-item.count := order-item.count - 1
```

Draw the add to cart button.
@TODO: Don't show this on devices supporting gestures.
```
search @browser
  menu-item = [#menu-item #buyable item]

bind @browser
  menu-item.children += [#button #add-item-btn sort: 10 item icon: "plus-round" style: [margin-right: -10]]
```

Draw the remove from cart button.
@TODO: Don't show this on devices supporting gestures.
```
search
  [#app order]
  count = if [#order-item order item count] then count
          else 0

search @browser
  menu-item = [#menu-item #buyable item]

bind @browser
  menu-item.children += [#button #remove-item-btn sort: -1 item icon: "minus-round" class: [disabled: is(count = 0)] style: [margin-left: -10 margin-right: 10]]
```

### Toggleable
Add event hooks for enabling/disabling menu items.

```
search @event @browser
  [#click element: [#menu-item #toggleable item]]
  not ([#click element: [class:"item-image"]])
  not ([#click element: [class:"item-name"]])
  not ([#click element: [#sort-btn]])

search
  item = [#disabled]

commit
  item -= #disabled
```

```
search @event @browser
  [#click element: [#menu-item #toggleable item]]
  not ([#click element: [class:"item-image"]])
  not ([#click element: [class:"item-name"]])
  not ([#click element: [#sort-btn]])

search
  item = [not(#disabled)]

commit
  item += #disabled
```

### Modifiable
Modifiable items may be clicked to access the edit-item page for that item.

```
search @event @browser
  [#click element: [#menu-item #modifiable item]]
  [#click element: [class:"item-image"]]

search
  app = [#app]

commit
  app.page := "edit-item"
  app.current-item := item
```

```
search @event @browser
  [#click element: [#menu-item #modifiable item]]
  [#click element: [class:"item-name"]]

search
  app = [#app]

commit
  app.page := "edit-item"
  app.current-item := item
```

### Menu Sorting
Adds event hooks to add/remove item from the current order.

If the item is included in the current order, give it a badge indicating how many are being purchased.
```
search @browser @session
  menu-item = [#menu-item #sortable item]
  container = menu-item.children
  container = [#menu-item-text]
  menu-sort = item.menu-sort

bind @browser
  container.children += [#div class: "sort-badge" text:"{{menu-sort}}"]
```

Move an item up in the list.
@TODO: Support touch gestures.
```
search @browser @event @session
  [#click element:[#sort-up-btn item:clicked-item]]
  not([#click element:[#sort-down-btn]])
  [#menu-item #sortable item]
  above-clicked = [#menu menu-sort:clicked-item.menu-sort - 1]

commit
  clicked-item.menu-sort := clicked-item.menu-sort - 1
  above-clicked.menu-sort := above-clicked.menu-sort + 1
```

Move an item down in the list.
@TODO: Support touch gestures.
```
search @browser @event @session
  [#click element:[#sort-down-btn item:clicked-item]]
  not([#click element:[#sort-up-btn]])
  [#menu-item #sortable item]
  below-clicked = [#menu menu-sort:clicked-item.menu-sort + 1]

commit
  clicked-item.menu-sort := clicked-item.menu-sort + 1
  below-clicked.menu-sort := below-clicked.menu-sort - 1
```

Draw the up button.
@TODO: Don't show this on devices supporting gestures.
```
search @browser @session
  menu-item = [#menu-item #sortable item]
  not(item.menu-sort:1)
  container = menu-item.children
  container = [#sort-buttons]

bind @browser
  container.children += [#div #sort-up-btn #sort-btn sort: 10 item class:("btn","ion-chevron-up") style:[margin-right:-20]]
```

Draw the down button.
@TODO: Don't show this on devices supporting gestures.
```
search @browser @session
  menu-item = [#menu-item #sortable item]
  not(item.menu-sort: count[given:[#menu]])
  container = menu-item.children
  container = [#sort-buttons]

bind @browser
  container.children += [#div #sort-down-btn #sort-btn sort: 10 item class:("btn", "ion-chevron-down") style:[margin-right:-20]]
```

### Styles
Since the style differences for individual modes are so small, they've all been inlined.

```css
.pending-order {
  flex: 1 0 auto;
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  border-radius: 6px;
  box-shadow: 0px 0px 8px rgba(120,120,120,0.5) inset;
  margin-bottom: 20px;
  min-height: 75px;
  user-select: none;
}

.order-number {
  padding-left: 20px;
  font-size: 48px;
  font-weight: bold;
}

.order-status {
  font-size: 48px;
  padding-right: 20px;
}

.order-items {
  flex: auto;
  padding: 10px 0px 10px 20px;
  text-align: left;
}

.ion-android-time {
}

.ion-android-checkmark-circle {
  color: green;
}

.style-pending {
  background-color: #f4f4f4;
  border-left: 1px solid #b4b4b4;
  border-top: 1px solid #b4b4b4;
  border-right: 1px solid #c8c8c8;
  border-bottom: 1px solid #c8c8c8;
  color: #b4b4b4;
}

.style-ready {
  background-color: #ffffff;
  border: 1px solid #f0f0f0;
  color: #000000;
}

```


```css
.menu-item {
  display: flex;
  flex-direction: row;
  align-items: center;
  position: relative;
  padding: 10 20;
  min-height: 80;
  background: transparent;
}

.menu-item:hover { background: #F3F3F3; }
.menu-item:active { background: #E9E9E9; }

.menu-item.disabled { background: #DDD; opacity: 0.75; }
.menu-item.disabled .item-image { filter: saturate(25%); }

.menu-item .item-image {
  align-self: stretch;
  flex: 0 0 90px;
  height: 90px;
  margin: 0;
  margin-right: 10;
  background-size: cover;
  background-repeat: no-repeat;
  background-position: middle center;
  border-radius: 4px;
}

.menu-item .item-text {
  display: flex;
  flex: 1 1 auto;
  flex-direction: column;
  justify-content: center;
  font-size: 1.2rem;
}

.menu-item .item-name {
  color: #111;
  width: fit-content;
}

.menu-item .item-instructions {
  margin-bottom: -1.3em;
  color: #999;
  font-size: 0.8em;
}

.menu-item .item-description {
  font-size: 10pt;
}

.menu-item .item-flags {
  display: flex;
  flex-direction: row;
}

.menu-item .item-flags div {
  font-size: 1rem;
  padding-right: 5px;
}

.menu-item .item-cost { margin: 0 10; }
.menu-item .item-cost:before { content: "$"; }

.menu-item .qty-badge {
  position: absolute;
  left: 52;
  top: 10;
  width: 24;
  height: 24;
  padding-top: 2;
  z-index: 2;
  border-bottom-right-radius: 8px;
  border-top-left-radius: 4px;
  background: rgba(255, 255, 255, 1);
  text-align: center;
}

.menu-item .sort-badge {
  position: absolute;
  left: 20;
  top: 10;
  width: 24;
  height: 24;
  padding-top: 2;
  z-index: 2;
  border-bottom-right-radius: 8px;
  border-top-left-radius: 4px;
  background: rgba(255, 255, 255, 1);
  text-align: center;
}
```

## Food Flags
The food flags component is a toggle list of additional information about the item.

### Flags
Available flags in in the system.

```
commit
  [#food-flag flag: "v" name: "V" icon: "ion-leaf"]
  [#food-flag flag: "gf" name: "GF" icon: "ion-ios-rose"]
  [#food-flag flag: "spicy" name: "SPICY" icon: "ion-flame"]
```

Ensure every item has a flags object.

```
search
  item = [#menu not(flags)]

commit
  item.flags := []
```

Draw the food flags selector for an item

```
search @browser
  wrapper = [#food-flags item]
  toggleable? = if wrapper = [#toggleable] then true else false

search
  flag-entry = [#food-flag flag name]
  icon = if flag-entry.icon then flag-entry.icon else ""
  active? = if lookup[record: item.flags attribute: flag value: true] then true
  else if toggleable? then false
  // If the food flags aren't toggleable, there's no reason to show inactive flags.

bind @browser
  wrapper <- [#div class: "food-flags" children:
  [#div #food-flag class: "btn bubbly food-flag {{icon}}" class: [active: active?] flag item text: name]]
```

Toggleable food flags change the items flagged state on click.

```
search @event @browser
  [#click element: [#food-flag flag item]]

search
  item.dietary = flag

commit
  item.dietary -= flag

```

```
search @event @browser
  [#click element: [#food-flag flag item]]

search
  not(item.dietary = flag)

commit
  item.dietary += flag
```

```
search @browser
  home-food-flags = [#food-flags #display]

bind @browser
  home-food-flags.class += "display"
```

### Styles

```css

.food-flags {
  margin-left: 20;
}
.food-flags .food-flag {
  padding: 20;
  display: flex;
  flex-direction: row;
  justify-content: space-around;
}
.food-flag.active { background: #DDFFDD; border-color: #99FF99; }

.food-flags.display {
  display: flex;
  flex-direction: row;
}

```


### Navigation Buttons

When a button with a target is clicked, set the window to that target.

```
search @event @browser
  [#click element: [#button target]]
  window = [#app-window]
  
commit @browser
  window.target := target

```

There are some special cases when certain buttons are clicked

```
search
  app = [#app]
  item = [#menu name: "French Fries"]
  
commit
  app.current-item := item
```

# Sample Food Truck

### Sample Truck Settings

```eve
search 
  orders = [#order]

commit @browser
  [#app-window target: "Home"]

commit 
  [#app order: orders, settings: [name: "Mama Rob's" hero-image: "http://i.imgur.com/1lcSHmQ.jpg"]]
```

### Sample Orders

A sample user's order

```
search
  [#app order]
  item1 = [#menu name: "French Fries"]
  item2 = [#menu name: "Bacon Swiss Burger"]
    
commit
  [#order #my-order number: 8 status: "pending" items:
    [#order-item order item: item1 count: 2]
    [#order-item order item: item2 count: 1]]
```

Other orders

```eve
search 
  veggie = [#menu name:"Veggie Burger"]
  arnold = [#menu name:"Arnold Palmer"]
  burger = [#menu name:"Bacon Swiss Burger"]
  fries = [#menu name:"French Fries"]

commit
  [#order number:1 status:"pending" items:
    [#order-item item:veggie count:1]]
  [#order number:2 status:"pending" items:
    [#order-item item:veggie count:1]
    [#order-item item:fries count:1]]
  [#order number:3 status:"pending" items:
    [#order-item item:veggie count:1]
    [#order-item item:fries count:1]
    [#order-item item:arnold count:1]]
  [#order number:4 status:"pending" items:
    [#order-item item:burger count:1]
    [#order-item item:fries count:1]]
  [#order number:5 status:"pending" items:
    [#order-item item:veggie count:1]
    [#order-item item:arnold count: 2]
    [#order-item item:burger count: 2]
    [#order-item item:fries count:1]]
  [#order number:6 status:"pending" items:
    [#order-item item:burger count:1]
    [#order-item item:fries count:1]]
  [#order number:7 status:"pending" items:
    [#order-item item:burger count:1]
    [#order-item item:arnold count:1]
    [#order-item item:fries count:1]]

```

### Sample Menu Items

```
commit
   [#menu name:"Bacon Swiss Burger"
    image:"assets/burger.jpg"
    description:"A half pound Niman Ranch burger with melted Swiss, thick cut bacon, and housemade aioli and ketchup."
    cost:12.00
  menu-sort:1]

   [#menu name:"Veggie Burger"
    image:"assets/veggieburger.jpg"
    description:"Our totally vegan black bean and portabella mushroom burger on a gluten-free bun."
    cost:12.00
  menu-sort:2
    |dietary:("v", "gf")]

   [#menu name:"Chicken Salad Sandwich"
    image:"assets/sandwich.jpg"
    description:"Grandma’s chicken salad recipe with grapes and walnuts, served on wholesome 12 grain bread."
    cost:10.00
  menu-sort:3]

   [#menu
    name:"Charbroiled Chicken Wings"
    image:"assets/chickwings.jpg"
    description:"Brined, coated with our secret spice rub, then grilled until then skin is crispy and the meat is juicy."
    cost:10.00
  menu-sort:4
    |dietary:("spicy", "gf")]


   [#menu name:"French Fries"
    image:"assets/french-fries.jpg"
    description:"Hand-cut Kennebec fries with Cajun seasoning."
    cost:5.00
  menu-sort:5
    |dietary:("v", "gf")]


   [#menu
    name:"Garlic Parmesan Mac & Cheese"
    image:"assets/mac-n-cheese.jpg"
    description:"Elbow pasta smothered with aged cheddar, fresh garlic, and parsley."
    cost:10.00
  menu-sort:6
    |dietary:("v")]

  [#menu
    name:"Jill's Mexican Grilled Corn"
    image:"assets/corn.jpg"
    description:"Sweet corn grilled and covered with spicy adobo aioli."
    cost:7.00
  menu-sort:7
    |dietary:("v", "gf", "spicy")]


   [#menu
    name:"Gloria’s Beignets"
    image:"assets/fritters.jpg"
    description:"Our take on the classic - deep-fried yeast doughnuts topped with powdered sugar and drizzled with honey."
    cost:6.00
  menu-sort:8
    |dietary:("v")]


   [#menu
    name:"Arnold Palmer"
    image:"assets/drink.jpg"
    description:"Freshly brewed iced tea and freshly squeezed lemonade with just a touch of mint! The perfect thirst-quencher."
    cost:4.00
  menu-sort:9
    |dietary:("v", "gf")]

```

Components

Shared

Tags to Classes

Elements of any component type automatically apply the component name as a class.

```
search @components
  [#kind component]

search @browser
  element = [tag: component]
bind @browser
  element.class += component
```

White-listed tags ("kinds") for a certain component type are automatically applied as classes on that element.

```
search @components
  [#kind component kind]

search @browser
  element = [tag: component tag: kind]

bind @browser
  element.class += kind
```

Copy
Copy records copy a source record's attribute (if it exists) onto a destination record.
**NOTE**: This is currently limited only to @`browser`.

```eve
search @components
  [#copy source destination attribute]
  
search @browser
  lookup[record: source, attribute, value]
  
bind @browser
  lookup[record: destination, attribute, value]
```

Layout

```
commit @components
  [#kind #layout #group component: "row" kind: ("spaced", "default")]
  [#kind #layout #group component: "column" kind: ("spaced", "default")]
  [#kind #layout component: "spacer"]
```

All layout components are divs.

```
search @components
  [#kind #layout component]

search @browser
  element = [tag: component]

bind @browser
  element += #div
```

Row
Column
Spacer
Styles
```css

.row { display: flex; flex-direction: row; }
.row.spaced > * + * { margin-left: 10; }
.column { display: flex; flex-direction: column; }
.column.spaced > * + * { margin-top: 10; }
.spacer { display: flex; flex: 1; }

```

Window
Window components swap out their content for one of many possible panes. The panes are rendered on demand by looking for windows with the appropriate target. Windows serve many purposes -- from accordions and tab boxes to page routing (where each page is a pane in a single big window). Windows automatically listen for the #navigate event. so they work well in conjunction with the #nav-button component.

Setup

If a window (likely new) lacks a target, use the default target. If that isn't available, the user will need to direct the window somewhere useful, since the window can't know a priori about on-demand panes.

```
search @browser
  window = [#window not(target)]
  target = if window.default then window.default

commit
  window.target := target
```

If a window is targeting a pre-rendered pane, inject it into the window.

```
search @browser
  window = [#window target pane]
  pane.target = target

bind @browser
  window.children := pane
```

All windows are divs.
```
search @browser
  window = [#window]

bind @browser
  window += #div
```

Navigation
If a window receives a navigate event, update its target accordingly.

```
search @event @browser
  [#navigate window target]

commit @browser
  window.target := target
```

Button
Kinds
```
commit @components
  [#kind component: "button", kind: (
    "flat"
    "bubbly"
    "inset"
    "pill"
)]
```

Icons
Buttons with an icon apply it to their class.

```
search @browser
  element = [#button icon]
  solo? = if element.text then false
          if element.children then false
          else true

bind @browser
  element.class += "icon"
  element.class += "ion-{{icon}}"
  element.class += [icon-solo: solo?]
```

Nav Button
Nav Buttons are a simple button preset that allows easily navigating window panes. Clicking a nav button within a pane will trigger a navigate event on its window.

```
search @event @browser
  [#click element: [#nav-button target window]]

bind @event
  [#navigate window target]
```

If a window isn't explicitly specified for a nav button, we use any windows tagged default.

```
search @browser
  nav-button = [#nav-button target]
  window = if nav-button.window then nav-button.window
           else if window = [#window #default] then window

bind @browser
  nav-button.window := window
```

If a window targets a nav button's target, that button is active.

```
search @browser
  nav-button = [#nav-button target window: [#window target]]
  
bind @browser
  nav-button.class += "active"
  nav-button.disabled := true
```

Nav buttons regular buttons.

```
search @browser
  nav-button = [#nav-button not(#linked)]

bind @browser
  nav-button += #button
```

Toggle
Toggle buttons latch on and off like a checkbox. Their current state is a boolean in their checked attribute, much like an input. Custom styling may be applied by looking for #toggles with checked: true. Otherwise their active style will be applied by default.

A toggle button that is not checked is unchecked.

```
search @browser
  toggle = [#button #toggle]
  checked = if toggle.checked then toggle.checked else false
  
bind @browser
  toggle.checked := checked
```

Clicking a toggle button will toggle it.

```
search @event @browser
  toggle = [#button #toggle]
  e = [#click element: toggle]
  checked = if toggle.checked = true then false else true
  
commit @browser
  toggle.checked := checked
```

Checked toggles are marked active.

```
search @browser
  toggle = [#button #toggle checked: true]
  
bind @browser
  toggle.class += "active"
```

Styles
Reset logic adapted from: https://codepen.io/terkel/pen/dvejH.

```css

  a.button { display: inline-block; overflow: hidden; outline: none; }
  
.button {
  box-sizing: border-box;
  overflow: visible;
  padding: 0;

  background: none;
  border: none;
  outline: none;
  color: inherit;
  cursor: pointer;
  font: inherit;
  line-height: normal;

  -webkit-appearance: none;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
}
.button::-moz-focus-inner { border: 0; padding: 0; }

.button {
  padding: 0.5em 1em;
}

.button.flat:hover, .button.bubbly:hover, .button.inset:hover { background: rgba(0, 0, 0, 0.1); }
.button.flat:active, .button.bubbly:active, .button.inset:active, .button.active { background: rgba(0, 0, 0, 0.2); }

.row > .button + .button { margin-left: 10; }
.column > .button + .button { margin-top: 10; }

.button.icon:not(.icon-solo):before { padding-right: 0.5em; }

.button.bubbly {
  border: 1px solid rgba(0, 0, 0, 0.2);
  border-radius: 6px;
}

.button.inset {
  border: 1px solid rgba(0, 0, 0, 0.2);
  border-top: 1px solid rgba(0, 0, 0, 0.25);
  border-bottom: 1px solid rgba(0, 0, 0, 0.15);
  border-radius: 6px;
}
.button.inset:hover { box-shadow: inset 0 2px 1px 0px rgba(0, 0, 0, 0.25); }
.button.inset:active, .button.inset.active { box-shadow: inset 0 3px 1px 0px rgba(0, 0, 0, 0.25); }

.button.pill { border-radius: 0; }
.button.pill + .button.pill { margin: 0; }

.row > .button.pill + .button.pill { border-left-width: 0; }
.row > .button.pill:first-child { border-top-left-radius: 6px; border-bottom-left-radius: 6px; }
.row > .button.pill:last-child { border-top-right-radius: 6px; border-bottom-right-radius: 6px; }

.column > .button.pill + .button.pill { border-top-width: 0; }
.column > .button.pill:first-child { border-top-left-radius: 6px; border-top-right-radius: 6px; }
.column > .button.pill:last-child { border-bottom-left-radius: 6px; border-bottom-right-radius: 6px; }

```

Input
Kinds
```
commit @components
  [#kind component: "input", kind: (
    "flat"
    "bubbly"
    "inset"
)]
```

Forms
The form attribute is a simple way to manage user data submission. By giving each input in a form the same value for the form attribute and a unique name, you can easily retrieve the values of all the inputs without the hassle of assigning unique tags to each input. So long as at least one input exists with the given form id, a #form record will exist in the @forms DB that represents as up-to-date snapshot of the values of each input in the form.

Create a form for each unique form id.

```
search @browser
  [#input form]
  
bind @form
  [#form form data: []]
```

```
search @browser
  [#input form name value]
  
search @form
  [#form form data]
  
bind @form
  lookup[record: data, attribute: name, value]
```

Styles
```css

.input {
  padding: 0.5em 1em;
  background: transparent;
  border: none;
  outline: none;
}

.input.flat {
  background: white;
  border: 1px solid rgba(0, 0, 0, 0.2);
}

.input.bubbly {
  background: white;
  border: 1px solid rgba(0, 0, 0, 0.2);
  border-radius: 6px;
}

.input.inset {
  background: white;
  border: 1px solid rgba(0, 0, 0, 0.2);
  border-top: 1px solid rgba(0, 0, 0, 0.25);
  border-bottom: 1px solid rgba(0, 0, 0, 0.15);
  border-radius: 6px;
  box-shadow: inset 0 3px 1px 0px rgba(0, 0, 0, 0.25);
}
```

Dropdown
Conceptually, a dropdown is an input that, when focused, opens a menu beneath it. To facilitate this pattern without absolute positioning tricks, we build our dropdown as an input element within a div. When focused, we can position an element within this div but outside of document flow (that is, taking up zero height) to represent the menu itself.

```
commit @components
  [#kind component: "dropdown"]
```

All input kinds are valid dropdown kinds

```
search @components
  [#kind component: "input" kind]
  
bind @components
  [#kind component: "dropdown" kind]
```

Base
Dropdowns always contain an input and a button. When the dropdown is empty its icon is an arrow; but if it has an active query, it renders as an X button instead.

```
search @browser
  dropdown = [#dropdown]
  icon = if dropdown.query != "" then "close-circled"
         else "arrow-down-b"
  
bind @browser
  dropdown += #column
  dropdown.children +=
    [#row #dropdown-base dropdown class: "dropdown-base input" children:
      [#input #dropdown-input style: [flex: 1] dropdown]
      [#button #dropdown-button icon dropdown]]
```

If the dropdown has a placeholder or value, apply it to the input.

```
search @browser
  input = [#dropdown-input dropdown]
  
bind @components
  [#copy source: dropdown destination: input attribute: ("placeholder" "value")]
```

If the dropdown has an input style applied to it, apply it to the #dropdown-base.

```
search @components
  [#kind component: "input" kind]
  
search @browser
  dropdown = [#dropdown tag: kind]
  dropdown-base = [#dropdown-base dropdown]
  
bind @browser
  dropdown-base.class += kind
```

Dropdowns that aren't open are closed by default.

```
search @browser
  dropdown = [#dropdown]
  open = if dropdown.open then dropdown.open
         else false
  
bind @browser
  dropdown.open := open
```

The current input value is the dropdown's query attribute. It can be used to dynamically populate the dropdown menu (e.g. for autocomplete).

**NOTE**: I'd like to call the `query` attribute `search`, but the parser is a bit overzealous about that.

```eve
search @browser
  [#dropdown-input dropdown value]
  
bind @browser
  dropdown.query := value
```

Menu
Dropdowns embed their menu, hiding it if it is currently closed.

```
search @browser
  dropdown = [#dropdown menu open]
  
bind @browser
  dropdown.children += menu
  menu.sort := 1
  menu.class += "dropdown-menu"
  menu.class += [open]
```

Events
When a #dropdown-input is focused or a #dropdown-button is clicked, open the dropdown.

```
search @event @browser
  dropdown = [#dropdown open: false]
  event = [#click element: [#dropdown-input dropdown]]
            //else if [#click element: [#dropdown-button dropdown]] then true

commit @browser
  dropdown.open := true
```

When a click happens anywhere outside an open dropdown, close it.

```
search @event @browser
  [#click]
  dropdown = [#dropdown open: true]
  not([#click element: dropdown])
  
commit @browser  
  dropdown.open := none
```

If the #dropdown-button is clicked while it has a query, clear it.

```
search @event @browser
  [#click element: [#dropdown-button dropdown]]
  dropdown.query != ""
  input = [#dropdown-input dropdown]
  
bind @browser
  input.value := ""

```

Styles
```css

  .dropdown { position: relative; }
.dropdown .dropdown-base { padding: 0; }

.dropdown .dropdown-menu:not(.open) { display: none; }
.dropdown .dropdown-menu { position: absolute; top: 100%; left: 0; right: 0; margin-top: -4px; background: white; border: 1px solid #DDD; border-radius: 6px; box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2); z-index: 10; }
  
```

Modal
A modal is a special window that fills its entire container, on a higher z-level. When a modal contains content, it renders over its parent container with a shade obscuring the parent. If the modal window navigates away from content or the user clicks on the shade, the modal is dismissed.

```
commit @components
  [#kind component: "modal" kind: ("flat", "bubbly")]
```

A modal contains a shade and a window.

```
search @browser
  modal = [#modal]
  
bind @browser
  modal <- [#column children:
    [#div #modal-shade class: "modal-shade" modal]
    [#window #modal-window class: "modal-content" modal]]
```

A modal with content is open.

```
search @browser
  content = [#modal-window modal]
  open = if content.children then true
         else false
  
bind @browser
  modal.class += [open]
```

Events
Clicking a modal shade directly dismisses it.

```
search @event @browser
  [#click #direct-target element: [#modal-shade modal]]
  window = [#modal-window modal target]
  
commit @browser
  window.target := none
```

Styles
```css
.modal { display: none; position: absolute; top: 0; right: 0; bottom: 0; left: 0; align-items: center; justify-content: center; z-index: 20; }
.modal.open { display: flex; }
.modal .modal-shade { position: absolute; top: 0; right: 0; bottom: 0; left: 0; background: rgba(0, 0, 0, 0.2); -webkit-backdrop-filter: blur(2px); } 

.modal.flat .modal-content { z-index: 21; padding: 1em 2em; background: white; }

.modal.bubbly .modal-content { z-index: 21; padding: 1em 2em; background: white; border-radius: 6px; box-shadow: 0 3px 1px rgba(0, 0, 0, 0.2); }

```

### Handle Twitter login

Send credentials

```
search
  integration = [#integration name: "twitter" credentials: [username password]]

commit
  integration += #pending

commit @twitter
  [#login username password]
```

Handle login response from twitter. For now, just throw back a success. The following block would be part of the Eve twitter integration

```
search @twitter
  login = [#login username password]

commit @twitter
  login += #success
```

Get the response from twitter, and modify our internal twitter record

```
search @twitter
  [#login #success]

search
  integration = [#integration #pending name: "twitter"]
  app = [#app]

commit
  integration -= #pending
  integration += #enabled
  app.page := "settings"
```

## Editable

Every `#editable` is also a `#div`

```
search @browser
  editable = [#editable]

bind @browser
  editable += #div
```

If an editable has no text, display default text

```
search @browser
  editable = [#editable not(#editing) default]
  name = if editable.value = "" then default
         else if editable.value then editable.value
         else default

bind @browser
  editable.children := [#div text: name]
```

Clicking on an editable puts it in the #editing state, which renders an input box instead of a raw div.

```
search @event @browser
  [#click element]
  element = [#editable]
  not(element = [#locked])

commit @browser
  element += #editing
  element.class += "editing"
```

`#editables` in editing mode have an #input instead of a #div

```
search @browser
  editable = [#editable #editing]
  value = if editable.value then editable.value
          else ""

bind @browser
  editable.children := [#input value autofocus: true]
```

Pressing "enter" while editing an #editable will save its current text and revert the editable back to its original state

```
search @browser
  editable = [#editable children: [#input value]]

search @event
  event = [#keydown element: editable key: "enter"]

commit @browser
  editable -= #editing
  editable.value := value
```
