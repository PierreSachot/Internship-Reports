# WEEK SEVEN: Scripting Eclipse With EASE

## I- What's EASE for Eclipse?

What is a script ?

> Programs written for a special run-time environment that automate the
> execution of tasks that could alternatively be executed one-by-one by a human
> operator. \-Wikipedia

Scripts often are used to automate tedious tasks, like renaming a lot of files
in a folder to correspond to a certain pattern. Scripting is very usefull and
programmers know it and are fond of them. But what if you could script your
favorite Java IDE, Eclipse, to automate tasks like setting up a project or
creating Readme files for all projects in the Workspace? Well you actually can.

Using a plugin called EASE, which stands for Eclipse Advanced Scripting
Environment, you can EASEly script Eclipse using your favorite script language,
like Javascript or Python. You could even code and add your own Module to EASE
to further enhance you and the community's scripting experience.

Try it out by installing
[EASE](http://download.eclipse.org/ease/update/release), but take care not
trying to automate everything, or you will end up coding more scripts than
actually getting work done.

![xkcd n°1319](https://imgs.xkcd.com/comics/automation.png)

## II- How to code python inside Eclipse?

### Our start:
During this week, we worked for the most part on python developping with EASE. To start, we follow Kichwa Coders's tutorial ([here](https://github.com/jonahkichwacoders/EASE-Python-Examples) see EASE.htm) which explain EASE bases, on how to develop and what is possible with EASE.
So the first things we did was to create a script which create a dialog asking you your name. After that, we worked on a really interresting function, EASE allows to generate a new Eclipse feature, like in the pop-up menu. Those functions works pretty well, without creating any plugins.
One example of code:

The code below allows the user to select text and replace inside of the selection all the private by public just by doing a right click on it and select "Replace private with public".


```Python
# name           : Replace private with public
# popup          : enableFor(java.lang.Object)
# description    : Replace private keyword with public keyword for selection

loadModule('/System/UI')

def find_replace(find, replace):
    editor = getActiveEditor()
    doc = editor.getDocumentProvider().getDocument(editor.getEditorInput())
   	selection = editor.getSelectionProvider().getSelection()
    text = selection.getText()
    after = text.replace(find, replace)
    if text != after:
        doc.replace(selection.getOffset(), selection.getLength(), after)

executeUI("find_replace('private', 'public')")
```

You can see that this code is really simple, because you can have an access to every Eclipse elements by functions writed with modules. 
Create this function in Java would be really complicated to write and bigger than this code.

### Our project:

After mastering EASE, we wanted to create our stuff, the idea was to create easily with a script a new project, and put a January example inside it. We choose to put Dataset creation example. Here is the steps we needed to do:

1. Create a project - already available in EASE Modules
2. Change this project in a Java Project - not available in EASE Modules
3. Add JVM dependencies - not available in EASE Modules
4. Create a SRC folder - not available in EASE Modules
5. Download January and its dependencies - not available in EASE Modules
6. Add those libraries to our project - not available in EASE Modules
7. Create a new file - 	already available in EASE Modules
8. Change this file in a Java source file - not available in EASE Modules
9. Put our code inside this file - already available in EASE Modules

So, it's a lot of steps, with a lot of function not created for the moment inside EASE... This will create a big python code, but we will reduce it in the Third part of this week report.
Here is our code:

```Python
import urllib2
from distutils.sysconfig import project_base

loadModule("/System/Resources")
include('workspace://Utilities/java_array.py')

# https://dzone.com/articles/use-eclipse-jdt-dynamically

IFolder = org.eclipse.core.resources.IFolder
IProject = org.eclipse.core.resources.IProject
IProjectDescription = org.eclipse.core.resources.IProjectDescription
IWorkspaceRoot = org.eclipse.core.resources.IWorkspaceRoot
ResourcesPlugin = org.eclipse.core.resources.ResourcesPlugin
CoreException = org.eclipse.core.runtime.CoreException
IClasspathEntry = org.eclipse.jdt.core.IClasspathEntry
ICompilationUnit = org.eclipse.jdt.core.ICompilationUnit
IJavaProject = org.eclipse.jdt.core.IJavaProject
IPackageFragment = org.eclipse.jdt.core.IPackageFragment
IPackageFragmentRoot = org.eclipse.jdt.core.IPackageFragmentRoot
IType = org.eclipse.jdt.core.IType
JavaCore = org.eclipse.jdt.core.JavaCore
JavaModelException = org.eclipse.jdt.core.JavaModelException
JavaRuntime = org.eclipse.jdt.launching.JavaRuntime

global project


def create_java_project(name):
    global project
    
    project = getProject(name)
    if project.exists():
        raise Exception("Project {} already exists".format(name))
    project = createProject(name)
    description = project.getDescription()
    natureId = org.eclipse.jdt.core.JavaCore.NATURE_ID
    description.setNatureIds(java_array([natureId], java_type=java.lang.String))
    project.setDescription(description, None)

    javaProject = JavaCore.create(project)
    dico_addresses = {"january.jar":"january address", "commons1.jar" : "commons-lang-2.6 address",
                  "slf4j.jar" : "slf4j-api-1.8.0-alpha2 address", "commons2.jar" : "commons-math3-3.6.1 address"}

	for key in dico_addresses.keys():
    	download_file(dico_addresses[key], key)

    # set the build path
    buildPath = java_array([JavaCore.newSourceEntry(project.getFullPath().append("src")), JavaRuntime.getDefaultJREContainerEntry(), JavaCore.newLibraryEntry(project.getFullPath().append("january.jar"), None, None), JavaCore.newLibraryEntry(project.getFullPath().append("commons1.jar"), None, None), JavaCore.newLibraryEntry(project.getFullPath().append("commons2.jar"), None, None), JavaCore.newLibraryEntry(project.getFullPath().append("slf4j.jar"), None, None)], java_type=IClasspathEntry)
    javaProject.setRawClasspath(buildPath, project.getFullPath().append("bin"), None)

    # create folder by using resources package
    folder = project.getFolder("src")
    folder.create(True, True, None)

    return javaProject

def create_java_package(javaProject, packageName):
    srcFolder = javaProject.getPackageFragmentRoot(javaProject.getProject().getFolder("src"))
    javaPackage = srcFolder.createPackageFragment(packageName, True, None)
    return javaPackage

def create_java_class(javaPackage, className):
    # init code string and create compilation unit
    str = '''package {packageName};

import org.eclipse.january.dataset.DTypeUtils;
import org.eclipse.january.dataset.Dataset;
import org.eclipse.january.dataset.DatasetFactory;
import org.eclipse.january.dataset.DoubleDataset;
import org.eclipse.january.dataset.Random;


public class {className} {{

    public static void main(String[] args)
    {{
        //our class code
    }}

}}
'''.format(packageName=javaPackage.getElementName(), className=className)

    cu = javaPackage.createCompilationUnit("{className}.java".format(className=className), str,
                    False, None);

def download_file(url, name):
    global project
    with open(getWorkspace().getLocation().toString() + '/' +project.getFullPath().toString()+'/'+name,'wb') as f:
        f.write(urllib2.urlopen(url).read())
        f.close()
    print "Download Complete!"

javaProject = create_java_project("test")
packageName = "com.kichwacoders.{}".format("test")
javaPackage = create_java_package(javaProject, packageName)
className = "test"[0].upper() + "test"[1:] + "Demo"
create_java_class(javaPackage, className)



```

Here you can see that the EASE Method is nice but really complicated because there is no autocompletion, create your own code link to Eclipse is really complicated because of that... That is why you can create a module to generate a new function callable in python, this will be explain in the part below.

## III- How to create EASE Modules?

### EASE Module? What is it?

Here you can see all the EASE Modules:
[image ease modules]()

A EASE Module is a function that you can call in your favorite scripting language which will execute a Java code for exemple by using Py4j in our case. Here we want to simplify our previous code, by allowing the user to:
	- Create directly a Java project with JVM dependencies in one line
	- Create src folder easily or directly if he create a Java project
	- Download file and put them inside of his project
	- Implements dependencies

So to do that, the first thing to do is to create a plug-in project.
Then, add those dependencies:

[image dependencies]()

After that, the thing to do is to create an extension to ease modules, create a new module and add dependencies to it:

[Ease module]()

[new module]()

[module parameters]() 

[new dependencies]()

[dependencies parameters]()

When it's done you can begin to write your code inside your class - Here :JPCreator
We did an other class to save project entries informations:

Entries.java
```Java
import java.util.HashSet;
import java.util.Set;

import org.eclipse.core.resources.IProject;
import org.eclipse.core.runtime.IProgressMonitor;
import org.eclipse.core.runtime.NullProgressMonitor;
import org.eclipse.jdt.core.IClasspathEntry;
import org.eclipse.jdt.core.IJavaProject;

public class Entries {

	private Set<IClasspathEntry> entries;
	private IProject projet;
	private IJavaProject javaProject;
	private IProgressMonitor progressMonitor;
	
	 public Entries(IProject project)
	 {
		 this.entries = new HashSet<IClasspathEntry>();
		 this.projet = project;
		 this.javaProject = null;
		 this.progressMonitor = new NullProgressMonitor();
	 }
	
	public Set<IClasspathEntry> getEntries() {
		return entries;
	}

	public void setEntries(Set<IClasspathEntry> entries) {
		this.entries = entries;
	}

	public IProject getProject() {
		return projet;
	}

	public void setProject(IProject project) {
		this.projet = project;
	}


	public IJavaProject getJavaProject() {
		return javaProject;
	}

	public void setJavaProject(IJavaProject javaProject) {
		this.javaProject = javaProject;
	}

	public IProgressMonitor getProgressMonitor() {
		return progressMonitor;
	}

	public void setProgressMonitor(IProgressMonitor progressMonitor) {
		this.progressMonitor = progressMonitor;
	}
}

```

And our JPCreator class:
```Java
import org.eclipse.core.resources.IProject;
import org.eclipse.core.resources.IProjectDescription;
import org.eclipse.core.resources.IWorkspaceRoot;
import org.eclipse.core.resources.ResourcesPlugin;
import org.eclipse.core.runtime.CoreException;
import org.eclipse.core.runtime.IProgressMonitor;
import org.eclipse.jdt.core.IClasspathEntry;
import org.eclipse.jdt.core.JavaCore;
import org.eclipse.jdt.core.JavaModelException;
import org.eclipse.jdt.launching.JavaRuntime;

import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.net.URL;
import java.nio.channels.Channels;
import java.nio.channels.ReadableByteChannel;

public class JPCreator {

	public void createJavaSrcFolder(Entries entries) {
		new File(entries.getProject().getLocation().toString() + "/src").mkdirs();
		entries.getEntries().add(JavaCore.newSourceEntry(entries.getProject().getFullPath().append("src")));
	}

	public Entries createJavaProject(String project_name) throws CoreException {
		IWorkspaceRoot root = ResourcesPlugin.getWorkspace().getRoot();
		IProject project = root.getProject(project_name);
		Entries entries = new Entries(project);
		IProgressMonitor progressMonitor = entries.getProgressMonitor();
		project.create(progressMonitor);
		project.open(progressMonitor);

		// Creating JavaProject
		IProjectDescription description = project.getDescription();
		String[] natures = description.getNatureIds();
		String[] newNatures = new String[natures.length + 1];
		System.arraycopy(natures, 0, newNatures, 0, natures.length);
		newNatures[natures.length] = JavaCore.NATURE_ID;
		description.setNatureIds(newNatures);
		project.setDescription(description, progressMonitor);
		entries.setJavaProject(JavaCore.create(project));

		// Adding dependenciess
		entries.getEntries().add(JavaRuntime.getDefaultJREContainerEntry());
		entries.getJavaProject().setRawClasspath(
				entries.getEntries().toArray(new IClasspathEntry[entries.getEntries().size()]), progressMonitor);
		entries.getProject().refreshLocal(0, entries.getProgressMonitor());
		createJavaSrcFolder(entries);
		return entries;
	}

	public void createJavaDependencies(Entries entries, String libraryName) throws JavaModelException {
		entries.getEntries()
				.add(JavaCore.newLibraryEntry(entries.getProject().getFullPath().append(libraryName), null, null));
		entries.getJavaProject().setRawClasspath(
				entries.getEntries().toArray(new IClasspathEntry[entries.getEntries().size()]),
				entries.getProgressMonitor());

	}

	public String downloadFile(String url, Entries entries, String nom) throws IOException {
		FileOutputStream fos = null;
		URL website = new URL(url);
		ReadableByteChannel rbc = Channels.newChannel(website.openStream());
		fos = new FileOutputStream(entries.getProject().getLocation().toString() + "/" + nom);
		fos.getChannel().transferFrom(rbc, 0, Long.MAX_VALUE);
		System.out.println("Download complete!");
		fos.close();
		return nom;
	}

}
```

Now if you execute this plug-in by launching it as Eclipse application, in the Scripting view, you will have those new functions:

[New module]()

And this is our new script:

```Python
loadModule('/EASE-Project-Creator.JPCreator');
loadModule('/System/Resources');


className = "Example"
classCode = '''
import org.eclipse.january.dataset.DTypeUtils;
import org.eclipse.january.dataset.Dataset;
import org.eclipse.january.dataset.DatasetFactory;
import org.eclipse.january.dataset.DoubleDataset;
import org.eclipse.january.dataset.Random;


public class {className} {{

    public static void main(String[] args)
    {{
        our code
    }}

}}
'''.format(className=className)



entries = createJavaProject("JanuaryProject")
refreshResource(entries.getProject())
januaryFile = createFile("workspace://JanuaryProject/src/"+className+".java");
writeFile(januaryFile, classCode)

dico_addresses = {"january.jar":"https://github.com/PierreSachot/JanuaryGameOfLife/raw/master/january.jar", "commons1.jar" : "https://search.maven.org/remotecontent?filepath=commons-lang/commons-lang/2.6/commons-lang-2.6.jar",
                  "slf4j.jar" : "https://search.maven.org/remotecontent?filepath=org/slf4j/slf4j-api/1.8.0-alpha2/slf4j-api-1.8.0-alpha2.jar", "commons2.jar" : "https://search.maven.org/remotecontent?filepath=org/apache/commons/commons-math3/3.6.1/commons-math3-3.6.1.jar"}

for key in dico_addresses.keys():
    path = downloadFile(dico_addresses[key], entries, key)
    createJavaDependencies(entries, path)

refreshResource(entries.getProject())
```

You can see that the code is really small comparing to the previous version, and by creating this new EASE Module, you have simplify a lot of task you would may like to do again.

## Conclusion

EASE is a very usfull tool you can use to rapidly extend Eclipse to automate a
task, or to prototype an Eclipse Plug-in you want to write. Sadly EASE only is
in its early stages and there still is a lot of room for improvements. For
example there is no mixed Java/Python completiton making writing code a bit
harder if you haven't read the documentation relative to the EASE module you
are using. A lot more modules must be written in order to be able to use
Eclipse at it maximum through scripts. The Eclipse foundation recognised the
use for such a tool as it got a Community Award in 2016 for Most Innovative
Project.
