# Create Symbolic Link

Open PowerShell as Administrator, then run

```powershell
New-Item -ItemType SymbolicLink `
-Path "C:\Program Files\WindowsPowerShell\Modules\CustomHelperUtils" `
-Target "C:\path\to\gitrepos\utils\powershell\CustomHelperUtils"
```

Output should be:

```powershell
  Verzeichnis: C:\Program Files\WindowsPowerShell\Modules


Mode          LastWriteTime         Length Name
----          -------------         ------ ----
d----l        04.05.2022 07:14      CustomHelperUtils
```
