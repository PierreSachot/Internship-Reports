# Pierre Sachot internship report week 3

During this week, I worked with Yannick on fixing the CDT source not found editor problem.

##Problem description :

The problem we were against was that the source not found editor displayed everytime and sometimes it was not useful.

##How to resolve the problem ?
###DsfSourceDisplayAdapter :
This is the class which call the source not found editor, so here in the function openEditor, we needed to check preferences options to
display it. 

###CSourceNotFoundEditor :
This is the class which is called by openEditor, it is the source not found editor, in this class Yannick added a link to the preferences 
page to change the display choices. 
- The first thing to do was to create the "Preferences..." button and a text, Yannick did it in the `createButtons()` function.
- Second thing was to create the listener which will open the Preferences on the right page, this means the Debug page in our case. So to open 
the preferences on the right page, you need to do like this :
`PreferencesUtil.createPreferenceDialogOn(parent.getShell(), SourceLookupUIMessages.CSourceNotFoundEditor_8, null, null).open();`
the CSourceNotFoundEditor_8 is a String containing org.eclipse.cdt.debug.ui.CDebugPreferencePage which is the class name.

###CDebugPreferencePage :
This class is the one which contains the debug preferences page in the debug. I modify it to set the source not found preferences and to get 
them, that includes to modify the PreferenceMessages.properties which contains the String values of the buttons and the PreferenceMessage.java to declare 
them and use them. The last thing was to put the debug preferences values in CCorePreferenceConstants.
  - The first thing to do was to create a group where put radio buttons inside, this is in the function `createContents()`. 
  - The second thing was to create the variables which will store the preference value. This value is a String store in the CCorePreferenceConstants
  class.
  To get a preference String value, you need to use :
  `DefaultScope.INSTANCE.getNode(CDebugCorePlugin.PLUGIN_ID).get(CCorePreferenceConstants.YOUR_PREFERENCE_NAME, null);`
  And to store it :
  `InstanceScope.INSTANCE.getNode(CCorePlugin.PLUGIN_ID).put(CCorePreferenceConstants.YOUR_PREFERENCE_NAME, "Your text");`
  Here we created a preference named : SHOW_SOURCE_NOT_FOUND_EDITOR which can take 3 values, defined at the begining of the CDebugPreferencePage
  class : 
  `private String all_time = "all_time"; //$NON-NLS-1$
	private String sometimes = "sometimes"; //$NON-NLS-1$
	private String never = "never"; //$NON-NLS-1$`
  - The third thing was to know where to put the values and where to get them.
  So, you need to get them in the `setValues()` function because this is the function which get the preferences and set them correctly.
  Finally, to store a value, you need to add the code in `storeValues()`, like it's name, it will store the value inside of the preferences variables.
  - The last thing is really important, so **don't forget it** :
  You need to put the default value of the preference you want to add in `setDefaultValues()` to allows the user to get the original value of the preferences.
