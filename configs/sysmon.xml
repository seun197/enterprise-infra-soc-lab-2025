Invoke-WebRequest -Uri "https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml" -OutFile "C:\Tools\sysmonconfig-export.xml"
Rename-Item -Path "C:\Tools\sysmonconfig-export.xml" -NewName "sysmon.xml"

cd C:\Tools
.\Sysmon64.exe -accepteula -i sysmon.xml
