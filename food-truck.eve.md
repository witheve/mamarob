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
- Draw the menu title

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
- Draw top banner
- Draw name
- Clicking brings up a text form filled with the current truck name
- Draw back button
- Clicking returns you to the home screen
- Draw hero pic
- Clicking lets you choose photo from URL, phone storage, or live photo
- Draw description form
- Draw integration buttons
- If all credentials are present, icon is lit up
- Else, icon is grayed out
- Clicking brings up credentials overlay
- Draw credential forms
- Draw accept button
- Clicking applies entered credentials and reverts to truck settings screen

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
- Draw orders pending
- Draw order number
- Draw order summary
- Draw completion icon
- Swipe/click/double click to progress status
- When in the queue to be made, order should be gray, icon should be pending icon
- When order is complete, order should be lit up, icon should be finished icon
- When order has been served, order should be removed from the queue


### Sample Menu Data
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
