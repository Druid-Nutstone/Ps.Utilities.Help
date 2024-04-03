# Ps.Utilities Help 

# Overview 
PS.Utilies is a c# .NET standard 2.0 set of powershell cmdlets that wraps the following utilities :- 

<ol>
  <li><strong>Git</strong> - wraps many usefull git commands</li>
  <li><strong>Devops</strong> - Uses the azure api to manipulate azure devops projects/repositories and pipelines
  <li><strong>Excel</strong> - provides wrappers to read and write Excel files using the excellent <strong>Spread sheet Light</strong> library. Usefull for allowing users to populate input data using a toolthey are used tool. 
  <li><strong>DotNet</strong> - access to dotnet cli  
  <li><strong>Visual Studio</strong> - call visual studio command line (automatically finds the latest version of VS that is installed) 
  <li><strong>Sql</strong> - added functionality for SQLServer PS Library 
</ol>

see: [Spread sheet light](https://spreadsheetlight.com/)

# Installation
```
Install-Module PS.Utilities
```

## Cmdlet Summary
   | Cmdlet | Description | System |  
   | --- | --- | --- |
   | [Set-DevopsCredentials](#set-devopscredentials) | Sets session-wide credentials | Git/Devops
   | [Find-DevopsRepository](#find-devopsrepository) | locates the remote devops repository | Devops
   | [Get-DevopsProjects](#get-devopsprojects) | Returns a collection of devops projects | Devops
   | [Get-DevopsProject](#get-devopsproject) | Returns a single project | Devops
   | [Get-DevopsRepositories](#get-devopsrepositories) | Returns a collection of repositories from the given project | Devops
   | [Get-DevopsRepository](#get-devopsrepository) | Returns a Repository model | Devops
   | [Get-DevopsPipelines](#get-devopspipelines) | Returns a collection of pipelines | Devops |
   | [Invoke-DevopsPipeline](#invoke-devopspipeline) | invokes the given pipeline | Devops |
   | [Wait-DevopsPipeline](#wait-devopspipeline) | Waits for a devops pipeline to complete | Devops |
   | [New-DevopsPullRequest]() | TO DO | 
   | [Complete-DevopsPullRequest]() | TO DO |
   | [Test-Git](#test-git) | Test to see if git is installed | Git
   | [Install-Git](#install-git) | Installs Git | Git
   | [Copy-Repository](#install-git) | Clones a remote repository | Git
   | [Find-Branch](#find-branch) | Checks local branch | Git
   | [Get-Branches](#get-branches) | Gets a collection of local branches | Git
   | [Switch-Branch](#switch-branch) | Checks out local branch | Git
   | [Remove-Branch](#remove-branch) | Removes local branch | Git
   | [New-Branch](#new-branch) | Creates new local branch | Git
   | [Save-Repository](#save-repository) | Saves (commmits) local changes | Git
   | [Push-Repository](#push-repository) | pushes commited changes from local to remote repository | Git
   | [Sync-Repository](#sync-repository) | Pulls latest changes to local repo | Git
   | [Get-Tags](#get-tags) | Gets a List<string> of tags in the local repository | Git
   | [New-Tag](#new-tag) | Creates a new tag at the current commit of a local repository | Git
   | [New-Excel](#new-excel) | Creates a new Excel Application Instance | Excel | 
   | [Open-Excel](#open-excel) | Opens an existing spread sheet | Excel |
   | [Set-ExcelData](#set-exceldata) | Populates the given worksheet with an array of PS class data | Excel |
   | [Set-ExcelRowProperty](#set-excelrowproperty) | Sets the properties of the specified row and wporksheet | Excel
   | [Save-Excel](#save-excel) | Saves the given excelapplication to an excel file | Excel |
   | [Get-ExcelData](#get-exceldata) | Gets data from the given excelapplication and returns a .net List of type | Excel  
   | [Select-ExcelWorkSheet](#select-excelworksheet) | Makes the given worksheet the default worksheet for other operations | Excel | 
   | [Invoke-DotNetProcess](#invoke-dotnetprocess) | Runs the given local csproj project | DotNet
   | [Start-DotNetCommand](#start-dotnetcommand) | Runs any dotnet cli command and returns the results | DotNet 
   | [Invoke-VisualStudioBuild](#invoke-visualstudiobuild) | Builds a project or solution using visual studio (devenv) | Visual Studio |
   | [Start-VisualStudioCommand](#start-visualstudiocommand) | runs the specified visual studio command line against the given project or solution | Visual Studio | 
   | [Resolve-SqlResult](#resolve-sqlresult) | returns a list<> or single object mapped to a typed class from the result of Invoke-Sqlcmd call | Sql


## Common Cmdlets
# Set-DevopsCredentials
Sets session-wide credentials for both git and devops. You should call this as the first cmdlet in your script. The credentials will be stored for the entire PS session 

**Example** 
```
$patToken = "some-plain-text-Path-Token"
$userName = "aperson@org.com"
Set-DevopsCredentials -Username $userName -PlaintextPassword $patToken -Organisation "your-devops-organisation" 

## to use existing credentials of a previously pulled repository you can use
Set-DevopsCredentials -Organisation -FromRepository "C:\\somelocalRepository"

# .. do your git/devops stuff here

```

<details>
   <summary>Parameters</summary>

   | Parameter | Description |  
   | --- | --- |
   | -Username  | (optional) Username to use to authenticate with git / azure | 
   | -PlainTextPassword | (optional if using -FromRepository) the PAT token to use to authenticate with remptes devops | 
   | -Organisation | the devops organisation name (i.e https://devop://dev.azure.com/<em>**organisation**<em>/blah) |  
   | -FromRepository | An existing repository to use (fropm the remote.url git config definition)

</details>

&nbsp;

**Returns**
Nothing

---

## Devops Cmdlets
&nbsp;  
# Find-DevopsRepository
locates the remote devops repository specified by the -Name parameter. It will search all projects within the organisation defined in Set-DevopsCredentials

**Example** 
```
$repo = Find-DevopsRepository -Name $repoName
# use $repo.RemoteUrl to target remote url 
```
#### RepositoryModel
```
    public class RepositoryModel
    {
        public Guid Id { get; set; }

        public string Name { get; set; }

        public string Url { get; set; }

        public string DefaultBranch { get; set; }

        public long Size { get; set; }

        public string RemoteUrl { get; set; }

        public bool IsDisabled { get; set; }

        public Guid ProjectID { get; set; } 
    } 
```

<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Name | The nameof the remote repository   |  

</details>

&nbsp;

**Returns**
[RepositoryModel] The repository model found. Throws if not found  

---

&nbsp;  
# Get-DevopsProjects
Returns a collection of devops projects from Set-DevopsCredentials 
**Example** 
```
$projects = Get-DevopsProjects
```

#### ProjectModel
```
    public class ProjectModelCollection : List<ProjectModel>
    {
      public ProjectModel GetProjectByName(string name)
      {
          return this.FirstOrDefault(x => x.Name.Equals(name, StringComparison.OrdinalIgnoreCase));   
      }
    }    
    
    public class ProjectModel
    {
        public string Name { get; set; }

        public Guid Id { get; set; }

        public string Url { get; set; } 
    }
```

&nbsp;

**Returns**
[ProjectModelCollection] A collection of devops projects

---

&nbsp;  
# Get-DevopsProject
Returns a project definition from Set-DevopsCredentials 

**Example** 
```
$project = Get-DevopsProject
 
```
#### ProjectModel
```
    public class ProjectModel
    {
        public string Name { get; set; }

        public Guid Id { get; set; }

        public string Url { get; set; } 
    }
```

<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Projects | [ProjectModelCollection] (from Get-DevopsProjects) |  
| -Name | The nameof the remote project to find   |  

</details>

&nbsp;

**Returns**
[ProjectModel] A devop projects

---

&nbsp;  

# Get-DevopsRepositories
Returns a collection of repositories from a given devops project  

**Example** 
```
$projectRepositories = Get-DevopsRepositories -Name $projectName
foreach ($repo in projectRepositories) {
   Write-Host "$($repo.RemoteUrl)"
}
```

#### RepositoryModel
```
    public class RepositoryModel
    {
        public Guid Id { get; set; }

        public string Name { get; set; }

        public string Url { get; set; }

        public string DefaultBranch { get; set; }

        public long Size { get; set; }

        public string RemoteUrl { get; set; }

        public bool IsDisabled { get; set; }

        public Guid ProjectID { get; set; } 
    } 
```

<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Project | (optional) a devops project from a previous call to Get-DevopsProject(s) |  
| -Name | the name of the project to retrieve the repositories from |

</details>

&nbsp;

**Returns**
[RepositoryModelCollection] A devop Repository

---

&nbsp;  
# Get-DevopsRepository
Returns a repository from the given project 

**Example** 
```
$projectRepository = Get-DevopsRepository -ProjectName $projectName -Name $repositoryName
```
#### RepositoryModel
```
    public class RepositoryModel
    {
        public Guid Id { get; set; }

        public string Name { get; set; }

        public string Url { get; set; }

        public string DefaultBranch { get; set; }

        public long Size { get; set; }

        public string RemoteUrl { get; set; }

        public bool IsDisabled { get; set; }

        public Guid ProjectID { get; set; } 
    } 
```

<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Repositories | (optional) [RepositoryModelCollection] a collection of repositories from devops project from a previous call to Get-DevopsRepositories |  
| -ProjectName | (optional) The name of the project that contains the project |
| -Name | the name of the repository to retrieve | 

</details>

&nbsp;

**Returns**
[RepositoryModel] A devops Repository

---

&nbsp;  
# Get-DevopsPipelines
Returns a collection of devopspipelines associated with the given project or all pipelines in the organisation   

**Example** 
```

# to get ALL pipelines in the current organisation
$allPipelines = Get-DevopsPipelines 

# to get a specific pipeline (by name) across all projects
$pipeline = Get-DevopsPipelines -Name $pipelineModelName   

#to get all pipelines in a specific project
$projectPipelines = Get-DevopsPipelines -Project $projectName 

# or any combination of parameters !

```
#### PipelineModelCollection
```
    public class PipelineModelCollection : List<PipelineModel>  
    {
    }
    
    public class PipelineModel
    {
        public string Url { get; set; }

        public long Id { get; set; }

        public string Name { get; set; }

        public string Folder { get; set; }

        public Guid ProjectId { get; set; } 
    }
```

<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Projects | (optional) [ProjectModelCollection] a collection of projects |  
| -ProjectName | (optional) The name of the project that contains the pipelines |
| -Name | the name of the pipeline to retrieve | 

</details>

&nbsp;

**Returns**
[PipelineModelCollection] A collection of devops project pipelines

---

&nbsp;  
# Invoke-DevopsPipeline
Invokes a devops pipeline and returns a pipeline response including the id of the pipeline 

**Example** 
```
# fron a pipeline model 
$pipelineResult = Invoke-Pipeline -Pipeline $pipelineModel 

# a specific pipeline name 
$pipelineResult = Invoke-Pipeline -NAme $pipelineName
```
#### PipelineModelCollection
```
    public class RunPipelineResponseModel
    {
        public string CreatedDateString {  get; set; }

        public DateTime CreatedDate { get; set; } = DateTime.MinValue;  

        public string FinishedDateString { get; set; }

        public DateTime FinishedDate { get; set; } = DateTime.MinValue;

        public long Id { get; set; }

         public string Name  { get; set; }

         public string RunResultString { get; set; }

        public string RunStateString { get; set; }

        public RunResult RunResult { get; set; } = RunResult.Unknown;

        public RunState RunState { get; set; } = RunState.Unknown;

        public bool IsRunning { get; set; } = false;

        public bool IsCompleted { get; set; } = false;  

        public bool HasWorked { get; set; } = false;    

        public Guid ProjectId { get; set; } 
    }
```

<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Pipeline | (optional) [PipelineModel] pipeline model |  
| -Name | the name of the pipeline to invoke | 

</details>

&nbsp;

**Returns**
[RunPipelineResponseModel] The pipeline response 

---

&nbsp;  
# Wait-DevopsPipeline
Waits for a devops pipeline to complete and returns an indicator [bool] as to wether it has completed successfully.

**Example** 
```
$devopsPipelineResult = Invoke-DevopsPipeline -Name $myPipeline | Wait-DevopsPipeline

Write-Host "Pipeline has pipeline worked $($devopsPipelineResult)"

```
#### PipelineModelCollection
```
    public class RunPipelineResponseModel
    {
        public string CreatedDateString {  get; set; }

        public DateTime CreatedDate { get; set; } = DateTime.MinValue;  

        public string FinishedDateString { get; set; }

        public DateTime FinishedDate { get; set; } = DateTime.MinValue;

        public long Id { get; set; }

         public string Name  { get; set; }

         public string RunResultString { get; set; }

        public string RunStateString { get; set; }

        public RunResult RunResult { get; set; } = RunResult.Unknown;

        public RunState RunState { get; set; } = RunState.Unknown;

        public bool IsRunning { get; set; } = false;

        public bool IsCompleted { get; set; } = false;  

        public bool HasWorked { get; set; } = false;    

        public Guid ProjectId { get; set; } 
    }
```

<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Pipeline | (optional) [PipelineModel] pipeline model |  
| -Name | the name of the pipeline to invoke | 

</details>

&nbsp;

**Returns**
[RunPipelineResponseModel] The pipeline response 

---

## GIT Cmdlets
# Test-Git
Tests to see if git is installed.   

**Example** 
```
if (Test-Git) {
   # do something with git  
} else {
   Install-Git -AutoInstall
}
```
&nbsp;

**Returns**
the full path of the git executable OR null if it does not exist

---

&nbsp;

# Install-Git
Downloads and optionally installs the latest verion of git for windows  

**Example** 
```
$gitExe = Install-Git -AutoInstall 
```

<details>
   <summary>Parameters</summary>

   | Parameter | Description |  
   | --- | --- |
   | -AutoInstall | switch parameter which (if specified) installs git to the local users directory

</details>

&nbsp;

**Returns**
[string] the full path of the git executable

---

&nbsp;  
# Copy-Repository
Git Clones a remote devops repository to the specified local directory  

**Example** 
```
$repoName = "my-repo-name"
$repo = Find-DevopsRepository -Name $repoName
$repositoryPath = Copy-Repository -Url $repo.RemoteUrl `
                                  -Directory "C:\Deleteme" `
                                  -Force
```

<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Url | the remote url of the repository to clone (see **Find-DevopsRepository** to get this from a repo name | 
| -User | the user name to authenticate (not required if Set-DevopsCredentials is used |  
| -Password | the password to authenticate (not required if Set-DevopsCredentials is used |  
| -Directory | the local directory to clone into |  
| -Branch | the branch to pull |  
| -Force | (switch parameter) if the repos already exists it will be deleted and re-created |  

</details>

&nbsp;

**Returns**
[string] the full path of the local repository  

---

&nbsp;  

# Find-Branch
indicates if the specified local branch of the given directory exists.

**Example** 
```
if (Find-Branch -Directory $repositoryPath -Branch $branch) {
    # do something because local branch exists
}

```

<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Directory | the local repo directory  |  
| -Branch | the branch to find |  


</details>

&nbsp;

**Returns**
[bool] true if branch exists  

---

&nbsp;  
# Get-Branches
Gets a list of branches for the specified local reporitory 

**Example** 
```
$branches = Get-Branches -DirectoryName $repoName
foreach ($branch in $branches) {
   Write-Host "$($branch.Path)"
}

```

### GitBranchCollection
```
    public class GitBranch
    {
        public string Name { get; set; }

        public bool IsRemote { get; set; } = false;

        public bool IsCurrent { get; set; } = false;

        public string Path { get; set; }

    }
```

<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Directory | Local path of local repository   |  

</details>

&nbsp;

**Returns**
[GitBranchCollection] Array {List<>} of GitBranch

---

&nbsp;  

# Switch-Branch
Checks out the given branch from the given local repository

**Example** 
```
if (Find-Branch -Directory $repositoryPath -Branch $branch) {
   $rc = Switch-Branch -Directory $repositoryPath -Branch $branch
}
```

<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Directory | the local repo directory  |  
| -Branch | the branch to find |  


</details>

&nbsp;

**Returns**
[int] the return code of the switch operation  

---

&nbsp;  

# Remove-Branch
Removes the specified local branch

**Example** 
```
  Remove-Branch -Directory $repositoryPath -Branch $branch -Force

```

<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Directory | the local repo directory  |  
| -Branch | the branch to find |  
| -Force | (Switch) remove the branch even if there are outstanding changes/commits |  

</details>

&nbsp;

**Returns**
[int] return code of the delete operation  

---

&nbsp;  

# New-Branch
Creates a new branch for the given local repository AND checks it out. If a remote branch with the same name exists, it will checkout that branch

**Example** 
```
  # get remote repo
  $repo = Find-DevopsRepository -Name $repoName
  
  # create new branch based on the default remote repo branch (main/master etc)
  New-Branch -Directory $repositoryPath -Branch $branch -BaseBranch $repo.DefaultBranch

```

<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Directory | the local repo directory  |  
| -Branch | the branch to find |  
| -BaseBranch | The base branch from which to create the new branch from |  

</details>

&nbsp;

**Returns**
[int] return code of the new branch operation  

---

&nbsp;  

# Save-Repository
Commits any changes to the given local repository

**Example** 
```
  Save-Repository -Directory $repositoryPath -Message "A Commit message"

```

<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Directory | the local repo directory  |  
| -Message | The commit message |  

</details>

&nbsp;

**Returns**
[int] return code of the save operation  

---

&nbsp;  

# Push-Repository
Pushes changes to the remote repository. 

**Example** 
```
  Push-Repository -Directory $repositoryPath -Branch $branch
```

<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Directory | the local repo directory  |  
| -Branch | (optional) use if the branch you want to push to is NOT the current branch |  

</details>

&nbsp;

**Returns**
[int] return code of the save operation  

---
&nbsp;

# Sync-Repository
Pulls latest changes to local repository for the given branch. 

**Example** 
```
  $rc = Sync-Repository -Directory $repositoryPath -Branch $branch
```

<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Directory | The local repo directory  |  
| -Branch | (optional) Branch you want to pull changes for  |  

</details>

&nbsp;

**Returns**
[int] return code of the pull operation  

---

&nbsp;

# Get-Tags
Gets a collection of tags for the given local repository. 

**Example** 
```
  $tags = Get-Tags -Directory $repositoryPath -Branch $branch
```

<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Directory | The local repo directory  |  
| -Branch | (optional) Branch you want to get tags for  |  

</details>

&nbsp;

**Returns**
[List<system.string>] List of tags associated with this repo/branch  

---

&nbsp;

# New-Tag
Creates a new Tag at the current commit of a local repository 

**Example** 
```
  $rc = New-Tag -Directory $repositoryPath -Tag "23234" -Message "Tag description"
```

<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Directory | The local repo directory  |  
| -Tag | The tag name to add to the current commit  |  
| -Message | (optional) description to add to the tag

</details>

&nbsp;

**Returns**
[int] Return code of the tag operation  

---

&nbsp;
## Excel Cmdlets

&nbsp;
# New-Excel
Creates a new Excel Application and returns [ExcelApplication] 

**Example** 
```
  # create an empty excelapplication
  $excelApp = New-Excel

  # create a new excel application and create new (default) worksheet named "worksheet1"
  # set the hilight colour and freeze the top rwo 
  $excelApp = New Excel -Worksheet "worksheet1" -HilightColour LightGray -FreezeTopRow  
```
<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Worksheet | (optional) create a new worksheet |  
| -HilightColour | (optional switch) set the system.drawing.color (named) colour for highlighted rows\columns |
| -FreezeTopRow | (optional switch) set the top row to be frozen when the user scrools the worksheet |   
</details>

&nbsp;

**Returns**
[ExcelApplication] instance of an excel application

---

&nbsp;
# Open-Excel
Opens an existing excel file and returns [ExcelApplication] 

**Example** 
```
  $excelApp = New-Excel | Open-Excel -Filename "C:\\SomeFile.xlsx" 
```
<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Application | [ExcelApplication] |  
</details>

&nbsp;

**Returns**
[ExcelApplication] instance of the excel application

---

&nbsp;
# Set-ExcelData
Populates an excel Application worksheet (or default) with the contents of a PS class 

**Example** 
```
  class TestExcelData {
    [string]$Excel_String_Column
    [decimal]$Excel_Decimal_Column
    
    TestExcelData() {}

    TestExcelData([string]$Col1, [decimal]$Col2) {
        $this.Excel_String_Column = $Col1
        $this.Excel_Decimal_Column = $Col2
    }

   $dataAray = @()

   $dataAray += New-Object TestExcelData -ArgumentList "row 1 column 1", 1.0
   $dataAray += New-Object TestExcelData -ArgumentList "row 1 column 2", 1.1

   $excelApp = New-Excel | Set-ExcelData -Data $dataAray -UsePropertyNameAsHeader -AutoFitColumns 
} 
```
<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Application | [ExcelApplication] |  
| -Data | PS class Array of data |
| -UsePropertyNameAsHeader | (optional) the property names of the input class will be used as the header names (row 1) of the worksheet |
| -AutoFitColumns | (Optional) (SwitchParameter) alter columns to fit their content |
| -Worksheet | (optional) Switches to or creates a worksheet to hold the class data |

</details>

&nbsp;

**Returns**
[ExcelApplication] instance of the excel application

---

&nbsp;
# Set-ExcelRowProperty
Sets the row properties of a given row and or column returns [ExcelApplication] 
Must be called AFTER a spreadsheet has been populated 

**Example** 
```
  
  # bold every column in row 0 (header column)  
  $excelApp = $app | Set-ExcelRowProperty -Row 0 -Format Bold

  # highlight row 1 column 3
  $excelApp = $app | Set-ExcelRowProperty -Row 0 -Column 3 -Format HiLight 
```
<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Application | [ExcelApplication] | 
| -Row | row number (starts at 1) |  
| -Column | optional column to apply yhr format to 
| -Format | formats the row/column to one of Hilight/Bold/None
| -Worksheet | (Optional) The names of the worksheet 

</details>

&nbsp;

**Returns**
[ExcelApplication] instance of the excel application

---

&nbsp;
# Save-Excel
Saves the ExcelApplication to the given filename and returns [ExcelApplication] 

**Example** 
```
  New-Excel | Save-Excel -Filename "C:\\somewhere.xlsx"
```
<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Application | [ExcelApplication] | 
| -Filename | excel filenane.(replaced if it already exists) |  

</details>

&nbsp;

---

&nbsp;
# Get-ExcelData 
Returns a PS.Class array of data from an existing worksheetand returns [ExcelApplication] 
**note** each property of the given type is matched with a row header value. so only the columns that macth a property are returned  

**Example** 
```
  class TestExcelData {
    [string]$Excel_String_Column
    [decimal]$Excel_Decimal_Column
    
    TestExcelData() {}

    TestExcelData([string]$Col1, [decimal]$Col2) {
        $this.Excel_String_Column = $Col1
        $this.Excel_Decimal_Column = $Col2
   }

   $data = New-Excel `
     | Open-Excel -Filename $excelFile `
     | Get-ExcelData -DataType TestExcelData  -Worksheet "David"

  foreach ($dat in $data) {
      Write-Host "$($dat.Excel_String_Column) $($dat.Excel_Decimal_Column)"
  }   
```
<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Application | [ExcelApplication] | 
| -DataType | Type of the array to return. |  
| -Worksheet | (Optional) name of worksheet to extract data from 

</details>

&nbsp;

**Returns**
[Array] .net generic list of -DataType  

---

&nbsp;
# Select-ExcelWorkSheet 
Selects the given worksheet in the ExcelApplication instance and returns [ExcelApplication] 

**Example** 
```
   $excelApp = New-Excel `
     | Open-Excel -Filename $excelFile `
     | Select-ExcelWorkSheet -Worksheet "David"
```
<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Application | [ExcelApplication] | 
| -Worksheet | Name of worksheet to use 

</details>

&nbsp;

**Returns**
[ExcelApplication] The excelapplication instance

---

## Dotnet Cmdlets
&nbsp; 
# Invoke-DotNetProcess 
Runs the given dotnet process on the current machine (e.g for micro services)  

**Example** 
```
$rootProj = "C:\\somedirectory\\subdir\\"
$projToRun = "$($rootProj)myconsoleapp.csproj"

New-DotNetProcess -Project $projToRun `
                  -WindowStyle Minimized `
                  -Pull ` # pull lates version first
                  -Name "My Program" # -Wait 
```
<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Project | Full path to project | 
| -WindowStyle | Window style of console app (Minimized,Normal etc)
| -Pull | Pull the latest version of the 'current' branch |
| -Branch | Switch to this branch (before run/pull) |
| -Name | Update the window text name   

</details>

&nbsp;

**Returns**
[Process] The process that is started 

---

&nbsp; 
# Start-DotNetCommand 
Runs the given dotnet command within the same process OR in a new command window   

**Example** 
```
$dotnetCommand = "--version"

Start-DotNetCommand -Command $dotnetCommand `
                  -Directory "C:\\somehere\\nice" `
                  -NewWindow ` # run in new external cmd process 
                  -WindowStyle Minimized 
                  #-wait  if speified waits for the command to complete if newWindow specified
```
<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Command | full dotnet cli command  | 
| -Directory | optional starting directory |
| -NewWindow | (switch parameter) if specified starts the dotnet command in a seperate process |
| -WindowStyle | for external process what type of winow (Normal, Mazimized etc) |
| -Wait | (switch parameter) if specified waits for the process to end (if NewWindow is specified)  |
| -Console | (switch parameter) if (specified and it is Not an external process the result of the command is returned as an array of string (list<>)  

</details>

&nbsp;

**Returns**
<br>
[Process] if (-NewWindow) is specified it returns the Process associated with the command window 

[List<>] if -Console specified the result of the command is returned as a generic strin glist 

[rc] if the command fails the return code is returned 

---

## Visual Studio Cmdlets
&nbsp; 
&nbsp; 
# Invoke-VisualStudioBuild 
Builds the given project using the latest version of Visual Studio that is installed on the local machine. This tends to work better than dotnet build and msbuild     

**Example** 
```
$vsProject = "C:\\somewhere\nic\my.csproj"

$consoleLogs = Invoke-VisualStudioBuild -Project $vsProject `
                                        -Console
```
<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Project | full path to the solution or project file to build  | 
| -Console | (switch parameter) if specified returns a list<> of the ouput from the build |
 

</details>

&nbsp;

**Returns**
<br>
[List<>] if (-Console) is specified it returns the List<> (string) of the output 

[int] if (-Console) is not spefified returns the return code of the process 

---

&nbsp; 
# Start-VisualStudioCommand 
Executes the given devenv command against the given solution or project using the latest version of Visual Studio that is installed on the local machine. This tends to work better than dotnet build and msbuild     

**Example** 
```
$vsProject = "C:\\somewhere\nic\my.csproj"
$args = "/clean"

$consoleLogs = Start-VisualStudioCommand -Project $vsProject `
                                        -Arguments $args `
                                        -Console
```
<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Project | full path to the solution or project file to build  | 
| -Arguments | list of argument to pass to visual studio |  
| -Console | (switch parameter) if specified returns a list<> of the ouput from the build |
 

</details>

&nbsp;

**Returns**
<br>
[List<>] if (-Console) is specified it returns the List<> (string) of the output 

[int] if (-Console) is not spefified returns the return code of the process 

---

## SQL Cmdlets
&nbsp;

# Resolve-SqlResult 
Takes the result of a Invoke-Sqlcmd call and returns a populated (typed) object which can be a single object or an array of objects. Only the properties specified in the -ResultType are returned (case-insensitive)     

**Example** 
```
class SqlResult {
   [Guid]$Id
   [string]$Name
}

$sqlconnection = "server=xxx; User Id=blah ..."
$sql = "select * from xxx where yyy = 'asasd"

$result = Invoke-Sqlcmd -ConnectionString $sqlConnection -Query $sql

$resultRows = Resolve-SqlResult -Result $result -ResultType SqlResult

foreach ($row in resultRows) {
   Write-Host "$($row.Id) $($row.Name)"
}
 

```
<details>
   <summary>Parameters</summary>

| Parameter | Description |  
| --- | --- |
| -Result | The result from the Invoke-Sqlcmd call  | 
| -ResultType | The class type to populate with the results (single or list<>) |  
 

</details>

&nbsp;

**Returns**
<br>
[List<>] of ResultType  

[object] of ResultType 

---

&nbsp;
&nbsp;
&nbsp;

# Examples 

## GIT/Devops

# Clone, update, commit, push a repository and create a pull request , and complete it 
```
Import-Module PS.Utilities

$patToken = "YOUR-PAT-TOKEN"
$org = "YOUR-DEVOPS-ORGANISATION"

# set credentials for git and azure devops based on a pat token  
# lasts for the duration of the powershell session 
Set-DevopsCredentials -PlainTextPassword $patToken -Organisation $org 

$repoName = "Test-Repo"

# locate the repository within all of the projects available to the users pat token auth  
$repo = Find-DevopsRepository -Name $repoName

# clone the repository locally (delete if already there) 
$repositoryPath = Copy-Repository -Url $repo.RemoteUrl `
                                  -Directory "C:\Deleteme" `
                                  -Force

Write-Host "Repository cloned to $($repositoryPath) "

$branch = "DavidTest"

Write-Host "Creating Branch $($branch)"

# if there is not a branch defined - delete it  
if (Find-Branch -Directory $repositoryPath -Branch $branch) {
    Remove-Branch -Directory $repositoryPath -Branch $branch -Force
}

# re-create the branch - NOTE if there is a remote branch with the same name it will checkout that branch instead 
New-Branch -Directory $repositoryPath -Branch $branch -BaseBranch $repo.DefaultBranch

# add a new text file to the local repo within the newly created branch 
Write-Host "Add test file to new Branch"

Get-Date | Out-File -FilePath (Join-Path -Path $repositoryPath -ChildPath "testcommit.txt") -Append 

Write-Host "Commit A test file"

# commit the changes locally
Save-Repository -Directory $repositoryPath -Message "A test for david"

Write-Host "Push Changes"

# push the directory to remote devops instance 
Push-Repository -Directory $repositoryPath -Branch $branch

# create a pull request AND complete it! - not a real world example 
$pullRequestResponse = New-DevopsPullRequest -Directory $repositoryPath -Repository $repoName -SourceBranch $branch -TargetBranch "main" -Title "Test request title" -Description "Test pull request from powershell" | Complete-DevopsPullRequest -Verbose

Write-Host ($pullRequestResponse | Format-List | Out-String)

Write-Host ($pullRequestResponse.CreatedBy | Format-List | Out-String) 
```

&nbsp;
&nbsp;

## Excel 

### Create and read excel spreadsheet 
``` 
Import-Module PS.Utilities

class TestExcelData {
    [string]$Excel_String_Column
    [decimal]$Excel_Decimal_Column
    
    TestExcelData() {}

    TestExcelData([string]$Col1, [decimal]$Col2) {
        $this.Excel_String_Column = $Col1
        $this.Excel_Decimal_Column = $Col2
    }
}

$global:excelFile = "C:\\deleteme\\TestExcelPS.xlsx"

# create a excel spreadsheet 
function CreateSpreadsheet() {
   $dataAray = @()

   $dataAray += New-Object TestExcelData -ArgumentList "hello 1 this is a very long to see if it will expand ",123.1
   $dataAray += New-Object TestExcelData -ArgumentList "hello 2",3123123.1

   $excelApp = New-Excel -Worksheet "David" -FreezeTopRow -HilightColour LightGray `
                         | Set-ExcelData -Data $dataAray -UsePropertyNameAsHeader -AutoFitColumns `
                         | Set-ExcelRowProperty -Row 0 -Format Hilight ` # highlight header row
                         | Save-excel -Filename $excelFile
}

function LoadSpreadsheet() {
# load the spreadhseet and display the contents 

  $data = New-Excel `
     | Open-Excel -Filename $excelFile `
     | Select-ExcelWorkSheet -Worksheet "David" `
     | Get-ExcelData -DataType TestExcelData 

  foreach ($dat in $data) {
      Write-Host "$($dat.Excel_String_Column) $($dat.Excel_Decimal_Column)"
  }   
}


CreateSpreadsheet
LoadSpreadsheet
```


