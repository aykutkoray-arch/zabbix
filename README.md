**Generic Windows Process Monitoring Template - Simple Version**


REQUIREMENTS - Add to zabbix_agent2.conf:
-------------------------------------------------------
# Process counter - simple version (exe name or full path only)
UserParameter=proc.count.ex[*],powershell.exe -NoProfile -Command "try { $$n='$1'; $$p='$2'.Replace('/','\'); $$procs = Get-Process -Name $$n.Replace('.exe','') -EA Stop; if ($$p) { $$procs = $$procs | Where-Object { $$_.Path -like $$p } }; if ($$procs) { if ($$procs.Count) { $$procs.Count } else { 1 } } else { 0 } } catch { 0 }"

# Discovery - echo macro value
UserParameter=proc.discover.list[*],powershell.exe -NoProfile -Command "Write-Output '$1'"

Then restart agent: Restart-Service "Zabbix Agent 2"

IMPORTANT NOTES:
-------------------------------------------------------
1. Always use forward slash (/) in paths, NOT backslash (\)
2. Case-insensitive: c:/windows = C:/WINDOWS = c:/Windows
3. Each comma-separated entry creates a separate item
4. This is a simplified version - no wildcards, no command line filtering

MACRO FORMAT {$EXE_LIST}:
-------------------------------------------------------
Comma-separated entries (NO SPACES, NO PIPES!)

Formats:
1. exe_name      → monitors all instances (any path)
2. full_path     → monitors specific path only

IMPORTANT: Use COMMA (,) to separate entries, NOT pipe (|) or semicolon (;)

========================================================================
USAGE EXAMPLES:
========================================================================

Example 1: Monitor all notepad.exe processes (any path)
-------------------------------------------------------
{$EXE_LIST} = notepad.exe

System has:
- c:\windows\system32\notepad.exe (1 instance)
- c:\windows\notepad.exe (1 instance)
- c:\temp\notepad.exe (1 instance)

Result: 1 item created
- Process count: notepad.exe → 3


Example 2: Monitor specific path only
-------------------------------------------------------
{$EXE_LIST} = c:/windows/system32/notepad.exe

System has:
- c:\windows\system32\notepad.exe (2 instances)
- c:\temp\notepad.exe (1 instance)

Result: 1 item created
- Process count: c:/windows/system32/notepad.exe → 2


Example 3: Monitor multiple processes (separate items)
-------------------------------------------------------
{$EXE_LIST} = notepad.exe,calc.exe,mspaint.exe

Result: 3 items created
- Process count: notepad.exe → (count of all notepad)
- Process count: calc.exe → (count of all calc)
- Process count: mspaint.exe → (count of all mspaint)


Example 4: Monitor multiple specific paths
-------------------------------------------------------
{$EXE_LIST} = c:/windows/system32/notepad.exe,c:/windows/notepad.exe

System has:
- c:\windows\system32\notepad.exe (1 instance)
- c:\windows\notepad.exe (1 instance)
- c:\temp\notepad.exe (1 instance)

Result: 2 items created
- Process count: c:/windows/system32/notepad.exe → 1
- Process count: c:/windows/notepad.exe → 1


Example 5: Case-insensitive paths
-------------------------------------------------------
{$EXE_LIST} = c:/WINDOWS/System32/NOTEPAD.EXE

System has:
- C:\Windows\system32\notepad.exe (1 instance)

Result: 1 item created
- Process count: c:/WINDOWS/System32/NOTEPAD.EXE → 1
(Works because -like operator is case-insensitive)


Example 6: Mix of exe names and paths
-------------------------------------------------------
{$EXE_LIST} = notepad.exe,c:/windows/system32/calc.exe,mspaint.exe

Result: 3 items created
- Process count: notepad.exe → (all notepad, any path)
- Process count: c:/windows/system32/calc.exe → (only this specific path)
- Process count: mspaint.exe → (all mspaint, any path)


Example 7: Java monitoring (version-independent)
-------------------------------------------------------
{$EXE_LIST} = java.exe

System has:
- c:\program files\java\jdk1.8\bin\java.exe (1 instance)
- c:\program files\java\jdk11\bin\java.exe (1 instance)

Result: 1 item created
- Process count: java.exe → 2
(Monitors all Java processes regardless of version/path)


Example 8: Application in custom folder
-------------------------------------------------------
{$EXE_LIST} = c:/app/myservice.exe

System has:
- c:\app\myservice.exe (1 instance)

Result: 1 item created
- Process count: c:/app/myservice.exe → 1


========================================================================
TROUBLESHOOTING:
========================================================================

Test UserParameter manually:

# Test 1: All instances
zabbix_agent2.exe -t "proc.count.ex[notepad.exe,]"

# Test 2: Specific path (use forward slash)
zabbix_agent2.exe -t "proc.count.ex[notepad.exe,c:/windows/system32/notepad.exe]"

# Test 3: Case-insensitive
zabbix_agent2.exe -t "proc.count.ex[NOTEPAD.EXE,C:/WINDOWS/SYSTEM32/NOTEPAD.EXE]"

Common issues:
1. Backslash (\) not allowed → Use forward slash (/)
2. Case sensitivity? → No, it's case-insensitive
3. Process not found → Check exact exe name: Get-Process | Select Name
4. Count is 0 but process running → Check path is correct

Check if process is running:
Get-Process -Name notepad -ErrorAction SilentlyContinue | Select Name, Path

========================================================================
