# Ps.Utilities Help (This is NOT complete - i'm working on it!)

# Overview 
PS.Utilies is a c# .NET standard 2.0 set of powershell cmdlets that wrap common (windows azure) **developer** utilities' Currently it supports **GIT** and **AZURE Devops**

# Installation
```
Install-Module PS.Utilities
```
&nbsp;  

---
### Set-DevopsCredentials
Sets session-wide credentials for both git and devops 

**Example** 
```
$patToken = "some-plain-text-Path-Token"
Set-DevopsCredentials -PlaintextPassword $patToken -Organisation "your-devops-organisation" 
```

<details>
   <summary>Parameters</summary>

   | Parameter | Description |  
   | --- | --- |
   | -Username  |  Username to use to authenticate with git / azure | 
   | -PlainTextPassword | the PAT token to use to authenticate with remptes devops | 
   | -Organisation | the devops organisation name (i.e https://devop://dev.azure.com/<em>**organisation**<em>/blah) |  

</details>

&nbsp;

**Returns**
Nothing

---

&nbsp;  

### Install-Git
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

### Copy-Repository
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



