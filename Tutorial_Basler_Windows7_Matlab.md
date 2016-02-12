#Tutorial Installing Basler MatLab drivers for Windows 7 64-bit.

-------------------
last edited by : Simon Lansbergen, 12-2-2016.
last executed : 12-2-2016 (@PC_Home Simon)

> **Note:**

> This tutorial describes the necessary steps for installing the Basler camera drivers provided by Hannes Badertscher (ICOM Institute for Communication Systems at HSR University of Applied Sciences of Eastern Switzerland), which can be downloaded from [github]. 
>The tutorial is aimed for installation on a Windows 7 64-bit machine, with MatLab 2014a (or later).

>For further information on the camera drivers and MatLab functions, see the README.md in the github url or visit [mathworks].

> **Requirements:**

> For successful implementation of the Basler camera drivers the following preliminary requirements are mandatory:

>- A (fairly) clean Windows 7, **64-bit** installation, although make sure windows update it is up to date.
>- MatLab 2014b or later, **64-bit** edition.
>- USB 3.0 interface.
>- Github environment.
>- A tool to mount *.iso files into windows (e.g. magicISO)

> Make sure that both Windows 7 and MatLab are 64-bits. This is necessary for the *C++* compiler (in MatLab), to compile the final mex-files.

> **Important:**

> Stick to the suggested downloads and versions. While other version will not work, especially when 32-bits!
> >**Turn off windows update during installation!**



##Step 1.

####Download the following :

>- [Basler USB 3.0 Drivers] (Basler USB3 AIK Driver 4.2).
>- [Basler Pylon 4] (version 4.0.0, **64-bit**).
>- [Boost C++ Libraries] (version 1.55.0 build 2 **64-bit**. *boost_1_55_0-msvc-10.0-64.exe*).
>- [Windows .NET 4 Framework] web installer from Microsoft.
>- [Windows SDK 7.1 and .NET Framework 4] (ISO) from Microsoft (chose x64: *GRMSDKX_EN_DVD.iso*) again **64-bit**.
>- [Microsoft Visual C++ 2010 SP1] Compiler Update for Windows SDK 7.1.

####Clone HBadertscher/Matlab_BaslerCamDriver repository in Github.
The next step is to add [HBadertscher/Matlab_BaslerCamDriver] to your github repository and then clone it locally to the preferred location (e.g. c:\software). 

##Step 2.

Now everything is downloaded the first thing to install are the boost libraries, these are pre-compiled binaries that can be installed via a windows executable.

> After installation of the Boost libraries it is important to set the root folder where Boost is installed in windows environmental variables. These settings can be found in `Control-panel > System and security > System`, then on the left side; click `advanced  settings`. Now add a new system variable the name of the variable is:

>>BOOST_ROOT

> The value of the variable is the Boost root directory  :

>>( e.g. ) c :\software

When Boost is correctly installed we continue with the installation of the Basler USB 3.0 drivers. If you open the folder *64bit* after unzipping the download you will find Basler AIK setup64. Install only the driver, when asked: check all available packages for installation. When done, install the Pylon suite 4.0.0 64-bit suite.

> Now it is time to plug the Basler camera in your USB 3.0 slot. After the first use windows will install the USB camera and you can check either in the software or in the windows control panel `system > Device manager` whether the camera is installed correctly.


##Step 3.


As the camera is installed correctly in windows it is time to install Windows SDK 7.1 and .NET Framework 4. This part is a bit tricky and it is important to follow each step carefully.

> For the installation of Windows SDK 7.1 and .NET Framework 4 it is convenient to mount the *.iso in a tool like magicISO so it can be installed from a virtual drive.

There are several steps involved in installing Windows SDK 7.1 and .NET Framework 4 correctly. When the .iso file is mounted **do not** run the setup on the disc yet! It will not install the necessary *C++* packages. 

>1. Go to `Control panel > Add and remove Software` and remove all .NET packages from Windows (both x86 as well as x64).

>2. Uninstall all Microsoft Visual C++ 2010 redistributable packages (both x86 as well as x64). 

>4. Install Windows .NET Framework 4.

>4. Mount and install Windows SDK 7.1.

>5. Microsoft Visual C++ 2010 SP1 Compiler Update for Windows SDK 7.1 (optional, but recommended).

It may occur that after installation the file `ammintrin.h` is missing from the folder `C:\Program Files (x86)\Microsoft Visual Studio 10.0\VC\INCLUDE\`. Check whether this file exist and if it does not exist download the ammintrin.m file from this [thread] on matlab central, rename it to ammintrin.h and copy the renamed file to the readily mentioned directory.



##Step 4.

Open MatLab and the following in the command window:

```matlab
mex -setup c++
```

The output should by now look like this:

```matlab
>> mex -setup c++
MEX configured to use 'Microsoft Windows SDK 7.1 (C++)' for C++ language compilation.
Warning: The MATLAB C and Fortran API has changed to support MATLAB
	 variables with more than 2^32-1 elements. In the near future
	 you will be required to update your code to utilize the
	 new API. You can find more information about this at:
	 http://www.mathworks.com/help/matlab/matlab_external/upgrading-mex-files-to-use-64-bit-api.html.
```

From this point only continue when the same output is achieved. Otherwise Matlab will not compile the mex files by which MatLab can communicate with C++ libraries.


##Step 5.

In Matlab go to the tab *Home* and hit *Set Path*, than add a new folder including subfolders. Look for the folder where you synced the Basler camera driver repository to (e.g.  c:\software\Matlab_BaslerCamDriver ), add this folder and save.

Before we can compile the necessary mex files in MatLab we need to make some changes is the cloned Basler code. We start by opening `make.m` in the MatLab script editor, this file is located in `\Matlab_BaslerCamDriver`.

> Replace the following line (45) :
> ```C++
> Line 45 :       ' ','"',fullfile(getenv('BOOST_ROOT'),'stage\lib'),'"', ...
> ```

>with

> ```C++
> Line 45 :       ' ','"',fullfile(getenv('BOOST_ROOT'),'lib64-msvc-10.0'),'"', ...
> ```

> The installation of the Basler camera drivers assumes a fresh compilation of Boost rather than installing pre-compiled binaries, therefore the directory where the Boost libraries are located is different and need to be changed.

Secondly open `baslerGetParameter.cpp` in MatLab script editor, this file is also located in `\Matlab_BaslerCamDriver`.

> Replace the following line (45) :
> ```C++
> Line 13 : enum class ParamType {
> ```

>with

> ```C++
> Line 13 : enum ParamType {
> ```

> This change is due to compatibility problems with type `enum class` in Windows Visual *C++* 2010. If not changed, the build will cause errors and will fail to run.

##Step 6.

Finally we can compile the mex files in MatLab. Make sure the active MatLab directory is `\Matlab_BaslerCamDriver`. Now run the `make.m` script from the MatLab command window.

The `make.m` script gives the following output if ran correctly:

```C++
>> make
=> Creating Libraries
Building with 'Microsoft Windows SDK 7.1 (C++)'.
MEX completed successfully.
Building with 'Microsoft Windows SDK 7.1 (C++)'.
MEX completed successfully.
=> Creating Functions
Building with 'Microsoft Windows SDK 7.1 (C++)'.
MEX completed successfully.
Building with 'Microsoft Windows SDK 7.1 (C++)'.
MEX completed successfully.
Building with 'Microsoft Windows SDK 7.1 (C++)'.
MEX completed successfully.
Building with 'Microsoft Windows SDK 7.1 (C++)'.
MEX completed successfully.
Building with 'Microsoft Windows SDK 7.1 (C++)'.
MEX completed successfully.
>> 
```

##Step 7.

The last step covers the reinstallation of Microsoft .NET Framework packages. This can be done in two ways:

>1. Manually search for the missing .NET Framework installers. 
>2. Or, alternatively let Microsoft update find the missing components automatically 
>(this could take a while).

>> **Do not** forget to turn windows update on again before closing windows!

###End Note :

This includes the tutorial on how to install the Basler camera drivers in Windows 7.

>Tutorial written by Simon Lansbergen, 2016. 
>>The author claims no copyright nor liability on the provided steps and changes to the code and changes to PC software. The author has no connection with any of the companies relevant to this tutorial.




[//]: # (References.)

[github]: <http://github.com/HBadertscher/Matlab_BaslerCamDriver>
[mathworks]: <http://www.mathworks.com/matlabcentral/fileexchange/50681-basler-camera-driver>
[Basler USB 3.0 Drivers]: <http://www.baslerweb.com/en/support/downloads/software-downloads?type=0&series=0&model=0>
[Basler Pylon 4]: <http://www.baslerweb.com/en/support/downloads/software-downloads?type=18&series=0&model=0>
[HBadertscher/Matlab_BaslerCamDriver]:<http://github.com/HBadertscher/Matlab_BaslerCamDriver>
[Boost C++ Libraries]: <https://sourceforge.net/projects/boost/files/boost-binaries/1.55.0-build2/>
[Windows SDK 7.1 and .NET Framework 4]: <https://www.microsoft.com/en-us/download/details.aspx?id=8442>
[Windows .NET 4 Framework]: <https://www.microsoft.com/nl-nl/download/details.aspx?id=17718>
[Microsoft Visual C++ 2010 SP1]: <https://www.microsoft.com/en-us/download/details.aspx?id=4422>
[thread]: <http://www.mathworks.com/matlabcentral/answers/90383-fix-problem-when-mex-cpp-file>