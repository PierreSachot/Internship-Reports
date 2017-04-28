# Pierre Sachot internship report week 3

During this week, I worked with Yannick on fixing the CDT source not found editor problem.

## Problem description :

The problem we were against was that the source not found editor displayed everytime and sometimes it was not useful.
The three cases where :

	- When the source file is not found.
	
	- When the memory address is known but not the function name.
	
	- When the function name is known.
	
### That is the previous version of the Source not found editor :
![before](http://image.prntscr.com/image/e5ef587d4fb3417aa9594fdb8cb9fb0b.png)

And this is the new version : <br>
![after](http://image.prntscr.com/image/b9f55e3f3ba94e499dcc3421b594e12b.png)

### This is the previous version of the Debug preferences :
![before](http://image.prntscr.com/image/793a6e8862c6488b897867b4ab30b9f8.png)

And this is the new version :<br>
![after](http://image.prntscr.com/image/f407a70baf13440c8027ba6392ede376.png)



## How to resolve the problem ?

### CSourceNotFoundEditor :
This is the class which is called by openEditor, it is the source not found editor, in this class Yannick added a link to the preferences page to change the display choices. 
- The first thing to do was to create the "Preferences..." button and a text, Yannick did it in the `createButtons()` function.
- Second thing was to create the listener which will open the Preferences on the right page, this means the Debug page in our case. So to open 
the preferences on the right page, you need to do like this :
```Java
PreferencesUtil.createPreferenceDialogOn(parent.getShell(), SourceLookupUIMessages.CSourceNotFoundEditor_8, null, null).open();
```
the CSourceNotFoundEditor_8 is a String containing org.eclipse.cdt.debug.ui.CDebugPreferencePage which is the class name.

### CDebugPreferencePage :
This class is the one which contains the debug preferences page in the debug. I modify it to set the source not found preferences and to get 
them, that includes to modify the PreferenceMessages.properties which contains the String values of the buttons and the PreferenceMessage.java to declare 
them and use them. The last thing was to put the debug preferences values in CCorePreferenceConstants.
  - The first thing to do was to create a group where put radio buttons inside, this is in the function `createContents()`. 
  - The second thing was to create the variables which will store the preference value. This value is a String store in the CCorePreferenceConstants
  class.
  To get a preference String value, you need to use :
  ```Java
  DefaultScope.INSTANCE.getNode(CDebugCorePlugin.PLUGIN_ID).get(CCorePreferenceConstants.YOUR_PREFERENCE_NAME, null);
  ```
  
  And to store it :
  
  ```Java
  InstanceScope.INSTANCE.getNode(CCorePlugin.PLUGIN_ID).put(CCorePreferenceConstants.YOUR_PREFERENCE_NAME, "Your text");`
  ```
  
  Here we created a preference named : SHOW_SOURCE_NOT_FOUND_EDITOR which can take 3 values, defined at the begining of the CDebugPreferencePage class : 
  
  ```Java
	private String all_time = "all_time"; //$NON-NLS-1$
	private String sometimes = "sometimes"; //$NON-NLS-1$
	private String never = "never"; //$NON-NLS-1$
 ```
	
  - The third thing was to know where to put the values and where to get them.
  So, you need to get them in the `setValues()` function because this is the function which get the preferences and set them correctly.
  Finally, to store a value, you need to add the code in `storeValues()`, like it's name, it will store the value inside of the preferences variables.
  - The last thing is really important, so **don't forget it** :
  You need to put the default value of the preference you want to add in `setDefaultValues()` to allows the user to get the original value of the preferences.


### DsfSourceDisplayAdapter :
This is the class which calls the source not found editor, so here in the function openEditor, we needed to check preferences options to
display it.
Those checks need to be done in `openEditor()` function because this is the function which open the Source not found editor.
To do that, we created 2 cases the first one is when the user want to display the Editor all the time, and the second one is when the user only want to display it if the source file is not found. The last case doesn't need to be check because if the 2 others are false, it will not do it.

To do that, we did it like that :
![how to display source not found editor](http://image.prntscr.com/image/bb4a2112940a43429f7f1fe3f7b28e1a.png)
