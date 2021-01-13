# CMAppCreator

CMAppCreator's purpose is to add applications to a MEMCM/ConfigMgr environment in a standardized way, with minimal work effort (minimize on clicking dialogs present in the full adminconsole) and through a web app GUI. (yes I am fully aware of GUI's people have made in powershell and other tools which this does not replace, only add to the eco-system and giving something back to the ConfigMgr community :) )
The idea is to create a first test release of an application, meaning distribute, set certain common settings and deploy to a test user collection for immediate testing if wanted.
Scenarios for usage is a group responsible for managing MEMCM/ConfigMgr obviously, but also to let departments that are not comfortable using the full console experience i.e. servicedesk/helpdesk. You can even use a smartphone for doing this task due to the app being mobile friendly :)
The app impersonates the calling user (except for UNC file access) so make sure RBAC is configured correctly in MEMCM/ConfigMgr.

Note: I the author have used the app in production and tested it in a few different environments but I recommend you test it in your test environment first. I have also tried to do some error handling and squash some bugs but it can probably improve..

### Features overview

- Powershell App Deployment Tool detection (sets install, uninstall and repair commands automatically when configured source folder contains the PSADT toolkit)
- MSI integration (extracts Productkey automatically and sets detection), full list of MSI properties also extracted and can be viewed
- Extracts icons from EXE files if selected (for usage in software center to end users) and automatically resizes the icon to fit
- Imports .ICO or .PNG files automatically if encountered when selecting source folders from configured UNC path for usage in software center to end users
- Optional: Distributes application to configured DP Group, deploys to selected user collection, possible to set as interactive for deployment type, and enable repair and admin approval for deployment
- Detection methods (MSI and powershell supported)
- Requirements rules (disk requirement rule is automatically calculated on selected source folder and doubles it), also configures estimated execution time based on this. Primary user rule added as well
- Possible to set administrative categories and also set one or more user categories directly
- Sets custom security scope for the application if configured to do so
- Rudimentary logging included

## Supported Configurations
This app has been built to support the following versions of Microsoft Endpoint Configuration Manager:

- Microsoft Endpoint Configuration Manager (version 1910 and up has been tested only)

Make sure that .NET Framework 4.7.2 or higher is available on the member server you intend to host this web app on.
Note: The app can be deployed to another server other then the MEMCM infrastructure, since all MEMCM calls are impersonated by the calling user (with the exception of UNC file access which is handled by a required service account)

For the frontend the app is built using asp.net webforms, utilizing Bootstrap 4 for styling, jQuery for javascript and font awesome for a few icons. These are all configured against their respective CDN, so the server hosting the app requires internet connectivity.
For the backend, WiX toolset SDK DLLs is used for MSI integration, and a couple of the MEMCM / ConfigMgr adminconsole DLL's utilizing the ConfigMgr SDK functionalities (DLL's not provided here you have to add them yourself from your environment)

## Installation instructions

To successfully run this web app, you'll need to have IIS installed on a member server with ASP.NET enabled, .NET Framework 4.7.2 or higher and internet connectivity for downloading of Bootstrap4/jQuery/fontawesome libraries from their CDNs. Easiest way to get going is to install CMAppCreator on the same server where your Management Point role is hosted. You'll also need to have a service account for the application pool in IIS. The service account requires no rights in MEMCM/ConfigMgr, only read-access to the UNC file share for source files.

### 1 - Create folder structure
1. Download the project and compile and publish the solution in Visual Studio (you can download the free version called Visual Studio Community Edition) Note! if needed download the required DLLs at point 3 to a local path and point the referenced files to that location
2. Create a folder in <b>C:\inetpub</b> called <b>CMAppCreator</b>. Inside that folder, copy the files that you published from Visual Studio.
3. Locate below files from your ConfigMgr admin-console installation location and copy them to <b>C:\inetpub\CMAppCreator\bin</b>.
  - <b>AdminUI.AppManFoundation.dll</b>
  - <b>AdminUI.DcmObjectWrapper.dll</b>
  - <b>AdminUI.FeaturesUtilities.dll</b>
  - <b>AdminUI.WqlQueryEngine.dll</b>
  - <b>Microsoft.ConfigurationManagement.ApplicationManagement.dll</b>
  - <b>Microsoft.ConfigurationManagement.ApplicationManagement.MsiInstaller.dll</b>
  - <b>Microsoft.ConfigurationManagement.ManagementProvider.dll</b>
  - <b>Microsoft.ConfigurationManager.CommonBase.dll</b>

### 2 - Add an Application Pool in IIS
1. Open IIS management console, right click on <b>Application Pools</b> and select Add Application Pool.
2. Enter <b>CMAppCreator</b> as name, select the .NET CLR version <b>.NET CLR Version v4.0.30319</b> and click OK.
3. Select the new <b>CMAppCreator</b> application pool and select <b>Advanced Settings</b>.
4. In the <b>Process Model</b> section, specify the service account that will have access to the UNC file share in the <b>Identity</b> field and click OK.

### 3 - Add an Application to Default Web Site
1. Open IIS management console, expand <b>Sites</b>, right click on <b>Default Web Site</b> and select <b>Add Application</b>.
2. As for <b>Alias</b>, enter <b>CMAppCreator</b>.
3. Select <b>CMAppCreator</b> as application pool.
4. Set the physical path to <b>C:\inetpub\CMAppCreator</b> and click OK.

### 4 - Disable anonymous authentication and enable windows authentication
1. Select <b>CMAppCreator</b> in IIS management console
2. Under IIS, select <b>Authentication</b>
3. Disable Anonymous Authentication and enable Windows Authentication

### 5 - Restrict access to specific AD group (optional)
1. Select <b>CMAppCreator</b> in IIS management console
2. Under ASP.NET, select <b>.NET Authorization Rules</b>
3. Add Allow Rule
4. Select <b>Specified roles or user groups</b> and enter name of AD group

### 6 - Set Application Settings
1. Edit <b>web.config</b> and locate CM_App_Creator.Properties.Settings.
2. Enter values for each application settings: 
 - <b>SiteServer</b> The server where the SMS Provider is installed
 - <b>SiteCode</b> The site code of your site
 - <b>DPGroupName</b> DP Group you want to send content to
 - <b>UNCPath</b> FQDN path to your UNC file share containing source files
 - <b>CMAppFolder</b> Folder in ConfigMgr where to move created applications (if omitted apps are created in root folder)
 - <b>SecurityScope</b> A custom security scope ID (if omitted default scope is used)
 - <b>FolderNameDetection</b> Folder pattern (trigger) when "Set Content Folder" button should be visible, i.e. if you have folders named inst_r1, inst_r2 containing the source files and so on you set the name inst_ (if omitted "Set Content Folder" button is always visible except for the root folder in UNC path
 - <b>CollectionPrefix</b> A prefix for user collections i.e. setting it to "Test" will only display collections starting with that name (if omitted all user collections are displayed)
 - <b>DefaultLanguage</b> Language set for deployment type and language set in the Software Center tab, i.e en-US
 - <b>DisplayContactAndExecutionTime</b> Show or hide the section containing app owner, app contact and estimated execution time, valid values are True or False
 - <b>AddBranding</b> Enables branding for deployment type
 - <b>BrandingText</b> The actual text that is to be set in comments for deployment type.
