# Food Truck App

## User

start the app for the user, landing on the menu page. here we create a new
empty order record.
```
  commit [#app page:"homepage" order:[]]
```

### Website Home Page

Draw the menu
should include title art
- Draw the hero image
- Draw the title text on top of the image
- Draw the food truck description
we can add the additional imagery to the page if provided.
- Draw the Location box
- Draw “Location today:” text
- Draw the minimap
- If no location is given, put a question mark
wouldn't it be better to just not draw?
- If location is given, put a pin
- Clicking the map opens up default map program

Draw the homepage.

```eve
search @browser
   wrapper = [#page-wrapper page:"homepage"]

search
   item = [#menu not(#disabled) name image cost menu-sort]

bind @browser
  wrapper <- [children:
    // Hero
    [#div style: [
      height: 320
      background-image: "url(http://i.imgur.com/1lcSHmQ.jpg)"
      background-size: "cover"]]

    // Location
    [#div class: "flex-row" style: [align-items: "center" height: "5em" overflow: "hidden"  background: "#EEE"]
      children:
        [#h3 text: "Location Now:" style: [flex: "0 0 auto" padding: "0 20" margin: 0 font-size: "2em" font-weight: 200]]
        [#img #mapz src: "https://goo.gl/euvdoF"
          style: [flex: 1 background-size: "cover"]]]

    // Menu
    [#div class: "flex-row" style: [position: "relative" justify-content: "center" align-items: "center"] children:
      [#h3 text: "Menu" style: [
        margin: "10 0"
        font-size: "2em"
        font-weight: 200
        text-decoration: "underline"]]
      [#cart-btn]
      ]
    [#div #menu-pane style:[flex:"0 0 auto" flex-direction:"column"]
     children:
       [sort:menu-sort #menu-item #flags #buyable item
       ]]]
```

Draw the cart button.
```
search @browser
  wrapper = [#cart-btn]

bind @browser
  wrapper <- [#div class: "flex-row"
    style: [flex: "0 0 auto" position: "absolute" width: 70 right: 0]
    children:
      [#div #cart class: "btn" style: [
        height: 30 padding: "0 5"
        content:"url(assets/shopping-cart-icon-30.png)"]]]
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
  wrapper.children += [#div #total-items class: "qty-badge" text: "({{item-count}})"]
```

Navigate to the checkout page.

```
search @browser @event @session
  a = [#app page:"homepage" order]
  [#click element:[#cart]]

commit
  a.page := "checkout"
```


### Checkout Page

- Draw the top banner
- Back button in left
- Draw each menu item in cart
- Swiping left removes item
- Clicking item brings up item detail overlay
- Draw pay button
- Clicking pay button takes you to the Stripe payment page/overlay

display the nav button, which is unconditional
```eve
search @browser
  wrapper = [#page-wrapper page: "checkout"]

search
   [#app order]
bind @browser
   wrapper.children += [#div #from-checkout-to-home text:"back to menu!" style:[border:"2px solid black"]]
```


display the current order
- Draw a user queue banner at the very top if an order is pending for them
```eve
search @browser
  wrapper = [#page-wrapper page: "checkout"]

search
   [#app order]
   [#order-item order item count]
   not (count = 0)
   total = sum[value: count * item.cost per:order given:item]

bind @browser
   wrapper.children += [#div #checkout-container
     style:[flex-direction:"column" display:"flex"]
     children:
       [#div text:"{{item.name}} {{count}} x {{item.cost}} = {{count * item.cost}}" sort:1]
       [#div text:"total: {{total}}" sort:2]
       [#div #order-button text:"place order!" style:[border:"2px solid black" sort:3]]]
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

### Checkout Item Detail
- Draw the item photo
- Draw the item name
- Draw the item description
- Draw the “Special instructions” text form
- Draw the item health icons
- Draw the quantity
- Initial displayed quantity is however many the user ordered
- Quantity box is a text form
- Unit price is pulled from item listing
- Total price is always unit price times whatever quantity is currently in the quantity box, regardless if the user has hit the accept button yet
- Draw the Accept button/checkmark
- Clicking closes the overlay to return you to the default checkout view
- When clicked, apply any special instructions from the text form to the order
- When clicked, update the quantity to whatever value is in the quantity text form



### User Queue Page
- Draw “Order Confirmed!” text
- Draw queue status
- If order is not ready yet, display number of pending orders ahead of user
- If order is ready, display “Order #XX is ready!”
- Draw order recap
- Draw back button to go back to home page

```
search @browser
  wrapper = [#page-wrapper page: "user-queue"]

search
  order = [#order #my-order number items status]
  (ahead, message, my-status) = if status:"ready" then ("Your order is ready!","","ready")
    else if count[given: [#order status:"pending" number < order.number]] = 1 then ("1","order ahead of you!","ahead")
    else if ahead = count[given: [#order status:"pending" number < order.number]] then (ahead,"orders ahead of you!","ahead")
    else ("0","orders ahead of you!","ahead")

bind @browser
  wrapper.children += [#div class:"my-order" children:
    [#div class:"order-confirmed" text:"Order confirmed!"]
    [#div class:my-status text:ahead]
    [#div class:"msg" text:message]
    [#div class:"my-order-number" text:"Order #{{number}}"]
    [#div class:"my-food" text:"{{items.count}}x {{items.item.name}}"]
    [#div class:"spacer"]
  [#div #back-btn class:"back-btn" text:"❮ Back to Mama Rob's"]]

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
  [#div #back-btn class:"back-btn" text:"❮ Back to Mama Rob's"]]
```

This block lets you go back to the home page from the user queue page.

```
search @browser @event @session
  a = [#app page:"user-queue" order]
  [#click element:[#back-btn]]

commit
  a.page := "homepage"
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

## Owner

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
  wrapper = [#page-wrapper page: "owner"]

search
   item = [#menu name image cost menu-sort]

bind @browser
  wrapper <- [children:
  [#div style: [padding: 10] children:
    [#div #social-btn class: "btn bubbly" text: "Social Media"]
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

```css

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

### Item Listing
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
  wrapper = [#page-wrapper page: "edit-item"]

search
  [#app current-item: item]
  name = if item.name then item.name else ""
  image = if item.image then item.image else ""
  cost = if item.cost then item.cost else 0
  description = if item.description then item.description else ""
  state = if wrapper = [#owner-view] then "unlocked"
          else "locked"

bind @browser
  wrapper.class += "edit-item"
  wrapper <- [children:
    [#div class: "item-top-bar" children:
      [#img src: image style: [height: "100%" background: "#DDD"]]
      [#editable tag: state form: "edit-item" field: "name" class: "item-name" default: "name" value: name]
      [#div #submit-form form: "edit-item" item class: "btn submit-btn ion-checkmark"]]

    [#div class: "item-middle-bar" children:
      [#div style: [flex: 1] children:
        [#editable tag: state form: "edit-item" field: "description" class: "btn bubbly item-description" default: "description" value: description]
        [#editable tag: state form: "edit-item" field: "cost" class: "btn bubbly item-price" default: "price" value: cost]]

      [#food-flags #toggleable item]]
    [#div class: "flex-spacer"]
    [#div class: "item-bottom-bar" children:
      [#div #delete-item-btn item class: "btn item-delete-btn ion-trash-a"]]]
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

### Social Media

Draw the social media page. There are two cases we want to handle. If the truck isn't set up or there are no #enabled social media #integrations, then we should display a message prompting the owner to add those in.

```
search @browser @session
  wrapper = [#page-wrapper page: "social-flow"]
  // Is there a better way to do this? I want the message to display if either of these conditions are true
  not([#app settings: [truck-name]]
  [#integration #enabled])

bind @browser
  wrapper.children := [#div children:
    [#div text: "Set up your truck to use social features"]
    [#button #to-settings text: "Truck Settings"]]
```

Navigate to the settings page when the button is clicked

```
search
  app = [#app]

search @event @browser
  [#click element: [#button #to-settings]]

commit
  app.page := "settings"
```

When the truck has a name and at least one integration, draw the complete social interface

```
search @browser
  wrapper = [#page-wrapper page: "social-flow"]

search
  app = [#app settings: [truck-name]]

  // Integrations
  integration = [#integration #enabled]
  selected? = if integration = [#selected] then ""
              else "-outline"

bind @browser
  wrapper.children := [#div children:
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
  wrapper = [#page-wrapper page: "settings"]
  integration = [#integration]
  enabled? = if integration = [#enabled] then ""
             else "-outline"

bind @browser
  wrapper.children := [#div children:
  	[#div #page-header children:
  		[#editable #truck-name default: "Tap to name your truck"]
      [#button #edit-truck-name text: "edit"]
      [#button #to-home text: "home"]]
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

Clicking the home button returns you to the home screen

```
search
  app = [#app]

search @event @browser
  [#click element: [#button #to-home]]

commit
  app.page := "homepage"
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
  wrapper = [#page-wrapper page: "integration setup"]
  [#app integration]
bind @browser
  wrapper.children := [#div children:
    [#div class: "ion-social-{{integration.name}}"]
    [#div text: "Sign in to {{integration.name}}"]
    [#input #username placeholder: "Username"]
    [#br]
    [#input #password type: "password" placeholder: "Password"]
    [#br]
    [#button #submit-credentials text: "Submit"]
    [#button #to-settings text: "Cancel"]
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

Clicking cancel goes back to the settings page

```
search
  app = [#app]

search @event @browser
  [#click element: [#button #to-settings]]

commit
  app.page := "settings"
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


## Cashier

### Order Screen
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
  wrapper = [#page-wrapper page: "cashier-order"]

search
  item = [#menu not(#disabled) name image cost]

bind @browser
  wrapper.class += "cashier-order"
  wrapper <- [children:
    [#header class: "flex-row" style: [flex: "0 0 auto"] children:
      [#h1 text: "Mama Rob's"]
      [#div class: "flex-spacer"]
      [#employee-menu]]

    [#div class: "menu-items" children:
      [#menu-item #buyable item]]

    [#div class: "flex-spacer"]
    [#div class: "cashier-bottom-bar" children:
      [#div #finalize-order-btn class: "btn bubbly finalize-btn" text: "Finalize Order"]]]
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

## The Pass

### Orders Pending Queue

The pending orders queue is a portion of the food truck app that is used onboard the food truck itself to help whoever is working the pass to see what orders are currently being made and which orders are completed and ready for pickup. This portion of the app resembles TodoMVC in a way, as it needs to show a list of pending orders (todos) which can be progressed along according to their states of completion.

Orders are placed by the customer on their mobile device or by the cashier in the truck, but both end up in the @`orders` database, which assigns them the #`order` tag and an order `number` attribute. The contents of the order itself are kept in the [body? Don’t know which attribute would be here], and are visible in the order queue to help keep track of the order, but are not adjustable here. Finally, each order has a `status` flag which affects how the order is displayed in the queue. When they enter the @`orders` database, their default value is `pending`, and can be progressed to `ready` once the order is prepared and finally to `complete` once the customer has picked up the order.

This block draws orders in the browser that have an order number, items in the order, and a status that isn't `done`. It excludes orders that are `done` so that once an order is complete, it is simply no longer drawn on the screen. In the future if we want to add a fancier disappearing or completion animation, etc, this block might have to be changed but for now the simple solution suits our needs. There is also some included `if` logic that draws the right icon with each order and keeps them sorted such that orders ready for pickup are always at the top, and all remain sorted by order number.

```eve
search @browser
  wrapper = [#page-wrapper page: "order-queue"]

search
  order = [#order number items status]
  status != "done"
  icon = if status = "ready" then "ion-android-checkmark-circle"
             else if status = "pending" then "ion-android-time"
  order-style = if status = "pending" then "style-pending"
                            else if status = "ready" then "style-ready"
  index = sort[value: (status, number), direction: ("down", "up")]
bind @browser
  wrapper.children += [#div sort:index class:("pending-order" order-style) #pending-order order children:
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

## Stripe
This block draws the Stripe page.

```
search @browser
  wrapper = [#page-wrapper page: "stripe"]

search
   [#app order]
   [#order-item order item count]
   total = sum[value: count * item.cost per:order given:item]


bind @browser
  wrapper.children += [#div class:"stripe" children:
    [#div class:"stripe-title" children:
      [#img src:"http://i.imgur.com/QI1vh2o.jpg"]
      [#div text:"Mama Rob's"]]
    [#input class:"stripe-email" placeholder:"Email"]
    [#div class:"payment-info" children:
      [#input class:"stripe-card" placeholder:"Card Number"]
      [#input class:"stripe-exp" placeholder:"MM/YY"]
      [#input class:"stripe-cvc" placeholder:"CVC"]]
    [#div class:"stripe-pay-btn" text:"Pay ${{total}}"]
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

# Sample Menu Data

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

This block progresses the `status` of an order when the order is double clicked.

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

# Components

## App Wrapper
### Page Template

```
search
  [#app page]
bind @browser
  [#div #app-wrapper class: "app-wrapper" children:
    [#div #page-wrapper class: "page-wrapper" page]
    [#nav-panel sort: 2]]
```

### Nav Panel (debug)
Display a list of pages to switch between. In the actual application this will occur through  one of the four built in user flows.

```
search @browser
  wrapper = [#nav-panel]
bind @browser
  wrapper <- [#div class:"nav-panel" children:
    [#div #nav-btn page:"homepage" text:"Home"]
    [#div #nav-btn page:"checkout" text: "Checkout"]
    [#div #nav-btn page:"stripe" text: "Stripe"]
    [#div #nav-btn page:"user-queue" text:"User Queue"]
    [#div #nav-btn page:"order-queue" text:"Order Queue"]
    [#div #nav-btn page:"settings" text:"Truck Settings"]
    [#div #nav-btn page: "social-flow" text: "Social Media"]
    [#div #nav-btn page:"owner" text:"Owner"]
    [#div #nav-btn page: "edit-item" text: "Edit Item"]
    [#div #nav-btn page: "cashier-order" text: "Cashier"]]
```

Highlight the currently active page.

```
search @browser
  nav-btn = [#nav-btn page]

search
  [#app page]

bind @browser
  nav-btn.style += [background: "#606060" color: "white"]
```

Change the current page in response to a click.

```
search @browser @event @session
  view = [#app]
  click = [#click element:[#nav-btn page]]
commit
  view.page := page

```

```css
.app-wrapper {
  display: flex;
  flex-direction: column;
  position: absolute;
  top: 0; left: 0; right: 0; bottom: 0; min-height: 100%;
  padding: 20;
  background: #404040;
}

.page-wrapper {
  align-self: center;
  background: white;
  width: 432;
  height: 768;
  overflow-y: auto;
}

.nav-panel {
  display: flex;
  flex-direction: row;
  flex: 0 0 30px;
  order: -1;
  margin-bottom: 10px;
}

.nav-panel > div {
  color: #eee;
  font-size: 14px;
  line-height: 30px;
  border: 1px solid black;
  border-radius: 6px;
  margin-right: 10px;
  padding: 0px 8px 0px;
}

```

## Employee Menu

Just draw a hamburger icon when the menu is closed.

```
search @browser
  wrapper = [#employee-menu]

bind @browser
  wrapper <- [#div class: "employee-menu" children:
    [#div #employee-menu-btn menu: wrapper class: "ion-navicon-round btn menu-btn"]]
```

When the menu is open, draw a dropdown under the icon.

```
search @browser
  wrapper = [#employee-menu open: true]

search
  open = if [#app truck-open:true] then true else false

bind @browser
  wrapper.children += [#div class: "menu-pane" sort: 3 children:
  [#div #truck-open-btn menu: wrapper class: "btn bubbly" class: "truck-open-btn" open]
  [#div #nav-btn class: "btn" text: "Owner" page: "owner"]
  [#div #nav-btn class: "btn" text: "Truck Settings" page: "settings"]
  [#div #nav-btn class: "btn" text: "Social Media" page: "social-media"]
  [#div #nav-btn class: "btn" text: "Cashier" page: "cashier-order"]
  [#div #nav-btn class: "btn" text: "Queue" page: "order-queue"]
  ]
```

### Toggle dropdown

```
search @event @browser
  [#click element: [#employee-menu-btn menu]]

  open? = if menu.open = true then false
          else true

commit @browser
  menu.open := open?
```

If there's a click that isn't somewhere within an open menu, close it.

```
search @event @browser
  [#click]
  menu = [#employee-menu open: true]
  not([#click element: menu])

commit @browser
  menu.open := none
```

If we navigate away from an open menu, close it.

```
search @event @browser
  [#click element: [#nav-btn]]
  menu = [#employee-menu open: true]

commit @browser
  menu.open := none
```

### Toggle Open

```
search @event @browser
  [#click element: [#truck-open-btn]]

search
  app = [#app]
  open = if app.truck-open = true then false else true

commit
  app.truck-open := open
```

### Styles

```css

.employee-menu { display: flex; flex-direction: column; position: relative; }

.employee-menu .menu-pane { position: absolute; top: 100%; right: 0; padding: 20; margin-top: -10; margin-right: 10; background: white; border: 1px solid #CCC; border-radius: 6px; white-space: pre;}

.employee-menu .truck-open-btn { height: 2em; }
.employee-menu .truck-open-btn:before { align-self: center; }

.employee-menu .truck-open-btn[open="true"]{ background: rgba(10, 255, 10, 0.5); }
.employee-menu .truck-open-btn[open="true"]:before { content: "Open"; }

.employee-menu .truck-open-btn[open="false"]{ background: rgba(255, 10, 10, 0.5); }
.employee-menu .truck-open-btn[open="false"]:before { content: "Closed"; }



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
  menu-item.children += [#div #add-item-btn sort: 10 item class: "btn ion-plus-round" style: [margin-right: -10]]
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
  menu-item.children += [#div #remove-item-btn sort: -1 item class: "btn ion-minus-round" class: [disabled: is(count = 0)] style: [margin-left: -10 margin-right: 10]]
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

Draw the food flags selector for an item

```
search @browser
  wrapper = [#food-flags item]
  toggleable? = if wrapper = [#toggleable] then true else false

search
  flag-entry = [#food-flag flag name]
  icon = if flag-entry.icon then flag-entry.icon else ""
  active? = if item.dietary = flag then true
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

## Button

```css

.btn { display: flex; padding: 10; flex: 0 0 auto; }
.btn:hover { background: #F3F3F3; }
.btn:active { background: #E9E9E9; }
.btn.disabled { color: #999; }

.btn:before { padding-right: 10; }
.btn-icon-start:before { padding-right: 10; }
.btn-icon-end:before { position: relative; left: 100%; padding-right: 0; padding-left: 10;}

.btn.bubbly { position: relative; justify-content: flex-start; flex: 1; border: 1px solid rgba(0, 0, 0, 0.2); border-radius: 8px; }
.btn.bubbly > input { background: transparent; border: none; }
.btn.bubbly + .btn.bubbly { margin-top: 10; }
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

## Image Container

```
search @browser
  image-container = [#image-container]

bind @browser
  image-container += #div
```

An empty image container prompts the user to upload an image

```
search @browser
  image-container = [#image-container not(image)]
  prompt-text = if image-container.prompt then image-container.prompt
                else "Choose an image"

bind @browser
  image-container.children := [#div text: prompt-text]
```

Clicking on an image container opens a dialogue to change the image. Image sources are from a URL, local storage, or captured from the camera (if available).

```
search @browser
  image-container = [#image-container]

search @event
  event = [#click element: image-container]

commit @browser
  image-container += #editing

```

An #image-container in #editing mode displays options for uploading an image
@TODO - add local storage and camera upload options
```
search @browser
  image-container = [#image-container #editing]

bind @browser
  image-container.children := [#input]
```

Enter a URL into the input box to display it

```
search @browser
  image-container = [#image-container #editing children: [#input value]]

search @event
  [#keydown element: image-container, key: "enter"]

commit @browser
  image-container -= #editing
  image-container.image := value
```

If the image container is assigned an image, display it

```
search @browser
  image-container = [#image-container image]

bind @browser
  image-container.children := [#img src: image]

```
