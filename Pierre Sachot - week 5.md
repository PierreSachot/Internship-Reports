
The problem is, you don't have only one Hook inside of Eclipse, but a lot, you can see all hooks by following that :

[Eclipse Hooks](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%205/Screenshot_1.png?raw=true)

Now you need to find the right hook, for that I only watch inside others completion proposals plugins on which hook they were attaching to.
In my case, this is the : `javaCompletionProposalComputer`.

## Hooking with your plugin:

Firstly, you need to add in your MANIFEST that what you are creating is a Singleton. To do that, open the manifest file and do that:

[singleton](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%205/Screenshot_8.png?raw=true)

You need to add some dependencies :

  - Open your MANIFEST and go to "Dependencies"

[dependencies](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%205/Screenshot_11.png?raw=true)

  - Click on "Add.."
  - Add those dependencies:
  
    ` org.eclipse.jface.text
      org.eclipse.swt
      org.eclipse.jface`

 If you don't find some dependencies, add them directly inside the MANIFEST:

[dependencies](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%205/Screenshot_9.png?raw=true)

 Now you should have that in the dependencies: 

[dependencies](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%205/Screenshot_10.png?raw=true)

 Now you can Hook your plugin to the `javaCompletionProposalComputer`.

  - Go inside the "Extensions" bloc:

[extensions](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%205/Screenshot_12.png?raw=true)

  - Click on "Add..."

  - Search `javaCompletionProposalComputer`, select it and click "Finish":

  [Extension point](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%205/Screenshot_13.png?raw=true)

Now you have your plugin base, but you don't know how to load the Regex proposals and first of all, find where they are.

118
## Finding the Regex proposals:

I knew that the Regex proposals already exists inside Eclipse because you can enable them when you do a CTRL+F.

It took 2 days to find them, I used the debugger, by putting break points on function, and I finally 

## What's missing?
