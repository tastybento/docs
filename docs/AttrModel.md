Great Attribute Modeling
========================

When defining attributes in a Profile for Afero devices, it’s important to model the data in a way that supports easy client design and implementation, as well as good reporting. This page will lay out some best practices and an example to help you define a comprehensive and efficient attribute set for your device.

Modeling Guidelines
-------------------

Below are four guidelines to help you take maximum advantage of the Afero attribute model.

### Start with the Client Application

The best way to implement an efficient data model is to start with the client application team. Build the list of things they want to represent in the user interface and work backwards to determine the attributes required to support those features.

For most applications, you can simply take the interesting local variables you have in your firmware and make them Afero attributes; such as a Boolean that holds whether the device is powered on or off.

### Store One Value Per Attribute

In general, it’s a good idea to store only one value per attribute. This makes for a cleaner history and reporting data, as well as making things better for the client applications. The one time you may want to consider storing multiple values in an attribute is when they are all part of a single transaction. For example, a command attribute that takes a command type followed by command parameters is better sent as one attribute so that all the data can be parsed at the same time.

Storing things like comma-separated strings or json objects as attribute values will definitely limit what can be done when reporting on those values. It will also increase the size of the attribute value and reduce response time.

### Use Smallest Size Needed

Whenever possible, express data in the smallest form possible. Usually this just means using the correct data type for the actual value. Expressing data types like numbers is not only inefficient from a size point of view, it complicates comparisons when doing reporting over a large data set.

### Minimize Update Frequency

For frequency, no more than one write/update per second is preferable. Much slower than that, if possible, is recommended. Remember, a high update rate can affect your overall solution cost. If a device reports attribute values rapidly during operation even if those values haven’t changed, those updates will cause excessive traffic between the MCU and ASR. Also, there’s no need to write a group of related attributes if only one has changed. Only send updates that are meaningful to you.

Writing to ASR does not mean the attribute has been synchronized with the Cloud; synchronization may take some time depending on the radio network or connectivity in general. So if you overwrite an attribute before it’s synchronized, the Cloud will not know about it (with an exception for attributes defined as [latch](AttrDef#Latching)). Read more about allowing adequate attribute write timing in [Don’t Forget to Call af\_lib\_loop()](MCUCodingTips#MCU-DoNotForgetafLib).

The MCU can sample as often as desired; there’s no limit on attribute reads since they come directly from ASR with no Cloud overhead. But again, **only push useful data**.

Modeling Method
---------------

One way to build a attribute model for your device is to organize all the attribute information in a table, one attribute per row. Assign each attribute a number using column _1_. MCU attributes can be numbered 1-1023.
<table>
    <thead>
        <tr>
            <th><em>1</em><br>No.</th>
            <th><em>2</em><br>Feature</th>
            <th><em>3</em><br>Description</th>
            <th><em>4</em><br>States</th>
            <th><em>5</em><br>Data Type</th>
            <th><em>6</em><br>Writability</th>
            <th><em>7</em><br>Default Value</th>
            <th><em>8</em><br>UI Control</th>
            <th><em>9</em><br>Send Frequency</th>
            <th><em>10</em><br>Dependencies</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>1</td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
        </tr>
        <tr>
            <td>2</td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
        </tr>
        <tr>
            <td>&vellip;</td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
        </tr>
        <tr>
            <td>1023</td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
        </tr>
    </tbody>
</table>

The information entered in the table will answer these questions:

*   Columns _2-4_ - What are all the device features and possible states?
*   Column _5_ - What data type is best suited for holding the state values?
*   Column _6_ - What feature states will the end user either view (read-only) or control (read/write) using the mobile app?
*   Column _7_ - Are there default values for any of the feature settings?
*   Column _8_ - How will the end user view or control each feature setting in the mobile app UI? (_column 8_)
*   Column _9_ - How frequently will an attribute value be sent to the Cloud; what event will trigger the send?
*   Column _10_ - Does the availability of a feature depend on the selection of any other features?

Smart Toaster Oven Example
--------------------------

In this example we’ll define the attributes for a smart midline toaster oven. We’ll step through the toaster oven features (a.k.a. attributes), numbering and naming them so we can use that information when defining these attributes in the Afero Profile Editor. The full table is shown at the end of the example in [The Toaster Oven Data Model](the-toaster-oven-data-model).

First, let’s ask, “What controls does the toaster oven provide?” The Start Button is pretty obvious, let’s begin with that.

### Feature 1: Start Button

Let’s use the table to define the Start button characteristics. We’ll later use this information when defining the attribute in the Profile Editor.

<table>
    <thead>
        <tr>
            <th>No.</th>
            <th>Feature</th>
            <th>Description</th>
            <th>States</th>
            <th>Data Type</th>
            <th>Writability</th>
            <th>Default Value</th>
            <th>UI Control</th>
            <th>Send Frequency</th>
            <th>Dependencies</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>1</td>
            <td>Start Button</td>
            <td>Button that starts/stops the currently-set action.</td>
            <td><ul class="af-ul-table">
    <li>Stop</li>
    <li>Start</li>
</ul>
            </td>
            <td>Boolean</td>
            <td>Read/Write</td>
            <td>Stop</td>
            <td>Menu</td>
            <td>On Change</td>
            <td>Any Cooking Mode is selected.</td>
        </tr>
    </tbody>
</table>

*   The Start button has only two states, Start and Stop, so we can model it with a Boolean attribute. Of course, we could use an Integer, or even a String, but remember that we should try to use the smallest attribute that is suited to the range of values required.
*   Being a physical control, the Start button allows the end-user to change the value of its associated attribute, so it must be defined as a Read/Write attribute.
*   We’ll give it a default value of Stop.
*   We’ve got some flexibility when deciding what sort of UI Control we want to use to represent the Start button in the mobile app UI. A Menu control is one option: it’s called “Menu” because it can be used to allow selection of one from many choices, but it is rendered as a group of buttons, and it works just fine for a two-state control. Let’s choose Menu.
*   Now, we need to know how often we should send the state of the attribute up to the Afero Cloud. In the case of the Start Button, there is no need to send that data at any time other than when it has changed, so we’ll pick a send frequency On Change.
*   Finally, this feature will only activate if a Cooking Mode has been selected (see Feature 2 below).

### Feature 2: Cooking Mode

The next feature we’ll look at is the cooking mode. On the physical oven unit, the end-user selects a cooking mode using a dial.

<table>
    <thead>
        <tr>
            <th>No.</th>
            <th>Feature</th>
            <th>Description</th>
            <th>States</th>
            <th>Data Type</th>
            <th>Writability</th>
            <th>Default Value</th>
            <th>UI Control</th>
            <th>Send Frequency</th>
            <th>Dependencies</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>2</td>
            <td>Cooking Mode Dial</td>
            <td>Selector for setting cooking mode.</td>
            <td>1 = Bake<br>
2&nbsp;=&nbsp;Convection<br>
3 = Broil<br>
4 = Toast</br>
5 = Warm<br>
6  =Reheat<br>
            </td>
            <td>SINT8</td>
            <td>Read/Write</td>
            <td></td>
            <td>Menu</td>
            <td>On Change</td>
            <td>Feature is available when unit is not cooking.</td>
        </tr>
    </tbody>
</table>

*   This toaster oven offers six cooking options: Bake, Convection, Broil, Toast, Warm, and Reheat. To accommodate these options, we need an 8-byte signed integer (SINT8). Each baking option will map to a numeric value, as shown.
*   Because the end-user will be setting the cooking method, this attribute must be Read/Write.
*   We don’t want to set a default value for this setting, because the availability of some features are dependent on the cooking method selected. For example, the features _Toast Shade_, _Toast Slices_, and _Bagel_ (below) are dependent on the end-user’s selecting the Toast cooking method.
*   Because there are six options, the obvious UI control choice is Menu.
*   Like with the Start Button, we only need to notify the Afero Cloud when the setting has changed, so the send frequency will be On Change.
*   This feature has no dependencies on any other attribute states, except that no other cooking mode be running.

### Feature 3: Toast Slices

This toaster oven requires you set how many slices you want to toast in the oven at once. The toaster can hold up to six slices, but the MCU code accommodates two options.

<table>
    <thead>
        <tr>
            <th>No.</th>
            <th>Feature</th>
            <th>Description</th>
            <th>States</th>
            <th>Data Type</th>
            <th>Writability</th>
            <th>Default Value</th>
            <th>UI Control</th>
            <th>Send Frequency</th>
            <th>Dependencies</th>
        </tr>
    </thead>
<tbody>
        <tr>
            <td>3</td>
            <td>Number of Toast Slices</td>
            <td>Toggle to set number of slices you plan to toast.</td>
            <td>1&ndash;3<br>
4&ndash;6</td>
            <td>Boolean</td>
            <td>Read/Write</td>
            <td>1&ndash;3</td>
            <td>Menu</td>
            <td>On Change</td>
            <td>Cooking Mode <em>Toast</em> is selected.</td>
        </tr>
    </tbody>
</table>

*   The two options for _number of slices_ are: 1-3 and 4-6. A Boolean data type can accommodate these two possible values.
*   As usual, because the user must be able to select number of slices, the attribute must be Read/Write.
*   By default, we’ll say 1–3 slices.
*   With two options that must be clearly spelled out, let’s use a Menu UI Control.
*   We’ll notify the Cloud when the setting changes (On Change).
*   For this feature to be selectable, the Cooking Mode _Toast_ must be selected.

### Feature 4: Toast Shade

<table>
    <thead>
        <tr>
            <th>No.</th>
            <th>Feature</th>
            <th>Description</th>
            <th>States</th>
            <th>Data Type</th>
            <th>Writability</th>
            <th>Default Value</th>
            <th>UI Control</th>
            <th>Send Frequency</th>
            <th>Dependencies</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>4</td>
            <td>Toast Shade</td>
            <td>Selector for setting toast “doneness”.</td>
            <td>Range from 1-10, increments of 1, where labels show:<br>
&nbsp;&nbsp;1 = Defrost<br>
&nbsp;&nbsp;3 = Light<br>
&nbsp;&nbsp;5 = Medium<br>
&nbsp;&nbsp;7 = Dark<br>
10 = Very Dark</td>
            <td>SINT8</td>
            <td>Read/Write</td>
            <td>Medium (5)</td>
            <td>Slider</td>
            <td>On Change</td>
            <td>Cooking Mode <em>Toast</em> is selected.</td>
        </tr>
    </tbody>
</table>

*   Toast shade has range of settings, from defrost to very dark. We can constrain user selections to discrete values, 1-10, so a data type of SINT8 will do the trick.
*   Because this is a setting the user makes, it must be Read/Write.
*   Let’s select middle-of-the-road Medium (5 in the range) as the default setting.
*   With a range of options to display, we’ll use a Slider in the mobile app.
*   There’s no need to notify the Cloud unless the setting has changed.

### Feature 5: Bagel

The density of a bagel vs. toast means toasting a bagel to a given doneness takes longer. This toaster oven allows the end-user to specify a bread vs. bagel option via a button.

<table>
    <thead>
        <tr>
            <th>No.</th>
            <th>Feature</th>
            <th>Description</th>
            <th>States</th>
            <th>Data Type</th>
            <th>Writability</th>
            <th>Default Value</th>
            <th>UI Control</th>
            <th>Send Frequency</th>
            <th>Dependencies</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>5</td>
            <td>Bagel</td>
            <td>Toggle to set bagel function (adds time to toast cycle).</td>
            <td><ul class="af-ul-table">
    <li>Bagel</li>
    <li>Bread</li>
</ul>
            </td>
            <td>Boolean</td>
            <td>Read/Write</td>
            <td>Off</td>
            <td>Switch</td>
            <td>On Change</td>
            <td>Cooking Mode <em>Toast</em> is selected.</td>
        </tr>
    </tbody>
</table>

*   The Bagel setting has two possible states: Bagel or Bread. A Boolean data type can hold the two values.
*   This attribute is settable by the end-user, so must be Read/Write.
*   The assumption being that most people will be toasting bread, the default state will be Bread.
*   Let’s use the Switch UI control for this one. Switch controls are nice for simple states, like On/Off or True/False.
*   The Cloud only needs to be notified On Change.

### Feature 6: Degree Units

This oven accommodates the entry and reporting of both Celsius and Fahrenheit temperature scales. Using the physical oven controls, the end-user sets their preference using a toggle switch.

<table>
    <thead>
        <tr>
            <th>No.</th>
            <th>Feature</th>
            <th>Description</th>
            <th>States</th>
            <th>Data Type</th>
            <th>Writability</th>
            <th>Default Value</th>
            <th>UI Control</th>
            <th>Send Frequency</th>
            <th>Dependencies</th>
        </tr>
    </thead>
<tbody>
        <tr>
            <td>6</td>
            <td>Degree Units</td>
            <td>Toggle between &deg;C and &deg;F.</td>
            <td>&deg;C<br>
&deg;F</td>
            <td>Boolean</td>
            <td>Read/Write</td>
            <td>&deg;F</td>
            <td>Menu</td>
            <td>On Change</td>
            <td>Feature is always available.</td>
        </tr>
    </tbody>
</table>

*   The Degree Units setting has two possible values (°C and °F), so a Boolean data type will be sufficient.
*   The attribute must be Read/Write so the end-user can toggle it.
*   Default depends on where the toaster oven is being used. Since this model is being sold in Canada, we will make the default °C.
*   We will use a Menu UI control for this setting, so we can clearly label °C vs °F.
*   The Cloud only needs to be updated On Change.
*   This feature is not tied to any other functions, so is always available.

### Feature 7: Current Temp

The oven’s current temperature should always be available to the end-user once a Cooking Mode selection is made for either Bake, Convection, or Broil.

<table>
    <thead>
        <tr>
            <th>No.</th>
            <th>Feature</th>
            <th>Description</th>
            <th>States</th>
            <th>Data Type</th>
            <th>Writability</th>
            <th>Default Value</th>
            <th>UI Control</th>
            <th>Send Frequency</th>
            <th>Dependencies</th>
        </tr>
    </thead>
<tbody>
        <tr>
            <td>7</td>
            <td>Current Temp</td>
            <td>Display of current oven temperature.</td>
            <td>130&ndash;500&nbsp;&deg;F&nbsp;(54-250 &deg;C)</td>
            <td>SINT8</td>
            <td>Read-Only</td>
            <td></td>
            <td>Value</td>
            <td>On Change</td>
            <td>Cooking Mode <em>Bake</em>, <em>Convection</em>, or <em>Broil</em> is selected.</td>
        </tr>
    </tbody>
</table>

*   The oven temperature can range from 130-500 °F (or 54-250 °C), so a SINT8 data type will work.
*   Because the Current Temp display reflects the actual oven temperature and the end-user won’t be controlling it, it can be Read-Only.
*   There is no default current temperature, only actual current temperature, so no default value for this attribute.
*   We will use a Value UI control to display the current temperature since it will be a read-only display of a string.
*   The Current Temp isn’t needed by the Cloud until the end-user has selected a Cooking Mode and then the Start Button.
*   The current temperature is meaningless until Cooking Mode _Bake_, _Convection_, or _Broil_ is selected.

### Feature 8: Set Temp

For Cooking Modes _Bake_ and _Convection_, the end-user must select a baking temperature. For these modes, the oven is preset to 150 °F or 65 °C. The end-user can adjust the target temperature up to 500 °F or 250 °C. Once the target temperature is reached, a bell rings once. (If the user selects _Broil_ cooking mode, the target temperature is not adjustable, but is set to 500 °F or 250 °C., automatically.)

<table>
    <thead>
        <tr>
            <th>No.</th>
            <th>Feature</th>
            <th>Description</th>
            <th>States</th>
            <th>Data Type</th>
            <th>Writability</th>
            <th>Default Value</th>
            <th>UI Control</th>
            <th>Send Frequency</th>
            <th>Dependencies</th>
        </tr>
    </thead>
    <tbody> 
        <tr>
            <td>8</td>
            <td>Set Temp</td>
            <td>Selector to set target oven temperature.</td>
            <td>100&ndash;500&nbsp;&deg;F</td>
            <td>SINT8</td>
            <td>Read/Write</td>
            <td>150&nbsp;&deg;F&nbsp;or 65 &deg;C</td>
            <td>Slider</td>
            <td>On Start</td>
            <td>Cooking Mode <em>Bake</em> or <em>Convection</em>.</td>
        </tr>
    </tbody>
</table>

*   The temperature will range from 130-500 °F (54-250 °C), so a SINT8 data type will work.
*   The end-user will be setting the temp, so must be Read/Write.
*   The oven will default to a setting of 150 °F or 65 °C.
*   The user must be able to select a temperature from the range of possible oven temp values, so a Slider will work. We can use increments of 25 °F, which means there will be approximately 15 steps in the Slider.
*   The Cloud needs to get an update once the end-user has selected the Start Button, after setting a temperature or accepting the default.
*   The end-user has selected a Cooking Mode of _Bake_ or _Convection_.

### Feature 9: Timer

The user will be able to set a timer while using Cooking Modes _Bake_, _Convection_, or _Broil_. The timer can be set for up to four hours, in minute/hour increments.

<table>
    <thead>
        <tr>
            <th>No.</th>
            <th>Feature</th>
            <th>Description</th>
            <th>States</th>
            <th>Data Type</th>
            <th>Writability</th>
            <th>Default Value</th>
            <th>UI Control</th>
            <th>Send Frequency</th>
            <th>Dependencies</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>9</td>
            <td>Timer</td>
            <td>Selector to set count-down timer.</td>
            <td>0&ndash;60 mins</td>
            <td>SINT8</td>
            <td>Read/Write</td>
            <td></td>
            <td>Slider</td>
            <td>On Change</td>
            <td>Cooking Mode <em>Bake</em>, <em>Convection</em>, or <em>Broil</em> is selected.</td>
        </tr>
    </tbody>
</table>

*   The timer can set for up to four hours, so we will use a data type of SINT8.
*   The timer must be set by the end-user so it will be Read/Write.
*   The default setting is zero (00:00).
*   A slider will work the best for setting a timer. Set points can be placed every 10 minutes.
*   The Cloud should get notified each time the end-user changes the default setting.
*   The timer setting is available after the user has selected a Cooking Mode option of _Bake_, _Convection_, or _Broil_.

### Feature 10: Timer Ring

Once a set timer has run down, it will ring until the end-user shuts it off.

<table>
    <thead>
        <tr>
            <th>No.</th>
            <th>Feature</th>
            <th>Description</th>
            <th>States</th>
            <th>Data Type</th>
            <th>Writability</th>
            <th>Default Value</th>
            <th>UI Control</th>
            <th>Send Frequency</th>
            <th>Dependencies</th>
        </tr>
    </thead>
<tbody>
        <tr>
            <td>10</td>
            <td>Timer Ring</td>
            <td>Ringer that goes off when timer reaches zero. Rings until shut off.</td>
            <td><ul class="af-ul-table">
    <li>Silent</li>
    <li>Ring</li>
</ul>
            </td>
            <td>Boolean</td>
            <td>Read/Write</td>
            <td>Silent</td>
            <td>Menu</td>
            <td>On Change</td>
            <td>Timer must have run down to zero to ring.</td>
        </tr>
    </tbody>
</table>

*   The timer is either On (Ringing) or Off (Stopped), so a Boolean data type will work.
*   The user must be able to turn the ringer of so it must be Read/Write.
*   Default state is Stop (silent).
*   For clarity, we’ll use a Menu control with Stop as the only selectable option.
*   The Cloud must be notified when the end-user taps Stop.
*   Timer Ring is triggered once the set timer has run down to zero.

### The Toaster Oven Data Model

The table below lays out the data model. You can even use the _No._ and _Feature_ column contents to number and name your attributes as you enter them in the Profile Editor.

<table>
    <thead>
        <tr>
            <th>No.</th>
            <th>Feature</th>
            <th>Description</th>
            <th>States</th>
            <th>Data Type</th>
            <th>Writability</th>
            <th>Default Value</th>
            <th>UI Control</th>
            <th>Send Frequency</th>
            <th>Dependencies</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>1</td>
            <td>Start Button</td>
            <td>Button that starts/stops the currently-set action.</td>
            <td><ul class="af-ul-table">
    <li>Start</li>
    <li>Stop</li>
</ul>
            </td>
            <td>Boolean</td>
            <td>Read/Write</td>
            <td>Stop</td>
            <td>Menu</td>
            <td>On Change</td>
            <td>Any Cooking Mode is selected.</td>
        </tr>
        <tr>
            <td>2</td>
            <td>Cooking Mode</td>
            <td>Selector for setting cooking mode.</td>
            <td>1 = Bake<br>
2 = Convection<br>
3 = Broil<br>
4 = Toast<br>
5 = Warm<br>
6 = Reheat</td>
            <td>SINT8</td>
            <td>Read/Write</td>
            <td>Bake</td>
            <td>Menu</td>
            <td>On Change</td>
            <td>Feature is available when unit is not cooking.</td>
        </tr>
        <tr>
            <td>3</td>
            <td>Number of Toast Slices</td>
            <td>Toggle to set number of slices you plan to toast.</td>
            <td>1&ndash;3<br>
4&ndash;6</td>
            <td>Boolean</td>
            <td>Read/Write</td>
            <td>1&ndash;3</td>
            <td>Menu</td>
            <td>On Change</td>
            <td>Cooking Mode <em>Toast</em> is selected.</td>
        </tr>
        <tr>
            <td>4</td>
            <td>Toast Shade</td>
            <td>Selector for setting toast “doneness”.</td>
            <td>Range from 1&ndash;10, increments of 1, where labels reflect:</br>
&nbsp;&nbsp;1 = Defrost<br>
&nbsp;&nbsp;3 = Light<br>
&nbsp;&nbsp;5 = Medium<br>
&nbsp;&nbsp;7 = Dark<br>
10 = Very Dark</td>
            <td>SINT8</td>
            <td>Read/Write</td>
            <td>Medium (5)</td>
            <td>Slider</td>
            <td>On Change</td>
            <td>Cooking Mode <em>Toast</em> is selected.</td>
        </tr>
        <tr>
            <td>5</td>
            <td>Bagel</td>
            <td>Toggle to set bagel function (adds time to toast cycle).</td>
            <td><ul class="af-ul-table">
    <li>Bagel</li>
    <li>Bread</li>
</ul>
            </td>
            <td>Boolean</td>
            <td>Read/Write</td>
            <td>Off</td>
            <td>Switch</td>
            <td>On Change</td>
            <td>Cooking Mode <em>Toast</em> is selected.</td>
        </tr>
        <tr>
            <td>6</td>
            <td>Degree Units</td>
            <td>Toggle between &deg;C and &deg;F.</td>
            <td>&deg;C<br>
&deg;F</td>
            <td>Boolean</td>
            <td>Read/Write</td>
            <td>&deg;F</td>
            <td>Menu</td>
            <td>On Change</td>
            <td>Feature is always available.</td>
        </tr>
        <tr>
            <td>7</td>
            <td>Current Temp</td>
            <td>Display of current oven temperature.</td>
            <td>100&ndash;500 &deg;F</td>
            <td>SINT8</td>
            <td>Read-Only</td>
            <td>None</td>
            <td>Value</td>
            <td>On Change</td>
            <td>Cooking Mode <em>Bake</em>, <em>Convection</em>, or <em>Broil</em> is selected.</td>
        </tr>
        <tr>
            <td>8</td>
            <td>Set Temp</td>
            <td>Selector to set target oven temperature.</td>
            <td>100&ndash;500 &deg;F</td>
            <td>SINT8</td>
            <td>Read/Write</td>
            <td>150 &deg;F or 65 &deg;C</td>
            <td>Slider</td>
            <td>On Start</td>
            <td>Cooking Mode <em>Bake</em> or <em>Convection</em> is selected.</td>
        </tr>
        <tr>
            <td>9</td>
            <td>Timer</td>
            <td>Selector to set count-down timer.</td>
            <td>0&ndash;60 mins</td>
            <td>SINT8</td>
            <td>Read/Write</td>
            <td>None</td>
            <td>Slider</td>
            <td>On Change</td>
            <td>Cooking Mode <em>Bake</em>, <em>Convection</em>, or <em>Broil</em> is selected.</td>
        </tr> 
        <tr>
            <td>10</td>
            <td>Timer Ring</td>
            <td>Ringer that goes off when timer reaches zero. Rings until shut off.</td>
            <td><ul class="af-ul-table">
    <li>Silent</li>
    <li>Ring</li>
</ul>
            </td>
            <td>Boolean</td>
            <td>Read/Write</td>
            <td>Silent</td>
            <td>Switch</td>
            <td>On Change</td>
            <td>Timer must have run down to zero to ring.</td>
        </tr>
    </tbody>
</table>

Note that in addition to the information gathered above, before you can complete your device’s Profile and UI presentation using the Afero Profile Editor, you will need to specify a few more details, such as attribute value max size, temperature setting increments, and so on, depending on how your MCU code functions.

Updated July 30, 2021

  

© 2015-2021 Afero | [Legal](https://www.afero.io/html/home/privacy.html) | [Privacy](https://www.afero.io/html/home/privacy.html#privacy) | [Afero Home](https://www.afero.io)

[![Afero, Inc.](static/aflib/images/afero-logo.svg)]()
