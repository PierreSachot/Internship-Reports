# Pierre Sachot internship report week 3

During this week, I started to work with Yannick on the CDT project which is the C/C++ Eclipse version coded in Java. We started to work on 
fixing bugs to understand the architecture, how it works, and understand how [Bugzilla](https://bugs.eclipse.org/bugs/) and cHudson website](https://hudson.eclipse.org/) works. More
specificly we worked on the [Windows topic](https://hudson.eclipse.org/cdt/job/cdt-master-windows/lastCompletedBuild/testReport/) on [Hudson](https://hudson.eclipse.org/) tring to solve some tests which don't pass. 
After that we work on adding a fonctionality because of a [bug report](https://bugs.eclipse.org/bugs/show_bug.cgi?id=515296) which allows the
user to display or not the **File not found Editor** in the debugger when you jump to an unknow address. We needed to add it to the preferences
and in the File not found Editor when it was display. Now we continue on this problem to give to the user a more explicit message when he is on it,
to know if the file, the function name or the address isn't know. 
More than that, we need to add 2 new check boxes, to allows the user to choose if he wants to display it everytime or only when the file isn't found.

This is a really interessant topic, because we are working on an big open source project, it allows us to see how the code was write and 
to create interesting functions inside of this program.
