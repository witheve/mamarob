# Food Truck App

## User

start the app for the user, landing on the menu page. here we create a new
empty order record.

```
commit 
  [#app page:"settings" order:[] settings: []]
```

### Website Home Page

Draw the menu
should include title art
- Draw the hero image
- Draw the title text on top of the image
- Draw the food truck description
we can add the additional imagery to the page if provided
- Draw the Location box
- Draw “Location today:” text
- Draw the minimap
    - If no location is given, put a question mark
      wouldn't it be better to just not draw?
    - If location is given, put a pin
      - Clicking the map opens up default map program

Draw the homepage.

```
search
   [#app page:"homepage"]
   item = [#menu name image cost]

bind @browser
   [#div style: [display:"flex" flex:"0 0 auto" flex-direction:"column"]
    children:
      //checkout
      [#div #checkout style: [display: "flex" flex: "0 0 auto" flex-direction: "row"] children:
        [#div #cart style:[width:30 height: 30 content:"url(assets/shopping-cart-icon-30.png)"]]]
      //menu
      [#div #menu-pane style:[display:"flex" flex:"0 0 auto" flex-direction:"column" overflow-y: "auto", height: "100%"]
       children:
         [#menu-item #description #buyable item]]]
```

Display a quantity badge on the shopping cart.

```
search
  [#app page:"homepage" order]
  [#order-item order item count]
  item-count = sum[value: count per:order given:item]
  item-count > 0

search @browser
  parent = [#div #checkout]

bind @browser
  t = [#div #total-items class: "qty-badge"]
  parent.children += t
  t.text := "{{item-count}}"
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
```
search
   [#app page:"checkout" order]
bind @browser
   [#div #from-checkout-to-home text:"back to menu!" style:[border:"2px solid black"]]
```


display the current order
- Draw a user queue banner at the very top if an order is pending for them
```
search
   [#app page:"checkout" order]
   [#order-item order item count]
   not (count = 0)
   total = sum[value: count * item.cost per:order given:item]
bind @browser
   [#div #checkout-container
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

### Stripe Page
- Draw email field
- Draw CC number field
- Draw MM/YY field
- Draw CVV field
- Draw “Save with Stripe” checkbox
- Draw “Confirm Order!” button
- Confirming order processes payment
- If payment succeeds, add the order from user cart into pending orders
- Go to User Queue page
- If payment fails, display error and stay on Stripe page

### User Queue Page
- Draw “Order Confirmed!” text
- Draw queue status
- If order is not ready yet, display number of pending orders ahead of user
- If order is ready, display “Order #XX is ready!”
- Draw order recap
- Draw back button to go back to home page

### Push Notifications
- If the user has an order pending that becomes ready, push a notification to alert them that their food is ready

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

### Social Media
- Draw top banner
- Draw truck name
- Draw back button
- Clicking takes you back to the home screen
- Draw message text form
- Draw picture icon
- Clicking lets you choose photo from phone storage or live photo
- Draw social media icons
- Facebook
- Twitter
- Instagram
- All off by default
- Clicking turns on that site for posting
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
- Draw “Post!” button
- If the message form contains text, at least one social media icon is toggled on, and a time is selected, make button clickable
- When clicked, if time selected is “Now”, commit the message to social media
- When clicked, if time selected is a later date, add message to social media queue to be posted at that time
- Draw social media queue
- Draw thumbnail photo if there’s a photo on the post
- Draw preview text
- Draw scheduled time to post
- Swipe left to remove from the queue
- Click to parse post details into the social media form and remove message from the queue
- Replace any contents that may already be present with contents from queued post

### Truck Settings

Structure the page

```
search @browser @session
  [#app #owner app page: "settings"]
  integration = [#integration]
  enabled? = if integration = [#enabled] then ""
             else "-outline"

bind @browser
  app.children := [#div children:
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
  [#app app integration page: "integration setup"]
  
bind @browser
  app.children := [#div children:
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

### Stripe Page
- Draw email field
- Draw CC number field
- Draw MM/YY field
- Draw CVV field
- Draw “Confirm Order!” button
- Confirming order processes payment
- If payment succeeds, add the order from user cart into pending orders, clear the cart, and return to order screen
- If payment fails, display error and stay on Stripe page

## The Pass

### Orders Pending Queue

The pending orders queue is a portion of the food truck app that is used onboard the food truck itself to help whoever is working the pass to see what orders are currently being made and which orders are completed and ready for pickup. This portion of the app resembles TodoMVC in a way, as it needs to show a list of pending orders (todos) which can be progressed along according to their states of completion.

Orders are placed by the customer on their mobile device or by the cashier in the truck, but both end up in the @`orders` database, which assigns them the #`order` tag and an order `number` attribute. The contents of the order itself are kept in the [body? Don’t know which attribute would be here], and are visible in the order queue to help keep track of the order, but are not adjustable here. Finally, each order has a `status` flag which affects how the order is displayed in the queue. When they enter the @`orders` database, their default value is `pending`, and can be progressed to `ready` once the order is prepared and finally to `complete` once the customer has picked up the order.

This block draws orders in the browser that have an order number, items in the order, and a status that isn't `done`. It excludes orders that are `done` so that once an order is complete, it is simply no longer drawn on the screen. In the future if we want to add a fancier disappearing or completion animation, etc, this block might have to be changed but for now the simple solution suits our needs. There is also some included `if` logic that draws the right icon with each order and keeps them sorted such that orders ready for pickup are always at the top, and all remain sorted by order number.

```
search
  order = [#order number items status]
  status != "done"
  icon = if status = "ready" then "ion-android-checkmark-circle"
             else if status = "pending" then "ion-android-time"
  order-style = if status = "pending" then "style-pending"
                            else if status = "ready" then "style-ready"
  index = sort[value: (status, number), direction: ("down", "up")]
bind @browser
  [#div sort:index class:("pending-order" order-style) #pending-order order children:
    [#div class:"order-number" text:number]
    [#div class:"order-items" children:
        [#div text:"{{count[given: items, per: (order, items.item)]}}x {{items.item.name}}"]]
    [#div class:(icon "order-status")]]
```

This block is some mocked up order data and will assuredly be replaced by the actual orders database once all the different parts of the app are integrated.

```
search
    veggie = [#menu name:"Veggie Burger"]
  arnold = [#menu name:"Arnold Palmer"]
  burger = [#menu name:"Bacon Swiss Burger"]
  fries = [#menu name:"French Fries"]
commit
    [#order number:10 status:"pending" items:
    [#order-item item:veggie]]
  [#order number:11 status:"pending" items:
    [#order-item item:veggie]
    [#order-item item:fries]]
  [#order number:12 status:"pending" items:
    [#order-item item:veggie]
    [#order-item item:fries]
    [#order-item item:arnold]]
  [#order number:13 status:"pending" items:
    [#order-item item:burger]
    [#order-item item:fries]]
  [#order number:14 status:"pending" items:
    [#order-item item:veggie]
    [#order-item item:arnold]
    [#order-item item:arnold]
    [#order-item item:burger]
    [#order-item item:burger]
    [#order-item item:fries]]
  [#order number:15 status:"pending" items:
    [#order-item item:burger]
    [#order-item item:fries]]
  [#order number:16 status:"pending" items:
    [#order-item item:burger]
    [#order-item item:arnold]
    [#order-item item:fries]]

```

This block progresses the `status` of an order when the order is double clicked.

```
search @browser @event @session
    clicks = [#double-click element:[#pending-order order]]
  newstatus = if order.status = "pending" then "ready"
                        else if order.status = "ready" then "done"
commit
    order.status := newstatus
```

This block links the CSS that styles the order queue.

```
commit @browser
    [#link rel:"stylesheet" type:"text/css" href:"/assets/style.css"]
```


## Sample Menu Data

```
commit
   [#menu name:"Bacon Swiss Burger"
    image:"assets/burger.jpg"
    description:"A half pound burger made from Niman Ranch beef with melted Swiss, thick cut bacon, and housemade aioli and ketchup."
    cost:12]

   [#menu name:"Veggie Burger"
    image:"assets/veggieburger.jpg"
    description:"Our totally vegan black bean and portabella mushroom burger, topped with avocado, grilled onion, vegan cheese, and soy mayo, all on a gluten-free bun"
    cost:12
    #gluten-free
    #vegetarian]

   [#menu name:"Chicken Salad Sandwich"
    image:"assets/sandwich.jpg"
    description:"Grandma’s chicken salad recipe with grapes and walnuts, served on wholesome 12 grain bread"
    cost:10]

   [#menu
    name:"Charbroiled Chicken Wings"
    image:"assets/chickwings.jpg"
    description:"Brined for 24 hours and coated with our secret spice rub, then grilled until then skin is crispy and the meat is juicy"
    cost:10
    #gluten-free
    #spicy]


   [#menu name:"French Fries"
    image:"assets/french-fries.jpg"
    description:"Hand-cut Kennebec fries with Cajun seasoning"
    cost:5
    #gluten-free
    #vegetarian]


   [#menu
    name:"Garlic Parmesan Mac & Cheese"
    image:"assets/mac-n-cheese.jpg"
    description:"Penne pasta smothered with aged cheddar, fresh garlic, and parsley"
    cost:10
    #vegetarian]


   [#menu
    name:"Gloria’s Beignets"
    image:"assets/fritters.jpg"
    description:"Our take on the classic - deep-fried yeast doughnuts topped with powdered sugar and drizzled with honey"
    cost:6
    #vegetarian]


   [#menu
    name:"Arnold Palmer"
    image:"assets/drink.jpg"
    description:"Freshly brewed iced tea and freshly squeezed lemonade with just a touch of mint! The perfect thirst-quencher."
    cost:4
    #gluten-free
    #vegetarian]
```

# Components

## Menu Item

Menu items are present on most pages of the site. On each, they're roughly the same. Different styles and behaviors can be enabled by adding optional tags to the component.

```
search @browser
  menu-item = [#menu-item item]
  mode = if menu-item = [#description] then "description"
         if menu-item = [#instructions] then "instructions"
         if menu-item = [#buyable] then "buyable"
         else "normal"

search
  item = [#menu name image cost]

bind @browser
  menu-item <- [#div class: "menu-item" children:
    [#div class: "item-image" style: [background-image: "url({{image}})"]]
    [#div #menu-item-text item class: "item-text" | mode children:
      [#div class: "item-name" text: name]]
    [#div class: "item-cost" text: cost]]
```

### Description
Adds the items description beneath its name, if available.

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
  [#app page:"homepage" order]
  [#click element:[#menu-item #buyable item]]
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
  [#app page:"homepage" order]
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

### Editable

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
  
commit @browser
  element += #editing
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

### Image Container

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
### Styles
Since the style differences for individual modes are so small, they've all been inlined.

```css
.menu-item {
  display: flex;
  flex-direction: row;
  align-items: center;
  position: relative;
  padding: 10 20;
  max-width: 500;
  min-height: 80;
  background: white;
}

.menu-item:hover { background: #F3F3F3; }
.menu-item:active { background: #E9E9E9; }

.menu-item .item-image {
  align-self: stretch;
  flex: 0 0 90px;
  max-height: 90px;
  margin: 0 -10;
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
  font-size: 1.1rem;
}

.menu-item .item-name { color: #111; }

.menu-item .item-instructions {
  margin-bottom: -1.3em;
  color: #999;
  font-size: 0.8em;
}

.menu-item .item-description {
  font-size: 10pt;
}

.menu-item .item-cost { margin-left: 10; }
.menu-item .item-cost:before { content: "$"; }

.menu-item .qty-badge {
  position: absolute;
  left: 42;
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

## Button

```css

.btn { display: flex; padding: 10; flex: 0 0 auto; }
.btn:hover { background: #F3F3F3; }
.btn:active { background: #E9E9E9; }
.btn.disabled { color: #999; }
```