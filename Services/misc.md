# Home Assistant

## Create an iOS widget to open and close the garage door

1. create 'action' in the app, found under Settings/Companion App with a simple name (e.g., "Garage"). Change the description, icon, and colour (you can edit this later)
2. create an automation with an event type `ios.action_fired` , and event data `actionName: "Garage"`, and action `call a service 'Cover: Toggle'`, choose entity `Garage Door`
3. create a widget on iOS, the 'action' should show up with the name, icon, and colour you chose in 1 