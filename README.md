# abaqus-subroutine-debugging

Debug the abaqus subroutines

## Enable debugging feature in Abaqus

Go the the Abaqus configuration file `abaqus_v6.env` or `win86_64.env` in the Abaqus path, for me, it locates in the following path:

```
C:\SIMULIA\EstProducts\2021\win_b64\SMA\site
```

 The file is like this:

```python
#-*- mode: python -*-

#############################################################################
#                                                                           #
#     Compile and Link commands for ABAQUS on the Windows 64 Platform       #
#                                                                           #
#############################################################################

# import os # <-- Debugging
# os.environ['GLOBAL_ENABLE_FPE'] = 'TRUE' # <-- Debugging

# Location of the /include directory in the ABAQUS installation
abaHomeInc = os.path.abspath(os.path.join(os.environ.get('ABA_HOME', ''), os.pardir)) 

compile_cpp=['cl', '/c', '/W0', '/MD', '/TP',
             '/EHsc', '/DNDEBUG', '/DWIN32', '/DTP_IP', '/D_CONSOLE',
             '/DNTI', '/DFLT_LIC', '/DOL_DOC', '/D__LIB__', '/DHKS_NT',
             '/D_WINDOWS_SOURCE', '/DFAR=', '/D_WINDOWS', '/DABQ_WIN86_64', '%P',
             # '/O1', # <-- Optimization
             # '/Zi', # <-- Debug symbols
             '/I%I', '/I'+abaHomeInc]

compile_fmu=['win64CmpWrp', '-m64', '-msvc9', 'cl', '/LD', 
             '/D_WINDOWS', '/TC', '/W0',  '/I%I', '/I'+abaHomeInc]

## Fortran compile command for User Subroutines

compile_fortran=['ifort',
                 '/c', '/fpp', '/extend-source', 
                 '/DABQ_WIN86_64',  '/DABQ_FORTRAN',
                 '/iface:cref', '/recursive',
                 '/Qauto',  # <-- important for thread-safety of parallel user subroutines
                 '/align:array64byte',
                 '/Qpc64',                    # set FPU precision to 53 bit significand
                 '/Qprec-div', '/Qprec-sqrt', # improve precision of FP divides and sqrt
                 '/Qfma-',                    # disable floating point fused multiply-add
                 '/fp:precise',               # floating point model: precise 
                 '/Qimf-arch-consistency:true', # math library consistent results
                 '/Qfp-speculation:safe',     # floating point speculations only when safe
                 '/Qprotect-parens',          # honor parenthesis during expression evaluation
                 '/Qfp-stack-check',          # enable stack overflow protection checks
                 '/reentrancy:threaded',      # important for thread-safety
                 #'/Qinit=zero','/Qinit=arrays',  # automatically initialize all arrays to zero
                 #'/Qinit=snan', '/Qinit=arrays', # automatically initialize all arrays to SNAN
                 '/QxSSE3', '/QaxAVX',        # generate SSE3, SSE2, and SSE instructions
                 # '/Od', '/Ob0',  # <-- Disable Optimization for Debugging
                 # '/Zi',          # <-- Debugging Information
                 '/include:%I', '/include:'+abaHomeInc, '%P']

link_sl=['LINK',
         '/nologo', '/NOENTRY', '/INCREMENTAL:NO', '/subsystem:console', '/machine:AMD64',
         '/NODEFAULTLIB:LIBC.LIB', '/NODEFAULTLIB:LIBCMT.LIB',
         '/DEFAULTLIB:OLDNAMES.LIB', '/DEFAULTLIB:LIBIFCOREMD.LIB', '/DEFAULTLIB:LIBIFPORTMD.LIB', '/DEFAULTLIB:LIBMMD.LIB',
         '/DEFAULTLIB:kernel32.lib', '/DEFAULTLIB:user32.lib', '/DEFAULTLIB:advapi32.lib',
         '/FIXED:NO', '/dll',
         # '/debug', # <-- Debugging
         '/def:%E', '/out:%U', '%F', '%A', '%L', '%B', 
         'oldnames.lib', 'user32.lib', 'ws2_32.lib', 'netapi32.lib',
         'advapi32.lib', 'msvcrt.lib', 'vcruntime.lib', 'ucrt.lib']

link_exe=['LINK',
          '/nologo', '/INCREMENTAL:NO', '/subsystem:console', '/machine:AMD64', '/STACK:20000000',
          '/NODEFAULTLIB:LIBC.LIB', '/NODEFAULTLIB:LIBCMT.LIB', '/DEFAULTLIB:OLDNAMES.LIB', '/DEFAULTLIB:LIBIFCOREMD.LIB',
          '/DEFAULTLIB:LIBIFPORTMD.LIB', '/DEFAULTLIB:LIBMMD.LIB', '/DEFAULTLIB:kernel32.lib',
          '/DEFAULTLIB:user32.lib', '/DEFAULTLIB:advapi32.lib',
          '/FIXED:NO', '/LARGEADDRESSAWARE',
          # '/debug', # <-- Debugging
          '/out:%J', '%F', '%M', '%L', '%B', '%O',
          'oldnames.lib', 'user32.lib', 'ws2_32.lib', 'netapi32.lib',
          'advapi32.lib', 'msvcrt.lib', 'vcruntime.lib', 'ucrt.lib']

## Link command to be used for ABAQUS/MAKE on machines without the Fortran compiler.
## Uncomment it to enable.
#
#link_exe=['LINK', '/nologo', '/INCREMENTAL:NO', '/subsystem:console', '/machine:AMD64', '/NODEFAULTLIB:LIBC.LIB', '/NODEFAULTLIB:LIBCMT.LIB',
#          '/DEFAULTLIB:OLDNAMES.LIB', '/DEFAULTLIB:MSVCRT.LIB', '/DEFAULTLIB:kernel32.lib', '/DEFAULTLIB:user32.lib', '/DEFAULTLIB:advapi32.lib',
#          '/FIXED:NO', '/LARGEADDRESSAWARE', '/out:%J', '%F', '%M', '%L', '%B', '%O',
#         'oldnames.lib', 'user32.lib', 'ws2_32.lib', 'netapi32.lib',
#         'advapi32.lib', 'msvcrt.lib', 'vcruntime.lib', 'ucrt.lib']

# Remove the temporary names from the namespace
del abaHomeInc
```

Uncomment all the lines that end with comment of Debugging.

## Insert some extra codes to pause in your subroutine when running Abaqus  

Add some codes like this to pause at the subroutine after the declaration of variables, like this:

```fortran
WRITE(*, *) "DEBUGGING, PRESS ENTER TO CONTINUE"
READ(*, *)
```

## Run Abaqus command to submit the job

Use the Abaqus command the submit the job, like this

```bash
abaqus job=Job-1 user=FRIC_coulomb int
```

If you have added some code to pause in the subroutine, the program will stop in the command window, like this

```bash
10/30/2021 11:51:14 PM
Begin Analysis Input File Processor
10/30/2021 11:51:14 PM
Run pre.exe
10/30/2021 11:51:17 PM
End Analysis Input File Processor
Begin Abaqus/Standard Analysis
10/30/2021 11:51:17 PM
Run standard.exe
DEBUGGING, PRESS ENTER TO CONTINUE
```

## Attach to process in Visual Studio

Open your subroutine file in Visual Studio, click `Debug -> Attach to Process`, choose the Abaqus program `standard.exe` or `explicit_dp.exe`.

## Resume running the subroutine and debug in Visual Studio

Add a breakpoint in the subroutine file in Visual Studio, resume running the subroutine in the command window, i.e., press enter to continue, the subroutine will stop in Visual Studio, and now you can debug your subroutine in Visual Studio!

## Warning

Abaqus CAE cannot be opened if the debug mode is activated, change the configuration file to the old version and you can open Abqus CAE again.
