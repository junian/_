<p align="center"><img align="center" src="./img/hippos-mascot.png" /></p>

<h1 align="center">Reverse Engineering Case Study: HipPOS Hidden API</h1>

<p align="center">How to discover HipPOS app hidden API by peeking at the source code.</p>

<p align="center"><img alt="GitHub language count" src="https://img.shields.io/github/languages/count/junian/junian.github.io" /></p>

## Background

My client needed to find the hidden API within the HipPOS desktop app to use for their own web app. Here's how I uncovered it.

## Tools

- [de4dot](https://github.com/de4dot/de4dot): .NET de-obfuscator and unpacker
- [ILSpy](https://github.com/icsharpcode/ILSpy): .NET decompiler
- [Visual Studio Code](https://code.visualstudio.com)
- [Visual Studio](https://visualstudio.microsoft.com)

## Step 1: Deobfuscate the Binaries

First, I detected if the binaries were protected by an obfuscator.

<!--
![Deobfuscate](./img/step-01-de4dot-deobfuscate.gif)
-->

Using de4dot, I identified that the binaries were protected by .NET Reactor. I then cleaned the binaries.

```shell
$ de4dot.exe  --one-file -r .\HipPOS\ -ru -ro .\HipPOS-clean\

de4dot v3.1.41592.3405

Detected .NET Reactor (C:\HipPOS\CorePOS.Business.dll)
Detected .NET Reactor (C:\HipPOS\CorePOS.Data.dll)
Detected .NET Reactor (C:\HipPOS\Hippos-Sync.exe)
Detected .NET Reactor (C:\HipPOS\HipPOS-Updater.exe)
Detected .NET Reactor (C:\HipPOS\HipPOS.exe)
Cleaning C:\HipPOS\CorePOS.Business.dll
Cleaning C:\HipPOS\CorePOS.Data.dll
Cleaning C:\HipPOS\Hippos-Sync.exe
Cleaning C:\HipPOS\HipPOS-Updater.exe
Cleaning C:\HipPOS\HipPOS.exe
Renaming all obfuscated symbols
Saving C:\HipPOS-clean\CorePOS.Business.dll
Saving C:\HipPOS-clean\CorePOS.Data.dll
Saving C:\HipPOS-clean\Hippos-Sync.exe
Saving C:\HipPOS-clean\HipPOS-Updater.exe
Saving C:\HipPOS-clean\HipPOS.exe
```

## Step 2: Decompile the Binaries and Retrieve the C# Source Code

Next, I copied all `dll` libraries from `C:\HipPOS\` to `C:\HipPOS-clean\`, ensuring not to replace the deobfuscated binaries.

<!--
![Copy Dependencies](./img/step-02-1-copy-dependencies.gif)
-->

I opened ILSpy, selected the necessary files, and saved the source code.

- CorePOS.Business.dll
- CorePOS.Data.dll
- Hippos-Sync.exe
- HipPOS-Updater.exe
- HipPOS.exe

<!--
![ILSpy Select Files](./img/step-02-2-ilspy-select-files.gif)
-->

For each binary, I right-clicked and selected **Save Code**.

<!--
![ILSpy Save Source Code](./img/step-02-3-ilspy-save-source-code.gif)
-->

I saved the code to `C:\HipPOS-src\`.

## Step 3: Find Decryption Method and Hidden REST API

I needed to identify how to decrypt a configuration file containing elements like an SQL Server connection string. To do this, I located the functions that interact with the .config file and examined the code to find the decryption method.

For discovering hidden REST APIs, I used Visual Studio Code to search for hipposhq.com within the project files and identified the classes and functions that make REST API calls.

## Step 4: Build a Simple WinForms App to Test HipPOS REST API

I built a simple desktop app to verify if I used the correct functions.

It was a basic GUI with functions copied from the decompiled code from Steps 2 and 3.

<!--
![Tools for HipPOS Screenshot](./img/tools-for-hippos-screenshot.png)
-->

## Step 5: Write Python Script for Web App Integration with HipPOS REST API

Once the REST API was verified, I wrote a Python script to integrate with the client's web app.

---

Looking to do something similar?
Let's discuss on [Upwork](https://www.junian.dev/upwork/).
