In ms exchange IMAP successful or fail logins with IP source are not fixed in the events log.
How to stop IMAP brute force on MS exchange 2013+(may be work on 2010) servers:
1) IMAP Logs on 
"Set-ImapSettings -Server "SERVERNAME" -ProtocolLogEnabled $true"
2) change logs size to 20mb
"Set-PopSettings -Server "SERVERNAME" -LogPerFileSizeQuota 20000000"
3) Restart IMAP
"Stop-service msExchangeIMAP4"
"Start-service msExchangeIMAP4"
4)check settings and remember logs path (on 2013 it c:\Program Files\Microsoft\Exchange Server\V15\Logging\Imap4\)
"Get-ImapSettings | format-list"
5)change in wail2ban_config.ini 
[Application]
18456=MSSQL Logins
to 
[Application]
1035
6)
run script that will check the logs and create events:

"$DebugPreference = "continue"
$path = "c:\Program Files\Microsoft\Exchange Server\V15\Logging\Imap4\"

do{ 

$file = Get-ChildItem -Path $path | Sort-Object LastWriteTime | Select-Object -Last 1
$string = Select-String "AuthFailed:" "c:\Program Files\Microsoft\Exchange Server\V15\Logging\Imap4\$file" | Select-Object -ExpandProperty line | select -last 1 
if (!$string) {Start-Sleep -Milliseconds 2000}
if (!$string) {continue}
$old = $string.Split(",")
Start-Sleep -Milliseconds 200

$string2 = Select-String "AuthFailed:" "c:\Program Files\Microsoft\Exchange Server\V15\Logging\Imap4\$file" | Select-Object -ExpandProperty line | select -last 1 
$new = $string2.Split(",")



if ($old[1] -ne $new[1]) { Write-EventLog –LogName Application –Source “wail2ban” –EntryType Warning –EventID 1035 –Message "$($new[0]), $($new[1]), $($new[4]), $($new[5]), $($new[11])."
 Write-Debug "$($new[0]), $($new[1]), $($new[4]), $($new[5]), $($new[11])."}


else {Start-Sleep -Milliseconds 200}
}
while ($true)"
