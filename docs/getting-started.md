# Getting started

## The basics

SAPHRON simulations can either be run using the API or using [JSON](http://json.org/) input files.
JSON is a lightweight text format which is easy for humans to read and write and for machines to 
parse and generate. Almost every programming language offers some level of JSON support, making it 
particularly convenient to script or automate input file generation using, say, Python (more on that later).
If you've never used JSON before, don't worry. Throughout the documentation we make no assumptions about 
the user's knowledge of JSON and provide clear easy-to-follow examples.

The SAPHRON executable has two modes: validation and execution. An input file's syntax can be verified 
by running SAPHRON with the `-v` flag. This is a great way to make sure there are no input file errors 
before submitting a job to a HPC cluster. A typical example of this is shown below.

```bash
$ saphron -v < input.json 
                                                                         
 **********************************************************************      
 *         ____      _     ____   _   _  ____    ___   _   _          *      
 *        / ___|    / \   |  _ \ | | | ||  _ \  / _ \ | \ | |         * 
 *        \___ \   / _ \  | |_) || |_| || |_) || | | ||  \| |         *  
 *         ___) | / ___ \ |  __/ |  _  ||  _ < | |_| || |\  |         *    
 *        |____/ /_/   \_\|_|    |_| |_||_| \_\ \___/ |_| \_|         *
 *                                                                    *      
 *   Statistical Applied PHysics through Random On-the-fly Numerics   *
 *                                                                    *     
 *  Version 0.0.7 Revision: a2fe4783e74ca5460c9825553a955b8835d4197b  *      
 **********************************************************************      
                                                                         
 > Validating JSON...                                               OK!
 > Building world(s)...                                             OK!
 * Building world "world0"...
 * Setting size to [20.000000, 20.000000, 20.000000] Å.
 * Setting neighbor list radius to 5.000000 Å.
 * Setting skin thickness to 2.000000 Å.
 * Setting seed to 471762238.
 * Setting temperature to 1.500000K.
 * Initialized 3000 particle(s) of type "LJ".
 > Building forcefield(s)...                                        OK!
 * Initialized 1 forcefield(s).
 > Building move(s)...                                              OK!
 * Initialized Translate move.
 > Building observer(s)...                                          OK!
 * Initialized DLMFile observer.
 * Set sampling frequency to 1 sweeps.
 * Initialized XYZ observer.
 * Set sampling frequency to 1 sweeps.
 > Building simulation...                                           OK!
$ 
```

SAPHRON will provide a basic summary of the settings supplied in the input file. To actually run the 
simulation, simply remove the `-v` flag, and off it goes. At the completion 
of a simulation, some general CPU usage statistics are provided to help the user optimize their 
input files.

#### A note on ensembles

One of the most important things to remember is that in SAPHRON, **ensembles are not 
defined explicitly**. To offer maximum flexibility, an ensemble is a consequence of the 
moves specified by the user, not the other way around. Check the examples section for 
ways of reproducing common ensembles in SAPHRON.

## JSON

A SAPHRON input file is a valid JSON document. Here we will define a bit of terminology 
relating to JSON. Take the following JSON structure as an example, obtained from 
[Wikipedia](https://en.wikipedia.org/wiki/JSON#JSON_sample):

```json
{
	"firstName": "John",
	"lastName": "Smith",
	"age": 25,
	"address": {
		"streetAddress": "21 2nd Street",
		"city": "New York",
		"state": "NY",
		"postalCode": "10021"
	},
	"phoneNumber": [
		{
			"type": "home",
			"number": "212 555-1234"
		},
	    {
			"type": "fax",
			"number": "646 555-4567"
		}
	],
	"gender": {
  		"type": "male"
  	}
}
```

The first pair of curly brackets define the *root* section, we will signify this using `#`.
An item in the hierarchy, such as the street address, can be referenced like this: 
`#/address/streetAddress`. 

Before moving on to SAPHRON specific JSON, we'll mention a few more things about JSON.

- Square brackets `[]` in JSON refer to *arrays*, while curly brackets refer to *objects*. 
They can be thought of as Python lists and dictionaries respectively. That would make 
`#/phoneNumber` an array of phone number objects, each containing a type 
and a number. The fax number can be referenced by `#/phoneNumber/1/number`, where `1` is 
the array index beginning from zero.

- Items in a JSON object (Python dictionary) are unique. In the example above, `#/age` can 
only be defined once - it is a key in the root tree. Defining `#/age` again will not throw 
an error, but instead the last definition will override any previous definitions. This is 
actually very powerful behavior. It means a user can import a general forcefield for example 
and override whatever parameters they wish. The exact behavior of the merging process is 
described in detail in the user guide.

- Types matter in JSON. Notice how `#/age` is specified by a number that is not surrounded in
quotes. This is a number, more specifically an integer. `#/address/postalCode` on the other 
hand is a string, even though the contents of the string are all numbers. Some fields in SAPHRON 
may require the input to be a string, integer or number and the user should be aware of this difference.

## Units 

SAPHRON supports both real and reduced units. For a detailed breakdown of the real units
used in a simulation, check out the [simulation](user-guide/simulation.md) page in the 
user guide.

## Particles

A particle is a general object that can be manipulated in a simulation. Molecules, 
atoms and spins are all *types* of particles. This abstraction is part of what gives 
the SAPHRON engine its flexibility. Particles can also be thought of as composite 
objects. They can have children, like a molecule, or be a "primitive", like an atom or 
a spin. Currently, the SAPHRON input file supports one level of depth - basically 
a molecule/atom hierarchy. 

The structure of a particle is defined by its topology, or *blueprint*. A blueprint is nothing 
but a description of a particle's topology. For something simple, like a spin or Lennard-Jones
particle, it would look like this: 

```json
"blueprints" : {
	"LJ" : {
		"masss" : 1.0,
		"charge" : 0.0
	}
}
```

The above example shows an neutral unit LJ particle. To introduce a bit of terminology: "LJ" is 
the *species* name of the particle. A more complex topology, like that of SPCE water would be

```json
"blueprints" : {
	"H2O" : {
		"children" : [
			{
				"species" : "O",
				"charge" : -0.84760,
				"mass" : 15.9994
			},
			{
				"species": "H1",
				"charge" : 0.42380,
				"mass" : 1.00794
			},
			{
				"species": "H2",
				"charge" : 0.42380, 
				"mass" : 1.00794
			}
		]
	}
```

Let's go through this in detail: we have defined a particle of type "H2O" (which is a molecule), with 
three children defined in an array. Each child has a species, charge and mass specified. A few things 
stand out. The first is that we have not defined any bonds, bends or positions of the water molecule. 
In SAPHRON particles are rigid by default; this includes molecules. Since SPCE is a rigid model, 
we do not need to specify any bonds or bends. Specifying bonds are used for evaluating bonded interactions. 

!!!note
	SAPHRON does not yet support flexible (nearest)-neighbor interaction scaling/exclusion. The default
	behavior is to exclude all nonbonded interactions for bonded pairs. Nonbonded interactions are 
	evaluated for all non-bonded pairs within a molecule.

The internal coordinates within the H2O molecule are also not specified. When specifying a world (see below), 
the coordinates of primitive particles, in this case atoms, are defined in the system (think XYZ file). 
This is done since flexible molecules may have different configurations. In a sense, "H2O" can 
be thought of as a container for three atoms. That means non-primitive particles cannot have a defined 
charge, mass or position. The three quantities are calculated properties from the children.

## Worlds 

A simulation box is called a "world" in SAPHRON. An arbitrary number of worlds are supported, each containing 
an arbitrary number of particles. Along with containing particles, a world also has properties, some specified 
and some calculated. Since a world is responsible for providing a move with a random particle, the user can define 
a seed for the random number generator. User defined properties of a world include dimensions, temperature, and
the chemical potential of species within the world. This means that different worlds can have different temperatures 
and species chemical potentials, allowing for some very creative "ensembles". Calculated properties include 
pressure, energy, dimensions (in non-constant volume ensembles) and in some cases chemical potential.

There are two other important properties that required for a world: neighbor list cutoff and skin thickness.
Each forcefield is responsible for specifying its cutoff radii (they can be vary between worlds), but the 
world is responsible for generating the neighbor lists for its particles. The neighbor list cutoff in a world
should be equal to or exceed the largest specified forcefield cutoff radius, "rcmax". Another subtle point is the 
skin thickness. This value should be chosen to be, *at maximum* the difference between "rcmax" and the neighbor 
list cutoff. Make the value any smaller and you run the risk of excluding particles from an energy calculation 
that should be included. However, this is not guaranteed since it is of a stochastic nature. These rules are 
not enforced by SAPHRON to offer the user the flexibility for those who like to live on the edge!

Using the LJ blueprint from above, we define a world with 300 LJ atoms

```json
"worlds" : [
	{
		"type" : "Simple",
		"dimensions" : [10, 10, 10],
		"seed" : 300, 
		"nlist_cutoff" : 3.5,
		"skin_thickness": 0.5,
		"components" : [
			["LJ", 100]
		], 
		"particles" : [
			[1, "LJ", [4.881841346822,6.371867327468,4.106571184116]],
			[2, "LJ", [2.530762553678,3.610971102018,5.829908319858]],
			[3, "LJ", [1.947804017482,6.637837197361,9.390609698797]],
			[4, "LJ", [9.765735290537,8.258298245790,7.792001193077]],
			[5, "LJ", [3.097831629217,9.321038282277,9.453535007431]],
			...
		]
	}
]
```

Notice how we've defined a world type. For now "Simple" is the only option, which represents an orthorhombic 
box, but non-orthorhombic geometry support is coming soon. `#/worlds/0/components' defines the number of each *high level* 
species in the box. In this case, LJ is both the high level type and primitive. Under '#/worlds/0/particles' a unique ID 
is given to each primitive particle, the species is defined and finally the coordinates are given. For SPCE water this 
would look like

```json
"worlds" : [
	{
		"type": "Simple",
		"dimensions": [20, 20, 20],
		"seed": 300,
		"nlist_cutoff": 10.0,
		"skin_thickness" : 0,
		"components" : [
			["H2O", 100]
		],
		"particles" : [
			[1, "O", [4.7730209484199992, 1.5620128626400014, 1.5253242702800005]],
			[2, "H1", [4.0459440059599991, 0.9341800149600008, 1.8031462188700003]],
			[3, "H2", [5.1428368446299997, 1.2737801963600006, 0.6420586630000003]],
			[4, "O", [8.0901735057700002, 4.5440428079500004, 1.2051669896099995]],
			[5, "H1", [8.7068419886699999, 5.2474487816600011, 0.8516984360500004]],
			[6, "H2", [7.8721620900899989, 3.8950436691600006, 0.4762835006500001]],
			...
		]
	}
]
``` 

Here the number of H2O molecules is specified and the coordinates of each atom is given under `#/worlds/0/particles`. 
Since there are 100 H2O molecules, SAPHRON would expect 300 atoms in the order in which they appear in the blueprint. 
More options are available for a world and individual particle directors can also be specified. Details are available 
in the [user guide](user-guide/worlds.md). There are additional options to pack a world with particles. 

## Observers

Observers are objects that "observe" a simulation as it proceeds and take notes. In other words, observers can be 
thought of as loggers. SAPHRON offers a number of different observers, including a JSON observer which writes 
checkpoint JSON files, a general delimited file observer which writes various properties and an XYZ observer which 
writes snapshots of worlds to XYZ files. There can be an arbitrary number of observers logging different 
properties at independent frequencies. An example is shown below

```json
"observers" : [
	{
		"type" : "DLMFile",
		"frequency" : 1,
		"prefix" : "test",
		"flags" : {
			"simulation" : 1,
			"world" : 1
		}
	},
	{
		"type" : "JSON",
		"frequency" : 1000,
		"prefix" : "test_checkpt"
	},
	{
		"type" : "XYZ", 
		"frequency" : 100,
		"prefix" : "test_xyz"
	}
]
```
We've defined three different observers above. The first is a fixed-width space delimited logger recording 
simulation and world properties every iteration. The second is a JSON observer writing checkpoint files every 
1000 iterations. The third is an XYZ observer recording snapshots every 100 iterations. Note the `prefix`
specification which is used for the file names. Also, multiple observers of the same type can be defined to 
log the same or different properties independent of one another. A detailed list of options and flags is located 
in the [user guide](user-guide/observers.md).

## Input file structure

A SAPHRON input file consists of a collection of JSON objects and arrays. The main 
sections are outlined below:

* `#` - The JSON root encompasses everything and defines an entire simulation.
* `#/worlds` - Array defining the different "worlds", or boxes, in the system.
* `#/moves` - Array defining the moves to be performed on the worlds.
* `#/forcefields` - Array defining the bonded, nonbonded and electrostatic interactions.
* `#/blueprints` - Object containing the blueprints, or topologies, of the particles in the simulation.
* `#/observers` - Array defining all of the observers, or loggers, that will collect data throughout the simulation.
* `#/DOS` - Object containing settings for DOS simulations. 
* `#/Histogram` - Object containing settings for histograms.

An empty shell containing some of the above looks like this:

```json
{
	"blueprints" : {
	
	},
	"forcefields" : {
		"nonbonded" : [
		],
		"bonded" : [
		],
		"electrostatic" : [
		]
	},
	"worlds" : [
	],
	"moves" : [
	],
	"observers" : [
	]
}
```
