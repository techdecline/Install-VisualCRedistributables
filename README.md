# Install-VisualCRedistributables.ps1
Deploying the Microsoft Visual C++ Redistributables in any complex desktop environment kinda sucks because there are so many versions that might be required. I got tired of updating my MDT deployment share with the redistributables manually, so wrote a script to automate the process.

This script will download the Visual C++ Redistributables listed in an external XML file into a folder structure that represents major release, processor architecture and update release (e.g. SP1, MFC, ATL etc.). The script defines the available redistributables and can be updated with each release with no changes made to the script.

*NOTE:* some validation of the Redistributables listed in the XML file is required, as not all may need to be installed in your environment.

This can be run to download and optionally install the Visual C++ (2005 - 2017) Redistributables as specified in the external XML file passed to the script.

The basic structure of the XML file should be as follows (an XSD schema is included in the repository):

    <Redistributables>
    <Platform Architecture="x64" Release="" Install="">
    <Redistributable>
    <Name></Name>
    <ShortName></ShortName>
    <URL></URL>
    <Download></Download>
    </Platform>
    <Platform Architecture="x86" Release="" Install="">
    <Redistributable>
    <Name></Name>
    <ShortName></ShortName>
    <URL></URL>
    <Download></Download>
    </Redistributable>
    </Platform>
    </Redistributables>

Each major version of the redistributables is grouped by <Platform> that defines the major release, processor architecture and install arguments passed to the installer.

The properties of each redistributable are defined in each <Redistributable> node:
- Name - the name of the redistributable as displayed on the download page. Not used in the script, but useful for reading the XML file.
- ShortName - the redistributable will be downloaded to Release\Architecture\ShortName
- URL - this is the URL to the page at microsoft.com/downloads. Not used in the script, but useful for referencing the download as needed
- Download - this is the URL to the installer so that the script can download each redistributable

In the future, I may add the MSI product code to each redistributable to enable the script to check whether a redistributable is already installed instead of installing it. This may also be useful for defining the redistributables as applications in Configuration Manager (but you should be adding the redistributables to your reference images anyway.

## Parameters
### Xml
The XML file that contains the details about the Visual C++ Redistributables. This must be in the expected format.

Example: download the Visual C++ Redistributables listed in VisualCRedistributables.xml to the current folder.

    .\Install-VisualCRedistributables.ps1 -Xml ".\VisualCRedistributables.xml"

### Path
Specify a target folder to download the Redistributables to, otherwise use the current folder.

Example: download the Visual C++ Redistributables listed in VisualCRedistributables.xml to C:\Redist.

    .\Install-VisualCRedistributables.ps1 -Xml ".\VisualCRedistributables.xml" -Path C:\Redist


### Install
By default the script will only download the Redistributables. This allows you to download the Redistributables for seperate deployment (e.g. in a reference image). Add -Install to install each of the Redistributables as well.

Example: download (to the current folder) and install the Visual C++ Redistributables listed in VisualCRedistributables.xml.

    .\Install-VisualCRedistributables.ps1 -Xml ".\VisualCRedistributables.xml" -Install:$True

The Redistributables will installed in the order specified in the XML file.

## Results
Here is an example of the end result with the Redistributables installed. Note that 2015 and 2017 are the same major version (14.x), so once 2017 is installed, 2015 will not be displayed in the programs list.

Visual C++ Redistributables 2005 to 2015 installed:

![Visual C++ Redistributables 2005-2015](https://raw.githubusercontent.com/aaronparker/Install-VisualCRedistributables/master/images/2005-2015.PNG "Visual C++ Redistributables 2005-2015")

Visual C++ Redistributables 2005 to 2017 (including 2015) installed:

![Visual C++ Redistributables 2005-2017](https://raw.githubusercontent.com/aaronparker/Install-VisualCRedistributables/master/images/2005-2017.PNG "Visual C++ Redistributables 2005-2017")
