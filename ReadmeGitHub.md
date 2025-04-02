# PS.Git
PS.Git is a set of C# cmdlets that use the Lib2GitSharp library and Github API (native) to provide a single set of cmdlets to manipulate both local and remote gitbug repositories.

it does __NOT__ rely on external Git ot Github cli's.

Currently it only supports GitHub PAT tokens to authenticate actions that require authentication.

It also only supports Powershell 7.50 (and above) and a windows X64 client.

## Using PS.Selenium 
```
   Install-Module PS.Git
   Import-Module PS.Git 
```   

## Cmdlet Summary
   | Cmdlet | Description | 
   | --- | --- | 
   | [New-GitConnection](#new-gitconnection) | This __Must__ be the first cmdlet called. it initiates a git connection (both local and remote) and is used by __ALL__ other cmdlets  
   | [Set-GitConnection](#set-gitconnection) | Sets any of the parameters for a git connection (see __New-GitConnection__) for parameter list 
   | [Invoke-Clone](#invoke-clone) | Clones a remote repository locally parameter list 
   | [New-Branch](#new-branch) | Creates (or checks out) a local branch   
   | [Get-Tags](#get-tags) | Retrieves a collection of tags from the local repository      
   | [New-SemanticTag](#new-tag) | Creates a semantic version tag (using standard notation vx.x.x)      

&nbsp;

# New-GitConnection 

Used by all other cmdlets and mst be the first cmdlet called in a PS session. The connection object it creates is stored in a memorycache object and is available to all cmdlets within the PS session. It can be altered by other cmdlets during the session or altered directly using the __Set-GitConnection__ cmdlet.    

The minium parameters you should set are __-Origranisation__ or __-User__ and __-Token__ if the environment variable  __GitToken__ is not set  

```
 New-GitConnection -Organisation "myOrganisation" `
            -Repository "myReo" `
            -CredetialUser "myUser" ` 
            -Token "asdasd" ` 
            -TestConnection`
            ...
            ....
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

# New-Tag

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

