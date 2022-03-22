Define the Device Type
======================

After selecting a project on the Startup window, the Profile Editor opens to the **Device Type definition** window:

![Device Type Definition Window](static/custom/images/APE-DefineDvcType.png)

Add Your Information
--------------------

1.  In the Define the Device Type section, complete the fields using the information in the table below:
    
<table>
    <thead>
        <tr>
            <th>Field</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Device Type Name</td>
            <td><p>This name is a string; by default it assumes the name you gave it when creating the project, but you can change it here. Note that if you do change it here, the project name displayed in Profile Editor will also update to reflect the new name. Here are some important things to know about Device Type:</p>
                <ul class="af-ul">
                    <li><p>In the mobile app, the default name given the device is the Device Type Name. This name will be used only initially as it can be changed to a “friendly name” by the end-user when adding the device to their account, or by editing the device name on the device Settings screen. You, as the developer, can also change this friendly name on the Publish window. So use a descriptive name here that makes sense to you and your end-users. Also, the friendly name is what appears in the Inspector tool to identify your devices.</p>
                    <p>Note that if the end-user does change the friendly name on the mobile app, the Device Type Name itself will not change. The only place you can change the Device Type Name is in this field; and only after the Profile has been published does any change you make reflect on the Publish window and in the Afero system.</p></li>
                    <li>The Device Type is an important identifier. Once you publish a Profile with a given Device Type, an ID is associated with the Device Type that remains unchanged as long as the Device Type is active. As pointed out above, you can always rename your Device Type using this field, but the <code>devicetypeID</code> will not change.</li>
                    <li>Device Type names must be unique across your projects. (Read more in  <a id="1549412226.68" href="/SelectProject#DOC-158">Before You Proceed</a>.)</li>
                    <li>The Device Type is used by the OTA Manager to associate an MCU firmware Image Type with devices.</li>
                </ul>
            </td>
        </tr>
        <tr>
            <td>Description</td>
            <td>This description is a string, internal only and not displayed to the end-user. Type a description that is useful to you.</td>
        </tr>
        <tr>
            <td>Project Folder</td>
            <td>The directory on your local filesystem where your project is saved. You can’t edit the location; it’s displayed for information purposes only.</td>
        </tr>
        <tr>
            <td>Header File Output</td>
            <td>If you are using a device that has a separate MCU, a header file containing the device description (attributes) will be generated when you publish your project. By default, the header file will be saved to your project directory, but you can use this setting to save the header file to a different directory. For example, you might wish to save the file directly into the project directory for the associated MCU code (which will rely on the header). </td>
        </tr>
        <tr>
            <td>Device Icon</td>
            <td>This icon will be visible to the end-user. Select a meaningful image to represent your device. Click <span class="UIText">Select</span> to see all available icons; highlight the icon you want then click <span class="UIText">Use This One!</span> Your selected icon will be displayed in the Preview pane to the right, to give you an idea of how it will look on a smartphone. It will also be displayed to the left of your project name in the Navigation pane.</td>
        </tr>
    </tbody>
</table>
    
2.  The Power Optimization section provides controls and information regarding battery power consumption.
    
!!!! note
     The Power Optimization calculations apply only to projects using either the Modulo-1 or Modulo-1B modules.
    
   *   Select an Advertising Interval (in seconds) for Bluetooth. If you’re running on battery power, the shorter the interval, the more battery power is consumed. However, at higher intervals the responsiveness of the device will be negatively affected due to lag time.
   *   Once you’ve selected an interval, the Daily Average Current field will display the projected average current required by Modulo-1 or Modulo-1B for that interval while idle (the module is said to be idle when performing no function other than advertising).
   *   Click the Show advanced power calculator link at the bottom of the pane to open the Approximate Battery Lifetime estimator. Use this calculator to estimate the runtime of an Modulo-1 or Modulo-1B based on the battery type you’re using. For a more accurate forecast of the battery life, type the number of connections per day and additional power the device may be using. You can also check what the runtimes might be for different battery capacities.
        
   ![Approximate Battery Life Calculator](static/custom/images/APE-ApproxBatteryLife.png)
        
3.  Once you’ve defined the Device Type, you’ll want to save the definition.
    
    *   Clicking Cancel reverts any changes you made on the page.
    *   Clicking Save initiates validation of the new data and, if validation is successful, the data is saved to your local project directory.
    
!!!! note
     Every time you save, your data is validated. If validation fails for any reason, an error message will appear then move to the left. Click the error description in the message to highlight the field that needs correcting.
    
      
    

Next: [Define the Device Attributes](AttrDef)

Updated July 30, 2021

  

© 2015-2021 Afero | [Legal](https://www.afero.io/html/home/privacy.html) | [Privacy](https://www.afero.io/html/home/privacy.html#privacy) | [Afero Home](https://www.afero.io)

[![Afero, Inc.](static/aflib/images/afero-logo.svg)]()
