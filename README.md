# TheSydeKick-System Development Kit
Initiated by Marko Kosunen, marko.kosunen@aalto.fi 7.8.2017

* OBS: THE SCRIPTS TO BE SOURCED ARE WRITTEN FOR T-SHELL *
if you're using some other shell, change to tcsh or modify the scripts to be 
compliant with your shell.
> tcsh

TheSyDeKick release 1.5 has been tested with Python v3.6

## Configuration quickstart

Go to TheSDK directory and run 
> ./init_submodules.sh
> ./pip3userinstall.sh
> ./configure

- Edit the TheSDK.config file so that the commands for python 
invocations are correct. By default LSF submissions are enabled in TheSDK.config. If you do not have LSF, please disable it from TheSDK.config

The variables defines the commands used in Makefiles for simulation
submission main thing to decide here is if you have LSF compliant cluster
environment or not. Modify commands accordingly.

- The simplest possible simulation is defined in Entities/myentity/myentity/__init__.py
To test your Python installation and configuration:
> cd Entities/myentity
> ./configure && make all

You should see a input and output waveforms of a buffer model.

- Configure circuit simulators ( vsim, eldo, spectre etc.) tools to your path, 
modify sourceme.csh if needed and source it
> source sourceme.csh

- To test your environment:
> cd Entities/inverter 
> ./configure && make clean && make all

If you wish to test the Python functionality only, edit Entities/inverter/inverter/__init__.py
and Change the line
```
models=[ 'py', 'sv', 'vhdl', 'eldo', 'spectre' ]
```
to
```
models=[ 'py' ]
```
and run 
> ./configure && make clean && make all
## How to use TheSyDeKick

TheSyDeKick is a multi-tool simulation and developement environment for developing systems. 
It targets to using a single control environment to simulate,design and measure the 
system components with various tools by using a single "Control environment" for
control, analysis, and visualization of the results.

Implementation the "Control environment" is written in Object-oriented
Python. Python selected based on its good support for computing and signal processing, and support for
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
- All component descriptions, called Entities, regardless of the used tool/language are 
located under Entities directory.
- Git submodules are initiated with script `init_submodules.sh`. This is to give controlled  
method to select what submodules to init.
- Things are configured with script named `configure`, that generates the Makefile.
- Things are executed with make <recipe>
- `configure && make` structure is used because by always following that we never need 
to document how to do configurations and executions. 

The main feature of TheSyDeKick is how to connect these objects together. 
- IO's are pointers to a Data field of an IO class instance.
- Drivers write to that data field.
- Input read from that data field.

Following this guideline your entities retain compatibility with othe TheSyDeKick entities.
See `Entities/inv_sim/inv_sim/__init__.py` for reference.

- Entities are documented with docstrings. To read the documentation, do:
```
cd Entities/rtl
./configure && make doc
firefox ./doc/build/html/index.html
```
### How to create and test new entity

Create a new entity with:
```
cd ./Entities
../thesdk_helpers/initentity <NAME>
```

Test the new entity:
```
cd <NAME> && ./configure && make all
```

See  `../thesdk_helpers/initentity -h` for help
The new entity is created as a git project. Push it to your favourite repository

### What next?
Take yout time to get acquainted with `Entities/inverter` and `Entities/inv_sim`  
together with the documentation of Entities/thesdk, Entities/rtl and Entities/spice. Those should  
give you a picture how the things work. Create a new entity, and start playing a round with it.  

See also: https://github.com/TheSystemDevelopmentKit/TheSyDeKick_tutorial

## Class organization guideline
This is not a strict ruleset, rather a guideline how to alleviate your modeling tasks and support modularity.

The Entities and simulation setups are implemented as classes that  
cross-reference to each other without restrictions. (Hardware) modules are instantiated as object of that class.


* EXAMPLE of class hierarchy 
           
        system_parameter_class(thesdk,spectre,rtl)  
                   \                 /      /  
                    \               /      /  
                        "system_sim       /  
                            |            /  
                          "system"      /  
                        /     \        /  
                "entity1"  "entity2"  
                  |  
               "entity3"  


- TheSyDeKick classes are intended to collect methods common to "TheSyDeKick"-framework.
They should NOT contain anything specific to a particular design. 
- Rtl class defines properties and methods that are required to 
run verilog and vhdl simulations.
- Spice class defines properties and methods that are required to 
run eldo and spectre simulations.
- If component has an  rtl model, it should  be a a subclass of rtl. If component does not have rtl as a superclass, rtl-requirements do not apply. 

- Design specific classes are freely defined by the designer
    - The "system_parameter_class" may used as super class for the "system_sim" and
"system" (not subcomponent entities) to define the properties that typically 
        1. Are common to whole system design. 
        2. Need not to be altered between simulations, but are most often
        propagated through property inheritance.

    - Typically a simulation is controlled by "system_sim" class that controls 
the simulation providing methods like  "run" and  "plot". This class usually contains a
"design under test", which is a instance of "system" class, and methods requiered to run the simulations.
(See: `Entities/inv_sim/inv_sim/__init__py` . As the test case for inv_sim is extremely simple, the DUT
is constructed inside it with 'define_simple method'. for more complex systems this is not preferred way.
This method should be in 'system' class.)
       
    - System is described in "system" class that determines the 
sub-components and the interconnections in between them, and methods to 
"run" the "system", i.e. how the signals propagate and in which order 
the methods of components are executed. Take  a look at `Entities/inv_verter/inv_verter/__init__.py`
and `Entities/inv_sim/inv_sim/__init__.py`

- Class attributes are controlled and propagated by class constructor by copying the  
selected properties from immediate "parent". The properties that are to be copied are determined  
by "proplist" attribute. By doing this isntead of using inherited classes, we keep entities independent of  
their use environment i.e. they can be used freely in other desings. Still we can automate the propagation  
of the parameters.

- Component entities Entity1-Entity-3 are not subclasses to sim or system class as they should be
independent of each other and transferrable between systems. 

- The "system_sim" class is should not be a superclass to system class, as the "system" definitions 
are independent of how it is simulated.

