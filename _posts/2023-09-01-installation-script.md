---
layout: post
title:  "A simple script for running executables"
categories: [tools]
tags: [windows,shell-scripting,powershell]
---



# A installation script

After every new installation on a new computer, we need to install a specific set of apps which are located on a designated network path, _which is already mapped before execution._

## Preparing the environment

Before we start the script, we need to set the correct permission:
1. Open PowerShell as Admin
2. input:
	Set-ExecutionPolicy RemoteSigned
3. Then execute the script

## Setting the location and variables

As we can see bellow we set the location of the mapped network path, and also set some variables as the path to some _.exe_

```powershell
Set-Location \\192.168.1.1\Trabalho\Publica\TI
$local_instaladores = "T:\Publica\TI\"
$local_antivirus = $local_instaladores+"Nova pasta\Antivirus\"
$local_openvpn = $local_instaladores+"OVPN\"
```

## The function

A simple function that has 3 parameters:

The local
: where the executable is located, normally we use the variable above

The executable
: the name of the executable itself, _something.exe_

The program name
: Just the name that will show in the prompt messages

```powershell
function Install-Exe([string]$local,[string]$executavel, [string]$programa) {
    Write-Host "[INSTALANDO] $programa" -BackgroundColor Black -ForegroundColor White
    
    Start-Process -Wait -FilePath $local$executavel -ArgumentList "/install" -PassThru
    #& $local$executavel /install
    if ($?) {
        Write-Host "[Instalado com sucesso]" -BackgroundColor Green -ForegroundColor Black
    } else {
        Write-Host "[Falha na instalacao]" -BackgroundColor Red -ForegroundColor Black
    }
}
```
### Step by Step

1. It writes on the prompt which program will install

2. Start the process of installation

3. Writes on prompt the exit: success or failure

## Calling the function

This is the correct way to use parameters within powershell, i didnt know.

```powershell
Install-Exe $local_instaladores ChromeSetup.exe "Google Chrome"
...
```

## Requirements
1. correct path to executables
2. powershell knowledge


