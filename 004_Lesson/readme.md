# Lesson 4: Heat Transfer

## Background

A steady-state heat transfer analysis of a unit block is performed. The top edge (B) of the block is prescribed with a distributed flux, while the bottom edge (A) has a prescribed temperature. All other edges are insulated. The thermal conductivity of the block material is <img src="/004_Lesson/tex/417e9d7f078cdb8c16592aee5f8c7687.svg?invert_in_darkmode&sanitize=true" align=middle width=39.21220214999999pt height=22.831056599999986pt/>. 

The flux associated with conduction across a surface is given by<sup>[a](#myfootnote1)</sup>: 

<img src="/004_Lesson/tex/ae408535435f2a09ffe3f2fa5136a230.svg?invert_in_darkmode&sanitize=true" align=middle width=114.88108664999999pt height=45.072403200000004pt/>

where <img src="/004_Lesson/tex/2f128f854fd9ff3109e6b9c75fa629a0.svg?invert_in_darkmode&sanitize=true" align=middle width=13.54268354999999pt height=14.15524440000002pt/> is the surface flux, <img src="/004_Lesson/tex/c2e026b8a86e00d5c521886fb6a64d89.svg?invert_in_darkmode&sanitize=true" align=middle width=10.502226899999991pt height=14.611878600000017pt/> is the surface normal, <img src="/004_Lesson/tex/4d05b05c2efa8462d93f554065731f8f.svg?invert_in_darkmode&sanitize=true" align=middle width=9.97711604999999pt height=22.831056599999986pt/> is the conductivity tensor, and <img src="/004_Lesson/tex/27e556cf3caa0673ac49a8f0de3c73ca.svg?invert_in_darkmode&sanitize=true" align=middle width=8.17352744999999pt height=22.831056599999986pt/> is the temperature. For our 1-dimensional problem, the above can be reduced to a scalar equation:

<img src="/004_Lesson/tex/97f0a98c672fd59814696638bd7a99ef.svg?invert_in_darkmode&sanitize=true" align=middle width=175.60577264999998pt height=45.072403200000004pt/>

since <img src="/004_Lesson/tex/417e9d7f078cdb8c16592aee5f8c7687.svg?invert_in_darkmode&sanitize=true" align=middle width=39.21220214999999pt height=22.831056599999986pt/> and <img src="/004_Lesson/tex/e937ba1d3e4f9e2abb84171f9ab72a0d.svg?invert_in_darkmode&sanitize=true" align=middle width=53.23049984999998pt height=22.465723500000017pt/>. This gives:

<img src="/004_Lesson/tex/24f88f149536bd362616cb16039705a9.svg?invert_in_darkmode&sanitize=true" align=middle width=93.00760919999999pt height=22.831056599999986pt/>. 

which can be used to check the accuracy of the thermal conduction procedure.

## Node definition

Nodes are defined using *NODE:

	*NODE, NSET=GLOBAL
		   1,          0.,          0.,          0.
		   2,         0.5,          0.,          0.
		   3,          1.,          0.,          0.
		   4,          1.,         0.5,          0.
		   5,          0.,         0.5,          0.
		   6,         0.5,         0.5,          0.
		   7,          0.,          0.,        -0.5
		   8,         0.5,          0.,        -0.5
		   9,          1.,          0.,        -0.5
		  10,          1.,         0.5,        -0.5
		  11,          0.,         0.5,        -0.5
		  12,         0.5,         0.5,        -0.5
		  13,          1.,          1.,          0.
		  14,          0.,          1.,          0.
		  15,          0.,         0.5,          0.
		  16,          1.,         0.5,          0.
		  17,          1.,          1.,        -0.5
		  18,          0.,          1.,        -0.5
		  19,          0.,         0.5,        -0.5
		  20,          1.,         0.5,        -0.5

## Element definition

The block is composed of three 8-node brick elements of type DC3D8 (note the D in front). 

	*ELEMENT, TYPE=DC3D8, ELSET=ALL
		  1,      7,      8,     12,     11,      1,      2,      6,      5
		  2,      8,      9,     10,     12,      2,      3,      4,      6
		  3,     19,     20,     17,     18,     15,     16,     13,     14

All the elements are grouped in the element set ALL.		  

## Node groups

We define the group for the nodes at the base of the model on which we will be prescribing the nodal temperatures.

	*************************************************
	** Nodes at the base of the model
	*************************************************
	*NSET, NSET=EDGEA
	1,2,3,7,8,9

## Element groups
	
We create the element groups for the top half and bottom half of the model. These elements will be used to define surfaces for thermal contact and prescribed flux as described in Surface interaction below.

	*************************************************
	** Bottom Blocks
	*************************************************
	*ELSET, ELSET=BOT_BLOCK
	1,2
	*************************************************
	** Top Block
	*************************************************
	*ELSET, ELSET=TOP_BLOCK
	3
		  
## Element property

	*SOLID SECTION,ELSET=ALL,MATERIAL=M1	

## Material definition

The sole material property associated with all elements is the thermal conductivity.

	*MATERIAL,NAME=EQUIL
	*CONDUCTIVITY
	1., 

To make things more interested, we also test the use of a non-conforming mesh where the block is split into top and bottom halves, which are meshed separately and tied together<sup>[b](#myfootnote1)</sup>. 

## Surface interaction

For the interaction properties of the tied surface is specified using *SURFACE INTERACTION:

	*Surface Interaction, name=Thermal_Contact	
	*Gap Conductance
	 1.e6,   0.
	 1.e6,   0.05

The conductance properties of the surface is prescribed using the keyword *GAP CONDUCTANCE. Here, a large thermal conductivity (<img src="/004_Lesson/tex/876fd957d8cc6f84f8dc48ba76a1a494.svg?invert_in_darkmode&sanitize=true" align=middle width=24.09255749999999pt height=21.18721440000001pt/>) of the tied surface is specified at zero distance. Note: Abaqus requires two pairs of points are needed; the second line is just a repeat at some nonzero distance away from the surface.
		
## Thermal surfaces

Using the element groups defined earlier, we define the following surfaces:

	************************************************
	** Top surface of top block
	************************************************
	*Surface, type=ELEMENT, name=TOP_BLOCK_TopSurf
	TOP_BLOCK, S5
	************************************************
	** Bottom surface of top block
	************************************************
	*Surface, type=ELEMENT, name=TOP_BLOCK_BotSurf
	TOP_BLOCK, S3
	************************************************
	** Top surface of bottom blocks
	************************************************
	*Surface, type=ELEMENT, name=BOT_BLOCK_TopSurf
	BOT_BLOCK, S5

## Thermal contact
		
The top and bottom blocks are tied together using *CONTACT PAIR:
	
	*Contact Pair, interaction=Thermal_Contact, type=SURFACE TO SURFACE
	BOT_BLOCK_TopSurf, TOP_BLOCK_BotSurf
		
The slave surface is associated with the bottom half and the master surface is associated with the top half of the block.

The block has 1 element at the top half and 2 elements at the bottom half. The base of the block (side A) is prescribed a temperature, which is ramped from 0 to 500 within duration of 1.0. The opposite side (top; side B) of the block has a uniform distributed flux <img src="/004_Lesson/tex/2f128f854fd9ff3109e6b9c75fa629a0.svg?invert_in_darkmode&sanitize=true" align=middle width=13.54268354999999pt height=14.15524440000002pt/>, which is prescribed using the keyword *DSFLUX and ramped from 0 to 10 using the keyword *AMPLITUDE.

## Amplitude function definition

We define an amplitude function, named <em> RAMPQ </em>, to ramp the heat flux from <img src="/004_Lesson/tex/29632a9bf827ce0200454dd32fc3be82.svg?invert_in_darkmode&sanitize=true" align=middle width=8.219209349999991pt height=21.18721440000001pt/> to <img src="/004_Lesson/tex/b0c08f9b595a704efb907fc688034d80.svg?invert_in_darkmode&sanitize=true" align=middle width=16.438418699999993pt height=21.18721440000001pt/> from time <img src="/004_Lesson/tex/29632a9bf827ce0200454dd32fc3be82.svg?invert_in_darkmode&sanitize=true" align=middle width=8.219209349999991pt height=21.18721440000001pt/> to <img src="/004_Lesson/tex/034d0a6be0424bffe9a6e7ac9236c0f5.svg?invert_in_darkmode&sanitize=true" align=middle width=8.219209349999991pt height=21.18721440000001pt/>:

	*************************************************
	** Ramp function
	*************************************************
	*AMPLITUDE, NAME=RAMPQ, DEFINITION=TABULAR
	0.,0.,    1.,10.

## Analysis step

Heat transfer analysis is performed using the keyword <em> *HEAT TRANSFER </em> and with the option <em> STEADY STATE </em>. The  time stepping parameters are identical to those used in static analyses (for e.g., see [Lesson 1](./../01_Lesson)).

	*STEP,INC=50
	*HEAT TRANSFER,STEADY STATE
	0.1,1.0,0.25,0.1
	**
	** Prescribed nodal temperature at base of model
	**
	*BOUNDARY
	EDGEA,11,11,500.
	**
	** Prescribed flux at the top surface of the model
	**
	*DSFLUX, AMPLITUDE=RAMPQ, OP=MOD
	TOP_BLOCK_TopSurf,S,1.

The boundary conditions are the nodal temperatures (degree of freedom <img src="/004_Lesson/tex/7ee94e64f8d5936cc5f263d0ed987bee.svg?invert_in_darkmode&sanitize=true" align=middle width=16.438418699999993pt height=21.18721440000001pt/>) of <img src="/004_Lesson/tex/25df05ed4b8476cb9a1a3db76ae8f22c.svg?invert_in_darkmode&sanitize=true" align=middle width=24.657628049999992pt height=21.18721440000001pt/> at the base of the model. The reference value of the heat flux of <img src="/004_Lesson/tex/f58ed17486d1735419372f2b7d091779.svg?invert_in_darkmode&sanitize=true" align=middle width=21.00464354999999pt height=21.18721440000001pt/> is prescribed at the top of the model using the keyword <em> *DSFLUX </em>. During analysis, this reference valued will be scaled by the amplitude function <em> RAMPQ </em> defined above.
	
## Viewing results	

After performing the steady-state heat transfer analysis, we can visually check the nodal temperatures of the model.

![](./abaqus_input_files/Lesson004_Step1_Frame11_NT11.png)

At the end of the analysis, the prescribed temperature at the base is <img src="/004_Lesson/tex/0e51e3366d6fb575fb54c79363d576af.svg?invert_in_darkmode&sanitize=true" align=middle width=64.99995809999999pt height=22.831056599999986pt/> and the prescribed flux at the top is <img src="/004_Lesson/tex/3d767a357f45c1881feec4702c5d780c.svg?invert_in_darkmode&sanitize=true" align=middle width=52.72064489999999pt height=21.18721440000001pt/>. Therefore, the temperature at the top should be <img src="/004_Lesson/tex/6099b6c3a9463ee47dddec53d978c4f9.svg?invert_in_darkmode&sanitize=true" align=middle width=140.40478155pt height=22.831056599999986pt/>. The nodal temperatures at the top jibe with this analytical value.

## Exercise 

None

---
## Footnotes

<a name="myfootnote1">a</a>) Abaqus Theory Manual > Procedures > Heat Transfer > Convection/Diffusion

<a name="myfootnote2">b</a>) Ties are typically used to as a convenient means to transition between fine and coarse meshes. 

---
## Additional Comments on This Lesson (Links to Milo)

https://milo.sgh.com/community/software/blog/2019/05/24/elementary-abaqus-heat-transfer-analysis

