# CSourceNotFoundEditor Modifications

## Release

This is the new source not found editor for CDT 9.3 which is part of the Eclipse Oxygen release of June 2017.

## General
### What has changed ?

This new feature allows you do diasable the **source not found editor** or to
choose to open it up only in usefull cases.

#### Previous version
This editor was shown every time you were debugging and you were trying to jump into an external function with no debugging information.

The source not found editor was displayed in three cases:
  - Source you were jumping into is not found:
 
![source not found](https://github.com/PierreSachot/Internship-Reports/blob/master/images/CDT%20new%20CSourceNotFoundEditor/Screenshot_1.png?raw=true)

  - Function name known but can't find its source:

![know name function](https://github.com/PierreSachot/Internship-Reports/blob/master/images/CDT%20new%20CSourceNotFoundEditor/Screenshot_2.png?raw=true)

  - Unknown function name:
  
![unknow function name](https://github.com/PierreSachot/Internship-Reports/blob/master/images/CDT%20new%20CSourceNotFoundEditor/Screenshot_3.png?raw=true)

#### New version
Now when the source not found editor opens you can choose to go into the preferences :

![Preferences access](https://github.com/PierreSachot/Internship-Reports/blob/master/images/CDT%20new%20CSourceNotFoundEditor/Screenshot_4.png?raw=true)

Then this preference window will be shown :

![Preferences access](https://github.com/PierreSachot/Internship-Reports/blob/master/images/CDT%20new%20CSourceNotFoundEditor/Screenshot_5.png?raw=true)

Inside the red square, visible on the previous picture, you can see the newly
added preferences, they allow you to choose between displaying the editor all
the time, only if the source file isn't found or never.


## Bugs Fixed in this Release:

**See: [Bug 515296](https://bugs.eclipse.org/bugs/show_bug.cgi?id=515296)**

