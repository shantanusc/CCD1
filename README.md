PTC Windchill: Code and Configuration Deployment (CCD)
Complete Step-by-Step Reference
Extracted from PTC University courses: CCD 1, CCD 2, and CCD 3.
Covers the full pipeline: Dev server setup → CCD package build → load files → database schema → BAC objects → Java customization → on-premises deployment → Windchill+ deployment.

Table of Contents
Course 1

CCD Utility Overview
Prepare Development Servers
Add Files to the Windchill Codebase
Business Administrative Change (BAC) Overview
Create and Import Load Files
Course 2 6. Add Load Files to a CCD Package

Configure Database Schema
Add BAC Objects to a CCD Package
Create a Custom Module in a CCD Package
Course 3 10. Set Windchill Properties in a CCD Package

Add Java Customization to a CCD Package
Deploy a CCD Package to Local Windchill
Deploy a CCD Package to Windchill+
COURSE 1
1. CCD Utility Overview
What the CCD Utility Does
The CCD (Code and Configuration Deployment) utility packages all customizations — Java code, web files, load files, BAC packages, database schema changes, and Windchill property settings — into a single deployable unit called a CCD package.

Deployment Pipeline
Development Server → Local Integration Server → QA Server → Production Server

Development: Where customizations are built and tested.
Local Integration: First test target; CCD package is applied here first.
QA: User acceptance testing.
Production: Final deployment.
CCD Package Folder Structure
After running ant create.package, the CCD Root directory is created with this structure:

D:\CCDRoot\
├── configurations\
│   ├── deploy.xml
│   ├── loadFiles\
│   ├── loadXMLFiles\
│   ├── conf\
│   ├── resources\
│   └── xconf\
│       └── custom.site.xconf
├── generated\
│   ├── BAC\
│   └── db\
│       └── conf\
│           └── SchemaConfig.xml
└── [custom module]\
    ├── descriptor.xml
    └── main\
        ├── src\
        ├── src_web\
        └── resources\

Key ANT Targets
Command	What It Does
ant create.package	Generates a new empty CCD package structure
ant all	Runs compile + deploy + load data + apply schema + restart (full pipeline)
ant compile	Compiles Java source code only
ant deploy	Copies files to Windchill codebase and copies BAC package + load files to staging; does NOT execute them
ant load.data	Executes LoadFileSet load files
ant apply.schema	Applies database schema changes
ant wnc.restart	Restarts the Windchill server
When to Use ant all vs Individual Targets
Situation	Use
First-time deploy to a clean target (no prior CCD data)	ant all — safe because no objects exist yet
Re-deploying to a target that already has the data	Individual targets only: ant deploy, ant apply.schema, ant wnc.restart
Deploying only codebase/property changes (no new data)	ant deploy
Deploying only schema changes	ant deploy + ant apply.schema
Deploying only Java/JSP/resource files	ant compile + ant deploy + ant wnc.restart
Caution: ant all re-imports the BAC package and re-executes all load files every time it runs. If objects from those files already exist in Windchill, you will get uniqueness/duplicate errors. Always use individual targets on any system that has been deployed to before.

2. Prepare Development Servers
Step 1 — Acquire a Windchill+ Development Server
Contact PTC to provision a Windchill+ development instance if you do not already have one.
All Windchill+ environments use the domain name ptcmscloud.com.
Step 2 — Set wt.customizationSource.dir.path in customizationTools.properties
Navigate to D:\ptc\Windchill_13.0\Windchill\bin\customizationTools.
Open customizationTools.properties in a text editor.
Set the property:
wt.customizationSource.dir.path=D:\\CCDRoot

Save the file.
Step 3 — Understand the CCD Directory Layout
D:\ptc\Windchill_13.0\Windchill\bin\customizationTools — home of ANT scripts and customizationTools.properties.
D:\CCDRoot — the CCD customization root directory.
D:\ptc\Windchill_13.0\Windchill — Windchill home directory.
3. Add Files to the Windchill Codebase
Which Folders Are Deployed to the Codebase
When ant deploy runs, the CCD utility copies files from the CCD root to the Windchill codebase. Files in these module folders are deployed:

CCD Source Location	Deploys To
[module]\main\resources\	Windchill\codebase\
[module]\main\src_web\	Windchill\codebase\
[module]\main\src\	Compiled to .class files → packaged into .jar → Windchill\custom\lib\
Exercise: Upload an Icon via the CCD Tool
Copy your icon file (e.g., NnIcon.png) to:
D:\CCDRoot\[customModule]\main\resources\netmarkets\[subdirectory]\

Open a Windchill Shell.
Navigate to D:\ptc\Windchill_13.0\Windchill\bin\customizationTools.
Run:
ant deploy

Wait for BUILD SUCCESSFUL.
Verify the icon file appears in Windchill\codebase\netmarkets\[subdirectory]\.
Exercise: Generate an Empty CCD Package
Open a Windchill Shell.
Set the working directory:
D:\ptc\Windchill_13.0\Windchill\bin\customizationTools

Run:
ant create.package

Wait for BUILD SUCCESSFUL.
Verify the D:\CCDRoot folder was created with the expected structure.
4. Business Administrative Change (BAC) Overview
What BAC Is
The Business Administrative Change (BAC) utility exports and imports Windchill configuration objects such as:

Type definitions and subtypes
Lifecycle templates
Workflow templates
Object initialization rules
Preferences
Global attributes and enumerations
BAC Limitations
BAC does not support:

Users, groups, or organizations
Libraries, products, or projects
Icons or codebase files
Database schema changes
For those, use load files or the CCD codebase deploy.

Import BAC Objects via the Windchill UI
Sign in to Windchill as an administrator (e.g., wcadmin).
Go to Site > Utilities > Business Administrative Change.
Set the From date and To date to bracket the exported objects.
Select the desired Object Types.
Click Search, then select the objects.
Click Create Package and name it (e.g., NnV1).
Download the resulting .zip BAC package.
Set ignoreGUIDList=All Before BAC Import
Open a Windchill Shell.
Run:
xconfmanager -t codebase/wt.properties -s com.ptc.windchill.bac.ignoreGUIDList=All -p

ignoreGUIDList=All — What It Does and Does Not Fix
Scenario	Without ignoreGUIDList=All	With ignoreGUIDList=All
Import a BAC package for the first time	✅ Works	✅ Works
Import the same BAC package a second time	❌ Error — duplicate GUID	✅ Skips duplicates silently (no error, no re-import)
Import a different BAC package that contains objects with the same GUID	❌ Error	✅ Skips those objects
Important distinction: ignoreGUIDList=All does not re-import duplicate objects — it silently skips them. It also does not fix duplicate errors caused by LoadFromFile/LoadFileSet data (those are separate and will still error if the same load file is run twice against the same data).

Import a BAC Package via the UI
Go to Site > Utilities > Business Administrative Change.
Click Import.
Browse to and select the BAC .zip file.
Click OK / Import.
Note: With ignoreGUIDList=All set, importing the same BAC package twice will not throw an error — the duplicate objects are silently skipped and the existing data is left unchanged. However, load file objects (users, parts, libraries) are independent of the GUID list and will still fail if you try to create the same named object twice.

5. Create and Import Load Files
Load File Tools Comparison
Tool	When to Use
LoadFromFile	Load a single XML file
LoadFileSet	Load multiple XML files in dependency order
BAC	Import configuration objects (types, lifecycles, workflows, OIRs, preferences)
Load File Types (by Windchill object)
Load files are XML files that create Windchill objects. Common types:

File	Creates
NnUsers.xml	User accounts
NnGroupMemberships.xml	License group memberships
NnCreators.xml	Library/product creators
NnLibraries.xml	Windchill libraries
NnDemoPart.xml	Demo parts
Step 1 — Create a User CSV File
Create a CSV with columns: username, fullName, email, etc.

Example NnUsers.csv:

username,fullName,email,organization
nnour,Noura Nour,nnour@example.com,Nifty Nutrition
nnox,Nael Nox,nnox@example.com,Nifty Nutrition

Step 2 — Convert CSV to XML Using CSV2XML
Open a Windchill Shell and run:

windchill wt.load.util.CSV2XML NnUsers.csv NnUsers.xml

This converts the CSV to a Windchill-compatible XML load file.

Step 3 — Load a Single File Using LoadFromFile
windchill wt.load.LoadFromFile -d NnUsers.xml -u <admin_user> -p <admin_password> -CONT_PATH "wctype:wt.org.WTOrganization|name=PTC"

Parameters:

-d — path to the XML file
-u — administrator username
-p — administrator password
-CONT_PATH — container path (context where objects are loaded)
Step 4 — Load Multiple Files in Order Using LoadFileSet
Create a loadFileSet.xml that lists load files in dependency order:

<LoadFileSet>
  <containerPath>wctype:wt.pdmlink.PDMLinkProduct|name=Nifty Nutrition</containerPath>
  <username>wcadmin</username>
  <loadFile>NnUsers.xml</loadFile>
  <loadFile>NnGroupMemberships.xml</loadFile>
  <loadFile>NnCreators.xml</loadFile>
  <loadFile>NnLibraries.xml</loadFile>
  <loadFile>NnDemoPart.xml</loadFile>
</LoadFileSet>

Then run:

windchill wt.load.LoadFileSet -d loadFileSet.xml -u <admin_user> -p <admin_password>

Order matters: Load users before groups, groups before creators, and creators before libraries/parts that reference them.

Step 5 — Troubleshoot Load Errors
Check the log file path printed after the build (e.g., D:\ptc\Windchill_13.0\Windchill\buildlogs\xxxx\Windchill_Customization.log).
Open the log in Notepad++.
Press Ctrl+F, search for loadFileSet to confirm load files executed.
Search for fail or ERROR to locate specific errors.
Open the MethodServer log (D:\ptc\Windchill_13.0\Windchill\logs\MethodServer-log4j.log) for deeper error details.
COURSE 2
6. Add Load Files to a CCD Package
How Load Files Work in a CCD Package
When you include load files in a CCD package, they are executed automatically when the CCD package is deployed with ant all or ant load.data.

Step 1 — Copy Load XML Files to the CCD Package
Copy your load XML files to:

D:\CCDRoot\configurations\loadFiles\custom\Nn\

(Create the Nn subdirectory as needed.)

Step 2 — Configure loadFileSet.xml
Open or create:

D:\CCDRoot\configurations\loadFiles\loadFileSet.xml

Structure:

<LoadFileSet>
  <containerPath>wctype:wt.pdmlink.PDMLinkProduct|name=Nifty Nutrition</containerPath>
  <username>wcadmin</username>
  <loadFile>custom/Nn/NnUsers.xml</loadFile>
  <loadFile>custom/Nn/NnGroupMemberships.xml</loadFile>
  <loadFile>custom/Nn/NnCreators.xml</loadFile>
  <loadFile>custom/Nn/NnLibraries.xml</loadFile>
  <loadFile>custom/Nn/NnDemoPart.xml</loadFile>
</LoadFileSet>

Paths inside <loadFile> are relative to the loadFiles\ folder.

Step 3 — Deploy the CCD Package with Load Files
Open a Windchill Shell.
Navigate to:
D:\ptc\Windchill_13.0\Windchill\bin\customizationTools

Run:
ant all -Dadministrator.username=wcadmin

Wait for BUILD SUCCESSFUL.
Note: If prompted, enter wcadmin as the Windchill administrator password.

Step 4 — Troubleshoot a CCD Load File Import Failure
Note the log file path shown before the BUILD FAILED message.
Open it in Notepad++.
Search (Ctrl+F) for loadFileSet — confirm around line 70–80 that it executed successfully.
Search for fail — look around lines 220–230 for any BAC import failures.
If a BAC import failed, open the most recent MethodServer log in:
D:\ptc\Windchill_13.0\Windchill\logs\

Sort by Date modified. Open the most recent MethodServer-log4j.log.
Select View > Word Wrap in Notepad++.
Search for ERROR and scroll to the last true error.
Common cause of BAC build failure: The same BAC package was loaded twice (e.g., it was already present from a previous exercise). The LoadFromFile-based objects will still load correctly; only the BAC portion fails.

7. Configure Database Schema
What Database Schema Changes CCD Supports
The CCD utility can apply these changes to the Windchill database:

Alter column sizes
Add custom indexes or sequences
Add attribute columns (soft attribute columns)
Caution: Very few database customizations are PTC-supported. Unsupported customizations may affect database integrity. On Windchill+, you are restricted to CCD-supported customizations only.

Where the Schema Config File Lives
D:\CCDRoot\generated\db\conf\SchemaConfig.xml

This file is generated when you run ant create.package. It contains no customizations by default.

Step 1 — Edit SchemaConfig.xml to Add Columns
Open D:\CCDRoot\generated\db\conf\SchemaConfig.xml in Notepad++.
Add your column definitions before the </SchemaConfig> closing tag:
<StandardAttributeColumns>
  <ClassName name="wt.part.WTPart">
    <Long count="2"/>
    <String count="1" Size="100"/>
  </ClassName>
</StandardAttributeColumns>

This adds 2 numeric columns and 1 string column (100 chars) to the WTPART table.

Save the file, overwriting the original.
Step 2 — Deploy Schema Changes
Open a Windchill Shell.
Navigate to D:\ptc\Windchill_13.0\Windchill\bin\customizationTools.
Run:
ant deploy

Wait for BUILD SUCCESSFUL.
Then run:
ant apply.schema

Wait for BUILD SUCCESSFUL.
Note: ant deploy copies the schema file to Windchill; ant apply.schema then sends it to the database. Do not use ant all here — it will also attempt to re-import BAC/load files.

Step 3 — Verify Schema Changes (Optional — Oracle SQL Developer)
Open Oracle SQL Developer.
Connect to the WIND database with username/password installds.
Navigate to Connections > TNS > WIND > Tables (Filtered) > WTPART.
Right-click COLUMN_NAME, select Sort, then sort alphabetically.
Columns named PTC_LNG_1TYPEINFWTPART, PTC_LNG_2TYPEINFWTPART, and PTC_STR_1TYPEINFWTPART should now be present.
Caution: Never write directly to the Windchill database using SQL Developer or any database tool. Only read/inspect using this tool.

8. Add BAC Objects to a CCD Package
Two Ways to Import a BAC Package
Method	When to Use
Windchill UI (BAC Utility)	Small/minor change; no release cycle needed
Embedded in CCD Package	Larger change requiring UAT and release; includes items BAC alone cannot handle (icons, codebase files, users, libraries)
Step 1 — Export Configuration Objects to a BAC Package (in Windchill UI)
Sign in to Windchill as wcadmin.
Go to Site > Utilities > Business Administrative Change.
Set From date to yesterday and To date to tomorrow.
In the Object Type field, select each of:
Life Cycle Template
Object Initialization Rule
Preference
Type Definition
Workflow Template
Click Search.
Select the relevant rows from the results table.
Click Create Package (N changes).
Name the package (e.g., NnV1) and optionally add a description.
Click Create.
Download the resulting .zip file; copy it to the root of the W:\ drive (or your working location).
Caution: Do not explicitly include global attributes or enumerations — they are automatically included with their parent type definition. Adding them again causes a uniqueness error on import.

Step 2 — Copy the BAC Package into the CCD Package
Copy the BAC .zip file (e.g., NnV1[timestamp]_PTC BAC Target_[seq].zip) to:
D:\CCDRoot\generated\BAC\

Delete any older BAC package files in that folder (e.g., the previous NnAttributes BAC). Including both would import the same objects twice and cause errors.
Step 3 — Deploy the CCD Package (with BAC embedded)
Open a Windchill Shell.
Navigate to D:\ptc\Windchill_13.0\Windchill\bin\customizationTools.
Run:
ant all -Dadministrator.username=wcadmin

Wait for BUILD SUCCESSFUL.
9. Create a Custom Module in a CCD Package
What a Custom Module Is
Every CCD package includes at least one custom module — a named subdirectory in the CCD root. Files in the custom module are deployed by the CCD utility to the Windchill codebase.

Step 1 — Rename the Default Custom Module Folder
In Windows File Explorer, navigate to D:\CCDRoot.
Rename the customModule folder to your desired module name, e.g., niftyNutritionPartModule.
Step 2 — Edit descriptor.xml
Open D:\CCDRoot\niftyNutritionPartModule\descriptor.xml in Notepad++.
Change the <name> tag:
<module>
  <name>niftyNutritionPartModule</name>
  <version/>
  <require/>
</module>

Save the file.
Note: The folder name and the <name> in descriptor.xml must match exactly. Any folder in the CCD root that is not named configurations or generated is treated as a custom module and must have a matching descriptor.xml.

Step 3 — Add Custom Action Configuration Files
These files (actions.xml, actionmodels.xml) define menu options visible in Windchill.

In Windows File Explorer, go to W:\WCDP-CCD-Lab-Files\ (lab files location).
Copy these files:
NnPartClient-actionmodels.xml
NnPartClient-actions.xml
Create the destination folder:
D:\CCDRoot\niftyNutritionPartModule\main\resources\niftyNutritionActions\

Paste both files into that folder.
Step 4 — Deploy to Windchill
Open a Windchill Shell.
Navigate to D:\ptc\Windchill_13.0\Windchill\bin\customizationTools.
Run:
ant deploy

Caution: Do not run ant all — it would re-import the BAC package and load files, causing uniqueness errors.

Wait for BUILD SUCCESSFUL.
Step 5 — Verify the Files Are in the Windchill Codebase
Navigate to:
D:\ptc\Windchill_13.0\Windchill\codebase\niftyNutritionActions\

Confirm NnPartClient-actionmodels.xml and NnPartClient-actions.xml are present.
Note: The action files appear in the Windchill UI but show as blank because no resource bundle (text/icons) has been added yet. That is added in Course 3.

COURSE 3
10. Set Windchill Properties in a CCD Package
How Windchill Properties Work
Windchill stores application settings in .properties files inside the Windchill codebase (e.g., wt.properties, wvs.properties, db.properties). These files are managed by the xconfmanager utility — never edit them directly.

Key flow:

CCD Root\configurations\xconf\custom.site.xconf
    → (ant deploy) →
Windchill Root\site.xconf
    → (xconfmanager -p) →
Windchill Root\codebase\wt.properties (and other .properties files)

Check a Current Property Value
xconfmanager -d <property.name>

Examples:

xconfmanager -d com.ptc.netmarkets.util.misc.customActions
xconfmanager -d com.ptc.netmarkets.util.misc.customActionModels
xconfmanager -d com.ptc.windchill.bac.ignoreGUIDList

Step 1 — Edit custom.site.xconf in the CCD Package
Open D:\CCDRoot\configurations\xconf\custom.site.xconf in Notepad++.
The file from the lab contains sample AddToProperty tags. You will add entries like:
<AddToProperty name="com.ptc.netmarkets.util.misc.customActions"
               value="niftyNutritionActions/NnPartClient-actions.xml"/>
<AddToProperty name="com.ptc.netmarkets.util.misc.customActionModels"
               value="niftyNutritionActions/NnPartClient-actionmodels.xml"/>
<AddToProperty name="com.ptc.windchill.bac.ignoreGUIDList"
               value="All"/>

Note: AddToProperty is used for multivalued properties (like customActions). For Windchill+ development instances, there is also a WindchillPlus property that must be set — check the lab file for the correct entry.

Save the file to D:\CCDRoot\configurations\xconf\, replacing the existing file.
Step 2 — Deploy the Properties to Windchill
Open a Windchill Shell.
Navigate to D:\ptc\Windchill_13.0\Windchill\bin\customizationTools.
Run:
ant deploy

Wait for BUILD SUCCESSFUL.
Step 3 — Verify Properties Were Set
xconfmanager -d com.ptc.netmarkets.util.misc.customActions
xconfmanager -d com.ptc.netmarkets.util.misc.customActionModels
xconfmanager -d com.ptc.windchill.bac.ignoreGUIDList

Both customActions and customActionModels should now show the values you added.

Step 4 — (Optional) View Custom Actions in the Windchill UI
In a Windchill Shell, run:
ant wnc.restart

Close all Chrome windows. Wait for Windchill to restart (a few minutes).
Sign in to Windchill as nnour / ptc.
Go to the Folders page of the Nifty Nutrition Library.
The custom action appears in the toolbar (blank tooltip, no icon yet — the resource bundle is added next).
11. Add Java Customization to a CCD Package
How Java Customizations Work in CCD
Add Java source files (.java) to the custom module under main\src\.
Add JSP files to main\src_web\ (for web pages).
The CCD utility compiles the source files and packages them into a .jar under Windchill\custom\lib\.
D:\CCDRoot\niftyNutritionPartModule\main\src\com\nn\custompart\CustomPartResource.java
    → (ant compile + ant deploy) →
D:\ptc\Windchill_13.0\Windchill\custom\lib\niftyNutritionPartModule.jar
                                           \com\nn\custompart\CustomPartResource.class

Important for Windchill+: Java customizations must be added as source code. Windchill+ will not accept pre-compiled .jar or .class files. On-premises Windchill will accept pre-compiled files, but source code is strongly recommended.

Step 1 — Examine the Actions XML to Identify Needed Java Files
Open D:\CCDRoot\niftyNutritionPartModule\main\resources\niftyNutritionActions\NnPartClient-actions.xml.
Find the custom action (e.g., downloadNnParts). Note the two dependencies it requires:
A resource bundle class: com.nn.custompart.CustomPartResource
A JSP page URL: NnActions/downloadNnParts.jsp
Step 2 — Add the Java Resource Bundle to the CCD Package
Open the resource bundle source file: W:\WCDP-CCD-Lab-Files\CustomPartResource.java in Notepad++.
Note the package declaration: package com.nn.custompart;
In Notepad++, choose File > Save As.
Create the folder structure matching the package:
D:\CCDRoot\niftyNutritionPartModule\main\src\com\nn\custompart\

Save CustomPartResource.java in the new custompart folder.
Step 3 — Add the JSP File to the CCD Package
Copy W:\WCDP-CCD-Lab-Files\downloadNnParts.jsp to:
D:\CCDRoot\niftyNutritionPartModule\main\src_web\NnActions\

Create the NnActions subfolder as needed.
Step 4 — Compile and Deploy the CCD Package
Open a Windchill Shell.

Navigate to D:\ptc\Windchill_13.0\Windchill\bin\customizationTools.

Run:

ant compile

Wait for BUILD SUCCESSFUL.
(The compile target compiles source to a temp directory in the CCD package.)

Run:

ant deploy

Wait for BUILD SUCCESSFUL.
(The deploy target moves the compiled files to Windchill and deletes the temp directory.)

Run:

ant wnc.restart

You do not need to wait for this to finish before proceeding.

Step 5 — Verify the Compiled JAR Is in Windchill
In File Explorer, navigate to:
D:\ptc\Windchill_13.0\Windchill\custom\lib\

Confirm niftyNutritionPartModule.jar exists.
To inspect it, copy niftyNutritionPartModule.jar to W:\, rename the copy to niftyNutritionPartModule.zip, and open it.
Verify that com\nn\custompart\CustomPartResource.class exists inside.
Step 6 — Verify the Custom Action in Windchill UI
Wait for Windchill to restart.
Sign in as nnour / ptc.
Go to the Folders page of the Nifty Nutrition Library.
Hover over the Download Nifty Nutrition Parts icon in the Folder Contents toolbar.
Confirm the tooltip reads "Download Nifty Nutrition Parts" (from the resource bundle).
Click Actions > Download Nifty Nutrition Parts.
Verify AllNiftyNutritionParts.csv downloads.
12. Deploy a CCD Package to Local Windchill
Overview
Before deploying to Windchill+, always validate the CCD package on an on-premises (local) Windchill system. On-premises testing is fast and provides full access to server logs. Windchill+ deployments are time-consuming and harder to troubleshoot.

Step 1 — Archive the CCD Package
In Windows File Explorer, right-click the D:\CCDRoot folder.
Select Send to > Compressed (zipped) folder.
This creates CCDRoot.zip.
Optionally, save CCDRoot.zip to cloud storage (Google Drive, OneDrive) for safekeeping.
Important for Windchill+ uploads: The root of the .zip file must contain configurations and generated folders directly — not a CCDRoot subfolder. Correct:

CCDRoot.zip/
  configurations/
  generated/
  niftyNutritionPartModule/

Incorrect: CCDRoot.zip/CCDRoot/configurations/...

Step 2 — Transfer the CCD Package to the Target System
On the target system (the reset/clean VM representing your integration/QA server), download CCDRoot.zip from cloud storage.
Extract CCDRoot.zip to the root of the D:\ drive.
Verify the following folders exist:
D:\CCDRoot\configurations\
D:\CCDRoot\generated\
D:\CCDRoot\niftyNutritionPartModule\
Step 3 — Set the Customization Root on the Target
Open Windows File Explorer, type D: in the navigation bar.
Browse to D:\ptc\Windchill_13.0\Windchill\bin\customizationTools.
Right-click customizationTools.properties, select Edit with Notepad++.
Change:
wt.customizationSource.dir.path=${dir.sep}opt${dir.sep}ptc${dir.sep}customization

to:
wt.customizationSource.dir.path=D:\\CCDRoot

Save the file.
Step 4 — Deploy to the Target System
Open a Windchill Shell on the target system.
Navigate to D:\ptc\Windchill_13.0\Windchill\bin\customizationTools.
Run:
ant all -Dadministrator.username=wcadmin

Wait for BUILD SUCCESSFUL. (This may take 10–15 minutes, including a Windchill server restart.)
Tip: If an error occurs, check both:

The last log in [Windchill Home]\buildlogs\
The MethodServer log in [Windchill Home]\logs\
Step 5 — Test the Target Environment
Validate these four areas after deployment:

Test 1 — User Import

Sign in to Windchill as nnour / ptc (login should succeed).
Go to Nifty Nutrition > Utilities > Participant Administration.
Click Add participants to the table; search for nnox (Nael Nox). Verify the user exists.
Test 2 — Library and Part Import

Click Browse > Recent Libraries > View All.
Confirm Nifty Nutrition Library exists.
Click the library; verify the test part exists.
Test 3 — Life Cycle and Workflow

Click New Part; select Type: Nifty Nutrition Part, Name: nnTest, Main Ingredient: Yeast.
Click Attach new local file, select any file, click Finish.
Verify the part is in Purging state (the first lifecycle state).
Click the Home icon, then click Approve on the workflow task.
Click Complete Task.
Refresh. Verify the part is now in Final state.
Test 4 — Download CSV Customization

Select the nnTest part checkbox.
Select Actions > Save As, name it nnTest2, click OK.
Click the Download Nifty Nutrition Parts icon.
Open AllNiftyNutritionParts.csv in OpenOffice Calc.
Under Separator Options, select Comma, click OK.
Verify a spreadsheet with your two Nifty Nutrition parts displays.
13. Deploy a CCD Package to Windchill+
This section is for Windchill+ (SaaS/PTC Cloud) customers only. If you use on-premises Windchill exclusively, you may skip this section.

Windchill+ Deployment Pipelines
Windchill+ uses two pipelines, both managed by PTC Cloud Services:

Pipeline	Purpose	Persistence
Integration Pipeline	Functional testing of CCD package in Windchill+	Non-persistent; reverts automatically after retention period (typically 7 days)
Production Pipeline	UAT + production deployment	Two stages: QA/integration → then production; production changes are permanent
What You Need Before Uploading
PTC Azure Storage BLOB access — an SAS connection string from a PTC customer support case.
Static external IP address — your upload machine must have a whitelisted static IP (PTC whitelists it when you open the support case).
Tested CCD package — a zipped CCD package that has been successfully validated on an on-premises system.
Manifest file — a YAML file specifying pipeline name, package name, deployment times, approvers, etc.
Procedure 1 — Acquire Azure Storage BLOB Access (Reference Only)
Open a PTC Support Case:

Sign in to PTC eSupport at https://www.ptc.com/support using your PTC customer credentials.
Open a new support case.
In the "Do you want to open a case with PTC Cloud Services?" field, select Yes – Request an activity (refresh, build, access, upgrade, etc.).
In the Description field, include:
A request for an Azure SAS token to upload a CCD package to your Windchill+ instance.
Your Windchill+ URL (e.g., https://ptc-peeu.ptcmscloud.com/Windchill).
The procedure you use to access Windchill+ (URLs and sign-in selections), including the administrator username — do not include passwords.
The static external IP address of the machine you will use to upload the CCD package.
Click Next, enter your Service Contract Number (SCN).
Select Access Request as the Service Request Type.
Select your Windchill+ product release and datecode.
Set an appropriate impact level, then click Open a Case.
PTC Customer Support will respond with an SAS connection string. Record it.
Procedure 2 — Create and Validate the Manifest File
Integration pipeline manifest (manifest_CCDIntegration.yml):

#Describes the deployment of build pipeline
deploy_pipe : int1
package_description : Package for integration deployment
package_name : CCDRoot.zip
package_type : CCD
duration : 2
date_time_int : 2025/04/22 14:00:00
notify : user@yourcompany.com
approvers : approver@yourcompany.com
retention_period : 7

Production pipeline manifest (manifest_CCDProduction.yml):

#Describes the deployment of build pipeline
deploy_pipe : pipeline1
package_description : Package for production deploy
package_name : CCDRoot.zip
package_type : CCD
duration : 2
date_time_qa : 2025/04/22 14:00:00
date_time_prod : 2025/05/22 14:00:00
notify : approver@yourcompany.com
approvers : approver@yourcompany.com
retention_period : 30

Manifest field reference:

Field	Description
deploy_pipe	Pipeline name provided by PTC customer support
package_name	Exact filename of the zipped CCD package (case-sensitive)
package_type	Always CCD
duration	Timeout in hours; set to at least 2× your on-premises deployment time
date_time_int	Scheduled deployment time to integration (YYYY/MM/DD HH:MM:SS)
date_time_qa	Scheduled deployment time to QA (production pipeline)
date_time_prod	Scheduled deployment time to production
notify	Comma-separated email addresses that receive pipeline notifications
approvers	Comma-separated email addresses that can approve/revert deployments
retention_period	Days before integration instance auto-reverts (7 or 30)
Manifest filename rules: Must start with manifest_ and end with .yml.
Date tip: Set date_time to a time in the past to deploy as soon as possible.

Procedure 3 — Upload the CCD Package to the Azure Pipeline
Start Microsoft Azure Storage Explorer (free download from Microsoft).
Click the Open Connect Dialog icon.
Select Blob container or directory.
Select Shared access signature URL (SAS).
Click Next.
Type a display name (e.g., MyWindchillPlusPipeline).
Paste the SAS connection string from PTC customer support.
Click Next, then Connect.
Validate the zip structure before upload:

The root of the .zip must contain configurations and generated directly, not a CCDRoot subfolder.
Upload the CCD package:

Navigate to data > builds in the Azure Storage Explorer.
Open Windows File Explorer to the folder containing CCDRoot.zip.
Drag CCDRoot.zip into Azure Storage Explorer (the builds folder).
Upload the manifest file:

Navigate to data > builds > deploy in Azure Storage Explorer.
Drag the manifest file (e.g., manifest_CCDIntegration.yml) into the deploy folder.
PTC Cloud Services automatically detects the manifest upload and begins processing.
What Happens After Upload — Integration Pipeline
First email (within ~1 hour): Deployment request received; shows scheduled servers and dates.
CloudWAVE validation email (~2 hours after first): PTC validates the CCD package against Windchill+ guardrails.
If it passes: deployment proceeds.
If it fails: deployment is cancelled; you must fix the package and re-upload.
Deployment proceeds to the integration Windchill+ environment.
Test and approve: Approvers receive an email with options to approve moving to production or revert.
After retention period: The integration instance automatically reverts to its previous state.
What Happens After Upload — Production Pipeline
First deploys to a QA/integration instance.
After the QA instance is approved (approver sends the approve email), and date_time_prod is reached, the package deploys to production.
Production is permanent — it cannot be reverted automatically. PTC maintains backups; a support call is required to restore from backup.
Pipeline Logs
Deployment logs are in Azure Storage at:

data/builds/logs/RITM[number]/

Download and inspect these if:

The CloudWAVE scan fails (the log shows which guideline was violated).
The deployment passes CloudWAVE but fails during apply (check MethodServer logs in the archive).
Two Non-Automated Windchill+ Production Operations
These require a PTC customer support service call:

Operation	What It Does
Rehost production to integration/QA	Copies production instance (data + customizations) to integration/QA; useful to keep dev teams aligned with the latest production state
Restore production from backup	Reverts production to a PTC-maintained backup; all data added since backup is lost
Quick Command Reference
ANT Targets (run from D:\ptc\Windchill_13.0\Windchill\bin\customizationTools)
# Full deploy cycle (compile + deploy + load data + apply schema + restart)
ant all -Dadministrator.username=wcadmin
# Generate a new empty CCD package
ant create.package
# Compile Java source only
ant compile
# Deploy files to Windchill codebase (no restart, no data load)
ant deploy
# Apply database schema changes
ant apply.schema
# Run load files (LoadFileSet)
ant load.data
# Restart Windchill
ant wnc.restart

xconfmanager Commands (run in Windchill Shell)
# Read a property value
xconfmanager -d <property.name>
# Set a property and propagate to .properties files
xconfmanager -s <property.name>=<value> -p
# Set, save to site.xconf, and propagate
xconfmanager -s <property.name>=<value> -t <target.file> -p

Load File Commands
# Convert CSV to XML
windchill wt.load.util.CSV2XML input.csv output.xml
# Load a single XML file
windchill wt.load.LoadFromFile -d file.xml -u wcadmin -p wcadmin -CONT_PATH "wctype:wt.org.WTOrganization|name=PTC"
# Load a set of files in order
windchill wt.load.LoadFileSet -d loadFileSet.xml -u <admin_user> -p <admin_password>

Key File & Folder Locations
Item	Path
CCD Root	D:\CCDRoot\
Customization Tools	D:\ptc\Windchill_13.0\Windchill\bin\customizationTools\
Properties config	D:\ptc\Windchill_13.0\Windchill\bin\customizationTools\customizationTools.properties
Windchill Home	D:\ptc\Windchill_13.0\Windchill\
Windchill Codebase	D:\ptc\Windchill_13.0\Windchill\codebase\
Custom JARs	D:\ptc\Windchill_13.0\Windchill\custom\lib\
Windchill Logs	D:\ptc\Windchill_13.0\Windchill\logs\
Build Logs	D:\ptc\Windchill_13.0\Windchill\buildlogs\
Load Files in CCD	D:\CCDRoot\configurations\loadFiles\
Schema Config	D:\CCDRoot\generated\db\conf\SchemaConfig.xml
BAC Packages in CCD	D:\CCDRoot\generated\BAC\
Custom Properties	D:\CCDRoot\configurations\xconf\custom.site.xconf
Lab Files (VM)	W:\WCDP-CCD-Lab-File
