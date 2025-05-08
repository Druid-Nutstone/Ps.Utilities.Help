# PS.Git
PS.Git is a set of C# cmdlets that use the Lib2GitSharp library and Github API (native) to provide a single set of cmdlets to manipulate both local and remote github repositories.

it does __NOT__ rely on external Git ot Github cli's.

### Authentication 
Currently it only supports GitHub PAT tokens to authenticate actions that require authentication. 
it does , however , implement Get-Token -Script { your stuff } so that you can implement Oauth 
or your own mechanism to get a PAT token   

## Pre-requisites 

It also only supports Powershell 7.50 (and above) and a windows X64 client.

## Debugging 

If you want more debugging for any action you can use the -Verbose parameter on any cmdlet 
Or to set the __Global__ verbose variable 

i.e $VerbosePreference = "Continue"

## HELP 

all Cmdlets now have in-built help. to display help for any cmdlet add the -Help switch parameter 
i.e 
__New-GitConnection -Help__ will display help specific to New-GitConnection

Or 

Get-GitHelp -List (display's available cmdlets in PS.Git) 

Get-GitHelp -Console (displays ALL help for all cmdlets on the console)

__Note__ this will be more up-to-date than this documentation!

## Using PS.Git 
```
   Install-Module PS.Git
   Import-Module PS.Git 
```   

##
##
# Cmdlet Categories
   
## Branches
   
| Cmdlet | Description |
|---|---|
| [Get-Branch](#get-branch) | Checks out the named (or default branch) if it exists. Any uncommited changes on the current branch will be REMOVED if the -Force parameter is specified otherwise an error will be thrown. |
| [Get-Branches](#get-branches) | Returns either a list of local branches or a filtered list based on the supplied ScriptBlock. The script block will be provides a LocalBranchCollection via a parameter |
| [Get-CurrentBranch](#get-currentbranch) | Returns the current branch. The script block will be provides a LocalBranchCollection via a parameter |
| [New-Branch](#new-branch) | Creates and checks out a new branch. Or , if the branch exists, Checks it out. |
| [Test-Branch](#test-branch) | Tests wether a local branch existrs or not |
## Connection
   
| Cmdlet | Description |
|---|---|
| [Get-GitConnection](#get-gitconnection) | Returns specific or all of the properties of the git-connection |
| [Get-GitToken](#get-gittoken) | Defined ScriptBlock that returns a string that is a valid Git hub Token.  That is then stored in the git connection object |
| [New-GitConnection](#new-gitconnection) | Creates a global git connection that can be used both locally and remotely It is available throughout the powershell session for all other cmdlets |
| [Set-GitConnection](#set-gitconnection) | Sets one or more git connection properties |
## Release
   
| Cmdlet | Description |
|---|---|
| [Get-Release](#get-release) | Returns a list of Releases (GitReleaseCollection) or a single release (GitReleaseItem) Use the -Latest, -Name and -Tag parameters to narrow the search |
| [Get-ReleaseDownloads](#get-releasedownloads) | Downloads the release content of a release and returns a (GitDownloadReleaseResponse) which contains fileinfo information about the files downloaded By default all release artifacts are download , and , if they are zipped , they are unzipped to the target -Path |
| [New-Release](#new-release) | Creates a new remote release |
## Repository
   
| Cmdlet | Description |
|---|---|
| [Add-RepositoryUser](#add-repositoryuser) | Adds to the the repositoryrequest a valid user whom is available to review |
| [Get-Repositories](#get-repositories) | Gets a list of remote repositories for the current user/Organisation |
| [Invoke-NewRepository](#invoke-newrepository) | Creates a new remote repository base on the input from a GitRepositoryRequest object |
| [New-Repository](#new-repository) | Creates a new GitRepositoryRequest request |
| [Remove-Repository](#remove-repository) | Removes a remote/local repository from the current user/organisation and or the local repository |
| [Test-Repository](#test-repository) | Tests for the existence of a remote repository |
| [Get-PullRequest](#get-pullrequest) | Returns details about a specific pull request. Use in conjunction with Get-PullRequests |
| [Get-PullRequests](#get-pullrequests) | Returns index details about pull requests (open or otheriwse) for the current repository It will return (GitPullResponse) or (GitPullRequestCollection) for multiple entries if there are none found an empty object is returned (or error is -ErrorAction is 'Stop' |
| [New-PullRequest](#new-pullrequest) | Creates a new pull request. It will return (GitPullResponse) or (GitPullRequestCollection) for multiple entries if there are none found an empty object is returned (or error is -ErrorAction is 'Stop' |
| [Merge-PullRequest](#merge-pullrequest) | Merges a pullrequest into  |
| [Merge-Remote](#merge-remote) | Merges remote branches |
| [Invoke-Commit](#invoke-commit) | Commits any changes in the current repository. with an optional tag and optionall pushes the changes |
| [Invoke-Pull](#invoke-pull) | Pulls the latest changes for the specified repo and branch. if the branch specified is not the current branch - it is checkout |
| [Invoke-Push](#invoke-push) | Pushes any commited changes on the current repository |
| [Test-Changes](#test-changes) | Tests if there any any uncommited chages to the current local repository |
| [Invoke-Clone](#invoke-clone) | Clones a remote repository locally. It is an itelligent clone, in that , if the repository already exists locally, it will not clone it again. but , pull the latest version from the remote if the -Force parameter is set, it will remove the local repo (including uncomitted changes) and clone again |
## Tags
   
| Cmdlet | Description |
|---|---|
| [Get-LatestTag](#get-latesttag) | Returns the latest tag (semantic) of the current respository. you can then increment it if there are no tags an error is thrown UNLESS erroraction is set to 'continue' in which case $null is returned |
| [Get-Tags](#get-tags) | Returns a GitTagCollection object tags in the current repo |
| [Invoke-Tag](#invoke-tag) | Creates a tag locally and remotely based on the current commit. It returns a SemanticVersion or null  |
| [New-SemanticTag](#new-semantictag) | Returns a SemanticVersion that that can be used as a tag. It uses the standard GitHub semantic versioning scheme. (v)major.minor.patch if the -Tags option (or value from pipeline) is specified the tag will be incremented before it is returned |
| [New-Tag](#new-tag) | Returns a new SemanticVersion that that can be used as a tag. |
| [Remove-Tag](#remove-tag) | Removes an existing tag from the local and remote repository |
| [Test-Tags](#test-tags) | Returns a bool if the repository has any tags. use the -SemanticTagsto check for semantic tags only |
## Utility
   
| Cmdlet | Description |
|---|---|
| [Copy-ToRepository](#copy-torepository) | Utility to copy files from a local path to the current repository. The -Path parameter allows filespec and or root directory definitions This needs to be updated to allow for the path being a directory and the directory copied as well |
## VisualStudio
   
| Cmdlet | Description |
|---|---|
| [Get-ProjectControl](#get-projectcontrol) | Returns a VisualStudioControlGroup from the given path (full or partial)  if the -Path parameter does not contains .csproj - the first .cspro file gound is used The returned object exposes the follwing Methods :-      GetProjectType() - returns the string project type      GetProjectOrganisation() - returns the string organisation      GetProjectUser() - returns the string user      GetProjectRepository() - returns the string repository      GetProjectWorkFlow() - returns the workflow to invoke      GetProjectWorkFlowRepository() - returns the workflow repository      GetProjectProjectPath() - returns the relative propject path (directory) of the target project (not including the .csproj)      IsOrg() - returns a boolean if the connection is an organisation |
| [Get-ProjectProperties](#get-projectproperties) | Returns the a collection of custom ProjectGroup entries from a visual studio .csproj file So to define nuget properties in the project:-     <PropertyGroup Label="Control">        <ProjectType>Nuget</ProjectType>        <OrganisationOrUser>userororg</OrganisationOrUser>        <Repository>TestRepository</Repository>      </PropertyGroup>     <PropertyGroup Label="Nuget">        <Version>1.0.0</Version>        <PackageId>TestPackage</PackageId>        .....     </PropertyGroup> |
| [Get-ProjectPropertyValue](#get-projectpropertyvalue) | Returns A specific property value from a projectgroup group |
| [Get-ProjectType](#get-projecttype) | Returns the Project type enum of a csproj project from the Control ProjectGroup |
| [Set-ProjectPropertyValue](#set-projectpropertyvalue) | Sets A specific property value in a projectgroup group |
## WorkFlow
   
| Cmdlet | Description |
|---|---|
| [Get-WorkFlowLogs](#get-workflowlogs) | Returns the logs from a workflow run. Etiehr to a file or the console. |
| [Get-WorkFlows](#get-workflows) | Returns all the defined workflows for the given repository. |
| [Invoke-WorkFlow](#invoke-workflow) | Runs a given workflow and optionally waits until it is complete You would typically use this cmdlet with Set-Workflow and Set-WorkFlowInputVariable  If the workflow has required variables, and they are not set. An error occurs. |
| [Set-WorkFlowInputVariable](#set-workflowinputvariable) | Sets the given variable value (or an Environment Value) to the current workflow request and returns the updated (GitWorkFlowRequest) object |
| [Set-WorkFlow](#set-workflow) | Initialises a GitWorkFlowRequest object with the variables required from the workflow definition |
   
# Branches Detail
   
   
   
   
# Get-Branch
   
> Checks out the named (or default branch) if it exists.
> Any uncommited changes on the current branch will be REMOVED if the -Force parameter is specified
> otherwise an error will be thrown.
   
## ** Example **
   
```
Get-Branch -Branch 'mybranch' -Force # remove any uncommited chages
```
## ** Parameters **
   
   
**-Origin&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;If specified the origin branch (main branch) is used
   
**-Force&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Forces any uncommited changes to be removed
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Get-Branches
   
> Returns either a list of local branches or a filtered list based on the supplied ScriptBlock.
> The script block will be provides a LocalBranchCollection via a parameter
   
## ** Example **
   
```
Get-Branches -Script {
  param($Branches)
    return $Branches | Where-Object { $_.Name -like 'feature/*' }
}
```
## ** Parameters **
   
   
**-Script&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[ScriptBlock]
   
&nbsp;&nbsp;&nbsp;The filter script to be called iwth the branch list object
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Get-CurrentBranch
   
> Returns the current branch.
> The script block will be provides a LocalBranchCollection via a parameter
   
## ** Example **
   
```
$currentBranch = Get-CurrentBranch
```
## ** Parameters **
   
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# New-Branch
   
> Creates and checks out a new branch. Or , if the branch exists, Checks it out.
   
## ** Example **
   
```
$branchok = New-Branch -Branch 'mynewbranch'
```
## ** Parameters **
   
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Test-Branch
   
> Tests wether a local branch existrs or not
   
## ** Example **
   
```
if ((Test-Branch -Branch 'mybranch')) {
    Write-Host 'Branch exists'
}
```
## ** Parameters **
   
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
# Connection Detail
   
   
   
   
# Get-GitConnection
   
> Returns specific or all of the properties of the git-connection
   
## ** Example **
   
```
Get-GitConnection -RepositoryOwner # get the org or user of the connection
```
## ** Parameters **
   
   
**-RepositoryOwner&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Returns either the organisation or user of the connection
   
**-Token&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Returns the defined PAT token as a string
   
**-IsOrg&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;retuns a boolean if the connection is for an origanisation
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Get-GitToken
   
> Defined ScriptBlock that returns a string that is a valid Git hub Token. 
> That is then stored in the git connection object
   
## ** Example **
   
```
Get-GitToken -Script {
    return SomeWayToGetATokenThatsReturnsAString
}
```
## ** Parameters **
   
   
**-Script&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[ScriptBlock]
   
&nbsp;&nbsp;&nbsp;A user defined ScriptBlock that returns a valid Git PAT token
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# New-GitConnection
   
> Creates a global git connection that can be used both locally and remotely
> It is available throughout the powershell session for all other cmdlets
   
## ** Example **
   
```
New-GitConnection -Organisation 'MyOrg'
```
## ** Parameters **
   
   
**-TestConnection&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Tests the connection
   
**-TokenScript&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[ScriptBlock]
   
&nbsp;&nbsp;&nbsp;A ScriptBlock that returns a valid Git token. This will be used in subsequent GitHub calls
   
**-Organisation&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The Organisation name of the remote repository
   
**-User&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The User of the name of the remote repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The Remote Repository name
   
**-Token&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The GitHub PAT token used for authentication
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The Root path of the local repository - the repository name will be appended automatically
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The branch to action against (defaults to origin/main)
   
**-DefaultBranch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The remote origin branch. Gets populated automatically by some cmdlets
   
**-Email&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The email to use when commiting/pushing
   
**-CredentialUser&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;Not used. the PAT token is used. Here for future oauth development
   
**-CredentialPassword&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;Not used. the PAT token is used. Here for future development
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Set-GitConnection
   
> Sets one or more git connection properties
   
## ** Example **
   
```
Set-GitConnection -Repository 'somerepo' -Branch 'mybranch
```
## ** Parameters **
   
   
**-Organisation&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The Organisation name of the remote repository
   
**-User&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The User of the name of the remote repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The Remote Repository name
   
**-Token&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The GitHub PAT token used for authentication
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The Root path of the local repository - the repository name will be appended automatically
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The branch to action against (defaults to origin/main)
   
**-DefaultBranch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The remote origin branch. Gets populated automatically by some cmdlets
   
**-Email&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The email to use when commiting/pushing
   
**-CredentialUser&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;Not used. the PAT token is used. Here for future oauth development
   
**-CredentialPassword&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;Not used. the PAT token is used. Here for future development
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
# Release Detail
   
   
   
   
# Get-Release
   
> Returns a list of Releases (GitReleaseCollection) or a single release (GitReleaseItem)
> Use the -Latest, -Name and -Tag parameters to narrow the search
   
## ** Example **
   
```
$releases = Get-Release -Repository -Latest
```
## ** Parameters **
   
   
**-Latest&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;If defined returns the latest release
   
**-Name&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;Filter on the name of the release
   
**-Tag&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;Filter on a specific tag
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Get-ReleaseDownloads
   
> Downloads the release content of a release and returns a (GitDownloadReleaseResponse)
> which contains fileinfo information about the files downloaded
> By default all release artifacts are download , and , if they are zipped , they are unzipped to the target -Path
   
## ** Example **
   
```
$downloads = Get-Release -Latest | Get-ReleaseDownloads -Repository -Latest
```
## ** Parameters **
   
   
**-Release&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[GitReleaseItem]
   
&nbsp;&nbsp;&nbsp;GitReleaseItem from Get-Release
   
**-Path&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;Required target path for downloads. The directory structure will be created if it does not exist
   
**-Tag&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;Optional Tag to search
   
**-ReleaseId&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[Int32]]
   
&nbsp;&nbsp;&nbsp;Release id of the releasew to download
   
**-Latest&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Get leatest release
   
**-ExtractZip&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[Boolean]
   
&nbsp;&nbsp;&nbsp;Extract all zip files found in the release
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# New-Release
   
> Creates a new remote release
   
## ** Example **
   
```
$releaseResponse = New-ReleaseLatest -UseLatestTag -ReleaseName 'myrelease' -Description 'my first release'
```
## ** Parameters **
   
   
**-NewTag&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;String tag that does not conform to semantic tag rules
   
**-Tag&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[GitTag]
   
&nbsp;&nbsp;&nbsp;Semantic tag
   
**-UseLatestTag&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;If specified uses the latest tag from the repository
   
**-ReleaseName&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;Mandatory release name
   
**-Description&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;Mandatory description
   
**-Draft&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Specifies this release is a draft
   
**-PreRelease&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Specifies this release is a pre-release
   
**-MakeLatest&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;This release is the latest
   
**-Assets&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String[]]
   
&nbsp;&nbsp;&nbsp;An Array of file names to be added to the release
   
**-AssetDirectory&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;A directory (source) that is used to populate the assets. This will be Zipped
   
**-AssetZipName&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;Optional the name of the zip to create from the sset directory
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
# Repository Detail
   
   
   
   
# Add-RepositoryUser
   
> Adds to the the repositoryrequest a valid user whom is available to review
   
## ** Example **
   
```
$repositoryRequest = Add-RepositoryUser -Name 'fred.bloggs' -UserType User
```
## ** Parameters **
   
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[GitRepositoryRequest]
   
&nbsp;&nbsp;&nbsp;Mandatory the GitRepositoryRequest object
   
**-Name&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;Manadatory the name of the user
   
**-UserType&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[RepositoryUserType]
   
&nbsp;&nbsp;&nbsp;The type of user User or Team
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Get-Repositories
   
> Gets a list of remote repositories for the current user/Organisation
   
## ** Example **
   
```
$repositoryList = Get-Repositories -Script {
     param($repoList)
     return $repoList | Where $_Name -eq 'myrepo'
}
```
## ** Parameters **
   
   
**-Script&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[ScriptBlock]
   
&nbsp;&nbsp;&nbsp;Optional script to filter the returned reposioty list
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Invoke-NewRepository
   
> Creates a new remote repository base on the input from a GitRepositoryRequest object
   
## ** Example **
   
```
$repositoryResult = New-Repository -Name 'frank' | Invoke-Repository
```
## ** Parameters **
   
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[GitRepositoryRequest]
   
&nbsp;&nbsp;&nbsp;Mandatory The GitRepositoryRequest object to create the repository from
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# New-Repository
   
> Creates a new GitRepositoryRequest request
   
## ** Example **
   
```
$repositoryList = New-Repository -Name 'frank'
```
## ** Parameters **
   
   
**-Name&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;Mandatory The unique name of the repository
   
**-Private&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;The repository is Private
   
**-Description&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The descrition of the repository content
   
**-MinimumRewiewers&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[Int32]
   
&nbsp;&nbsp;&nbsp;The minimum number of reviewers required to fullfill a pull request (default 0 = none)
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Remove-Repository
   
> Removes a remote/local repository from the current user/organisation and or the local repository
   
## ** Example **
   
```
$deleteResponse = Remove-Repository -Name 'myrepo'
```
## ** Parameters **
   
   
**-Name&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;Mandatory the name of the repository to remove
   
**-Action&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[RepositoryRemoveAction]
   
&nbsp;&nbsp;&nbsp;Which Repository to Remove :- Remote , local or Both
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Test-Repository
   
> Tests for the existence of a remote repository
   
## ** Example **
   
```
$repoExists = Test-Repository -Name 'myrepo'
```
## ** Parameters **
   
   
**-Name&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Get-PullRequest
   
> Returns details about a specific pull request. Use in conjunction with Get-PullRequests
   
## ** Example **
   
```
$pullRequests = Get-PullRequests -Repository 'myRepo' -Open
$pullRequestResponseDetail = $pullRequests | Get-PullRequest
if ($pullRequestResponseDetail.IsBlocked) {
   Write-Host 'Cannot merge - waiting for reviewers'
}
```
## ** Parameters **
   
   
**-PullResponse&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[GitPullResponse]
   
&nbsp;&nbsp;&nbsp;GitPullResponse object from pipeline
   
**-PullNumber&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[Int32]]
   
&nbsp;&nbsp;&nbsp;The unique pull number from a pull request
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Get-PullRequests
   
> Returns index details about pull requests (open or otheriwse) for the current repository
> It will return (GitPullResponse) or (GitPullRequestCollection) for multiple entries
> if there are none found an empty object is returned (or error is -ErrorAction is 'Stop'
   
## ** Example **
   
```
$pullRequests = Get-PullRequests -Repository 'myRepo' -Open
```
## ** Parameters **
   
   
**-PullRequest&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[GitPullResponse]
   
&nbsp;&nbsp;&nbsp;A specific pull request object
   
**-Open&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Filter on Open pull requests
   
**-SourceBranch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;Filter on a source branch name
   
**-TargetBranch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;Filter on a target branch
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# New-PullRequest
   
> Creates a new pull request.
> It will return (GitPullResponse) or (GitPullRequestCollection) for multiple entries
> if there are none found an empty object is returned (or error is -ErrorAction is 'Stop'
   
## ** Example **
   
```
$pullRequests = Get-PullRequests -Repository 'myRepo' -Open
```
## ** Parameters **
   
   
**-Title&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;Mandatory Title of pull request
   
**-Description&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;Mandatory Description of pull request
   
**-Reviewers&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String[]]
   
&nbsp;&nbsp;&nbsp;Optional Array of required reviewers. They must be active in the Organisation/User
   
**-TeamReviewers&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String[]]
   
&nbsp;&nbsp;&nbsp;Optional Array of required team reviewers. They must be active in the Organisation/User
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Merge-PullRequest
   
> Merges a pullrequest into 
   
## ** Example **
   
```
$mergeResponse = Merge-PullRequest -Number 4 -Comment 'Merging branch'
```
## ** Parameters **
   
   
**-PullRequest&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[GitPullResponse]
   
&nbsp;&nbsp;&nbsp;Optional GitPullResponseObject
   
**-Number&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[Int32]]
   
&nbsp;&nbsp;&nbsp;Optional pull request number
   
**-Comment&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;Mandatory merge commment
   
**-LeaveBranches&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Optional switch to leave local and remote branches with the branch intact
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Merge-Remote
   
> Merges remote branches
   
## ** Example **
   
```
$mergeResponse = Merge-Remote -SourceBranch 'testbranch' -DestinationBranch 'main' -Comment 'remote merge' -DeleteBranches
```
## ** Parameters **
   
   
**-Comment&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;
   
**-SourceBranch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;
   
**-DestinationBranch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;
   
**-DeleteBranches&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Invoke-Commit
   
> Commits any changes in the current repository. with an optional tag and
> optionall pushes the changes
   
## ** Example **
   
```
$pushCommitResult = Invoke-Commit -Message 'my commit' -Tag 'v1.0.0' -Push -ShowProgress
```
## ** Parameters **
   
   
**-Tag&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The tag to tag the commit with (can be semantic or free-form)
   
**-Message&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The message associated with the commit. this is REQUIRED
   
**-Push&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;If specified the commit will be pushed to the remote
   
**-ShowProgress&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;If specified shows a manualy progress of the push
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Invoke-Pull
   
> Pulls the latest changes for the specified repo and branch.
> if the branch specified is not the current branch - it is checkout
   
## ** Example **
   
```
$mergeStatus = Invoke-Pull -Branch 'mybranch'
```
## ** Parameters **
   
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Invoke-Push
   
> Pushes any commited changes on the current repository
   
## ** Example **
   
```
$pushed = Invoke-Push -ShowProgress
```
## ** Parameters **
   
   
**-ShowProgress&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Shows the progress of the push action
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Test-Changes
   
> Tests if there any any uncommited chages to the current local repository
   
## ** Example **
   
```
$pushed = Invoke-Push -ShowProgress
```
## ** Parameters **
   
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Invoke-Clone
   
> Clones a remote repository locally.
> It is an itelligent clone, in that , if the repository already exists locally, it will not clone it again.
> but , pull the latest version from the remote
> if the -Force parameter is set, it will remove the local repo (including uncomitted changes) and clone again
   
## ** Example **
   
```
$result = Invoke-Clone -LocalRepository 'C:\MyRepos' -Force
```
## ** Parameters **
   
   
**-ShowProgress&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Shows progress of the clone process
   
**-Force&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Re-clones the repo even if it is already in the -LocalRepository location
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
# Tags Detail
   
   
   
   
# Get-LatestTag
   
> Returns the latest tag (semantic) of the current respository. you can then increment it
> if there are no tags an error is thrown UNLESS erroraction is set to 'continue'
> in which case $null is returned
   
## ** Example **
   
```
$newTag = Get-LatestTag -Increment -ErrorAction 'Continue'
```
## ** Parameters **
   
   
**-Increment&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Is specified and the tag is semantic. it weill return the next incremented tag version
   
**-AsString&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Returns the tag as a string. and increments it if -Increment is specified
   
**-IncrementType&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SemanticVersionType]
   
&nbsp;&nbsp;&nbsp;The part of the Semantic (Major, Minor, Patch) to increment
   
**-AsNativeString&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Is specified and the tag is semantic. it will return the next incremented tag version without the 'v' prefix
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Get-Tags
   
> Returns a GitTagCollection object tags in the current repo
   
## ** Example **
   
```
$tagCollection = Get-Tags
```
## ** Parameters **
   
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Invoke-Tag
   
> Creates a tag locally and remotely based on the current commit. It returns a SemanticVersion or null 
   
## ** Example **
   
```
$tagCreated = Invoke-Tag -TagString 'my oh my'
```
## ** Parameters **
   
   
**-TagString&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;A free form tag to add to the current commit
   
**-Tag&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SemanticVersion]
   
&nbsp;&nbsp;&nbsp;Uses a passed in SemanticVersion
   
**-Increment&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Gets the last tag in the repository , increments it and uses that
   
**-Comment&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;Optional comment or release note to be added to the tag. If not specified just the tag name is used
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# New-SemanticTag
   
> Returns a SemanticVersion that that can be used as a tag.
> It uses the standard GitHub semantic versioning scheme.
> (v)major.minor.patch
> if the -Tags option (or value from pipeline) is specified the tag will be incremented before it is returned
   
## ** Example **
   
```
$newTag = Get-LatestTag -Increment -ErrorAction 'Continue'
```
## ** Parameters **
   
   
**-Tags&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[GitTagCollection]
   
&nbsp;&nbsp;&nbsp;Optional GitTagCollection (Get-Tags)
   
**-TagIncrement&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SemanticVersionType]
   
&nbsp;&nbsp;&nbsp;The semantic type Major, Minor, Path to increment when -Tags are specified
   
**-NativeTag&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;A non-version tag (i.e x.x.x without the 'v')
   
**-Major&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[Int32]
   
&nbsp;&nbsp;&nbsp;
   
**-Minor&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[Int32]
   
&nbsp;&nbsp;&nbsp;
   
**-Patch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[Int32]
   
&nbsp;&nbsp;&nbsp;
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# New-Tag
   
> Returns a new SemanticVersion that that can be used as a tag.
   
## ** Example **
   
```
$newTag = New-Tag -Tag '1.2.3' -Increment
```
## ** Parameters **
   
   
**-Major&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[Int32]
   
&nbsp;&nbsp;&nbsp;The Major version
   
**-Minor&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[Int32]
   
&nbsp;&nbsp;&nbsp;The Minor version
   
**-Patch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[Int32]
   
&nbsp;&nbsp;&nbsp;The Patch Version
   
**-Tag&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;An Existing tag (vx.x.x) or (x.x.x)
   
**-TagIncrement&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SemanticVersionType]
   
&nbsp;&nbsp;&nbsp;The semantic type Major, Minor, Path to increment when -Tags are specified
   
**-AsString&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;If specified the tag is returned as a string (vx.x.x)
   
**-Increment&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;If specified the tag is Incremented by the value of the -TagIncrement (Major, Minor, Patch (default)). The initil -Tag must be specified
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Remove-Tag
   
> Removes an existing tag from the local and remote repository
   
## ** Example **
   
```
$tagRemovalResult = Remove-Tag -Name 'mytag'
 # or 
$tagRemovalResult1 = Remove-Tag -Last # remove last tag
```
## ** Parameters **
   
   
**-Last&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Remove the last tag
   
**-Name&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;Name of the tag
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Test-Tags
   
> Returns a bool if the repository has any tags.
> use the -SemanticTagsto check for semantic tags only
   
## ** Example **
   
```
if (!(Test-Tags -LocalRepository 'blah')) {
    Write-Host 'No tags found'
}
```
## ** Parameters **
   
   
**-SemanticTags&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Only tests against valid semantic tags
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
# Utility Detail
   
   
   
   
# Copy-ToRepository
   
> Utility to copy files from a local path to the current repository.
> The -Path parameter allows filespec and or root directory definitions
> This needs to be updated to allow for the path being a directory and the directory copied as well
   
## ** Example **
   
```
$result = Copy-ToRepository -Path 'C:\myfiles\*.ps1'
```
## ** Parameters **
   
   
**-Path&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The source filespec of files to copy
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
# VisualStudio Detail
   
   
   
   
# Get-ProjectControl
   
> Returns a VisualStudioControlGroup from the given path (full or partial)
>  if the -Path parameter does not contains .csproj - the first .cspro file gound is used
> The returned object exposes the follwing Methods :-
>      GetProjectType() - returns the string project type
>      GetProjectOrganisation() - returns the string organisation
>      GetProjectUser() - returns the string user
>      GetProjectRepository() - returns the string repository
>      GetProjectWorkFlow() - returns the workflow to invoke
>      GetProjectWorkFlowRepository() - returns the workflow repository
>      GetProjectProjectPath() - returns the relative propject path (directory) of the target project (not including the .csproj)
>      IsOrg() - returns a boolean if the connection is an organisation
   
## ** Example **
   
```
$controlGroup = Get-ProjectControl -Path 'C:\myrepos\ps.git'
```
## ** Parameters **
   
   
**-Path&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;A fully or partial qualified path to search fro csproj file
   
**-ProjectCollection&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[VisualStudioProjectCollection]
   
&nbsp;&nbsp;&nbsp;The VisualStudioProjectCollection object
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Get-ProjectProperties
   
> Returns the a collection of custom ProjectGroup entries from a visual studio .csproj file
> So to define nuget properties in the project:-
>     <PropertyGroup Label="Control">
>        <ProjectType>Nuget</ProjectType>
>        <OrganisationOrUser>userororg</OrganisationOrUser>
>        <Repository>TestRepository</Repository>
> 
>     </PropertyGroup>
>     <PropertyGroup Label="Nuget">
>        <Version>1.0.0</Version>
>        <PackageId>TestPackage</PackageId>
>        .....
>     </PropertyGroup>
   
## ** Example **
   
```
$projectProperties = Get-ProjectProperties -ProjectPath 'C:\repos\someproject.csproj'
```
## ** Parameters **
   
   
**-ProjectPath&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The Visual Studio .csproj file that contains the custom ProjectGroup properties
   
**-ProjectCollection&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[VisualStudioProjectCollection]
   
&nbsp;&nbsp;&nbsp;The VisualStudioProjectCollection object
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Get-ProjectPropertyValue
   
> Returns A specific property value from a projectgroup group
   
## ** Example **
   
```
$projectCollection = Get-ProjectProperties -ProjectPath 'C:\repos\someproject.csproj'
$version = $projectCollection | Get-ProjectPropertyValue -Group Nuget -Property 'Version'
```
## ** Parameters **
   
   
**-Group&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[VisualStudioKnownGroup]
   
&nbsp;&nbsp;&nbsp;
   
**-Property&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;
   
**-ProjectCollection&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[VisualStudioProjectCollection]
   
&nbsp;&nbsp;&nbsp;The VisualStudioProjectCollection object
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Get-ProjectType
   
> Returns the Project type enum of a csproj project from the Control ProjectGroup
   
## ** Example **
   
```
$projectThatContainsProjectType = Get-ProjectType -Path 'C:\repos\someproject.csproj' -ProjectType Nuget
Write-Host $projectThatContainsProjectType
```
## ** Parameters **
   
   
**-Path&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The Visual Studio Root directory that contains .csproj files
   
**-ProjectType&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[VisualStudioKnownGroup]]
   
&nbsp;&nbsp;&nbsp;The type of project type to find (Nuget, PowerShell etc 
   
**-ProjectCollection&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[VisualStudioProjectCollection]
   
&nbsp;&nbsp;&nbsp;The VisualStudioProjectCollection object
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Set-ProjectPropertyValue
   
> Sets A specific property value in a projectgroup group
   
## ** Example **
   
```
$projectCollection = Get-ProjectProperties -ProjectPath 'C:\repos\someproject.csproj'
$version = $projectCollection | Get-ProjectPropertyValue -Group Nuget -Property 'Version'
$projectCollection | Set-ProjectPropertyValue -Group Nuget -Property 'Version' -Value '1.0.0'
```
## ** Parameters **
   
   
**-ProjectCollection&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[VisualStudioProjectCollection]
   
&nbsp;&nbsp;&nbsp;
   
**-Group&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[VisualStudioKnownGroup]
   
&nbsp;&nbsp;&nbsp;
   
**-Property&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;
   
**-Value&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
# WorkFlow Detail
   
   
   
   
# Get-WorkFlowLogs
   
> Returns the logs from a workflow run. Etiehr to a file or the console.
   
## ** Example **
   
```
$workFlowResult | Get-WorkFlowLogs -ToFile 'C:\temp\workflowlog.txt
```
## ** Parameters **
   
   
**-WorkFlowResponse&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[GitWorkFlowResponse]
   
&nbsp;&nbsp;&nbsp;The GitWorkFlowResponse from the workflow
   
**-WorkFlowId&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[Int64]]
   
&nbsp;&nbsp;&nbsp;The unique id of the workflow run
   
**-WorkFlowName&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the workflow in the remote repository
   
**-TargetPath&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The target (local) directory where the logs will be copied to
   
**-AsString&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;If specified , the logs are amalgamated and written to the current console
   
**-ToFile&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;If specified the amalgamated logs are written to the specified file
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Get-WorkFlows
   
> Returns all the defined workflows for the given repository.
   
## ** Example **
   
```
Get-WorkFlows -Repository 'myrepo'
```
## ** Parameters **
   
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Invoke-WorkFlow
   
> Runs a given workflow and optionally waits until it is complete
> You would typically use this cmdlet with Set-Workflow and Set-WorkFlowInputVariable
> 
> If the workflow has required variables, and they are not set. An error occurs.
   
## ** Example **
   
```
$workflowResult = Set-WorkFlow -Name 'CI' | `
Set-WorkFlowInputVariable -Name 'VAR1' -Value 'value1' | `
Invoke-WorkFlow -Wait
if ($workflowResult.IsSuccess) {
   Write-Host 'It Worked'
} else {
   Write-Host 'It Failed'
}
```
## ** Parameters **
   
   
**-GitFlow&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[GitWorkFlowRequest]
   
&nbsp;&nbsp;&nbsp;The input GitWorkFlowRequest - this is Mandatory
   
**-Wait&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;If specified waits until the workflow is complete. user -Verbose for progress
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Set-WorkFlowInputVariable
   
> Sets the given variable value (or an Environment Value) to the current workflow request and returns the updated (GitWorkFlowRequest) object
   
## ** Example **
   
```
$workflowResult = Set-WorkFlow -Name 'CI' | `
Set-WorkFlowInputVariable -Name 'VAR1' -Value 'value1' | `
Set-WorkFlowInputVariable -Name 'var2' -Value 'value2'
```
## ** Parameters **
   
   
**-GitFlow&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[GitWorkFlowRequest]
   
&nbsp;&nbsp;&nbsp;The current (GitWorkFlowRequest) this is Mandatory
   
**-Name&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the variable to set
   
**-Value&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The [string] value of the variable
   
**-EnvironmentVariable&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The environment variable (user or process) to get the value from
   
**-Optional&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;If specified no check is made for a valid value
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
   
   
# Set-WorkFlow
   
> Initialises a GitWorkFlowRequest object with the variables required from the workflow definition
   
## ** Example **
   
```
$workflowRequest = Set-WorkFlow -Name 'CI'
```
## ** Parameters **
   
   
**-Name&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The mandatory Name of the workflow within the current repository
   
**-Branch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the branch to action
   
**-LocalRepository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The root path to the local repository
   
**-Repository&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[String]
   
&nbsp;&nbsp;&nbsp;The name of the local and remote repository
   
**-Help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**
[SwitchParameter]
   
&nbsp;&nbsp;&nbsp;Display help for this cmdlet
   
---
