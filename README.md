# Industrial-Mixing-Process-In-OpenFOAM
Well, in the "About" it says 'simple', but anyone involved in CFD knows, that even this example is not that simple in CFD. The basic outline is: we want to mix a fluid with two different temperatures in a vessel and do everything with free and open-source software (design, simulation). Sounds simple, but in OpenFOAM we have to use at least compressibleInterFoam for this task. Compressible solvers have their own caveats, as you CFD aficionados know. They are inherently difficult, if for example a boundary condition is not defined well. It is very easy to arrive at a situation where solving the problem takes 'forever' (very small timeStep to converge) or some parameter goes haywire.
However, compressibleInterFoam can treat 2 species, one fluid and one gas. The fluid can exchange heat with itself and also with the gas (also vice-versa). In this example we simulate the filling of a tank with water, he tank will already have a certain level of water in it (T=300K), a pipe will add additional hotter water into the tank (T=360K). We do not stir the tank, although this would also be an interesting case. It would involve a MRF like the submarine example, but the result result would be a simple one as we basically may calculate the mixing temperature with pen and paper or with Excel (even in a time dependent and transient manner!). 
Now, let's take a look at our simulation domain. The tank is 3m high with a diameter of 1m, has a dia 150mm pipe connecting to the top. This pipe will fill the additional hot water into the vessel. The tank has a little stub pipe on top (dia 100mm), we will use it to blow off air pressure to keep the pressure inside constant. It also has a larger runoff-pipe with dia 250mm. In the first part of the simulation this runoff is closed and will be treated like a wall. Unfortunately, we don't have a valve like boundary condition which we could switch on or off (setting U=0 creates its own problems), so we have to split a simulation into two parts if we want to empty the vessel in a second step. In the first part we will fill some hot water into the already 40% (or so) filled tank in two steps. The velocity profile of the filling pipe looks like this:

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

Essentially, from 1 to 6s we fill in hot water and also from 11 to 16s. In the first step we obviously also have to fill the pipe, so hot water does not flow into the vessel for a few seconds. The we switch off the flow and turn it on again at 11s. Both times we use a ramp of 1s on turn-on. 

