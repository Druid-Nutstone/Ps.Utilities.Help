# Ps.Utilities Help 

# Overview 
PS.Utilies is a c# .NET standard 2.0 set of powershell cmdlets that wraps the following utilities :- 

<ol>
  <li>Git - wraps many usefull git commands</li>
  <li>Devops - Uses the azure api to manipulate azure devops projects/repositories and pipelines
  <li>Excel - provides wrappers to read and write Excel files using the excellent Spread sheet Light library.   
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
   | [Get-DevopsPipelines](#get-devopspipelines) | Returns a collection of pipelines |
   | [Invoke-DevopsPipeline](#invoke-devopspipeline) | invokes the given pipeline |
   | [Wait-DevopsPipeline](#wait-devopspipeline) | Waits for a devops pipeline to complete
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
&nbsp;
&nbsp;

## Examples 
# Clone, update, commit, push a repository and create a pull request , and complete it !
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



