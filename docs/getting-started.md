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

## Concepts

To better understand the documentation, we describe a few key concepts used in SAPHRON
which may not be immediately obvious to the user.

#### Particles

#### Worlds 

#### Observers

### Blueprints


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
`#/address/streetAddress`. Before moving on to SAPHRON specific JSON, it is important 
to recognize that square brackets `[]` in JSON refer to *arrays*, while curly brackets
refer to *objects*. They can be thought of as Python lists and dictionaries respectively. 
That would make `#/phoneNumber` an array of phone number objects, each containing a type 
and a number. The fax number can be referenced by `#/phoneNumber/1/number`, where `1` is 
the array index beginning from zero.

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
