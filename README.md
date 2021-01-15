# TheSydeKick-System Development Kit
Initiated by Marko Kosunen, marko.kosunen@aalto.fi 7.8.2017

*OBS: THE SCRIPTS TO BE SOURCED ARE WRITTEN FOR T-SHELL*
if you're using some other shell, change to tcsh or modify the scripts to be 
compliant with your shell.
> tcsh

## CONFIGURATION Quickstart
Go to TheSDK directory and run 
> ./init_submodules
> ./configure

- Edit the TheSDK.config file so that the commands for python 
invocations are correct. 
Variables in TheSDK.config

The variables defines the commands used in Makefiles for simulation
submission main thing to decide here is if you have LSF compliant cluster
environment or not. Modify commands accordingly.


- Configure latex, and simulators ( vsim, eldo, spectre etc.) tools to your path, 
modify sourceme.csh if needed and source it
> source sourceme.csh

-If you are using python, you may need to install some modules locally.
run ./pip3userinstall.sh or modify it to be compliant with your current python
version.
> ./pip3userinstall.sh


- To test your environment:
> cd Entities/inverter 
> ./configure && make clean && make all
(Dependency tracking for Python currently not functional)

By default LSF submissions are enabled in TheSDK.config. If you do not have LSF, please disable it from TheSDK.config and
> ./configure && make clean && make all
again

If you wish to test the Python functionality only, edit Entities/inverter/inverter/__init__.py
and Change the line
> models=[ 'py', 'sv', 'vhdl', 'eldo', 'spectre' ]
to
> models=[ 'py' ]
and run 
> ./configure && make clean && make all

* TO CREATE AND TEST NEW ENTITY

Create a new entity with:
> cd ./Entities
> ../thesdk_helpers/initentity <NAME>

Test the new entity:
> cd <NAME> && ./configure && make all

See  
> ../thesdk_helpers/initentity -h for help
The new entity is created as a git project. Push it to your favourite repository

## HOW TO USE TheSyDeKick

TheSyDeKick is a multi-tool simulation and developement environment for developing systems. 
It targets to using a single control environment to simulate,design and measure the 
system components with various tools by using a single "Control environment" for
control, analysis, and visualization of the results.

Implementation the "Control environment" is written in Object-oriented
Python. Python selected based on
its good support for computing and signal processing, and support for
interfaces to measurement equipment.. 

NAMING AND STRUCTURE
The files are organized in directories as follows
                      TheSDK
pip3userinstall.sh
init_submodules.sh
configure
TheSDK.config (generated by configure)
Entities                               
    |                                  
    entity1                            
        entity1                         
             |                          
             __init__.py                
             other_module.py            
        vhd                             
            |                           
            entity1.vhd                 
            tb_entity1.vhd                                     
        sv                                                
            |                                   
            entity1.sv                          
            tb_entity1.sv
        spice
            |
            entity1.cir
            tb_entity1.cir
        Simulations
            rtlsim
                |
                work

Guidelines to follow
- All simulations configured, performed and results processed under Simulations directory
- All component descriptions, called Entities, regardless of the used tool/language are 
  located under Entities directory.


## CLASS ORGANIZATION
The Entities and simulation setups are implemented as classes that  
cross-reference to each other without restrictions. 

*EXAMPLE of class dependencies and relations
        
           
> system_abstract_class(thesdk,spectre,rtl)
>           \                 /      /
>            \               /      /
>                "system_sim       /
>                     |           /
>                  "system"      /
>                 /     \       /
>        "entity1"  "entity2"
>           |
>       "entity3"


- Classes are documented with docstrings. To read the dogumentation, e.g:
> cd Entities/rtl
> ./configure && make doc
> firefox ./doc/build/html/index.html

- thesdk is a class to collect methods common to "TheSyDeKick"-framework.
It should NOT contain anything specific to
a particular system. 
It should be superclass to all classes in TheSDK, 
- rtl class defines properties and methods that are required to 
run verilog and vhdl simulations.

- spice class defines properties and methods that are required to 
run eldo and spectre simulations.


- If component does not have rtl as a superclass, rtl-requirements do not 
apply. Consequently, if component has an  rtl model, it MUST  be a a subclass of rtl. 

- "system_abstract_class" may used as super class for the "system_sim" and
    "system" to define the properties that typically 
        1. Are common to whole systems. 
        2. Need not to be altered between simulations, but are most often
        propagated through property inheritance.

- Typically a simulation is controlled by "system_sim" class that controls 
the simulation providing methods like  "run" and  "plot".
See: Entities/inv_sim/inv_sim/__init__py 

- Component properties are controlled and propagated by class constructor by copying the
    selected properties from immediate "parent". The properties that are to be copied are determined 
    by property "proplist".

## COMPONENT HIERARCHY GUIDELINE
Take  a look at 
> Entities/inv_verter/inv_verter/__init__.py
and
> Entities/inv_sim/inv_sim/__init__.py

- System is described in "system"  class that determines the 
sub-components and the interconnections in between them, and methods to 
"run" the "system", i.e. how the signals propagate and in which order 
the methods of components are executed.

- If the system is simple, "sim" class may also construct the system. 
(see Entities/inv_sim/inv_sim/inv_sim.)

### Guidelines to be followed:

- Components are not subclasses to sim or system class as they should be
    independent of each other and transferrable between systems. 

- sim class is (usually) not a superclass to system class, as the "system" definitions 
    are independent of how it is simulated.

## WHAT NEXT?:
Take yout time to get acquainted with Entities/inverter and Entities/inv_sim
together with the documentation of Entities/thesdk, Entities/rtl and Entities/spice. Those should 
give you a picture how the things work.

See also: https://github.com/TheSystemDevelopmentKit/TheSyDeKick_tutorial
