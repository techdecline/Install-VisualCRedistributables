# Install-VisualCRedistributables.ps1
Deploying the Microsoft Visual C++ Redistributables in any complex desktop environment kinda sucks because there are so many versions that might be required. I got tired of updating my MDT deployment share with the redistributables manually, so wrote a script to automate the process.

This script will download the Visual C++ Redistributables listed in an external XML file into a folder structure that represents major release, processor architecture and update release (e.g. SP1, MFC, ATL etc.). The script defines the available redistributables and can be updated with each release with no changes made to the script.

*NOTE:* some validation of the Redistributables listed in the XML file is required, as not all may need to be installed in your environment.

This can be run to download and optionally install the Visual C++ (2005 - 2017) Redistributables as specified in the external XML file passed to the script.

You can find Install-VisualCRedistributables.ps1 and the XML files in the \bin folder.

The basic structure of the XML file should be as follows (an XSD schema is included in the repository):

    <Redistributables>
    <Platform Architecture="x64" Release="" Install="">
    <Redistributable>
    <Name></Name>
    <ShortName></ShortName>
    <URL></URL>
    <ProductCode></ProductCode>
    <Download></Download>
    </Platform>
    <Platform Architecture="x86" Release="" Install="">
    <Redistributable>
    <Name></Name>
    <ShortName></ShortName>
    <URL></URL>
    <ProductCode></ProductCode>
    <Download></Download>
    </Redistributable>
    </Platform>
    </Redistributables>

Each major version of the redistributables is grouped by <Platform> that defines the major release, processor architecture and install arguments passed to the installer.

The properties of each redistributable are defined in each <Redistributable> node:
- Name - the name of the redistributable as displayed on the download page. Not used in the script, but useful for reading the XML file.
- ShortName - the redistributable will be downloaded to Release\Architecture\ShortName
- URL - this is the URL to the page at microsoft.com/downloads. Not used in the script, but useful for referencing the download as needed
- ProductCode - this is the MSI Product Code for the specified VC++ App that will be used to import the package into Configuration Manager
- Download - this is the URL to the installer so that the script can download each redistributable

## Parameters
The script supports the -WhatIf and -Verbose parameters for testing and verbose output when using the parameter actions below.

There are 3 parameter sets that control the following actions:
1. Download only
2. Download and Install the redistributable to the current machine
3. Download and create ConfigMgr applications for the redistributables

### Download

#### Xml
The XML file that contains the details about the Visual C++ Redistributables. This must be in the expected format. If the redistributable exists in the target location, it will be skipped and not re-downloaded.

Example: download the Visual C++ Redistributables listed in VisualCRedistributables.xml to the current folder.

    .\Install-VisualCRedistributables.ps1 -Xml ".\VisualCRedistributables.xml"

#### Path
Specify a target folder to download the Redistributables to, otherwise use the current folder will be used.

Example: download the Visual C++ Redistributables listed in VisualCRedistributables.xml to C:\Redist.

    .\Install-VisualCRedistributables.ps1 -Xml ".\VisualCRedistributables.xml" -Path C:\Redist

### Install
To install the redistributables add the -Install parameter.

#### Install
By default the script will only download the Redistributables. This allows you to download the Redistributables for seperate deployment (e.g. in a reference image). Add -Install to install each of the Redistributables as well.

Example: download (to the current folder) and install the Visual C++ Redistributables listed in VisualCRedistributables.xml.

    .\Install-VisualCRedistributables.ps1 -Xml ".\VisualCRedistributables.xml" -Install

The Redistributables will installed in the order specified in the XML file.

#### Results
Here is an example of the end result with the Redistributables installed. Note that 2015 and 2017 are the same major version (14.x), so once 2017 is installed, 2015 will not be displayed in the programs list.

Visual C++ Redistributables 2005 to 2015 installed:

![Visual C++ Redistributables 2005-2015](https://raw.githubusercontent.com/aaronparker/Install-VisualCRedistributables/master/images/2005-2015.PNG "Visual C++ Redistributables 2005-2015")

Visual C++ Redistributables 2005 to 2017 (including 2015) installed:

![Visual C++ Redistributables 2005-2017](https://raw.githubusercontent.com/aaronparker/Install-VisualCRedistributables/master/images/2005-2017.PNG "Visual C++ Redistributables 2005-2017")

### Microsoft Deployment Toolkit
The script can create an application in an MDT share that can be used to install the Redistributables at deployment time. Useful when creating reference images.

#### CreateMDTApp
If specified, the script will create a single application for the Redistributables in an MDT deployment share. This makes a copy of the script and the XML file plus downloads the Redistributables into the target application folder.

#### MDTPath
The root path to the target MDT deployment share. This can be a local path or a UNC path.

#### Results
The MDT application should be a single application that installs all of the Redistributables or only those specified when running the script to create the application. The command line for the application is updated to reflect what was used to create the application.

![Visual C++ Redistributables in MDT](https://raw.githubusercontent.com/aaronparker/Install-VisualCRedistributables/master/images/VCRedist_MDT-App.PNG)

The application folder will include the script, the XML file and folders for each of the Redistributables specified.

![The application folder in the MDT deployment share](https://raw.githubusercontent.com/aaronparker/Install-VisualCRedistributables/master/images/VCRedist_MDT-Folder.PNG)

### Configuration Manager 
Support for downloading the Redistributables and creating applications in System Center Configuration Manager is also supported.

#### CreateCMApp
If specified enables automatic creation of Application Containers in Configuration Manager with a single Deployment Type containing the downloaded EXE file. The script must be run from a machine with the Configuration Manager console and the ConfigMgr PowerShell module installed.

#### SMSSiteCode
Specify SMS Site Code for the ConfigMgr app creation.

Example: Download Visual C++ Redistributables listed in VisualCRedistributables.xml and create ConfigMgr Applications for the selected Site.

    .\Install-VisualCRedistributables.ps1 -Xml ".\VisualCRedistributables.xml" -Path \\server1.contoso.com\Sources\Apps\VSRedist -CreateCMApp -SMSSiteCode S01

#### Results
This will look similar to the following in the Configuration Manager console:

![Visual C++ Redistributables in Configuration Manager](https://raw.githubusercontent.com/aaronparker/Install-VisualCRedistributables/master/images/VCredist_ConfigMgr.PNG)

# Known Issues

## -Release and -Architecture won't work with -CreateMDTApp
When the MDT application calls the script from powershell.exe using the -Release and -Architecture parameters at install time, the script fails parameter validation.

This does not happen if the script is called from from a running  PowerShell session. Until this issue is fixed, support for these parameters when creating an MDT application has been removed.

## Installation or Temporary File in the Root of C
The Visual C++ Redistributables 2005 and 2008 leave installer files or DLLs in the root of the system drive after installation.

This is documented in the following articles:

https://support.microsoft.com/en-us/help/927665/the-msdia80.dll-file-is-installed-in-the-root-folder-of-the-boot-drive-when-you-install-the-visual-c-2005-redistributable-package-by-using-the-vcredist-x64.exe-file-or-the-vcredist-ia64.exe-file

https://support.microsoft.com/en-us/help/950683/vcredist-from-vc-2008-installs-temporary-files-in-root-directory

This can be fixed by installing 2005 ATL update or later and 2008 SP1 or later. Redistributable listed in VisualCRedistributablesSupported.xml should not exhibit this behaviour.
