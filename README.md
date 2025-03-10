# Industrial-Mixing-Process-In-OpenFOAM
Well, in the "About" it says 'simple', but anyone involved with CFD knows, that even this example is not that simple in OpenFOAM. The basic outline is: we want to mix a fluid with two different temperatures in a vessel and do everything with free and open-source software (design, simulation). Sounds simple, but in OpenFOAM we have to use at least compressibleInterFoam for this task. Compressible solvers have their own caveats, as you CFD aficionados know. They are inherently difficult, if for example a boundary condition is not defined well. It is very easy to arrive at a situation where solving the problem takes 'forever' (very small timeStep to converge) or some parameter goes haywire.
However, compressibleInterFoam can treat 2 species, one fluid and one gas. The fluid can exchange heat with itself and also with the gas (also vice-versa). In this example we simulate the filling of a tank with water, he tank will already have a certain level of water in it (T=300K), a pipe will add additional hotter water into the tank (T=360K). We do not stir the tank, although this would also be an interesting case. It would involve a MRF like the submarine example, but the result result would be a simple one as we basically may calculate the mixing temperature with pen and paper or with Excel (even in a time dependent and transient manner!). 
Now, let's take a look at our simulation domain. The tank is 3m high with a diameter of 1m, has a dia 150mm pipe connecting to the top. This pipe will fill the additional hot water into the vessel. The tank has a little stub pipe on top (dia 100mm), we will use it to blow off air pressure to keep the pressure inside constant. It also has a larger runoff-pipe with dia 250mm. In the first part of the simulation this runoff is closed and will be treated like a wall. Our initial state is shown in the first image, we have colder water in the tank and hotter water is ready to be pumped into the vessel. The scalar clip of alpha.water shows all the water in the domain. The clip is colored with temperature (hot water = yellow, cold water = purple):
![t0s_sliceU_clipalpha_T](https://github.com/user-attachments/assets/d85252a2-f201-4c9d-84e5-a3e59148cdbd)


Unfortunately, we don't have a valve like boundary condition which we could switch on or off (setting U=0 creates its own problems), so we have to split a simulation into two parts, if we want to empty the vessel in a second step. In the first part we will fill some hot water into the already 40% (or so) filled tank in two steps. The velocity profile of the filling pipe looks like this:

| time / s      | velocity / m/s|
| ------------- | ------------- |
|  0.0          | (-0.1 0 0)    |
|  1.0          | (-1 0 0)      |
|  6.0          | (-1 0 0)      |
|  6.1          | (0 0 0)       |
|  10.0         | (-0.1 0 0)    |
|  11.          | (-1 0 0)      |
|  16.0         | (-1 0 0)      |
|  16.1         | (0 0 0)       |
|  21.          | (0 0 0)       |

Essentially, from 1 to 6s we fill in hot water and also from 11 to 16s. In the first step we obviously also have to fill the pipe, so hot water does not flow into the vessel for a few seconds (more clearly in the video). The we switch off the flow and turn it on again at 11s. Both times we use a ramp of 1s on turn-on which leads to a more docile behavior of the compressible solver. The first part of the simulation is finished at 21s. 
Let's take a look at an intermediate result at 15s:
![t15s_sliceU_clipalpha_T](https://github.com/user-attachments/assets/f34415bd-cfb8-4062-951a-b0ea3f5fdb51)

The filling via the pipe is nearly finished at 15s (finish = 16s). Clearly, a layer of mixed hot and cold water has developed. There's also a slice in the image, showing the velocity U. As the clip covers the slice in the regions where we have water, only the U of the air is visible. Just before the falling water you'll notice the air getting pushed downward, this also more clearly in the shared video, where we even see tiny waves develop at the water level in the vessel before the water arrives. 

Then we have to prepare our next part which will be draining the vessel from 21 to 30s with a velocity profile on the runoff pipe like this:

| time / s      | velocity / m/s|
| ------------- | ------------- |
|  21.0         | (0 0 0)       |
|  21.1         | (-0.1 0 0)    |
|  21.5         | (-2 0 0)      |
|  29.0         | (-2 0 0)      |
|  30.0         | (-0.1 0 0)    |

Before we start the second part, we have to prepare a second folder (for safety) and do a 'reconstructPar -latestTime' on our data from step 1 and copy the folders system/ constant/ and $latestTime/ to the newly created folder.
We will perform our second simulation in this folder by just launching the above shared script. Its main purpose is to change the boundary conditions on inlet, outlet_air (that's the blowoff stub on top) and outlet_water. As mentioned, outlet_water now really becomes an outlet for draining the vessel with above velocity profile. The inlet becomes a wall now. Let's take a look at a later result showing the same data when the tank is nearly drained:

![t29s_sliceU_clipalpha_T](https://github.com/user-attachments/assets/fb9e27f3-576c-42b1-ac11-3699b960e166)

Now what to make of all this? CFD has become a widely used tool, today even smaller companies use it to make calculations and estimates regarding the fluids in their machinery and processes. Do they have the money for a commercial CFD license that will cost up to 50000€ a year per engineer? Very often, they don't. Even in this simple setup we can estimate process times (time until fluid temperatures have settled in the vessel, time until we can pump the mixture, max. runoff velocity allowed until air also gets sucked into the runoff pipe etc.), pump times/powers, mixing times etc. So even this simple simulation can increase the knowledge about the processes in a facility before it is even built and tested for the first time. The shared videos of the first and second simulation can be watched in real time when the playback speed is 8x. 

emefff@gmx.at




