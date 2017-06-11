---
ID: 69
title: >
  CS0013 or CS0016 Compilation Errors in
  ASP.NET Web Applications
author: 唐睿
date: 2006-05-11 13:42:02 +0800
categories: [技术]
tags: [编程, .net, C#, ASP.net, IIS, Web]
layout: post
published: true
---

### SYMPTOMS

When you view a Microsoft ASP.NET Application in a Web browser, you may receive the following error messages:

For the Microsoft .NET Framework version 1.1, the error message is the following:

> CS0016: Could not write to output file 'c:\WINDOWS\Microsoft.NET\Framework\v1.1.4322\Temporary
> ASP.NET Files\application1\c11b43f6\cf3ec03\rizcntet.dll' . The directory name is invalid.
> For the .NET Framework 1.0, the error message is the following:<br />
>
> CS0013: Unexpected error writing metadata to file
'C:\WINDOWS\Microsoft.NET\Framework\v1.0.3705\Temporary ASP.NET Files\application2\3fc72f26\eb731247\ev2bslce.dll'.
The directory name is invalid.

### CAUSE

The system *TEMP* and *TMP* variables point to a folder that does not exist.

The compiler generates temporary files in the folder where the *TEMP* and the *TMP* variables point to before the files are copied to the Temporary ASP.NET Files folder. However, the folder where the system variables point to is deleted when you restart the computer. Therefore, the compiler cannot generate the temporary files.

### RESOLUTION

* Create a temporary folder under *%Systemroot%*, and then name it *Temp*.
* Grant full permissions on the Temp folder to the *aspnet* user account in .NET Framework 1.0 or to the *NETWORK SERVICE* user account in .NET Framework 1.1.
* Right-click *My Computer*, and then click *Properties*.
* On the *Advanced* tab, click *Environment Variables*.
* Select the *TEMP* variable under *System variables*, and then click *Edit*.
* Type *%SystemRoot%\TEMP* in the *Variable Value* box, and then click *OK*.
* Repeat steps 5 and 6 to edit the *TMP* variable. Click *OK* two times.
* Click *Start*, and then click *Run*.
* To reset Internet Information Services (IIS), type *iisreset* on the command prompt.

**Note**: If the error message that is mentioned in the "Symptoms" section of this article persists, restart the computer.

### MORE INFORMATION

Steps to Reproduce the Behavior

1. Start Microsoft Visual Studio .NET.
2. Create a new ASP.NET Web Application project by using Microsoft Visual C# .NET or Microsoft Visual Basic .NET, and then name the project *CompileTest*.
3. On the *Build* menu, click *Build Solution*.
4. Right-click *My Computer*, and then click *Properties*.
5. On the *Advanced* tab, click *Environment Variables*.
6. Select the *TEMP* variable under *System variables*, and then click *Edit*.
7. Type *<em>%SystemRoot%</em>\TEMP1* in the *Variable Value* box to point to the nonexistent TEMP1 folder, and then click *OK*.
8. Repeat steps 6 and 7 to edit the *TMP* variable to point to the nonexistent TEMP1 folder.
9. Click *OK* two times.
10. To notice one of the error messages that are mentioned in the "Symptoms" section of this article, visit the following URL: `http://localhost/CompileTest/WebForm1.aspx`
