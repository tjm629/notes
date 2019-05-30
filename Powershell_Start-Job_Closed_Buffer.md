# Powershell Start-Job error - closed buffer

Error message:

```Objects cannot be added to a closed buffer. Make sure the buffer is open for Add and Insert operations to succeed.```

I was working on a script that would pass the `PSRemotingJob`(s) on to another cmdlet.

The code originally was something like

```posh
$job = @{
    Name        = "Contoso_{0}_{1}" -f $Result.OuName, (Get-Date -Format "yyyyMMddhhmm")
    InputObject = $Result
    ScriptBlock = {
        Import-Module "Contoso.Foobar"
        $Input | ForEach-Object {
            $PsItem.Property | Do-SomethingInteresting -Credential $PsItem.Credential
        }
    }
}
$Jobs = Start-Job @job
```

But then I had a need to add additional actions to the ScriptBlock. I figured using the `-Credential` parameter a bunch of times wasn't the right move, so added the Credential to the Job, instead of from within it:

```posh
$job = @{
    Name        = "Contoso_{0}_{1}" -f $Result.OuName, (Get-Date -Format "yyyyMMddhhmm")
    Credential  = $Result.Credential
...
```

That's when I started to get the above error (`Objects cannot be added to a closed buffer...`).

I've learned that the Microsoft-provided cmdlets handle credentials in an inconsitent manner while implementing `-Credential` parameters into our corporate PowerShell modules.<sup>[1]</sup>

`Start-Job` in Powershell 4<sup>[2]</sup> doesn't seem to handle being passed an empty credential object (`[PsCredential]::Empty`) correctly. The fix for this error was to not pass `Start-Job` the empty credential object:

```posh
$job = @{
    Name        = "Contoso_{0}_{1}" -f $Result.OuName, (Get-Date -Format "yyyyMMddhhmm")
...
}

If ($Result.Credential -ne [PsCredential]::Empty) {
    $job["Credential"] = $Result.Credential
}

$Jobs = Start-Job @job
```

*<sup>[1]</sup> [@duffney](https://github.com/duffney) has a very helpful post '[How to Add Credential Parameters to PowerShell Functions](http://duffney.io/AddCredentialsToPowerShellFunctions)'.*

*<sup>[2]</sup> A quick test on my workstation shows that this issue is resolved in PowerShell 5.*

---tjm