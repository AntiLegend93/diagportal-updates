DiagPortal v1.5
===============

A Windows diagnostics program. It runs 85 checks on your PC, turns the results into readable pages with
advice on what to fix, and gives the PC a health score out of 100 for six areas.

  Windows 10 (version 1809 or newer) and Windows 11, 64-bit.
  The scan asks for administrator permission, because many checks cannot run without it.
  Nothing is installed or changed on the PC by a scan. It only reads and measures.
  (The speed and stress tests briefly use the drive, processor, memory and internet connection.)


WHAT IT CHECKS (85 checks)
--------------------------
  Storage ................ drive health and wear (SMART), free space, partitions, drive speed test,
                           large files and space you can clear, backup and restore status
  Performance ............ hardware, memory, startup programs and boot time, boot breakdown,
                           processor stress test (throttling), memory test, graphics, display, sleep blockers
  Security ............... Defender, firewall rules, exposed network services, exploit protections,
                           encryption, certificates, Secure Boot and TPM, administrator accounts,
                           failed sign-ins, browser extension audit, ransomware readiness, hosts file and proxy
  Network ................ interfaces, routes, DNS (cache, servers, benchmark), Wi-Fi quality and signal,
                           packet loss and latency, internet speed test
  Updates and software ... pending updates, update history, update health, installed programs, hotfixes
  Stability and hardware . crash history, repeating error events, reliability signals, device problems,
                           drivers (signed, old, failed), battery wear, BIOS, clock sync

Every check writes its own page. The Executive Summary lists what needs attention, with plain-English
advice and a button that opens the right Windows setting. It also lists the checks that found nothing wrong.

Quick Scan skips the slow checks (system file scan, update search, speed tests, memory test, large file
search, firewall review and account audit). A full scan runs everything.

Privacy: reports stay on the PC. The internet speed test downloads and uploads random test data with
Cloudflare's speed test service (speed.cloudflare.com). Nothing else leaves the PC except the update
check, which reads one small text file from GitHub.


THE TOP BAR
-----------
  One row of buttons, grouped:
    Home, Back  |  Quick Scan, Full Scan (Cancel Scan appears while a scan runs)  |  Results, Share, Tools menus
    ... then Search, the Settings gear and Donate on the right.
  Results menu ... System Summary, Previous Runs, Compare Runs
  Share menu ..... Save as PDF, Copy Summary, Export to Share, Support Bundle, Open Reports Folder
  Tools menu ..... Schedule Scans, Compare PCs, Check for Updates, Discord
  Settings gear .. the Plain English switch


THE WHOLE THING LIVES IN THIS ONE FOLDER
----------------------------------------
  BUILD.bat        double-click to build everything
  GET_HASH.bat     works out the checksum for an update (see RELEASING AN UPDATE)
  app\             the program (C# project): open app\DiagPortal.csproj in Visual Studio to edit
  suite\           the scan engine, ONE copy:
                     Run_Diagnostics.bat   runs the checks in order
                     Build_Reports.ps1     builds the pages, scores and advice
                     version.txt           the version
                     modules\NN.bat        one small file per check (01 to 85)
                     modules\ps\           the PowerShell scripts that checks 65 to 85 use (and a shared helper)
  installer\       the Inno Setup script (optional)
  release\         created by BUILD.bat; this is where the finished files appear


HOW TO BUILD
------------
  1. Put the whole DiagPortal_Clean folder anywhere (for example your Desktop).
  2. Optional: copy your icon to app\Assets\AppIcon.ico (the build works without it).
  3. Double-click BUILD.bat. It needs the .NET 8 SDK.
  4. Open the release folder:
       release\DiagPortal\DiagPortal.exe   test it here
       release\DiagPortal_Share.zip        send this to a friend (no install needed)
       release\DiagPortal_Setup_1.5.exe    only if Inno Setup is installed

The build prints some PRI263 warnings about the .ps1 files. They are harmless.


CHANGING THINGS LATER
---------------------
  Edit a scan check ........ suite\modules\NN.bat, or suite\modules\ps\NN_Name.ps1 for checks 65 to 85
  Edit the report builder .. suite\Build_Reports.ps1 (page list, area scores, advice rules)
  Edit the app ............. files in app\
  Add a check .............. add modules\NN.bat, add a "call :step NN" line in Run_Diagnostics.bat,
                             and add the page to the layout list in Build_Reports.ps1
  Bump the version ......... suite\version.txt and AppVersion in installer\DiagPortal.iss
  Then run BUILD.bat again. Nothing else needs copying anywhere.

Two things to know when writing PowerShell checks: they run elevated under Windows PowerShell 5.1, and
security software looks closely at scripts that touch startup entries, memory or the network. If a check
is blocked, its page now says so and shows the reason.


IF A CHECK SAYS IT COULD NOT COLLECT DATA
-----------------------------------------
  Open that check's page. It shows the reason from Windows PowerShell. The usual causes are:
    - antivirus blocked the script (look in Windows Security > Protection history)
    - the check needs a Windows feature this PC does not have (for example, no Wi-Fi adapter)
    - there is no internet connection (internet speed test)
  The Executive Summary lists every check that could not collect data.


CLEAN UP THE OLD MESS
---------------------
  Once this builds, move your old folders somewhere else (or delete them) so there is only one copy:
  old Run_Diagnostics.bat / modules / Build_Reports.ps1 copies, old publish folders, old .iss and
  Build_Installer.bat, and the old project folder.


BEFORE SENDING A RELEASE TO SOMEONE
-----------------------------------
  The release is built from clean files, so it carries no reports or history of yours. If you zip a folder
  you have run yourself, delete HTML_Reports, Runs, progress.txt, cancel.flag, schedule.txt, settings.txt,
  ignored.txt and crash.log first.


RELEASING AN UPDATE (automatic install)
---------------------------------------
The app checks https://raw.githubusercontent.com/AntiLegend93/diagportal-updates/refs/heads/main/latest.txt
every time it starts. If the version on line 1 is newer than suite\version.txt, it offers to download and install.

1. Change the version in suite\version.txt (for example v1.5) and in installer\DiagPortal.iss (#define AppVersion "1.5").
2. Run BUILD.bat. You get release\DiagPortal_Setup_1.5.exe.
3. On GitHub: repo diagportal-updates > Releases > Draft a new release. Tag v1.5. Attach the .exe. Publish.
4. Double-click GET_HASH.bat. It works out the checksum and copies the three lines for latest.txt to the clipboard.
5. Edit latest.txt on GitHub. Three lines, last one optional:
       v1.5
       https://github.com/AntiLegend93/diagportal-updates/releases/download/v1.5/DiagPortal_Setup_1.5.exe
       <the 64-character checksum>
   Do this LAST, after the release is published. Saving it is what tells everyone's app about the update.

Rules: line 2 must be a direct github.com link ending in .exe, or the app only opens it as a web page.
If line 3 is present and does not match the download, the app discards the file and does not install.
