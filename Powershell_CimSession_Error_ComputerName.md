# Get Remote Computer Name when catching errors with Get-CimInstance, Start-ScheduledTask, etc.

**$errorRecord.Exception.ErrorData.CimSystemProperties.ServerName**

```
Try { Start-ScheduledTask -CimSession (Get-CimSession) -TaskName 'Example_Task' -ErrorAction Stop }
Catch [Microsoft.Management.Infrastructure.CimException] {
  $remotename = $PsItem.Exception.ErrorData.CimSystemProperties.ServerName
}
```
