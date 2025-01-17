# NSPanel ioBroker Integration

## Features

- Thermostat Card
- Entity Card (Temperature, Switches and sensors, the script tries to figure the unit of measurement automatically)
- Grid Card
- Detail Card (only switch and normal dimmer)
- Live update (when value was changed in the backend and the page is currently open)
- Screensaver Page with Time, Date and Weather Information.

## Requirements
- ioBroker
  - MQTT Broker/Client
  - Javascript
  - devices (default)
  - all devices needs to be defined in the devices panel
  - supported device roles are light, dimmer, blind, thermostat

## Note
Currently the names are pulled from the objects data field common.name.de.
If you use a different language please search and replace the "common.name.de" with your language. 
You can find this in the device raw settings.

  
## Installation
- Import this script into the ioBroker javascript instance and choose Typescript.
- Make sure the version of the adapter is not to old.
- Find the config variable and update to your needs.
- The format strings are not used right now.
- Make sure your device is connected with the mqtt instance. I didn't get it working with the sonoff adapter, but I didn't tried it too long.
- Create a state with a mqtt client or create one per hand. The mqtt adapter will not create the state CustomSend
    - you only need to send a dummy message to cmnd/<yourPanel>/CustomSend 
    - then the state will be created 

## Update the screensaver string
The screensaver string which is send to the display looks something like this:
weatherUpdate,?23?11 °C?26?54%?Batterie?4?12 %?PV?23?123W
All fields are seperated by a question mark. In detail the fields are:
weatherUpdate,?Icon?Text?Icon (default humidity)?Text next to the last icon?Text for the left icon on the right side?Icon?Text under the icon?Text for the right icon on the left side?Icon?Text under the icon

See the icons currently usable in the following table:

[Icon Table](../HMI#icons-ids)

You can change the string and devices in the config object.

## Hardware buttons
If you like you can add special pages for the buttons.

First you need to add this rule to Tasmota:

```
Rule2 on Button1#state do Publish tele/%topic%/RESULT {"CustomRecv":"event,button1"} endon on Button2#state do Publish tele/%topic%/RESULT {"CustomRecv":"event,button2"} endon
Rule2
```

## Colors
You can define colors this way and use them later in the PageItem element
```
const BatteryFull: RGB = { red: 96, green: 176, blue: 62 }
const BatteryEmpty: RGB = { red: 179, green: 45, blue: 25 }
```
## The config element in the script which needs to be configured
```
var config: Config = {
    panelRecvTopic: "mqtt.0.tele.WzDisplay.RESULT",       // This is the object where the panel send the data to.
    panelSendTopic: "mqtt.0.cmnd.WzDisplay.CustomSend",   // This is the object where data is send to the panel.
    firstScreensaverEntity: { ScreensaverEntity: "alias.0.Wetter.HUMIDITY", ScreensaverEntityIcon: 26, ScreensaverEntityText: "Luft", ScreensaverEntityUnitText: "%" },
                                                        // Items which should be presented on the screensaver page
    secondScreensaverEntity: { ScreensaverEntity: "alias.0.Wetter.PRECIPITATION_CHANCE", ScreensaverEntityIcon: 19, ScreensaverEntityText: "Regen", ScreensaverEntityUnitText: "%" },
    thirdScreensaverEntity: { ScreensaverEntity: "alias.0.Batterie.ACTUAL", ScreensaverEntityIcon: 34, ScreensaverEntityText: "Batterie", ScreensaverEntityUnitText: "%" },
    fourthScreensaverEntity: { ScreensaverEntity: "alias.0.Pv.ACTUAL", ScreensaverEntityIcon: 32, ScreensaverEntityText: "PV", ScreensaverEntityUnitText: "W" },
    screenSaverDoubleClick: false,                        // Doubletouch needed for leaving screensaver.                                                           
    timeoutScreensaver: 15,                               // Timeout for screensaver
    dimmode: 8,                                           // Display dim
    locale: "de_DE",                                      // not used right now
    timeFormat: "%H:%M",                                  // not used right now
    dateFormat: "%A, %d. %B %Y",                          // not used right now
    weatherEntity: "alias.0.Wetter",
    defaultColor: Off,                                    // Default color of all elements
    defaultOnColor: RGB,                                  // Default on state color for items
    defaultOffColor: RGB,                                 // Default off state color for page
    temperatureUnit: "°C",                                // Unit to append on temperature sensors
<<<<<<< HEAD
    pages: [Wohnen, Strom,
        {
            "type": "cardThermo",
            "heading": "Thermostat",
            "useColor": true,
            "items": [<PageItem>{ id: "alias.0.WzNsPanel" }]
=======
    pages: [
        {
            "type": "cardEntities",                       // card type (cardEntities, cardThermo)
            "heading": "Testseite",                       // heading
            "useColor": false,                             // should colors be enabled on this page, can be overridden in PageItem
            "items": [                                    // items array (up to 4 on cardEntities, 1 for cardThermo)
                <PageItem>{ id: "alias.0.Rolladen_Eltern" },                // device which must be configured in the device panel. Use only the folder for the device, not the set, get states ...
                <PageItem>{ id: "alias.0.Erker" },
                <PageItem>{ id: "alias.0.Küche", useColor: true },
                <PageItem>{ id: "alias.0.Wand", useColor: true  }

            ]
        },
        {
            "type": "cardEntities",
            "heading": "Strom",
            "useColor": true,                             // should colors be enabled on this page, can be overridden in PageItem
            "items": [
                <PageItem>{ id: "alias.0.Netz" },
                <PageItem>{ id: "alias.0.Hausverbrauch", icon: 4, interpolateColor: true, offColor: BatteryFull, onColor: Red , maxValue: 1000 },
                <PageItem>{ id: "alias.0.Pv" },
                <PageItem>{ id: "alias.0.Batterie", icon: 34, interpolateColor: true, offColor: BatteryEmpty, onColor: BatteryFull }

            ]
        },
        {
            "type": "cardThermo",
            "heading": "Thermostat",
            "useColor": false,                            // should colors be enabled on this page, can be overridden in PageItem
            "item": "alias.0.WzNsPanel"                   // Needs to be a thermostat in the device panel
>>>>>>> 8a48ff35d408a7712a3052ee3cf8fc84e8b699c7
        }
    ],
    button1Page: button1Page,                             // A cardEntities, cardThermo or nothing. This will be opened when pressing button1 
    button2Page: button2Page                              // you guess it 
};
```

The pageItem element:
```
type PageItem = {
    id: string,                             // the element in ioBroker devices 
    icon: (string | undefined),             // the icon which should be displayed instead of the default detected. (not implemented)
    onColor: (RGB | undefined),             // the color the item will get when active
    offColor: (RGB | undefined),            // the color the item will get when inactive
    useColor: (boolean | undefined)         // override colors, only Grid pages has colors enabled per default
    interpolateColor: (boolean | undefined),// fade between color on and off, useColor on Page or PageItem must be enabled
    minValue: (number | undefined),         // the minimum value for the fade calculation, if smaller the minimum value will be used
    maxValue: (number | undefined),         // the maximum value for the fade calculation, if larger the maximum value will be used
    buttonText: (string | undefined)        // the Button Text, default is "Press"
}
```


If you want you can create dedicated objects, so you don't need to declare them again. Then you can use tehm in the pages array and button pages.

```
var button1Page: PageGrid =
{
    "type": "cardGrid",
    "heading": "Radio",
    "useColor": true,                             // should colors be enabled on this page, can be overridden in PageItem
    "items": [
        <PageItem>{ id: "alias.0.Radio.NJoy" },
        <PageItem>{ id: "alias.0.Radio.Delta_Radio" },
        <PageItem>{ id: "alias.0.Radio.NDR2" },
    ]
};
```

Pages array can look like this, so you can add the pages as object or define them in the array itself. This is up to you.

```
pages: [
        button1Page,
        {
            "type": "cardEntities",
            "heading": "Strom",
            "useColor": true,                             // should colors be enabled on this page, can be overridden in PageItem
            "items": [
                <PageItem>{ id: "alias.0.Netz" },
                <PageItem>{ id: "alias.0.Hausverbrauch" },
                <PageItem>{ id: "alias.0.Pv" },
                <PageItem>{ id: "alias.0.Batterie" }
            ]
        }]
```
