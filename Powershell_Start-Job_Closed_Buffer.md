# Powershell Start-Job error - closed buffer

```Objects cannot be added to a closed buffer. Make sure the buffer is open for Add and Insert operations to succeed.```

Solved this one after not finding anything else on the web for this particular cmdlet + error.

**Background:**

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

One thing I've learned while implementing `-Credential` parameters into our corporate PowerShell modules is that the Microsoft-provided cmdlets handle credentials in an inconsitent manner. (h/t [@joshduffney](http://duffney.io/AddCredentialsToPowerShellFunctions)).

`Start-Job` (at least in Powershell 4) doesn't seem to handle being passed an empty credential object (`[PsCredential]::Empty`) correctly. The fix for this error was to not pass `Start-Job` the empty credential object:

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

---tjm