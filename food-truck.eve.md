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
we can add the additional imagery to the page if provided
- Draw the Location box
- Draw “Location today:” text
- Draw the minimap
- If no location is given, put a question mark
wouldn't it be better to just not draw?
- If location is given, put a pin
- Clicking the map opens up default map program

Draw the homepage.

```eve
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

```eve
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
```eve
search
   [#app page:"checkout" order]
bind @browser
   [#div #from-checkout-to-home text:"back to menu!" style:[border:"2px solid black"]]
```


display the current order
- Draw a user queue banner at the very top if an order is pending for them
```eve
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

```
search
  [#app page:"user-queue"]
  order = [#order #my-order number items status]
  (ahead, message, my-status) = if status:"ready" then ("Your order is ready!","","ready")
    else if count[given: [#order status:"pending" number < order.number]] = 1 then ("1","order ahead of you!","ahead")
    else if ahead = count[given: [#order status:"pending" number < order.number]] then (ahead,"orders ahead of you!","ahead")
    else ("0","orders ahead of you!","ahead")
bind @browser
  [#div class:"my-order" children:
    [#div class:"order-confirmed" text:"Order confirmed!"]
    [#div class:my-status text:ahead]
    [#div class:"msg" text:message]
    [#div class:"my-order-number" text:"Order #{{number}}"]
    [#div class:"my-food" text:"{{count[given: items, per: (order, items.item)]}}x {{items.item.name}}"]
    [#div class:"spacer"]
  [#div class:"back-btn" text:"❮ Back to Mama Rob's"]]
```

```css
.my-order {
  border: 1px solid #000000;
  border-radius: 6px;
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

```

## The Pass

### Orders Pending Queue

The pending orders queue is a portion of the food truck app that is used onboard the food truck itself to help whoever is working the pass to see what orders are currently being made and which orders are completed and ready for pickup. This portion of the app resembles TodoMVC in a way, as it needs to show a list of pending orders (todos) which can be progressed along according to their states of completion.

Orders are placed by the customer on their mobile device or by the cashier in the truck, but both end up in the @`orders` database, which assigns them the #`order` tag and an order `number` attribute. The contents of the order itself are kept in the [body? Don’t know which attribute would be here], and are visible in the order queue to help keep track of the order, but are not adjustable here. Finally, each order has a `status` flag which affects how the order is displayed in the queue. When they enter the @`orders` database, their default value is `pending`, and can be progressed to `ready` once the order is prepared and finally to `complete` once the customer has picked up the order.

This block draws orders in the browser that have an order number, items in the order, and a status that isn't `done`. It excludes orders that are `done` so that once an order is complete, it is simply no longer drawn on the screen. In the future if we want to add a fancier disappearing or completion animation, etc, this block might have to be changed but for now the simple solution suits our needs. There is also some included `if` logic that draws the right icon with each order and keeps them sorted such that orders ready for pickup are always at the top, and all remain sorted by order number.

```eve
search
  [#app page:"order-queue"]
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

```eve
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
  [#order #my-order number:15 status:"pending" items:
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

### Sample Menu Data
```
commit
   [#menu name:"Bacon Swiss Burger"
    image:"assets/burger.jpg"
    description:"A half pound Niman Ranch burger with melted Swiss, thick cut bacon, and housemade aioli and ketchup."
    cost:12]

   [#menu name:"Veggie Burger"
    image:"assets/veggieburger.jpg"
    description:"Our totally vegan black bean and portabella mushroom burger on a gluten-free bun."
    cost:12
    #gluten-free
    #vegetarian]

   [#menu name:"Chicken Salad Sandwich"
    image:"assets/sandwich.jpg"
    description:"Grandma’s chicken salad recipe with grapes and walnuts, served on wholesome 12 grain bread."
    cost:10]

   [#menu
    name:"Charbroiled Chicken Wings"
    image:"assets/chickwings.jpg"
    description:"Brined, coated with our secret spice rub, then grilled until then skin is crispy and the meat is juicy."
    cost:10
    #gluten-free
    #spicy]


   [#menu name:"French Fries"
    image:"assets/french-fries.jpg"
    description:"Hand-cut Kennebec fries with Cajun seasoning."
    cost:5
    #gluten-free
    #vegetarian]


   [#menu
    name:"Garlic Parmesan Mac & Cheese"
    image:"assets/mac-n-cheese.jpg"
    description:"Elbow pasta smothered with aged cheddar, fresh garlic, and parsley."
    cost:10
    #vegetarian]


   [#menu
    name:"Gloria’s Beignets"
    image:"assets/fritters.jpg"
    description:"Our take on the classic - deep-fried yeast doughnuts topped with powdered sugar and drizzled with honey."
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

## Page view control
```
search
  [#app page]
bind @browser
  [#div class:"nav-panel" children:
    [#div #nav-btn page:"homepage" text:"Home"]
    [#div #nav-btn page:"checkout" text: "Checkout"]
    [#div #nav-btn page:"user-queue" text:"User Queue"]
    [#div #nav-btn page:"order-queue" text:"Order Queue"]]
```

```
search @browser @event @session
  view = [#app]
  click = [#click element:[#nav-btn page]]
commit
  view.page := page

```

```css
.nav-panel {
  display: flex;
  flex-direction: row;
  flex: 0 0 30px;
  order: -1;
  margin-bottom: 10px;
}

.nav-panel > div {
  font-size: 14px;
  line-height: 30px;
  border: 1px solid black;
  border-radius: 6px;
  margin-right: 10px;
  padding: 0px 8px 0px;
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
