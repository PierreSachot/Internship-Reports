# Week 5:

## Context:

This week I worked on Eclipse JDT, this is the Java version of [Eclipse](https://eclipse.org/).
My idea was to add a Regex completion proposal in Eclipse editor. In this document, I will tell you how did I do, what were my ideas and how to create your own stuff inside Eclipse JDT.

**Important: *My code isn't done, I need to include my stuff inside of the Eclipse structure, so I will continue this blog during the entire time I'm working on this new feature.***

## My ideas:

 1. Firstly, I wanted to add my stuff inside the Completion proposal loader, but this is a **really bad idea!**

This is a bad idea because your module will be load all the time, you can have the idea to create a global variable to enable or disable your changes, but if all developpers do that, Eclipse code would be a anthill...
Eclipse structure doesn't work like that, that's because you mustn't do that.
 
  2. Secondly, I wanted to add my code inside of an other completion proposal, but this is a **bad idea** too for the same reasons than before.
  
  3. Thirdly, Jonah Graham from Kichwa Coders company told me that in Eclipse everything is plugin.

This mean that every part of Eclipse are an external module that is load. Eclipse is make from plugins.

## Create your Eclipse plug-in:

 1. You need to download Eclipse's last version, and to follow this tutorial:
   - Eclipse last version: [Eclipse last build](http://download.eclipse.org/eclipse/downloads/) (Last integration Build)
   - Tutorial: [Eclipse JDT Integration](https://wiki.eclipse.org/JDT_UI/How_to_Contribute)
   
 2. Launch Eclipse's last version and do "File->New->Project...":
 
![Create Project](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%205/Screenshot_2.png?raw=true)
  
 3. Select "Plug-in Project" and click "Next >":
 
![Project selection](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%205/Screenshot_3.png?raw=true)
  
 4. Name your plug-in - here "tutorialPlugin" - and click "Next >":
 
![Name the project](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%205/Screenshot_4.png?raw=true)
  
 5. Enable "Generate an activator, a Java class that controls the plug-in's life cycle" and "This plug-in will make contributions to the UI" and click "Finish":
 
![Checkbox](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%205/Screenshot_5.png?raw=true)

 6. If the "Open Associated perspective?" is shown, click on "Open Perspective":

![associate perspective](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%205/Screenshot_6.png?raw=true)

Now you should have this view:

![Project view](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%205/Screenshot_7.png?raw=true)

Your plugin is now created, but how to link it to Eclipse? This is possible with something call "Hooks".

## What's a Hook:


	In computer programming, the term hooking covers a range of techniques used to alter or augment the behavior of an operating system, of applications, or of other software components by intercepting function calls or messages or events passed between software components. Code that handles such intercepted function calls, events or messages is called a hook.

   From Wikipedia.

In our case, a hook is something that allows the developper to create an external module that Eclipse can load. It's like a door which allow the developper to create a new feature and say to the program "Load this code".

The problem is, you don't have only one Hook inside of Eclipse, but a lot, you can see all hooks by clicking on the link bellow :

[Eclipse Hooks](https://help.eclipse.org/neon/index.jsp?topic=%2Forg.eclipse.platform.doc.isv%2Freference%2Fextension-points%2Findex.html&cp=2_1_1)

Now you need to find the right hook, for that I only watch inside others completion proposals plugins on which hook they were attaching to.
In my case, this is the : `javaCompletionProposalComputer`.

## Hooking with your plugin:

Firstly, you need to add in your MANIFEST that what you are creating is a Singleton. To do that, open the manifest file and do that:

![singleton](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%205/Screenshot_8.png?raw=true)

You need to add some dependencies :

  - Open your MANIFEST and go to "Dependencies"

![dependencies](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%205/Screenshot_11.png?raw=true)

  - Click on "Add.."
  - Add those dependencies:
  
```
org.eclipse.jface.text
org.eclipse.swt
org.eclipse.jface
```

 If you don't find some dependencies, add them directly inside the MANIFEST:

![dependencies](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%205/Screenshot_9.png?raw=true)

 Now you should have that in the dependencies: 

![dependencies](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%205/Screenshot_10.png?raw=true)

 Now you can Hook your plugin to the `javaCompletionProposalComputer`.

  - Go inside the "Extensions" bloc:

![extensions](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%205/Screenshot_12.png?raw=true)

  - Click on "Add..."

  - Search `javaCompletionProposalComputer`, select it and click "Finish":

![Extension point](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%205/Screenshot_13.png?raw=true)

Now you have your plugin base, but you don't know how to load the Regex proposals and first of all, find where they are.

## Finding the Regex proposals:

I knew that the Regex proposals already exists inside Eclipse because you can enable them when you do a CTRL+F.

It took 2 days to find them, I used the debugger, by putting break points on function, and I finally found that Regex are in a class name: `FindReplaceDocumentAdapterContentProposalProvider.java`.
To load every proposals, you need to do that:

```Java
FindReplaceDocumentAdapterContentProposalProvider finder = new FindReplaceDocumentAdapterContentProposalProvider(true);
IContentProposal[] regex = finder.getProposals("", 0);
```
Here, in the regex array, there is all the proposals.

Now you need to create your plugin using those propositions.

## Coding the plugin:

 1. Create a new class with the name you want - here I chose "RegexProposalComputer.java" - and implements `IJavaCompletionProposalComputer.java` interface:
 
![Regex class](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%205/Screenshot_14.png?raw=true)

 2. Create an other class which will be use to store the proposals from the `FindReplaceDocumentAdapterContentProposalProvider.java` and to use them -here we will name it `CompletionProposal.java`:
  - For that implements the `ICompletionProposal` class

![Completion proposal](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%205/Screenshot_15.png?raw=true)

  - Past this code inside:
  
```Java
import org.eclipse.jdt.ui.text.java.ContentAssistInvocationContext;
import org.eclipse.jface.text.IDocument;
import org.eclipse.jface.text.contentassist.ICompletionProposal;
import org.eclipse.jface.text.contentassist.IContextInformation;
import org.eclipse.swt.graphics.Image;

public class CompletionProposal implements ICompletionProposal {
	
	private final String displayString;
	private final String additionalString;
	private final ContentAssistInvocationContext context;
	
	public CompletionProposal(final ContentAssistInvocationContext context, final String toDisplay, final String additionalInformations)
	{
		displayString = toDisplay;
		additionalString = additionalInformations;
		this.context = context;
	}

	 @Override
     public String getDisplayString() {
       // this is the label shown in the completion list
       return displayString;
     }

     @Override
     public String getAdditionalProposalInfo() {
       // this is the additional piece of information shown when selected
       return additionalString;
     }

     @Override
     public IContextInformation getContextInformation() {
       // @return the context information for this proposal or <code>null</code>
       return null;
     }

     /** Inserts the proposed completion at the cursor position */
     @Override
     public void apply(IDocument document) {
       int position = context.getViewer().getSelectedRange().x;
       String currentText = document.get();
       int index = context.getInvocationOffset();
       String before = currentText.substring(0, index);
       String after = currentText.substring(index);
       document.set(before + getDisplayString() + after);
       context.getViewer().setSelectedRange(position + getDisplayString().length() + 1, -1);
     }

     @Override
     public org.eclipse.swt.graphics.Point getSelection(IDocument document) {
       // If it returns * <code>null</code>, no new selection is set.
       return null;
     }

     @Override
     public Image getImage() {
       //  @return the image to be shown or <code>null</code> if no image is desired
       return null;
     }

}
```

   3. Come back in your RegexProposal.java and add this code inside the `computeCompletionProposals()` function:

  ```Java
  List<ICompletionProposal> res = new ArrayList<ICompletionProposal>();   

  FindReplaceDocumentAdapterContentProposalProvider finder = new FindReplaceDocumentAdapterContentProposalProvider(true);
  IContentProposal[] regex = finder.getProposals("", 0);
  for(IContentProposal test : regex)
  {
     res.add(new CompletionProposal(context, test.getContent(), test.getDescription()));
  }
  return res;
  ```
The last thing to do is to change some settings inside the `plugin.xml`:
You need to follow the color code to understand why I put those informations at those places.

![plugin.xml](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%205/Screenshot_16.png?raw=true)

For the `categoryId` the identifier come from the MANIFEST:

![plugin.xml](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%205/Screenshot_17.png?raw=true)

![plugin.xml](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%205/Screenshot_18.png?raw=true)

Now you can start your plug-in, to do that, just do:

![plugin.xml](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%205/Screenshot_20.png?raw=true)

Then when you press "CTRL+SPACE", this completion proposal should show:

![plugin.xml](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%205/Screenshot_19.png?raw=true)

So what's missing? What need I do now?

## What's missing?

Now I need to add my plugin inside Eclipse JDT structure to have it directly without needed to download my plugin.
