# WEEK SEVEN: Scripting Eclipse With EASE

## I- What's EASE for Eclipse?

## II- How to code python inside Eclipse?

### Our start:
During this week, we worked for the most part on python developping with EASE. To start, we follow Kichwa Coders's tutorial ([here](https://github.com/jonahkichwacoders/EASE-Python-Examples) see EASE.htm) which explain EASE bases, on how to develop and what is possible with EASE.
So the first things we did was to create a script which create a dialog askip you your name. After that, we worked on a really interresting function, EASE allows to generate a new Eclipse feature, like in the pop-up menu. Those functions works pretty well, without creating any plugins.
One example of code:

The code bellow allows the user to select text and replace inside of the selection all the private by public just by doing a right click on it and select "Replace private with public".


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
	1- Create a project - already available in EASE Modules
	2- Change this project in a Java Project - not available in EASE Modules
	3- Add JVM dependencies - not available in EASE Modules
	4- Create a SRC folder - not available in EASE Modules
	5- Download January and its dependencies - not available in EASE Modules
	6- Add those libraries to our project - not available in EASE Modules
	7- Create a new file - 	already available in EASE Modules
	8- Change this file in a Java source file - not available in EASE Modules
	9- Put our code inside this file - already available in EASE Modules

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

Here you can see that the EASE Method is nice but really complicated because there is no autocompletion, create your own code link to Eclipse is really complicated because of that... That is why you can create a module to generate a new function callable in python, this will be explain in the part bellow.

## III- How to create EASE Modules?

## Conclusion
- not easy, no auto-completion
