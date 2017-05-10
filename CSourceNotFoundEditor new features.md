# CSourceNotFoundEditor Modifications

## Release

This is the new CSourceNotFound editor for CDT 9.3 which is part of the Eclipse Oxygen release of June 2017.

## General
### What's change ?

The new features are the possibility to disable the CSourceNotFoundEditor or to choose in which cases we want to open it up. 

#### Previous version
This editor was shown every time you were debugging and you were trying to jump into an external function.

The CSourceNotFound was display in three cases:
  - Source you were jumping into not found:
 
![source not found](https://github.com/PierreSachot/Internship-Reports/blob/master/images/CDT%20new%20CSourceNotFoundEditor/Screenshot_1.png?raw=true)

  - Function name know but can't find the source it:

![know name function](https://github.com/PierreSachot/Internship-Reports/blob/master/images/CDT%20new%20CSourceNotFoundEditor/Screenshot_2.png?raw=true)

  - Unknow function name:
  
![unknow function name](https://github.com/PierreSachot/Internship-Reports/blob/master/images/CDT%20new%20CSourceNotFoundEditor/Screenshot_3.png?raw=true)

#### New version
Now when this case is reproduce, you can choose to go into the preferences :

![Preferences access](https://github.com/PierreSachot/Internship-Reports/blob/master/images/CDT%20new%20CSourceNotFoundEditor/Screenshot_4.png?raw=true)

Then this window will be show :

![Preferences access](https://github.com/PierreSachot/Internship-Reports/blob/master/images/CDT%20new%20CSourceNotFoundEditor/Screenshot_5.png?raw=true)

Inside the red square, visible on the previous picture, you can see the new preferences we added, this allow you to display is all the time, only if the source file isn't found or never.


## Bugs Fixed in this Release:

**See: [Bug 515296](https://bugs.eclipse.org/bugs/show_bug.cgi?id=515296)**


