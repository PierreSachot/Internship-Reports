# Pierre Sachot internship report week 4

This week I worked with Yannick on fixing the CDT CSourceNotFoundEditor problem - the unwanted error message that Eclipse CDT shows when users are running the debugger and jumping into a function which is in another project file.

## Context:
When Eclipse CDT users were running the debugger on the C Project, a window was opening on screen. This window was both alarming in appearance and obtrusive. 
In addition, the message itself was unclear. For example, it could display "No source available for 0x02547", which is irrelevent to the user because he/she does not have an access to this memory address. Several users had complained about it and expressed a desire to disable the window (see: [stack overflow: "Eclipse often opens editors for hex numbers (addresses?) then fails to load anything"](http://stackoverflow.com/questions/43361654/eclipse-often-opens-editors-for-hex-numbers-addresses-then-fails-to-load-anyt/43412237)).
In this post I will show you how we replaced CSourceUserNot FoundEditor with a better user experience display.

## Problem description:

1- The problem we faced was that CSourceNotFoundEditor displayed on several occasions. For example:
	
- When the source file was not found
- When the memory address was known but not the function name
- When the function name was known
		
2- We also wanted to tackle that red link! Red lettering is synonymous with big problems - yet the error message was merely informing the user that the source could not be found, so we felt a less alarmist style of text would be more appropriate.
___

CSourceNotFoundEditor Dialog:

Previous version	|	New version
------------------------:|:------------------
![](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%204/Screenshot_1.png?raw=true) | ![](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%204/Screenshot_2.png?raw=true)

CSourceNotFoundEditor Preferences:

Previous version	|	New version
------------------------:|:------------------
![](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%204/Screenshot_3.png?raw=true) | ![](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%204/Screenshot_4.png?raw=true)

## How to resolve the problem ?

### CSourceNotFoundEditor:

CSourceNotFoundEditor is the class called by the `openEditor()` function, Yannick added a link to the debug preferences page inside it:

- The first thing to do was to create the "Preferences..." button and a text to go with it. Yannick did this in the `createButtons()` function.

- Next, we made it possible for the user to open the Preferences on the correct page - in our case, the Debug page - using this code:
```Java

PreferencesUtil.createPreferenceDialogOn(parent.getShell(), "org.eclipse.cdt.debug.ui.CDebugPreferencePage", null, null).open();

```

"org.eclipse.cdt.debug.ui.CDebugPreferencePage" is the class name we want to load in the debug preferences.

### CDebugPreferencePage:

This class is the one which contains the debug preferences page. I set about modifying it so that the CSourceNot Found preferences could be re-set and access to them enabled. This included the option to modify PreferenceMessages.properties which contains the String values of the buttons, and PreferenceMessage.java to declare them and use them. The last thing we did was to create a global value in CCorePreferenceConstants to get and set the display preferences. This we did in 4 stages:

- First we created a group for the radio buttons. This is in the function createContents().

- Second we created the variable intended to store the preference value. This value is a String store in the CCorePreferenceConstants class. To get a preference String value, you need to use:

```Java

DefaultScope.INSTANCE.getNode(CDebugCorePlugin.PLUGIN_ID).get(CCorePreferenceConstants.YOUR_PREFERENCE_NAME, null);

```
And to store it:

```Java

InstanceScope.INSTANCE.getNode(CCorePlugin.PLUGIN_ID).put(CCorePreferenceConstants.YOUR_PREFERENCE_NAME, "Your text");

```
Here we created a preference named: SHOW_SOURCE_NOT_FOUND_EDITOR which can take 3 values, defined at the begining of the CDebugPreferencePage class:

  ```Java
  /**
 * Use to display by default the source not found editor
 * @since 6.3
 */
public static final String SHOW_SOURCE_NOT_FOUND_EDITOR_DEFAULT = "all_time"; //$NON-NLS-1$

/**
 * Use to display all the time the source not found editor
 * @since 6.3
 */
public static final String SHOW_SOURCE_NOT_FOUND_EDITOR_ALL_THE_TIME = "all_time"; //$NON-NLS-1$

/**
 * Use to display sometimes the source not found editor
 * @since 6.3
 */
public static final String SHOW_SOURCE_NOT_FOUND_EDITOR_SOMETIMES = "sometimes"; //$NON-NLS-1$

/**
 * Use to don't display the source not found editor
 * @since 6.3
 */
public static final String SHOW_SOURCE_NOT_FOUND_EDITOR_NEVER = "never"; //$NON-NLS-1$

```

 - Third, we needed to find where to set the values and where to get them. So, to set the values on your components, use the `setValues()` function.To store a value, you will need to add your code in `storeValues()`, like it's name suggests it will store the value inside of the global preferences variable.

 - The fourth and final stage is really important: - You need to put the default value of the preference you want to add in setDefaultValues() to allows access to the original value of the preferences.
 
### DsfSourceDisplayAdapter:

This is the class which calls CSourceNotFoundEditor, so here in the function openEditor, we needed to check the preferences options in order to know if it was possible to display CSourceFoundEditor. These checks need to be carried out in openEditor() function because this is the function which opens the CSourceNotFoundEditor. To do that, we created two cases:
	
   - First case in which the user wants to display the Editor all the time
   - Second for when the user only wants to display it if the source file is not found 
   - The last case is an exclusion of the "all_time", so you don't need to check it because nothing is done in this case.
   
To do that, we did it like this:  
![how to display CSourceNotFoundEditor](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%204/Screenshot_5.png?raw=true)

### Conclusion
Now users have the capacity to disable CSourceNotFoundEditor window altogether or to choose for themselves when to display it. Thus saving time and improving the user experience of the Eclipse debugger. This is a great example of how working on an open source project can really benefit a whole community of users. But, a word of warning, CDT project isn't the easiest program to develop or the easiest to master, you need to understand other user's code and if you change it you need to retain its original logic and style.
Fiddly perhaps but well worth it! The user community will appreciate your efforts and the flow of coding future work will be smoother and more efficient. A better user experience for everyone.

