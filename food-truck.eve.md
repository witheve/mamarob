# Food Truck App

## User

start the app for the user, landing on the menu page. here we create a new
empty order record.
```
commit 
  [#app #owner page:"settings" order:[] settings: []]
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

Draw the menu items with image, name, cost, addition button
```
search
   [#app page:"homepage"]
   item = [#menu name image cost]
bind @browser
   [#div style:[display:"flex" flex:"0 0 auto" flex-direction:"row"]
         children:
         //menu
         [#div #menu-pane style:[display:"flex" flex:"0 0 auto" flex-direction:"column" overflow-y: "auto", height: "100%"]
               children:
               [#div item #item-box
                     style:[display:"flex" height:"200px" flex:"0 0 auto" flex-direction:"row"]
                           children:
                           [#div text:"{{name}} ${{cost}}"]
                           [#div name style:[max-width:"200px" background-size:"cover" display:"block" content:"url({{image}})"]]
                           [#div #add-item item text:"add to order!" style:[border:"2px solid black" width:80]]]]
         //checkout
         [#div #checkout children:
               [#div #cart style:[width:30 content:"url(assets/shopping-cart-icon-30.png)"]]]]
```

Draw the remove-from-cart button for each menu item if this item is in the cart
```
search
   [#app page:"homepage" order]
   order-item = [#order-item order item count]
   count != 0
search @browser
   d = [#div item #item-box]
bind @browser
   d.children += [#div #remove-item order-item  style:[border:"2px solid black" width:80] text:"remove from order! {{count}}" sort:6]
```

Add an item to the order after a click. in the original design this was a swipe right
```
search @browser @event @session
   [#app page:"homepage" order]
   [#click element:[#item-box item]]
   not([#click element:[#remove-item]])
   count = if [#order-item order item count:c] then c + 1 else 1
commit
   order-item = [#order-item order item]
   order-item.count := count
```

Handle the remove item from cart button
in the original design this was swipe left, but we dont have swipe machinery yet
```
search @browser @event @session
   [#app page:"homepage" order]
   [#click element:[#remove-item order-item]]
commit
   order-item.count := order-item.count - 1
```


display the item count next to the shopping cart
```
search
   [#app page:"homepage" order]
   [#order-item order item count]
   item-count = sum[value: count per:order given:item]
search @browser
    parent = [#div #checkout]
bind @browser
   t = [#div #total-items]
   parent.children += t
   t.text := "{{item-count}}"
```

navigate to the checkout page
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


Send the app to the browser

```
search
  app = [#app]

bind @browser
  app.app := [#div]
```

commit the initial app

```
search @browser @session
  [#app #owner app page: "settings"]

bind @browser
  app.children := [#div children:
  	[#div #page-header children:
  		[#editable #truck-name default: "Tap to name your truck"]
      [#button #edit-truck-name text: "edit"]
      [#button #home text: "home"]]
    [#div #page-body children:
      [#image-container #hero-pic prompt: "Tap to change your cover photo"]
      [#editable #description default: "Tap to add a description"]
      [#div #integrations children:
        [#div text: "Twitter"]
        [#div text: "Facebook"]
        [#div text: "Instagram"]
      ]]]
```

Save the name of the truck when the user sets it

```
search @browser
  [#truck-name value]
  
search
  [#app #owner settings]
  
commit
  settings.name := value
```

But editable into editing mode when the button is clicked

```
search @event @browser
  truck-name-editable = [#truck-name]
  [#click element: [#button #edit-truck-name]]
  
commit @browser
  truck-name-editable += #editing
```

- Clicking the home button returns you to the home screen

```
search
  app = [#app]
  
search @event @browser
  [#click element: [#button #home]]
  
commit
  app.page := "homepage"
```

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

## Utility

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
  image-container = [#image-container]
  prompt-text = if image-container.prompt then image-container.prompt
                else "Choose an image"
  
bind @browser
  image-container.children := [#div text: prompt-text]
```

clicking on an image container opens a dialogue to change the image

If the image container is assigned an image, display it
