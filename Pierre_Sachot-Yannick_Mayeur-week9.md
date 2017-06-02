# WEEK NINE: Add a new ToolChain to CDT

## I- Context

During this week, we needed to add MSYS2 toolchain to Eclipse CDT, we finally understand that it was already the case, but was made in MinGW toolchain, which is a really bad thing for the user which doesn't know what he is using. Previous developers didthat because MSYS2 is using MinGW.

## II- Where Toolchains are ?

Inside of Eclipse CDT, because it's a RCP application, everything is plugin, Toolchains too, that means that you can easily provide a new toolchain inside of your CDT, but you already need to understand how to do that...

So, first of all, the plugin name is: `org.eclipse.cdt.managedbuilder.gnu.ui`.

Inside of this plugin, you will need to change some importants element:

 - Create inside `org.eclipse.cdt.core` a new class to find the toolchain you want to add.
 - Create a new package to create a new `EnvironnementVariableSupplier` and a new `isToolchainSupported`
 - plugin.xml: this is where all the toolchains are declarated here and the projects examples.
 
 
## III- Create a new Toolchain

  - Create inside `org.eclipse.cdt.core` a new class to find the toolchain you want to add. Here is for MSYS2: 
  
```Java
package org.eclipse.cdt.internal.core;

import java.io.File;
import java.util.Collections;
import java.util.Map;
import java.util.WeakHashMap;

import org.eclipse.cdt.core.CCorePlugin;
import org.eclipse.cdt.core.envvar.IEnvironmentVariable;
import org.eclipse.cdt.core.settings.model.ICConfigurationDescription;
import org.eclipse.cdt.core.settings.model.util.CDataUtil;
import org.eclipse.cdt.utils.PathUtil;
import org.eclipse.cdt.utils.WindowsRegistry;
import org.eclipse.core.runtime.IPath;
import org.eclipse.core.runtime.Path;
import org.eclipse.core.runtime.Platform;

/**
 * A collection of MinGW-related utility methods.
 */
public class MSYS2 {
	public static final String ENV_MINGW_HOME = "MINGW_HOME"; //$NON-NLS-1$
	public static final String ENV_MSYS_HOME = "MSYS_HOME"; //$NON-NLS-1$
	private static final String ENV_PATH = "PATH"; //$NON-NLS-1$

	private static final boolean isWindowsPlatform = Platform.getOS().equals(Platform.OS_WIN32);

	private static String envPathValueCached = null;
	private static String envMinGWHomeValueCached = null;
	private static String minGWLocation = null;
	private static boolean isMinGWLocationCached = false;

	private static String envMinGWHomeValueCached_msys = null;
	private static String mSysLocation = null;
	private static boolean isMSysLocationCached = false;

	private final static Map<String/* envPath */, String/* mingwLocation */> mingwLocationCache = Collections
			.synchronizedMap(new WeakHashMap<String, String>(1));

	/**
	 * @return The absolute path to MinGW root folder or {@code null} if not
	 *         found
	 */
	private static String findMinGWRoot(String envPathValue, String envMinGWHomeValue) {
		String rootValue = null;
		// Look in MSYS2
		if (rootValue == null) {
			WindowsRegistry registry = WindowsRegistry.getRegistry();
			String uninstallKey = "SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Uninstall"; //$NON-NLS-1$
			String subkey;
			boolean on64bit = Platform.getOSArch().equals(Platform.ARCH_X86_64);
			String key32bit = null;
			for (int i = 0; (subkey = registry.getCurrentUserKeyName(uninstallKey, i)) != null; i++) {
				String compKey = uninstallKey + '\\' + subkey;
				String displayName = registry.getCurrentUserValue(compKey, "DisplayName"); //$NON-NLS-1$
				if (on64bit) {
					if ("MSYS2 64bit".equals(displayName)) { //$NON-NLS-1$
						String installLocation = registry.getCurrentUserValue(compKey, "InstallLocation"); //$NON-NLS-1$
						String mingwLocation = installLocation + "\\mingw64"; //$NON-NLS-1$
						File gccFile = new File(mingwLocation + "\\bin\\gcc.exe"); //$NON-NLS-1$
						if (gccFile.canExecute()) {
							rootValue = mingwLocation;
							break;
						} else {
							mingwLocation = installLocation + "\\mingw32"; //$NON-NLS-1$
							gccFile = new File(mingwLocation + "\\bin\\gcc.exe"); //$NON-NLS-1$
							if (gccFile.canExecute()) {
								rootValue = mingwLocation;
								break;
							}
						}
					} else if ("MSYS2 32bit".equals(displayName)) { //$NON-NLS-1$
						key32bit = compKey;
					}
				} else {
					if ("MSYS2 32bit".equals(displayName)) { //$NON-NLS-1$
						String installLocation = registry.getCurrentUserValue(compKey, "InstallLocation"); //$NON-NLS-1$
						String mingwLocation = installLocation + "\\mingw32"; //$NON-NLS-1$
						File gccFile = new File(mingwLocation + "\\bin\\gcc.exe"); //$NON-NLS-1$
						if (gccFile.canExecute()) {
							rootValue = mingwLocation;
							break;
						}
					}
				}
			}

			if (on64bit && key32bit != null) {
				String installLocation = registry.getCurrentUserValue(key32bit, "InstallLocation"); //$NON-NLS-1$
				String mingwLocation = installLocation + "\\mingw64"; //$NON-NLS-1$
				File gccFile = new File(mingwLocation + "\\bin\\gcc.exe"); //$NON-NLS-1$
				if (gccFile.canExecute()) {
					rootValue = mingwLocation;
				} else {
					mingwLocation = installLocation + "\\mingw32"; //$NON-NLS-1$
					gccFile = new File(mingwLocation + "\\bin\\gcc.exe"); //$NON-NLS-1$
					if (gccFile.canExecute()) {
						rootValue = mingwLocation;
					}
				}
			}
		}

		return rootValue;
	}

	private static String findMingwInPath(String envPath) {
		if (envPath == null) {
			// $PATH from user preferences
			IEnvironmentVariable varPath = CCorePlugin.getDefault().getBuildEnvironmentManager()
					.getVariable(ENV_PATH, (ICConfigurationDescription) null, true);
			if (varPath != null) {
				envPath = varPath.getValue();
			}
		}

		String mingwLocation = mingwLocationCache.get(envPath);
		// check if WeakHashMap contains the key as null may be the cached value
		if (mingwLocation == null && !mingwLocationCache.containsKey(envPath)) {
			// Check for MinGW-w64 on Windows 64 bit, see
			// http://mingw-w64.sourceforge.net/
			if (Platform.ARCH_X86_64.equals(Platform.getOSArch())) {
				IPath gcc64Loc = PathUtil.findProgramLocation("x86_64-w64-mingw32-gcc.exe", envPath); //$NON-NLS-1$
				if (gcc64Loc != null) {
					mingwLocation = gcc64Loc.removeLastSegments(2).toOSString();
				}
			}

			// Look for mingw32-gcc.exe
			if (mingwLocation == null) {
				IPath gccLoc = PathUtil.findProgramLocation("mingw32-gcc.exe", envPath); //$NON-NLS-1$
				if (gccLoc != null) {
					mingwLocation = gccLoc.removeLastSegments(2).toOSString();
				}
			}
			mingwLocationCache.put(envPath, mingwLocation);
		}

		return mingwLocation;
	}

	private static String findMSysRoot(String envMinGWHomeValue) {
		String msysHome = null;

		// Try under MSYS2
		if (msysHome == null) {
			// Give preference to msys64 on 64-bit platforms and ignore 64 on
			// 32-bit platforms
			WindowsRegistry registry = WindowsRegistry.getRegistry();
			String uninstallKey = "SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Uninstall"; //$NON-NLS-1$
			String subkey;
			boolean on64bit = Platform.getOSArch().equals(Platform.ARCH_X86_64);
			String key32bit = null;
			for (int i = 0; (subkey = registry.getCurrentUserKeyName(uninstallKey, i)) != null; i++) {
				String compKey = uninstallKey + '\\' + subkey;
				String displayName = registry.getCurrentUserValue(compKey, "DisplayName"); //$NON-NLS-1$
				if (on64bit) {
					if ("MSYS2 64bit".equals(displayName)) { //$NON-NLS-1$
						String home = registry.getCurrentUserValue(compKey, "InstallLocation"); //$NON-NLS-1$
						if (new File(home).isDirectory()) {
							msysHome = home;
							break;
						}
					} else if ("MSYS2 32bit".equals(displayName)) { //$NON-NLS-1$
						key32bit = compKey;
					}
				} else {
					if ("MSYS2 32bit".equals(displayName)) { //$NON-NLS-1$
						String home = registry.getCurrentUserValue(compKey, "InstallLocation"); //$NON-NLS-1$
						if (new File(home).isDirectory()) {
							msysHome = home;
							break;
						}
					}
				}
			}

			if (on64bit && key32bit != null) {
				String home = registry.getCurrentUserValue(key32bit, "InstallLocation"); //$NON-NLS-1$
				if (new File(home).isDirectory()) {
					msysHome = home;
				}
			}
		}
		return msysHome;
	}

	/**
	 * Find location where MinGW is installed. A number of locations is being
	 * checked, such as environment variable $MINGW_HOME, $PATH, Windows
	 * registry et al. <br>
	 * <br>
	 * If you use this do not cache results to ensure user preferences are
	 * accounted for. Please rely on internal caching.
	 * 
	 * @return MinGW root ("/") path in Windows format.
	 */
	public static String getMinGWHome() {
		if (!isWindowsPlatform) {
			return null;
		}

		IEnvironmentVariable varPath = CCorePlugin.getDefault().getBuildEnvironmentManager()
				.getVariable(ENV_PATH, (ICConfigurationDescription) null, true);
		String envPathValue = varPath != null ? varPath.getValue() : null;
		IEnvironmentVariable varMinGWHome = CCorePlugin.getDefault().getBuildEnvironmentManager()
				.getVariable(ENV_MINGW_HOME, (ICConfigurationDescription) null, true);
		String envMinGWHomeValue = varMinGWHome != null ? varMinGWHome.getValue() : null;

		// isMinGWLocationCached is used to figure fact of caching when all
		// cached objects are null
		if (isMinGWLocationCached && CDataUtil.objectsEqual(envPathValue, envPathValueCached)
				&& CDataUtil.objectsEqual(envMinGWHomeValue, envMinGWHomeValueCached)) {
			return minGWLocation;
		}

		minGWLocation = findMinGWRoot(envPathValue, envMinGWHomeValue);
		envPathValueCached = envPathValue;
		envMinGWHomeValueCached = envMinGWHomeValue;
		isMinGWLocationCached = true;

		return minGWLocation;
	}

	/**
	 * Find location where MSys is installed. Environment variable $MSYS_HOME
	 * and some predetermined locations are being checked. <br>
	 * <br>
	 * If you use this do not cache results to ensure user preferences are
	 * accounted for. Please rely on internal caching.
	 * 
	 * @return MSys root ("/") path in Windows format.
	 */
	public static String getMSysHome() {
		if (!isWindowsPlatform) {
			return null;
		}

		// Use $MSYS_HOME if defined
		IEnvironmentVariable varMsysHome = CCorePlugin.getDefault().getBuildEnvironmentManager()
				.getVariable(ENV_MSYS_HOME, (ICConfigurationDescription) null, true);
		String msysHomeValue = varMsysHome != null ? varMsysHome.getValue() : null;
		if (msysHomeValue != null) {
			return msysHomeValue;
		}

		String envMinGWHomeValue = getMinGWHome();

		// isMSysLocationCached is used to figure whether it was cached when all
		// cached objects are null
		if (isMSysLocationCached && CDataUtil.objectsEqual(envMinGWHomeValue, envMinGWHomeValueCached_msys)) {
			return mSysLocation;
		}

		mSysLocation = findMSysRoot(envMinGWHomeValue);
		envMinGWHomeValueCached_msys = envMinGWHomeValue;
		isMSysLocationCached = true;

		return mSysLocation;
	}

	/**
	 * Check if MinGW is available in the path.
	 *
	 * @param envPath
	 *            - list of directories to search for MinGW separated by path
	 *            separator (format of environment variable $PATH) or
	 *            {@code null} to use current $PATH.
	 * @return {@code true} if MinGW is available, {@code false} otherwise.
	 */
	public static boolean isAvailable(String envPath) {
		return isWindowsPlatform && findMingwInPath(envPath) != null;
	}

	/**
	 * Check if MinGW is available in $PATH.
	 *
	 * @return {@code true} if MinGW is available, {@code false} otherwise.
	 */
	public static boolean isAvailable() {
		return isWindowsPlatform && findMingwInPath(null) != null;
	}
}

```
  
  - Create a new package to create a new `EnvironnementVariableSupplier` and a new `isToolchainSupported`
  
Here is the new MSYS2 package:

![MSYS2 package](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%209/Screenshot_1.png)

The `EnvironnementVariableSupplier` is to set eclipse environnment variables for the toolchain, so you need to parameterize like that:

```Java
package org.eclipse.cdt.managedbuilder.gnu.MSYS2;

import org.eclipse.cdt.core.CCorePlugin;
import org.eclipse.cdt.core.envvar.IEnvironmentVariable;
import org.eclipse.cdt.core.settings.model.ICConfigurationDescription;
import org.eclipse.cdt.internal.core.MSYS2;
import org.eclipse.cdt.internal.core.envvar.EnvironmentVariableManager;
import org.eclipse.cdt.managedbuilder.core.IConfiguration;
import org.eclipse.cdt.managedbuilder.envvar.IBuildEnvironmentVariable;
import org.eclipse.cdt.managedbuilder.envvar.IConfigurationEnvironmentVariableSupplier;
import org.eclipse.cdt.managedbuilder.envvar.IEnvironmentVariableProvider;
import org.eclipse.cdt.managedbuilder.internal.envvar.BuildEnvVar;
import org.eclipse.core.runtime.IPath;
import org.eclipse.core.runtime.Path;

/**
 * @noextend This class is not intended to be subclassed by clients.
 */
public class MSYS2EnvironmentVariableSupplier implements IConfigurationEnvironmentVariableSupplier {
	private static final String ENV_PATH = "PATH"; //$NON-NLS-1$
	private static final String BACKSLASH = java.io.File.separator;
	private static final String PATH_DELIMITER = EnvironmentVariableManager.getDefault().getDefaultDelimiter();

	@Override
	public IBuildEnvironmentVariable getVariable(String variableName, IConfiguration configuration, IEnvironmentVariableProvider provider) {
		if (variableName.equals(MSYS2.ENV_MINGW_HOME)) {
			IEnvironmentVariable varMinGWHome = CCorePlugin.getDefault().getBuildEnvironmentManager().getVariable(MSYS2.ENV_MINGW_HOME, (ICConfigurationDescription) null, false);
			if (varMinGWHome == null) {
				// Contribute if the variable does not already come from workspace environment
				String minGWHome = MSYS2.getMinGWHome();
				if (minGWHome == null) {
					// If the variable is not defined still show it in the environment variables list as a hint to user
					minGWHome = ""; //$NON-NLS-1$
				}
				return new BuildEnvVar(MSYS2.ENV_MINGW_HOME, new Path(minGWHome).toOSString(), IBuildEnvironmentVariable.ENVVAR_REPLACE);
			}
			return null;

		} else if (variableName.equals(MSYS2.ENV_MSYS_HOME)) {
			IEnvironmentVariable varMsysHome = CCorePlugin.getDefault().getBuildEnvironmentManager().getVariable(MSYS2.ENV_MSYS_HOME, (ICConfigurationDescription) null, false);
			if (varMsysHome == null) {
				// Contribute if the variable does not already come from workspace environment
				String msysHome = MSYS2.getMSysHome();
				if (msysHome == null) {
					// If the variable is not defined still show it in the environment variables list as a hint to user
					msysHome = ""; //$NON-NLS-1$
				}
				return new BuildEnvVar(MSYS2.ENV_MSYS_HOME, new Path(msysHome).toOSString(), IBuildEnvironmentVariable.ENVVAR_REPLACE);
			}
			return null;

		} else if (variableName.equals(ENV_PATH)) {
			@SuppressWarnings("nls")
			String path = "${" + MSYS2.ENV_MINGW_HOME + "}" + BACKSLASH + "bin" + PATH_DELIMITER
					+ "${" + MSYS2.ENV_MSYS_HOME + "}" + BACKSLASH + "bin" + PATH_DELIMITER
					+ "${" + MSYS2.ENV_MSYS_HOME + "}" + BACKSLASH + "usr" + BACKSLASH + "bin";
			return new BuildEnvVar(ENV_PATH, path, IBuildEnvironmentVariable.ENVVAR_PREPEND);
		}

		return null;
	}

	@Override
	public IBuildEnvironmentVariable[] getVariables(IConfiguration configuration, IEnvironmentVariableProvider provider) {
		return new IBuildEnvironmentVariable[] {
				getVariable(MSYS2.ENV_MINGW_HOME, configuration, provider),
				getVariable(MSYS2.ENV_MSYS_HOME, configuration, provider),
				getVariable(ENV_PATH, configuration, provider),
			};
	}

}

```

The `isToolchainSupported` allows Eclipse to know if the toolchain is install on user's PC:

```Java
package org.eclipse.cdt.managedbuilder.gnu.MSYS2;

import org.eclipse.cdt.core.envvar.IEnvironmentVariable;
import org.eclipse.cdt.internal.core.MSYS2;
import org.eclipse.cdt.managedbuilder.core.IManagedIsToolChainSupported;
import org.eclipse.cdt.managedbuilder.core.IToolChain;
import org.eclipse.cdt.managedbuilder.internal.envvar.EnvironmentVariableManagerToolChain;
import org.osgi.framework.Version;

/**
 * @noextend This class is not intended to be subclassed by clients.
 */
public class MSYS2IsToolChainSupported implements IManagedIsToolChainSupported {
	private static final String ENV_PATH = "PATH"; //$NON-NLS-1$

	@Override
	public boolean isSupported(IToolChain toolChain, Version version, String instance) {
		IEnvironmentVariable var = new EnvironmentVariableManagerToolChain(toolChain).getVariable(ENV_PATH, true);
		String envPath = var != null ? var.getValue() : null;
		return MSYS2.isAvailable(envPath);
	}

}
```

  - plugin.xml: this is where all the toolchains are declarated here and the projects examples.
  
So there is a lot of things to create, the first thing is to declar the Toolchain, so inside of the Extension properties, create a new toolchain parameterize like that:

![MSYS2 package](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%209/Screenshot_2.png)

After that there is a lot of tools to create, so we just copy and past the same parameteres than for MinGW inside the xml view:

```XML
<toolChain
    archList="all"
    configurationEnvironmentSupplier="org.eclipse.cdt.managedbuilder.gnu.MSYS2.MSYS2EnvironmentVariableSupplier"
    id="cdt.managedbuild.toolchain.gnu.msys2.base"
    isToolChainSupported="org.eclipse.cdt.managedbuilder.gnu.MSYS2.MSYS2IsToolChainSupported"
    languageSettingsProviders="org.eclipse.cdt.managedbuilder.core.GCCBuildCommandParser;org.eclipse.cdt.managedbuilder.core.GCCBuiltinSpecsDetectorMinGW"
    name="%ToolChainName.MSYS2"
    osList="win32"
    targetTool="cdt.managedbuild.tool.gnu.cpp.linker.mingw.base;cdt.managedbuild.tool.gnu.c.linker.mingw.base;cdt.managedbuild.tool.gnu.archiver">
            <targetPlatform
      id="cdt.managedbuild.target.gnu.platform.msys2.base"
      name="%PlatformName.Dbg"
                binaryParser="org.eclipse.cdt.core.PE"            					  
      osList="win32"					  
      archList="all">
    </targetPlatform>
            <builder
                  id="cdt.managedbuild.tool.gnu.builder.msys2.base"
                  isAbstract="false"
                  isVariableCaseSensitive="false"
                  superClass="org.eclipse.cdt.build.core.internal.builder">
            </builder>
    <tool
      id="cdt.managedbuild.tool.gnu.assembler.msys2.base"
      superClass="cdt.managedbuild.tool.gnu.assembler">
    </tool> 		               		         
      <tool
        id="cdt.managedbuild.tool.gnu.archiver.msys2.base"
          superClass="cdt.managedbuild.tool.gnu.archiver">
  <enablement 
          type="ALL">
    <checkBuildProperty 
      property="org.eclipse.cdt.build.core.buildArtefactType"
      value="org.eclipse.cdt.build.core.buildArtefactType.staticLib"/>
  </enablement>
    </tool>                 
            <tool
                id="cdt.managedbuild.tool.gnu.cpp.compiler.msys2.base"
                superClass="cdt.managedbuild.tool.gnu.cpp.compiler">
            </tool>
            <tool
      id="cdt.managedbuild.tool.gnu.c.compiler.msys2.base"
                superClass="cdt.managedbuild.tool.gnu.c.compiler">
            </tool>
            <tool
                id="cdt.managedbuild.tool.gnu.c.linker.msys2.base"
                name="%ToolName.linker.mingw.gnu.c"
                superClass="cdt.managedbuild.tool.gnu.c.linker">
         <enablement 
          type="ALL">
          <not>
    <checkBuildProperty 
      property="org.eclipse.cdt.build.core.buildArtefactType"
      value="org.eclipse.cdt.build.core.buildArtefactType.staticLib"/>
    </not>
  </enablement>
  <outputType
             id="cdt.managedbuild.tool.gnu.c.linker.msys2.so.output.base"
             superClass="cdt.managedbuild.tool.gnu.c.linker.output.so"
             outputs="dll">
    </outputType>

            </tool>
            <tool
                id="cdt.managedbuild.tool.gnu.cpp.linker.msys2.base"
                name="%ToolName.linker.mingw.gnu.cpp"
                superClass="cdt.managedbuild.tool.gnu.cpp.linker">
         <enablement 
          type="ALL">
          <not>
    <checkBuildProperty 
      property="org.eclipse.cdt.build.core.buildArtefactType"
      value="org.eclipse.cdt.build.core.buildArtefactType.staticLib"/>
    </not>
  </enablement>
    <outputType
             id="cdt.managedbuild.tool.gnu.cpp.linker.msys2.so.output.base"
             superClass="cdt.managedbuild.tool.gnu.cpp.linker.output.so"
             outputs="dll">
    </outputType>
            </tool>                  

</toolChain> 
```

This provides all tools for makes working perfectly MinGW and by the same way MSYS2.

## IV- Projects types

We have a working Toolchain, but now we need to create some projects types. 
This means that you need to create some projects types like that:

```XML
<projectType
            buildArtefactType="org.eclipse.cdt.build.core.buildArtefactType.exe"
            id="cdt.managedbuild.target.gnu.msys2.exe"
            isAbstract="false"
            isTest="false"
            >                                  
         <configuration
               name="%ConfigName.Dbg"
               cleanCommand="rm -rf"
               id="cdt.managedbuild.config.gnu.msys2.exe.debug"
               parent="cdt.managedbuild.config.gnu.msys2.base"
               buildProperties="org.eclipse.cdt.build.core.buildType=org.eclipse.cdt.build.core.buildType.debug">
               <toolChain
               		 superClass="cdt.managedbuild.toolchain.gnu.msys2.base"
                     id="cdt.managedbuild.toolchain.gnu.msys2.exe.debug">
                  <targetPlatform
					  id="cdt.managedbuild.target.gnu.platform.msys2.exe.debug"
					  superClass="cdt.managedbuild.target.gnu.platform.mingw.base">
				  </targetPlatform>
                  <tool
                      id="cdt.managedbuild.tool.gnu.cpp.compiler.msys2.exe.debug"
                      superClass="cdt.managedbuild.tool.gnu.cpp.compiler.mingw.base">
                      <option
                          id="gnu.cpp.compiler.msys2.exe.debug.option.optimization.level"
                          superClass="gnu.cpp.compiler.option.optimization.level">
                      </option>
                      <option
						  id="gnu.cpp.compiler.msys2.exe.debug.option.debugging.level"
                          superClass="gnu.cpp.compiler.option.debugging.level">
                      </option>
                  </tool>
                  <tool
					  id="cdt.managedbuild.tool.gnu.c.compiler.msys2.exe.debug"
                      superClass="cdt.managedbuild.tool.gnu.c.compiler.mingw.base">
					  <option
						  id="gnu.c.compiler.msys2.exe.debug.option.optimization.level"
						  superClass="gnu.c.compiler.option.optimization.level">
					  </option>
					  <option
						  id="gnu.c.compiler.msys2.exe.debug.option.debugging.level"
						  superClass="gnu.c.compiler.option.debugging.level">
					  </option>
                  </tool>
                  <tool
                      id="cdt.managedbuild.tool.gnu.c.linker.msys2.exe.debug"
                      superClass="cdt.managedbuild.tool.gnu.c.linker.mingw.base">
                  </tool>
                  <tool
                      id="cdt.managedbuild.tool.gnu.cpp.linker.msys2.exe.debug"
                      superClass="cdt.managedbuild.tool.gnu.cpp.linker.mingw.base">
                  </tool>                  
				  <tool
					  id="cdt.managedbuild.tool.gnu.assembler.msys2.exe.debug"
					  superClass="cdt.managedbuild.tool.gnu.assembler.mingw.base">
				  </tool>   
               </toolChain>                                   
         </configuration>
         <configuration
               name="%ConfigName.Rel"
               cleanCommand="rm -rf"
               id="cdt.managedbuild.config.gnu.msys2.exe.release"
               parent="cdt.managedbuild.config.gnu.msys2.base"
               buildProperties="org.eclipse.cdt.build.core.buildType=org.eclipse.cdt.build.core.buildType.release">
               <toolChain
                     superClass="cdt.managedbuild.toolchain.gnu.msys2.base"
                     id="cdt.managedbuild.toolchain.gnu.msys2.exe.release">
                  <targetPlatform
					  id="cdt.managedbuild.target.gnu.platform.msys2.exe.release"
					  superClass="cdt.managedbuild.target.gnu.platform.mingw.base">
				  </targetPlatform>
                  <tool
                      id="cdt.managedbuild.tool.gnu.cpp.compiler.msys2.exe.release"
                      superClass="cdt.managedbuild.tool.gnu.cpp.compiler.msys2.base">
                      <option
                          id="gnu.cpp.compiler.msys2.exe.release.option.optimization.level"
                          superClass="gnu.cpp.compiler.option.optimization.level">
                      </option>
                      <option
                          id="gnu.cpp.compiler.msys2.exe.release.option.debugging.level"
                          superClass="gnu.cpp.compiler.option.debugging.level">
                      </option>
                  </tool>                      
                  <tool
					  id="cdt.managedbuild.tool.gnu.c.compiler.msys2.exe.release"
                      superClass="cdt.managedbuild.tool.gnu.c.compiler.mingw.base">
                      <option
                          id="gnu.c.compiler.msys2.exe.release.option.optimization.level"
                          superClass="gnu.c.compiler.option.optimization.level">
                      </option>
                      <option
                          id="gnu.c.compiler.msys2.exe.release.option.debugging.level"
                          superClass="gnu.c.compiler.option.debugging.level">
                      </option>
                  </tool>
                  <tool
                      id="cdt.managedbuild.tool.gnu.c.linker.msys2.exe.release"
                      superClass="cdt.managedbuild.tool.gnu.c.linker.mingw.base">
                  </tool>
                  <tool
                      id="cdt.managedbuild.tool.gnu.cpp.linker.msys2.exe.release"
                      superClass="cdt.managedbuild.tool.gnu.cpp.linker.mingw.base">
                  </tool>
				  <tool
					  id="cdt.managedbuild.tool.gnu.assembler.msys2.exe.release"
					  superClass="cdt.managedbuild.tool.gnu.assembler.mingw.base">
				  </tool>   
               </toolChain>                                                     
         </configuration>
      </projectType> 
```

and that for all the project types you need.

## V- Templates

We have a working Toolchain, but now we need to assign it to some templates or our toolchain will not be abble to be selected. 
to do that, it's really simple, you just need to find the templates inside of the extension point: `org.eclipse.cdt.core.templateAssociations` and add your templates like that:

![MSYS2 package](https://github.com/PierreSachot/Internship-Reports/blob/master/images/week%209/Screenshot_3.png)

and this in every templates.
