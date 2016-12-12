# Food Truck App

## User

### Website Home Page
- Draw the hero image
- Draw the title text on top of the image
- Draw the food truck description
- Draw the Location box
- Draw “Location today:” text
- Draw the minimap
- If no location is given, put a question mark
- If location is given, put a pin
- Clicking the map opens up default map program
- Draw the menu title
- Draw the shopping cart icon in corner
- Default is showing no number next to icon
- As items are added, count the items and display the count next to icon
- Clicking the icon takes you to the checkout page
- Draw menu items
- Draw photo
- Draw item name
- Swiping right adds one to the user cart
- Swiping left removes one from the user cart
- Draw a user queue banner at the very top if an order is pending for them

### Sample Menu
```
commit 
   [#menu name:"Bacon Swiss Burger"
    image:"https://pixabay.com/en/burger-food-meat-tasty-1428948/"
    description:"A half pound burger made from Niman Ranch beef with melted Swiss, thick cut bacon, and housemade aioli and ketchup."
    cost:12]

   [#menu name:"Veggie Burger"
    image:"https://pixabay.com/en/eat-burger-onion-avocado-veggi-1352880/"
    description:"Our totally vegan black bean and portabella mushroom burger, topped with avocado, grilled onion, vegan cheese, and soy mayo, all on a gluten-free bun"
    cost:12
    #gluten-free
    #vegetarian]

   [#menu name:"Chicken Salad Sandwich"
    image:"https://pixabay.com/en/bread-breakfast-brown-business-1239276/"
    description:"Grandma’s chicken salad recipe with grapes and walnuts, served on wholesome 12 grain bread"
    cost:10]

   [#menu
    name:"Charbroiled Chicken Wings"
    image:"https://pixabay.com/en/barbecue-bbq-charcoal-chicken-88340/"
    description:"Brined for 24 hours and coated with our secret spice rub, then grilled until then skin is crispy and the meat is juicy"
    cost:10
    #gluten-free
    #spicy]


   [#menu name:"French Fries"
    image:"https://pixabay.com/en/french-fries-salt-food-923687/"
    description:"Hand-cut Kennebec fries with Cajun seasoning"
    cost:5
    #gluten-free
    #vegetarian]


   [#menu
    name:"Garlic Parmesan Mac & Cheese"
    image:"https://pixabay.com/en/mac-and-cheese-pasta-food-cheese-521447/"
    description:"Penne pasta smothered with aged cheddar, fresh garlic, and parsley"
    cost:10
    #vegetarian]


   [#menu
    name:"Gloria’s Beignets"
    image:"https://pixabay.com/en/fritters-tradition-food-traditional-316488/"
    description:"Our take on the classic - deep-fried yeast doughnuts topped with powdered sugar and drizzled with honey"
    cost:6
    #vegetarian]


   [#menu
    name:"Arnold Palmer"
    image:"https://pixabay.com/en/beverages-cold-drink-fresh-ice-1866476/"
    description:"Freshly brewed iced tea and freshly squeezed lemonade with just a touch of mint! The perfect thirst-quencher."
    cost:4
    #gluten-free
    #vegetarian]
```

### Checkout Page
- Draw the top banner
- “Cart” in center
- Back button in left
- Draw each menu item in cart
- Swiping left removes item
- Clicking item brings up item detail overlay
- Draw pay button
- Clicking pay button takes you to the Stripe payment page/overlay

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
