Tutorial 2: Creating a Device Profile
=====================================

Before you start, we assume that you’ve already run through [Tutorial 1: Linking Modulo](Lesson1). If not, please start there.

!!! note
    This tutorial is designed for the Modulo-1 and Modulo-2 dev boards, but not for the Modulo-1B.


1.  If you haven’t already, download the Afero Profile Editor, for [Windows](http://cdn.afero.io/latest-ape/win) or [macOS](http://cdn.afero.io/latest-ape/mac). Open and sign in to the Profile Editor. The Startup window appears:
    
    ![Startup Window](static/custom/images/APE-StartScreen-Tut.png)
    
2.  For the purposes of this tutorial, we’re going to save you some typing by providing a pre-configured Modulo project.
    1.  First, you will need the appropriate Sample Project. The easiest way to get this is from the Profile Editor system Tools menu. From this menu, select Sample Projects > [GitHub Repository](http://github.com/aferodeveloper/APE-Project-Profiles). (All Modulo projects, and more, are available from this GitHub repository.)
    2.  On the GitHub web page that opens, click the green Clone or download button then select Download ZIP. Navigate to your Profile Editor project directory to save the zip file.
    3.  Once saved, go to the Profile Editor project directory on your filesystem and double-click the file to decompress it. You can keep all the projects in the unzipped project profiles directory, or delete the projects you’re not interested in.
    4.  Back in the Afero Profile Editor Startup window, select the Open button, navigate to the directory where you unzipped the project, and select the directory that holds the Modulo project you want to use in this lesson.

3.  Once the project is open, click Device Type in the Navigation pane in the left of the window. To be sure we’re all starting in the same view, your window should look something like this:

![Device Type Pane](static/custom/images/Mod2_DeviceType.png)
    
!!!! note
     Notice the Module Type of the Profile is specified right at the top of the pane (in the example, we’re working with a Modulo-2.) This module type was specified in the pre-built Profile you downloaded; when creating a Profile from scratch, you must specify the module type right from the beginning. Doing so is the first step in the New Profile window. It’s critical that the module type in the Profile match the hardware you’re using in your project!
    
      
In the Device Type view, you can set the device name, type a detailed device description, and set the device icon. Of these items, only the icon will be visible to the end-user. Notice the Preview panel on the right of the view – it shows the selected device icon as it will appear in the mobile app.
    
Since we’re working with a pre-configured Profile, you don’t need to make any changes, but if you like, feel free to do so. When you’re finished, click Save to save any changes you might have made.
    
4.  Click Attributes in the left-hand Navigation pane. You’ll see that the four Modulo GPIO pins are listed. Only two GPIO pins are needed for this project, pins 0 and 3. Click the On/Off switch to ON for these two pins; you’ll see them highlighted in the right-hand Preview pane:
    
    ![Preview Pane](static/custom/images/AttributeDefinition-Tut.png)
    
5.  For this project, we’ve defined two attributes: one for GPIO 0, which is connected to the LED on the Modulo; and one for GPIO 3, which is tied to the pushbutton. We’ll take a detailed look at the attributes now. Note that selected values are shown in white letters on an orange background.
   1.  Click LED Attribute, the attribute name for GPIO 0, to open its detailed view:
        
![Attribute Definitions](static/custom/images/Tut2_LED_Attr.png)

   *   Attribute Name is a required field for all attributes. We’ve named this one LED Attribute but you can change it in this field.
   *   To control the LED from the mobile app, Operation Mode must be Output. This is because Outputs are Read/Write, and sending a command from the mobile app requires writing to an attribute.
   *   GPIO 0 is given a Default Logic Level of 0, which sets the default state of the LED to OFF.
   *   Leave both Options, PWM (pulse-width modulation) and Pulse, deselected.
   *   Change the Active selector set to Low. Because the cathode of the LED is connected to GPIO 0, the LED turns on when the pin goes Low.
   *   Don’t select any of the Bind to Attributes buttons.
   
   2.  Click Button Attribute, the attribute name for GPIO 3, to open its detailed view:
        
![Attribute Definitions](static/custom/images/Tut2_Button_Attr.png)

   *   Attribute Name is a required field for all attributes. We’ve named this one Button Attribute but you can change it here.
   *   Leave GPIO3 Operation Mode set to Input. This will display the state of the button when pressed in the mobile app.
   *   Change the Active setting to Low. When pressed, the button connects GPIO 3 to ground so the pin is considered active when it goes Low.
   *   The Pull Up setting is required for to keep the GPIO pin inactive when the button is not pressed.
   *   With Is Toggle set, the status displayed by the mobile app will switch with every momentary button push. If Is Toggle is not set, the state displayed will always reflect the current button state.
   *   Leave the Debounce Time set to 0, the Active selector set to High, and don’t select any of the Bind to Attributes buttons.
   
   3.  Click Save when you are finished defining project attributes.

6.  To move on to defining the mobile app UI for this project, click UI Controls in the left-hand Navigation pane.
    
    You will probably not be surprised to find that two UI controls have been defined since our project uses two attributes. It’s common, though not a rule, to have one UI control for every attribute. As examples: some attribute values will be used “behind-the-scenes” to calculate the values of other attributes, and you can assign the same control to multiple attributes and multiple controls to the same attribute.
    
    There are two steps to defining the UI controls: a) we first define the “attribute options” for each attribute; and b) we then assign UI controls to each attribute.
    
    You might be wondering why defining attribute options is separate from assigning controls. The reason is so you can easily assign multiple controls to the same attribute without having to retype a lot of details (although that doesn’t apply to this Tutorial). To learn more about attribute options, read [Define the Mobile App UI > Define the Attribute Options](AppUIDef#Options).
    
      
    
1.  In the far-right pane under the heading Attribute Options, click the pencil icon in the LED:LED Attribute box to open its Attribute Option Details dialog:
        
![LED Attribute Options](static/custom/images/Les2-LEDAttrOpt.png)
        
You will see that we have defined the following:
 
   *   The label for this control will display as “LED”.
   *   The LED control is selected as Primary Operation, which means you can control the LED from the Home screen of the mobile app by double-tapping the device icon.
   *   Note that two Value Options have been defined. These present the two different states the on-board LED can have: 1 is On, 0 is Off. You can edit the labels if you want, but keep the values as defined.
   *   Check the Running State box next to the On state for the LED. This will change the color of the device in the mobile app when the LED is on.
   *   We don’t need to complete the Range Options for this attribute since we’ll be using a Menu control, which does not require range definitions.
   *   Click OK to close the Attribute Option Details dialog.
        
   Now, again in the far-right pane, click the pencil icon in the Button:Button Attribute box to open its Attribute Option Details dialog:
        
   ![LED Attribute Options](static/custom/images/Les2-ButtonAttrOpt.png)
   *   The label for this control will display as “Button”.
   *   This attribute happens to have the same Value Options as the LED attribute, but is not set as the Primary Operation.
   *   Again, we don't need any Range Options defined.
   *   Click OK to close the Attribute Option Details dialog.
2.  Now we will assign a UI Control to each attribute. There are many types of UI Controls available, but in this project are using the Menu control for both.
   
   ![Menu Controls](static/custom/images/Dfn_UI_Ctls.png)
   
   Click the first attribute, LED Attribute (I/O 0): Menu, to open it for editing.
   
   ![LED Control](static/custom/images/AttributeDef-LED.png)
   
   *   For attribute option we’ve selected LED, and for control type we’ve selected Menu.
   *   For View Style, we’ve selected INLINE, which means all menu options will be shown. (Popup View Style means the end-user must tap the control to reveal all menu options.) You can see examples by expanding View Style Preview (shown in the screenshot above).
    
   Now click Button Attribute (I/O 3): Menu to open it.
    
   ![Button Attribute](static/custom/images/AttributeDef-Button.png)
    
   Notice that we’ve also selected the INLINE View Style.
    
   6.  Click Save when you are finished defining UI controls.
   
7.  Click UI Control Groups in the left-hand Navigation pane to group your controls.
    
    The left-hand pane, Define the UI Control Groups, displays a representation of the device mobile app UI. At the top of both the mobile app and the Profile Editor is the “groups ribbon”; below are the controls for the selected group. Note that device Settings is a selection only shown in the mobile app groups ribbon, not in the Profile Editor.
    
   1.  We’ve already set up two groups, shown in the groups ribbon and labeled **LED** and **Button**. (We’ve already created groups for this project, but if we hadn’t, you would click![](static/custom/images/AddGroupIcon.png)to create a new group.)
   
   2.  On the right, you’ll see all the controls you defined on the UI Controls window, identified by their Default Label. These are the controls available for grouping.
   3.  Click the group labeled **LED** to make it active and ready for editing. The group will be highlighted in orange:
   
   ![UI Control Group](static/custom/images/Control_Groups_LED.png)
   
   Below the selected group, you’ll see the **LED** menu control. We’ve already moved the **LED** menu into the **LED** group, but if we hadn’t, you would add a control by dragging it from the Available UI Controls pane on the right into the controls display on the left, below the groups ribbon.
   
   4.  To remove a control from a group, simply drag it back to the Available UI Controls pane.
   
   A control can appear in multiple groups, but _must_ appear in at least one group or it won’t appear at all in the mobile app UI. That means that even if you have just one UI control, you’ll need to create a UI control group to contain it.
        
          
        
   5.  Click Save when you are finished defining UI control groups.
8.  Now that your device Profile is complete, click Publish in the left-hand Navigation pane to install it on your Modulo.
    
    If you intend to perform the next steps yourself (and you should!), you’ll need to set up some hardware:
    
    *   You should have your Modulo connected to USB (for power only at this time).
    *   You should have the Afero app running and open on your smartphone.
    
    Before we get started, take a look at the Publish window:
    
    *   This window shows you all the developer devices on your account. For each available device, the Status displays the Bluetooth signal strength, e.g. -40dBm, in an orange badge.
    *   The Device Activity log will show all attribute-related communication between device and service in real-time. It is initially populated by the latest cached values from the Cloud.
    
    **OK, time to Publish:**
    
    1.  Select the device(s) you want to update by selecting the corresponding orange checkbox(es).
    2.  Click Publish.
    3.  The update can take up to one minute. You’ll see an update message in the Device Activity and the SW Version will change once the update has completed.
9.  Try out the new mobile app UI for your device! Notice that the LED is on, which is indicated on the Home screen. Because it’s the Primary Operation defined with a Boolean attribute, tapping the large device icon on the Home screen will toggle the LED on and off.
    
    ![Mobile App](static/custom/images/ios_app.png)
    
    You should now be able to control the Modulo LED using the LED buttons on your smartphone, and the pushbutton on the Modulo board should toggle the On/Off Button control in the mobile app UI.
    

Next: [Tutorial 3: Afero + Arduino](Lesson3)

Updated July 30, 2021

  

© 2015-2021 Afero | [Legal](https://www.afero.io/html/home/privacy.html) | [Privacy](https://www.afero.io/html/home/privacy.html#privacy) | [Afero Home](https://www.afero.io)

[![Afero, Inc.](static/aflib/images/afero-logo.svg)]()