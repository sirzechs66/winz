HOW TO REBUILD winz.exe FROM THESE PARTS
========================================

What is in this repository
--------------------------
  parts/winz_001.bin ... parts/winz_029.bin
        Your installer, cut into 29 slices of 5 MiB (last one is smaller).
        They are plain data files, not archives - nothing to unpack.
        29 slices x 5 MiB keeps every upload/download small enough that servers
        do not time out.

  parts/winz.manifest.json
        The index: exact size and SHA-256 of every slice, plus the SHA-256 of
        the whole file. Required by the rebuild script.

  parts/SHA256SUMS.txt
        One line per slice (sha256  filename) - handy to check a download
        without running the script.

  join.ps1.txt
        The rebuild script, stored as .txt because browsers, mail scanners and
        antivirus filters routinely block *.ps1 downloads. Byte-identical copy.


Step 1 - get the files
----------------------
Easiest: green "Code" button on the repository page -> "Download ZIP", then
unzip it anywhere, for example D:\winz.

Or download the files one by one into a single folder while keeping the
"parts" subfolder:
  D:\winz\
      join.ps1.txt
      parts\
          winz.manifest.json
          winz_001.bin ... winz_029.bin

Nothing needs to be renamed except the script.


Step 2 - turn join.ps1.txt into join.ps1
----------------------------------------
Rename the file and delete the trailing ".txt":

      join.ps1.txt   ->   join.ps1

In Explorer: right-click -> Rename. If Windows hides the extension, first turn
on View -> Show -> File name extensions, otherwise you would end up with
"join.ps1.txt.txt".

Same bytes, only the name changes. This is all the "conversion" there is - a
PowerShell script is a plain text file.


Step 3 - join the parts
-----------------------
Open a terminal in the folder that contains join.ps1 and the parts subfolder,
then run:

      powershell -NoProfile -ExecutionPolicy Bypass -File .\join.ps1 .\parts

How to open that terminal: click the address bar of the Explorer window, type
"powershell" and press Enter.

What the script does:

  1. reads parts\winz.manifest.json,
  2. checks that every slice exists, has the recorded size and matches its
     SHA-256 - a damaged or missing slice is named, so only that file needs to
     be downloaded again,
  3. writes the result under a temporary name, verifies the SHA-256 of the
     complete file, and only then renames it.

Result:

      D:\winz\parts\winz.exe       143.84 MiB (150,822,912 bytes)
      SHA-256  3556fff271ddab3506e4bce05f8d6de44e8616929bd68644eb1cbae904c96d51

winz.exe is byte-identical to the original "Wingspan-3.0.6 Setup.exe": same
icon, same resources, only the file name differs. Check it yourself with:

      certutil -hashfile parts\winz.exe SHA256

Because it is an unsigned installer, Windows SmartScreen may show the usual
"unknown publisher" prompt when you run it. That is normal and unrelated to
the splitting.


Options
-------
  -OutputPath <file>   write the result somewhere else
  -VerifyOnly          only check the slices, do not rebuild
  -Force               overwrite an existing winz.exe


If something goes wrong
-----------------------
"running scripts is disabled on this system"
    Use the full command from Step 3 - "-ExecutionPolicy Bypass" applies to
    that single run only, needs no admin rights and changes no setting.

"Repair needed - missing / size / checksum"
    Download the named file(s) again and re-run. Nothing else has to be redone.

"No '*.manifest.json' found"
    The script was pointed at the wrong folder. Pass the folder that contains
    winz.manifest.json, e.g. .\parts

"No parts folder found"
    Use the folder that really holds the .bin files:
    powershell -NoProfile -ExecutionPolicy Bypass -File .\join.ps1 "D:\winz\parts"
