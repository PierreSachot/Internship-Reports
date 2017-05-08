# Pierre Sachot internship report week 4

This week I worked with Yannick on fixing the CDT CSourceNotFoundEditor problem.

## Context:
When the user was stepping into functions by running the debugger on the C Project, a window was opening on screen. This window was both alarming in appearance and obtrusive. 
In addition, the message itself was unclear. For example, it could display "No source available for 0x02547", this information isn't really relevent to the user because he doesn't have an access to this memory address. Several users had complain about it and expressed a desire to disable the window (see: [stack overflow: "Eclipse often opens editors for hex numbers (addresses?) then fails to load anything"](http://stackoverflow.com/questions/43361654/eclipse-often-opens-editors-for-hex-numbers-addresses-then-fails-to-load-anyt/43412237)).
In this post, we will show you how we replaced CSourceUserNot FoundEditor with a better user experience display.

## Problem description:

1- The problem we faced was that CSourceNotFoundEditor displayed on several occasions. For example:
	When the source file was not found
	When the memory address was known but not the function name
	When the function name was known
		
2- We wanted to tackle that red link ! Red lettering is synonymous with big problems ! And yet the error message was merely informing the user that the source could not be found we felt a less alarmist style of text would be more appropriate..
___
That is the previous version of CSourceNotFoundEditor:

Previous version	|	New version
------------------------:|:------------------
![](https://github.com/PierreSachot/Internship-Reports/blob/master/images/Screenshot_1.png?raw=true) | ![](https://github.com/PierreSachot/Internship-Reports/blob/master/images/Screenshot_2.png?raw=true)

## How to resolve the problem ?
### CSourceNotFoundEditor:

CSourceNotFoundEditor is the class called by `openEditor()` function, Yannick added a link to the debug preferences page inside it:

- The first thing to do was to create the "Preferences..." button and a text, Yannick did it in the `createButtons()` function.
- Next, we made it possible for the listener to open the Preferences on the correct page - in our case, the Debug page - using this code:

`PreferencesUtil.createPreferenceDialogOn(parent.getShell(), "org.eclipse.cdt.debug.ui.CDebugPreferencePage", null, null).open();`

"org.eclipse.cdt.debug.ui.CDebugPreferencePage" is the class name we want to load in the debug preferences.


This is the class which is called by openEditor, it is CSourceNotFoundEditor, in this class Yannick added a link to the preferences page to change the display choices. 
- The first thing to do was to create the "Preferences..." button and a text, Yannick did it in the `createButtons()` function.
- Second thing was to create the listener which will open the Preferences on the right page, this means the Debug page in our case. So to open 
the preferences on the right page, you need to do like this:
```Java
PreferencesUtil.createPreferenceDialogOn(parent.getShell(), SourceLookupUIMessages.CSourceNotFoundEditor_8, null, null).open();
```
the CSourceNotFoundEditor_8 is a String containing org.eclipse.cdt.debug.ui.CDebugPreferencePage which is the class name.

### CDebugPreferencePage:
This class is the one which contains the debug preferences page in the debug. I modify it to set the source not found preferences and to get 
them, that includes to modify the PreferenceMessages.properties which contains the String values of the buttons and the PreferenceMessage.java to declare 
them and use them. The last thing was to put the debug preferences values in CCorePreferenceConstants.
  - The first thing to do was to create a group where put radio buttons inside, this is in the function `createContents()`. 
  - The second thing was to create the variables which will store the preference value. This value is a String store in the CCorePreferenceConstants
  class.
  To get a preference String value, you need to use:
  ```Java
  DefaultScope.INSTANCE.getNode(CDebugCorePlugin.PLUGIN_ID).get(CCorePreferenceConstants.YOUR_PREFERENCE_NAME, null);
  ```
  
  And to store it:
  
  ```Java
  InstanceScope.INSTANCE.getNode(CCorePlugin.PLUGIN_ID).put(CCorePreferenceConstants.YOUR_PREFERENCE_NAME, "Your text");`
  ```
  
  Here we created a preference named: SHOW_SOURCE_NOT_FOUND_EDITOR which can take 3 values, defined at the begining of the CDebugPreferencePage class: 
  
  ```Java
	private String all_time = "all_time"; //$NON-NLS-1$
	private String sometimes = "sometimes"; //$NON-NLS-1$
	private String never = "never"; //$NON-NLS-1$
 ```
	
  - The third thing was to know where to put the values and where to get them.
  So, you need to get them in the `setValues()` function because this is the function which get the preferences and set them correctly.
  Finally, to store a value, you need to add the code in `storeValues()`, like it's name, it will store the value inside of the preferences variables.
  - The last thing is really important, so **don't forget it**:
  You need to put the default value of the preference you want to add in `setDefaultValues()` to allows the user to get the original value of the preferences.


### DsfSourceDisplayAdapter:
This is the class which calls CSourceNotFoundEditor, so here in the function openEditor, we needed to check preferences options to
display it.
Those checks need to be done in `openEditor()` function because this is the function which open CSourceNotFoundEditor.
To do that, we created 2 cases the first one is when the user want to display the Editor all the time, and the second one is when the user only want to display it if the source file is not found. The last case doesn't need to be check because if the 2 others are false, it will not do it.

To do that, we did it like that:
![how to display CSourceNotFoundEditor](http://image.prntscr.com/image/bb4a2112940a43429f7f1fe3f7b28e1a.png)
