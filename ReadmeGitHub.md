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

## Using PS.Selenium 
```
   Install-Module PS.Git
   Import-Module PS.Git 
```   

## Cmdlet Summary
   | Cmdlet | Description | 
   | --- | --- | 
   | [Get-GitToken](#get-gittoken) | Sets the git token based on a script function. This is a hookfor you to implement your own token processing (Oauth , internal token processing).     
   | [New-GitConnection](#new-gitconnection) | This __Must__ be the first cmdlet called. (apart from __Get-GitToken__) it initiates a git connection (both local and remote) and is used by __ALL__ other cmdlets  
   | [Set-GitConnection](#set-gitconnection) | Sets any of the parameters for a git connection (see __New-GitConnection__) for parameter list 
   | [Get-Repositories](#get-repositories) | Retrieves a list of repositories for the given user or organisation. With an optional Script filter to return a filtered list 
   | [Invoke-Clone](#invoke-clone) | Clones a remote repository locally parameter list 
   | [New-Branch](#new-branch) | Creates (or checks out) a local branch   
   | [Get-Branches](#get-branches) | Retrieves a list of local branches with an optional Script filter      
   | [New-SemanticTag](#new-semantictag) | Creates a semantic version tag (using standard notation vx.x.x)      
   | [Invoke-Commit](#invoke-commit) | Commits local changes (optionally with a tag)    
   | [Invoke-Push](#invoke-push) | Pushes the current local repo changes to the remote      
   | [Invoke-Pull](#invoke-pull) | Pulls the latest of a repo/branch      
   | [New-PullRequest](#new-pullrequest) | Creates a new pull request for the current remote repository  
   | [Get-PullRequests](#get-pullrequests) | Retrieves open or all pull remote requests      
   | [Merge-PullRequest](#merge-pullrequest) | Merges a previously created pull request into the target branch (usually origin)  
   | [New-Release](#new-release) | Creates a new GitHub release 
   | [Get-Release](#get-release) | Returns a release object (GitReleaseItem) by tagname , name or latest release  
   | [Get-ReleaseDownloads](#get-releasedownloads) | Downloads , to the specified path , all of the release assets.     

&nbsp;

# Get-GitToken 

Entry point for creating a git token. if the environment variable __GitToken__ 
is set for the user , it will just return that. 
Otheriwse it will execute the script defined by the __-Script__ parameter. 
The script defined __Must__ return a single string that is a valid git token. 
That token will then be stored in the user __GitToken__ user envrionment variable 

```
    $gitToken = Get-GitToken -Script {
      return my_way_of_getting_a_token
      # do something that returns a valid token
    }
```

### Parameters

__-Script__

A scriptBlock that returns a [string] representation of a valid GitHub Token.

&nbsp;

# New-GitConnection 

Used by all other cmdlets and must be the first cmdlet called in a PS session. 
before another direct git cmdlet is called. 

The connection object it creates is stored in a memorycache object 
and is available to all cmdlets within the PS session. 
It can be altered by other cmdlets during the session or altered directly 
using the __Set-GitConnection__ cmdlet.    

The minium parameters you should set are __-Origranisation__ or __-User__.
If the user environment __GitToken__ is set that will be used. 
If the target repository requires apat token you can set it using the __-Token__ 
parameter , or you can provide a script block that will return the token based on 
the request being made (__-TokenScript__)   

```
 New-GitConnection -Organisation "myOrganisation" `
            -Repository "myReo" `
            -CredetialUser "myUser" ` 
            -Token "asdasd" ` 
            -TestConnection`
            ...
            ....

 # using a script to set the token 
 New-GitConnection -Organisation "blah org" -TestConnection -TokenScript {
    param(
        [GitHub.Service.Services.Common.GitRequest]$Request
    )
    if ($Request.Organisation -eq "blah org") {
      return "asdasdasd"
    }
    return $Request.Token
}           
```

### Parameters

__ALL__ parameters are optional!

__-Organisation__

The name of the github organisation. If you don't specify this , you must specify the __-User__

__-User__

The name of the user's repo. 

__-Repository__

Then name of the GitHub (user or organisation) remote repository name. 

__-CredentialUser__

The name of the user to use to authenticate. This can be any name when using a token. (which is currently the only mechanism supported!)

__-CredentialPassword__

Not currently used - it will set the __-Token__ parameter if specified 

__-Token__

if specified , the plain-text token is used to authenticate. if __Not__ specified , the __user__ environment variable __GitToken__ will be used to populate the token.

__-LocalRepository__

The root (directory) path to the local repository. this is combined with the __-Repository__ parameter to create the actual full path of the local repository 

__-Branch__

The local branch to create/use. this is set automatally by other cmdlets (i.e __Set-Branch__)  

__DefaultBranch__

The defalt (origin) branch (e.g  master/main etc)

__Email__

Any valid email address to be used for commits and tags.

__-TesConnection__ (switch)

Tests the organisation/user remote repository to see if it is valid (New-GitConnection only)

__-TokenScript__

Inline or function script that returns a valid token. The script is passed the current (GitRequest) object which it can interogate

&nbsp;

# Set-GitConnection

Sets individual or multiple properties of the git connection 

```
   Set-GitConnection -Token "xxx" 
```

### Parameters

All parameters are the same as __New-GitConnection__

&nbsp;

# Get-Repositories 

Returns a list of repositories for the given user or organisation.
With an optional __-Script__ parameter that is a user-defined powershell script 
that can filter the returned object (GitRepositoryListCollection)

```
Get-Repositories -Script {
    param(
        [GitHub.Service.Models.GitRepositoryListCollection]$Repositories
    )
    return $Repositories | Where-Object { $_.Name -match "ps." }
}
```

### Parameters

__-Script__

A user defined powershell script that returns a filtered list of repositories 
(or anything else).  

&nbsp;

# Invoke-Clone 

Clones a remote repository. The __-Path__ and __-Repository__ parameters will automatically populate the Git-Connection properties for use by all other cmdlets.  

```
Invoke-Clone -Path "C:\Temp" -Repository "Test-Repo" 

# Clone to C:\Temp\Test-Repo

```

### Parameters

__-Path__

The __Root__ directory where the cloned repo will be written to. 

__-Repository__

the __Remote__ repository name

&nbsp;

# New-Branch 

Creates a new local branch on the local repostiory. if the branch exists it will be checked out.   

```
New-Branch -Branch "Test-David" 

# the local repository can be set with -Path and -Repository

```

### Parameters

__-Branch__ (required)

The name of the (local) branch to create. If the branch already exists it is checked out.

__-Path__ (optional)

The __Root__ directory where the cloned repo will be written to. 

__-Repository__ (optional)

the __Remote__ repository name

&nbsp;

# Get-Branches

Returns a (LocalBranchCollection) object containing a list of local branches.
you can optionally define an inline script that will take the branch collection 
and return a filtered list of branches (or whatever you want)

```
   $branchFiltered = Get-Branches -Script {
      param(
         [LocalBranchCollection]$Branches
      )
      # do something with the branches 
      return $Branches.Selesct(x => x.contains("fnafna").ToList()) 
   }
```

### Parameters

__-Path__ (optional)

The __Root__ directory where the cloned repo will be written to. 

__-Repository__ (optional)

the __Local__ repository name

__Script__ (optonal)

A script that accepts a (LocalBranchCollection) collection and returns 
a Filtered list of branches (or anything you want)



# Get-Tags

Gets a list of tags from the local repository. It returns a GitTagCollection object. t can be piped into New-Tag

```
   $tags = Get-Tags 
   # if not set use -Path and -Repository to set the local repository path
```


### Parameters

__-Path__ (optional)

The __Root__ directory where the cloned repo will be written to. 

__-Repository__ (optional)

the __Remote__ repository name

&nbsp;

# New-SemanticTag

Creates a new semantic tag that conforms to the GITHUB release version Vxx.xx.xx (Major , Minor , Patch) 

To create the 'Next' release , you can pipe the results of the Get-Tags cmdlet into this cmdlet 

```
   # initialise a new tag  

   $newTag = New-Tag -Minor 1
   # creates v1.1.0

   # get last tag and increment the latest patch version 
   $newTag = Get-Tags | New-SemanticTag -TagIncrment Patch
   #creates Vx.x.(x+1) or if no tags then v1.0.0 
```


### Parameters

__-Tags__ (can be piped)

A GitTagCollection object that contains the tags from the current local repository

__-TagIncrement__ 

enum that describes the semantic version portion to increment (Major, Minor, Patch)

__-Major__

int major portion to increment 

__-Minor__

int minor portion to increment

__-Patch__

int patch portion to increment

# Invoke-Commit
Creates a local commit. With any changes made to the local repository. you can optionally create a tag to be associated with the commit.

```
    Invoke-Commit -Tag $newTag -Message "Test commit one with tag" 
```


### Parameters

__-Tag__ 

A tag name (string) you want associated with the commit.  

__-Message__ 

The commit description. If the tag is used the same description is used for the commit and tag

__-Path__ (optional)

The __Root__ directory where the cloned repo will be written to. 

__-Repository__ (optional)

the __Remote__ repository name


# Invoke-Push
Pushes any commits from the local repository to the remote repository.

```
    Invoke-Push # optional -Path xx -Repository yy 
```


### Parameters

__-Path__ (optional)

The __Root__ directory where the cloned repo will be written to. 

__-Repository__ (optional)

the __Remote__ repository name

&nbsp 

# Invoke-Pull

invokes a pull against the current or specified repository and branch.
If there are otstanding commits an error will be thrown.

```
Invoke-Pull -Branch "SomeBranch"
```

### Parameters 

__Repository__ (optional) 

The local repository name 

__Path__ (optional)

The local root directory of the repository 

__Branch__ (optional)

The branch to pull against (it will be checkout first)

&nbsp;

# New-PullRequest
Creates a Remote pull request for the current repostiory. You can optinally specify the __-Repostiory__ and __-Branch__. If __-Branch__ is not specified , the default branch (origin) is used as the target of the subsequent merge.  

```
    # optional -Repository and -Branch
    New-PullRequest -Title "title" -Description "description"
```

### Parameters

__-Title__

The title of the pull request 


__-Description__ 

The Description of the pull request 

__-Branch__ (optional)

The target branch of the pull (merge) request. 

__-Repository__ (optional)

the __Remote__ repository name

&nbsp;

# Get-PullRequests

Returns a list of all pull requests for the given remote repository or the first 'open' pull request 

```
   # optional -Repostiory parameter to set remote repository name
   
   # get the first (and only) open pull request (GitPullResponse)
   $openPullRequest = Get-PullRequests -Open 

   # or 

   # get all pull requests (PullRequestCollection)
   $allRequests = Get-PullRequests 
```

### Parameters

__-Open__ (optional)

Returns a GitPullResponse object of the first open pull request 

__-Repository__ (optional)

The target remote repository 

&nbsp;

# Merge-PullRequest

Merges a pull request into the origin branch. It must have been authorised (if neccessary).

```
# to leave the remote and local branches intact after the merge 
#  use the -LeaveRemoteBranch swith parameter

$latestPullRequest = Get-PullRequests -Open 

$latestPullRequest | Merge-PullRequest -Comment "merged ok" 

# Or 

$mergeResult = Merge-PullRequest -PullRequest $latestPullRequest -Comment "merged ok"

```

### Parameters

__-PullRequest__ 

The GitPullResponse (can be Piped) 

__-Repository__

Source remote repository name

&nbsp;

# New-Release 

Creates a new github release from the a) latest tag b) specified tag c) a new tag

You can add assets directly to the release either individually or pass a directory and that directory will be zipped and added to the release.

There are also options to create a draft, pre-release and avoid making it the default latest release 

```
New-Release -ReleaseName "First release for powershell" `
             -Description "Release to see if powershell works" `
             -AssetDirectory "C:\Deleteme\PS.Selenium" `
             -AssetZipName "Selenium" `
             -UseLatestTag  
```

### Parameters 

__-ReleaseName__

Name to give the release

__-Description__

Description of the release

__AssetDirectory__

Path to the directory that contains a list of files/directories to be zipped and added to the release assets

__AssetZipName__

The name of the zip file that will be added to the release assets

__UseLatestTag__ (switch)

Tells the release to get the latest tag in the repository and use that as the release associated tag

__-Tag__

The GitTag instance to use as the tag 

__-NewTag__ (string)

A new Tag to asstiate with the release 

__-Draft__ (switch)

Does not publish the release , but tags it as a draft release

__-PreRelease__ (switch)

Specified that this release includes preRelease code 

__-MakeLatest__ (bool)

if you do not want this release to be the latest release set this to $false

__-Assets__ (string[])

Enumerable list of asset files to add to the release. These files will __NOT__ be zipped , but added to the release assets as individual assets 

&nbsp;

# Get-Release 

Returns (GitReleaseItem) object of a release. it can return the object based on it's a) name b) tag or the latest release (__-Latest__)

if no parameters are set (apart from __-Repository__) then a GitReleaseCollection object is returned

```
$latestRelease = Get-Release -latest
```

### Parameters

__-Latest__ (Switch)

Gets the 'latest' release 

__-Tag__ (optional)

Gets the release that has the 'latest' tag.

__-Name__ (optional)

Gets the release named -Name

__-Repository__ (optional)

Source remote repository name

# Get-ReleaseDownloads

Downloads __ALL__ the assets associated with a release to the specified local path. By default any zip files will be extracted.

it returns a GitDownloadReleaseResponse object , that contains an array of FileInfo objects for each file downloaded (and extracted) 

```
$release = Get-Release -Latest -Repository "myRepo"

$downloadedFiles = Get-ReleaseDownloads -Release $release -Path "C:\Temp"  
```

### Parameters 

__-Release__ (GitReleaseItem)

The target release to download (GitReleaseItem)

__-Path__ (string) 

The full directory path of the target. If the path exists it will be deleted and re-created. If it does not exist , it will be created 

__-ReleaseId__ (int)

The id of the release 

__-Latest__ (switch)

Downloads the 'latest' release from the repository 

__ExtractZip__ (bool) 

Set to $false is you don't want any zip files to be extracted
