# Food Truck App

```css
{for a good time, leave this here}
```

## Page Template
### Starting Page
Set the starting target window to the home page.

```
search @browser
  window = [#app-window]

commit @browser
  window.target := "Truck Settings"
```

### Pages
Add the different pages.

```
commit @debug
  [#page target: "Home"]
  [#page target: "Checkout"]
  [#page target: "Stripe"]
  [#page target: "User Queue"]
  [#page target: "Owner"]
  [#page target: "Edit Item"]
  [#page target: "Add Item"]
  [#page target: "Order Queue"]
  [#page target: "Cashier"]
  [#page target: "Truck Settings"]
  [#page target: "Social Media"]
```

### The App Window
The main app window and accompanying buttons.

```
search @debug
  [#page target]

bind @browser
  [#row style: [margin-bottom: 20] children:
            [#a class: "button inset pill" href: "#/{{target}}" style: [padding: 0] children: [#nav-button target text: target style: [padding: "0.5em 1em"]]]]
  [#window class: "app-window" #default #app-window]
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

## User View
### User Home Page
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
    [#div class:"hero-image" style: [background-image: "url({{settings.hero-image}})"]
    ]

    // Location
    [#div class: "location-row" children:
      [#h3 class:"location-now-text" text: "Location Now:"]
      [#img #mapz class:"location-map" src:"https://goo.gl/euvdoF"]
    ]

    // Menu
    [#div class: "menu-header" children:
      [#h3 text: "Menu" class: "menu-title"]
      [#button #cart-btn target: "Checkout" icon: "android-cart"]
    ]
    [#div #menu-pane class:"menu-pane" children:
      [sort:menu-sort #menu-item #flags #buyable item]
    ]
  ]
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

### Checkout Page
Display the nav button, which is unconditional.

```eve
search @browser
  window = [#app-window target: "Checkout"]

search
   [#app order]
bind @browser
   window.children += [#button target: "Home" icon: "ios-arrow-back" text:"Back to Menu!"]
```

Display the current order.

```eve
search @browser @session
  window = [#app-window target: "Checkout"]
  [#app order]
  [#order-item order item count]
  not (count = 0)
  total = sum[value: count * item.cost per:order given:item]
  item = [#menu not(#disabled) name image cost menu-sort]

bind @browser
  window.children += [#div #checkout-container class:"checkout"
    style:[flex-direction:"column" display:"flex"]
    children:
      [#div class:"checkout-food" children:
        [#menu-item #buyable item]]
  ]
  window.children += [#div class:"checkout-info" children:
      [#div class:"checkout-total" text:"Total: ${{total}}"]
      [#button target: "Stripe" class:"checkout-pay-btn" text:"Place Order!"]
  ]
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
    [#button target: "User Queue" class:"stripe-pay-btn" text:"Pay ${{total}}"]
  ]
```

This block commits the current cart to a new order record, which then appears in the Orders Pending Queue, and empties the cart to "reset" it.

```
search @browser @event @session
  [#app order]
  cart = [#order-item order item count]
  count > 0
  next-order = if c = count[given: [#order]] then c + 1
               else 1
  window = [#app-window target: "Stripe"]
  [#click element:[class:"stripe-pay-btn"]]

commit
  [#order #my-order number:next-order status:"pending" items:
    [#order-item item count]]
  cart := none
```

### User Queue Page
Draw the user queue page if they have an order tagged #`my-order`.

```
search @browser
  window = [#app-window target: "User Queue"]

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
  window = [#app-window target: "User Queue"]

search
  not ([#order #my-order])

bind @browser
  window.children += [#div class:"my-order" children:
    [#div class:"order-confirmed" text:"Oops! Looks like you haven't ordered yet."]
    [#div class:"spacer"]
  [#button target: "Home" text:"❮ Back to Mama Rob's"]]
```

### Styles
Styling for the user view pages.

```css
.hero-image {
  height: 320px;
  background-size: cover;
}

.app-window .location-row {
  align-items: center;
  height: 5em;
  overflow: hidden;
  background: #eee;
  display: flex;
  flex-direction: row;
}

.app-window .location-now-text {
  flex: 1 0 auto;
  padding: 0px 20px;
  margin: 0px;
  font-size: 2em;
  font-weight: 200;
}

.app-window .location-map {
  flex: 1;
  background-size: cover;
}

.menu-header {
  position: relative;
  justify-content: center;
  align-items: center;
}

.menu-pane {
  flex: 0 0 auto;
  flex-direction: column;
}

.app-window {
  align-self: center;
  background: white;
  width: 432;
  height: 768;
  overflow-y: auto;
}

.menu-title {
  margin: 10 0;
  font-size: 2em;
  font-weight: 200;
  text-decoration: underline;
  text-align: center;
}

.ion-android-cart {
  font-size: 24px;
  position: absolute;
  top: 2px;
  right: 0px;
  display: flex;
  flex: 1 1 auto;
  width: 90px;
  align-items: flex-end;
}

.ion-android-cart::before {
  font-size: 24px;
  padding-left: 10px;
}

.app-window {
  position: relative;
  overflow-y: scroll;
}

.checkout {
  max-height: 620px;
  overflow-y: scroll;
}

.checkout-info {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding-bottom: 20px;
  background-color: #fff;
  border-top: 1px solid #d1d1d1;
  z-index: 10;
}

.app-window .checkout-total {
  font-size: 1.2rem;
  align-self: center;
  padding: 10px 0px;
}

.app-window .checkout-pay-btn {
  border: 1px solid #307ac5;
  border-radius: 4px;
  padding: 10px 50px;
  background: linear-gradient(#46b1e5, #3099db);
  color: white;
  align-self: center;
  bottom: 0px;
}

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
### Owner App Home

```
search @browser
  window = [#app-window target: "Owner"]

search
   item = [#menu name image cost menu-sort]

bind @browser
  window <- [children:
  [#div class:"owner-tools-pane" children:
    [#button target: "Social Media" text: "Social Media"]
    [#button target: "Add Item" text: "Add New Menu Item"]
    [#editable class: "ion-android-search btn bubbly" default: "Enter your address"]]

  [#div class: "owner-menu-header" children:
    //[#div class: "ion-arrow-left-c btn-icon-start" text: "off"]
    [#div class: "owner-menu-text" text: "Menu"]
    //[#div class: "ion-arrow-right-c btn-icon-end" text: "on"] style: [padding-right: 10]
  ]

   [#div #menu-pane class:"owner-menu-pane" children:
     [sort:menu-sort class:"owner-page-items" #menu-item #toggleable #modifiable #flags #sortable item]]]

```

When the new item button is clicked, add a new item to the database and make it the currently active item

```
search @event @browser @session
  [#click element: [#button target: "Add Item"]]
  app = [#app]
  
commit  
  item = [#menu]
  app.current-item := item
```
### Edit Item

```
search @browser
  window = [#app-window target: "Edit Item"]

search
  [#app current-item: item]
  name = if item.name then item.name else ""
  image = if item.image then item.image else ""
  cost = if item.cost then item.cost else 0
  description = if item.description then item.description else ""
  state = if wrapper = [#owner-view] then "input"
          else "input"

bind @browser
  window.class += "edit-item"
  window <- [children:
    [#div class: "item-top-bar" children:
      [#img src: image style: [height: "100%" background: "#DDD"]]
  [tag: state form: "edit-item" field: "name" class: "item-name" default: "name" value: name text: name]]

    [#div class: "item-middle-bar" children:
      [#div class:"item-desc-price" children:
        [#h3 text: "Description"]
        [tag: state form: "edit-item" field: "description" class: "btn bubbly item-description" default: "description" value: description text: description]
        [#h3 text: "Price"]
        [tag: state form: "edit-item" field: "cost" class: "btn bubbly item-price" default: "price" value: cost text: cost]]

      [#food-flags #toggleable item]]
    [#div class: "item-bottom-bar" class: [                              hidden: toggle[value: true]] children:
      [#button #submit-form form: "edit-item" class: "edit-submit" item icon: "checkmark"]
      [#button #delete-item-btn item class: "edit-delete" icon: "trash-a"]]]
```



When the submit button is clicked, save the current form state to the item, then clear it.

```
search @event @browser
  [#click element: [#submit-form form item]]
  [form: "edit-item" field: "name" value: name]
  [form: "edit-item" field: "cost" value: cost]
  [form: "edit-item" field: "cost" value: description]
  window = [#app-window target: "Edit Item"]

search
  app = [#app]

commit @session @browser
  [#div text: "foo"]
  item.name := name
  item.description := description
  item.cost := cost
  window.target := "Owner"
  app.current-item := none
```

When the delete button is clicked, remove the current item from the menu and renumber any items below it for sorting purposes.

```
search @event @browser @session
  [#click element: [#delete-item-btn item:clicked-item]]
  items-below = [#menu menu-sort > clicked-item.menu-sort]
  window = [#app-window target: "Edit Item"]

commit @session @browser
  clicked-item := none
  items-below.menu-sort := items-below.menu-sort - 1
  window.target := "Owner"
```

### Add Item

```
search @browser
  window = [#app-window target: "Add Item"]

search
  [#app current-item: item]
  
bind @browser
  window.class += "edit-item"
  window <- [children:
  [#div class: "item-top-bar" children:
  [#img src: "" style: [height: "100%" background: "#DDD"]]
  [#input form: "add-item" field: "name" class: "item-name" placeholder: "Item Name"]]
[#div class: "item-middle-bar" children:
  [#div class:"item-desc-price" children:
    [#h3 text: "Description"]
    [#input form: "add-item" field: "description" class: "btn bubbly item-description" placeholder: "A short item description"]
    [#h3 text: "Price"]
    [#input form: "add-item" field: "cost" class: "btn bubbly item-price" placeholder: "$"]]

  [#food-flags #toggleable item]]
[#div class: "item-bottom-bar" children:
  [#button #submit-form form: "add-item" class: "submit-button" icon: "checkmark"]
  [#button #delete-item-btn class: "delete-button" icon: "android-cancel"]]]

```

```
search @event @browser @session
  [#click element: [#submit-form form: "add-item"]]
  [form: "add-item" field: "name" value: name]
  [form: "add-item" field: "description" value: description]
  [form: "add-item" field: "cost" value: cost]
  app = [#app current-item: item]
  menu-sort = count[given: [#menu]] + 1
  window = [#app-window target: "Add Item"]

commit @browser
  window.target := "Owner"
  
commit
  item <- [image: "assets/veggieburger.jpg" name description cost menu-sort]
  app.current-item := none
```


### Order Queue

The pending orders queue is a portion of the food truck app that is used onboard the food truck itself to help whoever is working the pass to see what orders are currently being made and which orders are completed and ready for pickup. This portion of the app resembles TodoMVC in a way, as it needs to show a list of pending orders (todos) which can be progressed along according to their states of completion.

Orders are placed by the customer on their mobile device or by the cashier in the truck, but both end up in the @`orders` database, which assigns them the #`order` tag and an order `number` attribute. The contents of the order itself are kept in the [body? Don’t know which attribute would be here], and are visible in the order queue to help keep track of the order, but are not adjustable here. Finally, each order has a `status` flag which affects how the order is displayed in the queue. When they enter the @`orders` database, their default value is `pending`, and can be progressed to `ready` once the order is prepared and finally to `complete` once the customer has picked up the order.

This block draws orders in the browser that have an order number, items in the order, and a status that isn't `done`. It excludes orders that are `done` so that once an order is complete, it is simply no longer drawn on the screen. In the future if we want to add a fancier disappearing or completion animation, etc, this block might have to be changed but for now the simple solution suits our needs. There is also some included `if` logic that draws the right icon with each order and keeps them sorted such that orders ready for pickup are always at the top, and all remain sorted by order number.

```eve
search @browser
  window = [#app-window target: "Order Queue"]

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
text

```
search @browser
  window = [#app-window target: "Cashier"]

search
  [#app settings]
  item = [#menu not(#disabled) name image cost menu-sort]

bind @browser
  window.class += "cashier-order"
  window <- [children:
    [#header class: "cashier-header"  children:
      [#h1 text: settings.name]
      [#employee-menu]]

    [#div class: "menu-items" children:
      [sort:menu-sort #menu-item #buyable item]]

    [#div class: "flex-spacer"]
    [#div class: "cashier-bottom-bar" children:
      [#button class:"checkout-pay-btn" target: "Stripe" #finalize-order-btn text: "Finalize Order"]]]
```

### Social Media
Draw the social media page. There are two cases we want to handle. If the truck isn't set up or there are no #enabled social media #integrations, then we should display a message prompting the owner to add those in.

```
search @browser @session
  window = [#app-window target: "Social Media"]
  not([#app settings: [truck-name]]
  [#integration #enabled])

bind @browser
  window.children := [#div children:
    [#div text: "Set up your truck to use social features"]
    [#button target: "Truck Settings" text: "Truck Settings"]]
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
  window = [#app-window target: "Truck Settings"]
  // Get settings to populate fields
  [#app settings]
  // Integrations
  integration = [#integration]
  enabled? = if integration = [#enabled] then ""
             else "-outline"
  name = if settings.name then settings.name else ""
  image = if settings.hero-image then settings.hero-image else ""
  description = if settings.description then settings.description else ""

bind @browser
  window.children := [#div class: "settings" children:
    [#h3 text: "Truck Name"]  	
    [#input #bubbly form: "settings" field: "truck name" placeholder: "name" value: name]
    [#h3 text: "Hero Image"]
    [#input #bubbly form: "settings" field: "hero image" placeholder: "image" value: image]
    [#h3 text: "Description"]
    [#input #bubbly form: "settings" field: "description" placeholder: "description" value: description]
    [#h3 text: "Integrations"]
    [#div #integrations children:
      [#button #integration class: "integration-button" integration icon: "social-{{integration.name}}{{enabled?}}"]]
    [#button target: "Owner" #save-settings class: "submit-button" icon: "checkmark"]]
```

Save the name of the truck when the user sets it

```
search @browser @session @event
  [#click element: [#save-settings]]
  [form: "settings" field: "truck name" value: name]
  name != settings.name

search
  [#app settings]

commit
  settings.name := name
```

Save truck description to settings when it changes

```
search @browser @session @event
  [#click element: [#save-settings]]
  [form: "settings" field: "description" value: description]
  description != settings.description

search
  [#app settings]

commit
  settings.description := description
```

Clicking on an integration opens up a page to enter credentials.

```
search @event @browser @session
  [#click element: [#integration integration]]
  window = [#app-window]
  app = [#app]

commit @browser
  window.target := "Integration Setup"
  app.integration := integration
```

Draw credential forms

```
search @browser @session
  window = [#app-window target: "Integration Setup"]
  [#app integration]
  
bind @browser
  window.children := [#div class: "settings" children:
    [#div class: "ion-social-{{integration.name}}"]
    [#h3 text: "Sign in to {{integration.name}}"]
    [#input #bubbly #username placeholder: "Username"]
    [#h3]
    [#input #bubbly #password type: "password" placeholder: "Password"]
    [#br]
    [#button class: "submit-button" #submit-credentials text: "Submit"]
    [#button class: "delete-button" target: "Truck Settings" text: "Cancel"]
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

### Styles
Styling for the owner view pages.eee

```css
.owner-tools-pane {
  padding: 10;
}

.owner-menu-header {
  padding: 10px 20px;
  background: #eee;
}

.owner-menu-text {
  text-align: center;
}

.owner-menu-pane {
  flex: 0 0 auto;
  flex-direction: column;
}

.item-desc-price {
  flex: 1;
}

.edit-item {
  display: flex;
  flex-direction: column;
}

.edit-item .item-top-bar {
  position: relative;
  display: flex;
  flex-direction: row;
  align-items: flex-start;
  height: 140;
  padding: 20;
}

.edit-item .item-top-bar > img {
  border-radius: 6px;
}

.edit-item .item-name {
  flex: 1;
  padding: 0 20;
  font-size: 1.5em;
  border: 1px solid #999;
  width: 200px;
  margin-left: 10px;
  padding: 10px;
  border-radius: 10px;
}

.edit-item .submit-btn {
  position: absolute;
  right: 0;
  top: 0;
  padding-top: 30;
  padding-right: 20;
}

.edit-item .item-middle-bar {
  display: flex;
  flex-direction: row;
  align-items: flex-start;
  padding: 0 20;
}

.edit-item .item-description {
  height: 5em;
  justify-content: flex-start !important;
}

.edit-item .item-price {
  width: 5em;
  justify-content: flex-start !important;
  padding-left: 20;
}

.edit-item .item-price:before {
  display: block;
  content: "$";
}

.edit-item .item-bottom-bar {
  display: flex;
  flex-direction: row;
  justify-content: flex-end;
  margin-top: 10;
}

.edit-item .item-delete-btn {
  display: flex;
  flex: 0 0 auto;
  padding-right: 20;
  padding-bottom: 30;
}

.submit-button {
  height: 50px;
  width: 50%;
  margin: 10px;
  border-radius: 10px;
  color: white !important;
  background: rgb(121,183,79) !important;
}

.delete-button {
  height: 50px;
  width: 50%;
  margin: 10px;
  border-radius: 10px;
  color: white !important;
  background: rgb(233,81,81) !important;
}

.cashier-header {
  display: flex;
  flex-direction: column;
  align-items: center;
  flex: 0 0 auto;
}

.cashier-order {
  display: flex;
  flex-direction: column;
}

.cashier-order header {
  border-bottom: 1px solid #DDD;
  box-shadow: 0 6px 6px -3px white;
  z-index: 1;
}

.cashier-order h1 {
  margin: 10;
}

.cashier-order .menu-btn {
  align-self: flex-start;
  padding: 20;
  font-size: 2em;
}

.cashier-order .menu-items {
  position: relative;
  overflow-y: auto;
}

.cashier-order .cashier-bottom-bar {
  flex: 1 0 65px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 10 0;
  border-top: 1px solid #DDD;
  box-shadow: 0 -6px 6px -3px white;
  z-index: 1;
}

.hidden {
 visibility: hidden; 
}

.integration-button {
 border: 1px solid #999 !important;
 margin: 10px;
 border-radius: 5px;
}

.settings {
 padding: 20px; 
}
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
  window = [#app-window]

search
  app = [#app]

commit @session @browser
  window.target := "Edit Item"
  app.current-item := item
```

```
search @event @browser
  [#click element: [#menu-item #modifiable item]]
  [#click element: [class:"item-name"]]
  window = [#app-window]

search
  app = [#app]

commit
  window.target := "Edit Item"
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

.menu-item .ion-chevron-up {
  margin-bottom: 10px;
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
  cursor: pointer;
}
.food-flags .food-flag {
  border: 1px solid #999;
  margin-bottom: 10px;
  border-radius: 10px;
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




## Sample Data
### Sample Truck Settings

```eve
search
  orders = [#order]

commit @browser
  [#app-window target: "Home"]

commit
  [#app order: orders, settings: [name: "Mama Rob's" hero-image: "http://i.imgur.com/1lcSHmQ.jpg" description: "Fun food at affordable prices"]]
```

### Sample Orders


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

The following block adds a default order.

```
search
  veggie = [#menu name:"Veggie Burger"]
  
commit
  [#order #my-order number:8 status:"pending" items:
    [#order-item item:veggie count:1]]
```

default current item

```
search
  current-item = [#menu name: "Veggie Burger"]
  
commit
  //[#app current-item]
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

```
commit @browser
  [#editable class: "foobar" default: "click to edit" value: "The form attribute is a simple way to manage user data submission. By giving each input in a form the same value for the form attribute and a unique name, you can easily retrieve the values of all the inputs without the"]
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

Twitter Login

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

```css

```
