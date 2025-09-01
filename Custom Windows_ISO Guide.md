# How to make a custom Windows ISO

## Tools
You will need a few tools to make this happen
1. 7-Zip

     **Install it at: https://7-zip.org/a/7z2501-x64.exe**

     ###### OR 
     **Run this command:** <pre><code>winget install --id=7zip.7zip -e</code></pre>


2. DISM

        This tool comes installed as standard with every Windows 11 Installation.
3. OSCDIMG

        You will need to install the Windows ADK (Deployment Tools only)
     **Install it at: https://go.microsoft.com/fwlink/?linkid=2289980**

## Commands

`Run CMD as Administrator to avoid any issues`

### ***Change Directory to `C:\`***
<pre><code>cd /d C:\</code></pre>


### ***Create working folders***
<pre><code>mkdir C:\WinISO\Mount\Custom</code></pre>
<pre><code>cd C:\WinISO\</code></pre>
<pre><code>mkdir CustomFiles</code></pre>
<pre><code>mkdir ISO_Files</code></pre>



`Save the files or folders that you want to add to the Windows ISO in the CustomFiles folder`

`Also save the Windows ISO that you want to modify in C:\WinISO`

### ***Extract the Original ISO***
<pre><code>"C:\Program Files\7-Zip\7z.exe" x C:\WinISO\Win11_24H2_EnglishInternational_x64.iso -oC:\WinISO\ISO_Files</code></pre>

**Tip:** Check the actual name of your ISO file (`Win11_24H2_EnglishInternational_x64.iso` for me) and adjust if needed.

### ***Inspect the Install Image***
<pre><code>dir C:\WinISO\ISO_Files\sources\install.*</code></pre>
<pre><code>dism /Get-WimInfo /WimFile:C:\WinISO\ISO_Files\sources\install.wim</code></pre>

`This command lists the available Windows editions in the WIM file. Find the index number for the edition you want to modify and use it in the next command.  If Windows 11 Home, use the next command as-is.`

### ***Mount the WIM File***
<pre><code>dism /Mount-Wim /WimFile:C:\WinISO\ISO_Files\sources\install.wim /Index:1 /MountDir:C:\WinISO\Mount</code></pre>

### ***Copy Custom Files***
<pre><code>xcopy C:\WinISO\CustomFiles\* C:\WinISO\Mount\Custom /E /I /Y</code></pre>

#### Command Explanation
***/E***: Copy all subdirectories

***/I***: Assume the destination is a folder (avoid prompt)

***/Y***: Suppress prompt to overwrite existing files

### ***Save Changes to the WIM***
<pre><code>dism /Unmount-Wim /MountDir:C:\WinISO\Mount /Commit</code></pre>

### ***Find `oscdimg`***
<pre><code>where /r "C:\Program Files (x86)\Windows Kits" oscdimg.exe</code></pre>

### ***Rebuild the ISO using oscdimg***
<pre><code>"C:\Program Files (x86)\Windows Kits\10\Assessment and Deployment Kit\Deployment Tools\amd64\Oscdimg\oscdimg.exe" -bC:\WinISO\ISO_Files\boot\etfsboot.com -u2 -udfver102 -bootdata:2#p0,e,bC:\WinISO\ISO_Files\boot\etfsboot.com#pEF,e,bC:\WinISO\ISO_Files\efi\microsoft\boot\efisys.bin C:\WinISO\ISO_Files C:\WinISO\Custom_Win11.iso</code></pre>

**Tip:** Name your new ISO file (`Custom_Win11.iso`) how you want.

`If oscdimg fails, double-check the path. It can vary by ADK version.`

#### Command Explanation
***-b...etfsboot.com***: Specify the boot sector for BIOS-based (legacy) booting

***-u2***: Enable `UDF` file system format, needed for larger files.

***-udfver102***: Set `UDF` version to *1.02* (broader compatibility with bootloaders)

        -bootdata:2#p0,e,b...#pEF,e,b...:

**-bootdata:** Flag that defines how the ISO should boot

**2**: Number of boot entries (dual boot: BIOS + UEFI)

**p0**: Platform ID 0 = BIOS

**pEF**: Platform ID EF = UEFI

**e**: Enable `El Torito` boot entry

**b...** Path to boot image

### ***Clean Up***
<pre><code>rd /s /q C:\WinISO\CustomFiles</code></pre>
<pre><code>rd /s /q C:\WinISO\ISO_Files</code></pre>
<pre><code>rd /s /q C:\WinISO\Mount</code></pre>

### ***Finished ISO***
After you finish installing from the custom ISO, the added files will be in `C:\Custom` on the installed system.