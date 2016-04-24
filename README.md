# Octopus
##Offline calculation of Lagrangian trajectories

This model is in a development stage. Email me for questions.

Jinbo Wang <jinbow@gmail.com>
Scripps Institution of Oceanography
August 26, 2015


There are two configurations: Lagrangian particle and Argo float. 

Use "make" to compile the code for Lagrangian particle simulation. Use "make argo" for simulating Argo float. 

Before running the model, go to scripts folder and run "python gen_data.py" to compute the necessary data needed by the code.


###Lagrangian particle simulation

#### Steps to run:

The model is currently setup for the Southern Ocean State Estimate (SOSE). But it can be easily modified to fit any MITgcm output or any C-grid model output. To run the code, first make sure the model knows where to find your model output and correctly reads them. 

 1. Edit the parameters in **src/size.h** to fit the data you have.
 1. Edit cpp_options.h to include or exclude features. Make sure **isArgo** is turned off in **cpp_options.h**: **#undef isArgo**
 1. Within **src/** folder. Compile the code using
     >make

   You will get an excutable file named **O.particle**.

 1. Prepare the initialization file using **scritps/init_particl_xyz.py**. Copy the binary file to **src/**.
 1. If you run particle with the SOSE 1/6th degree simulation, download necessary binary files:
    >bash sync_data.sh

    otherwise, go to scripts folder and run
    >python gen_data.py

    to generate the binary files.
    
 1. After running gen_data.py, you should get a list of binary files in your **pth_data_out** folder specified in **data.nml** including

  1. reflect_x.bin
  1. reflect_y.bin
  1. z_to_k_lookup_table.bin
  1. k_to_z_lookup_table.bin.

 1. Set parameters in the namelist file **data.nml**. The parameters are explained line by line in **src/data.nml.explained**.
 1. In the **src/** folder, run the model

    >./O.particle

 1. Outputs are saved in the folder  **output_dir** specified in **data.nml**.

###Argo simulation

#### Steps to run:
Most of the steps are the same as for Lagrangian particle simulation with few exceptions. 

1. Make sure **isArgo** is defined in **cpp_options.h**:  **#define isArgo**
1. Use *"make argo"*  to compile the code. 
1. You will get a excutable **O.argo** after successfully compiling the code. 
1. Run the code using

   >./O.argo

###Parameterizations

####Laplacian diffusion

Use random walk to mimic the Laplacian diffusion. The random number with normal distribution is generated by **random.f90**. The value of the horizontal diffusion is set in **Khdiff** in **data.nml**.

1. Before compiling the code, use **#define use_Laplacian_diffusion** in **cpp_options.h** to turn it on. 
1. Use **Khdiff** and **Kvdiff** to set the Laplacian diffusivity in horizontal and vertical, respectively. The unit is m^2/s.

####Mixed Layer processes

**Use this with caution**

This is a rather ad-hoc solution. No existing study has confirmed the robustness of this parameterization. 

The idea is that the vertical mixing induced by mixed layer instability, which is usually parameterized by KPP (large et al. 1997), is missing for velocity-based Lagrangian trajectory integration. If we use particle density to represent passive tracers, we need to add random walk to particle position to mimic the parameterized vertical mixing in coarse resolution models. It is computationally too expensive to explicitly calculate the vertical mixing according to for example a K-profile of KPP. Our ad-hoc solution is to find particles within mixed layer and reshuffle them using a random-displacement model. 

The model needs the following to run with mixed layer parameterization:

1. turn on the paramterization in **cpp_options.h**: **#define use_mixedlayer_shuffle**
1. prepare a data file with mixed layer depth (MLD)
1. specific the parameters in **data.nml**
 1. **fn_MLD='your mixed layer depth file name'**
 1. **dt_mld=432000,** the time interval (in seconds) of the MLD data. For example, if you saved you MLD every 5 days, then **dt_mld=432000**

####Meridional boundary condition

1. Use **#define reflective_meridional_boundary** to turn on the reflective meridional boundaries. Particles will keep bouncing back to the domain.
1. Use **#undef reflective_meridional_boundary** to turn it off. Particles will stay stagnant outside.

####Zonal boundary condition
Right now the zonal boundary condition is hard-coded as **periodic**.

####Reflective continent
Occasionally, particles enter continent, i.e., the dry cells. You can either keep them there or perform a rescue. 

1. Use **#define reflective_continent** to turn on reflective continent boundary condition. 
1. use **#undef reflective_continent** to turn it off. Particles will then stay stagnant once enter a dry grid cell. 

####Looping condition

With usually short duration of a numerical simulation, a common practice to obtain a Lagrangian simulation with a longer duration than the Eularian fields is to loop the velocity from the starting point. This is rather a poor-man's solution, and strongly discouraged. However, if it is preferred, there are too options.

1. Before compilation, use  **#define jump_looping** in **cpp_options.h** to turn on the jumping condition for looping, i.e., applying an artifical jump to make sure particle stay on the same isopycnal level before and after looping.
1. Use **undef jump_looping** to turn it off. The code will just use the first step velocity to continue to advect particles once it reaches the end of the Eularian field without doing anything on particle positions.




Good luck!


Jinbo Wang
April 24, 2016
~                                                                                                                                                                                                           
~                                                                                                                                                                                                           
~                                                                                                                                                                                                           
~                          
