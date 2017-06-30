% SYN2: synapse connectivity file format for large neuronal networks

# Abstract

SYN2 is a proposed file format specification adapted to store the neuronal connectivity (synapses) of large neuronal network.
It is design to be extensible and adapted to both point to point neuronal connectivity and detailed neuronal connectivity.


**Authors:**
Adrien Devresse <adrien.devresse@epfl.ch>
Till Schumann <till.schumann@epfl.ch>
James Gonzalo King <jamesgonzalo.king@epfl.ch>


**Date:** January 27, 2017

**Status:** Draft



# INTRODUCTION

## Terminology
This specification follows the wording of the IETF RFCs, as defined by [rfc2119](https://www.ietf.org/rfc/rfc2119.txt)


    * S: number of synapses
    * N: number of neuron
    * neuron_id: unique identifier (integer) of a neuron

## Motivations

It aims to unify and correct the known problems of the following file formats:

 * The BBP internal synapse file format ( NRN.h5 )
 * The Nest synapse file format
 * The BBP TouchDetector proprietary file format


# REQUIREMENTS

## Compatibility requirements

 * MUST be extensible to additional synapse properties WITHOUT breaking backward reader compatibility
 * MUST be adapted to store both point to point and detailed neuronal connectivity
 * MUST support heterogenous population of neurons ( namespaces )
 * MUST be able to store and retrive any kind of data and meta-data of the historical BBP file format

## Scale requirements

 * MUST be adapted to store TeraByte (up to Petabyte) scale neuronal synapse connectivity :
     * MUST be able to be loaded partially (out of core)
     * MUST be resistant to corruption
 * MUST support parallel I/O ( read and write ) operations from multiple nodes
 * MAY support compression if possible

## Performance requirements

 * MUST be able to query the following information with a maximum complexity of O(log(N_synapses))
    1. List of synapses associated to one neuron in pre-Synaptic view
    2. List of synapses associated to one neuron in post-Synaptic view
    3. List of synapses common to two neurons
    4. Any property associated to a given synapse identified by its synapse id

An acceptable query time for a query of type (1) in case of a local dataset SHOULD be < 1s


List of the main design requirements for this file format :


## Substainability requirements

* MUST be a portable file format independent of any computer architecture( big endian, small endian )
* MUST be programming language agnostic
* MUST be any database engine agnostic


## Usability requirements
* SHOULD be based on existing open standard
* SHOULD be a self-described layout



# SYN2 CONTAINER

The SYN2 defines a hierachy of datatypes that MAY be used on top of any kind of container supporting hierachical storage.
However, any implementation of SYN2 SHOULD support the [Hierarchical Data Format 5 (HDF5)](https://support.hdfgroup.org/HDF5/doc/H5.format.html) format as container.

Files in the SYN2 formats SHOULD use the .syn2 extension.

group, attributes and dataset in a SYN2 file MUST uses the UTF-8 encoding standard.

# SYN2 HIERARCHY AND DATATYPES

## Layout

The SYN2 containers MUST follow the following hierachy layout.

> /
>
> /synapses
>
> /synapses/{population_name}
>
> /synapses/{population_name}/properties
>
> /synapses/{population_name}/properties/{property_name}
>
> /synapses/{population_name}/indexes
>
> /synapses/{population_name}/indexes/{index_group_name}
>
> /synapses/{population_name}/indexes/{index_group_name}/{index_data}





### population_name

namespace for a population of synapses. A population of synapses MUST BE independant
of each other and MAY have their own properties. This allows the
representation of several heterogenous collections of synapses in a single file.
( e.g point to point, detailed, ... ).

In case of a single unamed population, the name "default" SHOULD be used.



### property_name
Each property of a synapse collection is representated by a separated dataset, column oriented.
The list of mandatory and optional supported properties are define in the [following section](#properties)

### index_group-Name
Each index group contains the associated index dataset related to the indexing of on or several associated [property_name](#property_name) dataset


## Properties {#prop_anchor}

### connected_neurons_pre
description: Pre-synaptic neuron connections. each row is the neuron_id identifier of the pre-synaptic neuron.

This property MUST be defined.


dataset_size: 1xS

datatype: INT64


### connected_neurons_post
description: Post-synaptic neuron connections. Each row is the neuron_id identifier of the post-synaptic neuron.

This property MUST be defined.


dataset_size: 1xS

datatype: INT64


### position
description: Synapse spatial position. x,y,z coordinates.

This property is optional


dataset size: 3xS

datatype: DOUBLE


### delay
description: Synapse propagation delay. Used in point to point and detailed models.

This property is optional


dataset size: 1xS

datatype: FLOAT


### weight
description: Synapse weight. Used in point to point model.

This property is optional


dataset size: 1xS

datatype: FLOAT


### u0
description: Synapse u0 parameter. Used in point to point model.

This property is optional


dataset size: 1xS

datatype: FLOAT


### conductance
description: The conductance of the synapse . Used in detailed model.

This property is optional


dataset size: 1xS

datatype: FLOAT

unit: nanosiemens


### u_syn
description: The u parameter in the TM model. Used in detailed model.

This property is optional


dataset size: 1xS

datatype: FLOAT

unit: model specific


### depression_time
description: The time constant of depression. Used in detailed model.

This property is optional


dataset size: 1xS

datatype: INT64

unit: milliseconds


### facilitation_time
description: The time constant of facilitation. Used in detailed model.

This property is optional


dataset size: 1xS

datatype: INT64

unit: milliseconds


### decay_time
description: The time constant of decay. Used in detailed model.

This property is optional


dataset size: 1xS

datatype: INT64

unit: milliseconds


### absolute_efficacy
description: The Absolute Synaptic Efficacy. Used in detailed model.

This property is optional


dataset size: 1xS

datatype: INT64

unit: millivolts


### n_mvr
description: Docked vesicles. Used in detailed model.

This property is optional


dataset size: 1xS

datatype: INT64

unit: number

### syn_type_id
description: Synapse type id as used in the recipe. Used in detailed model.
The id can be any arbitrary INT64 value refering to an external model id

This property is optional


dataset size: 1xS

datatype: INT64

unit: recipe specific id


### morpho_section_id_pre
description: section id of the touched segment associated with the pre-synaptic neuron.
The ID comes is associated with the morphology model.
Used in detailed model.


This property is optional


dataset size: 1xS

datatype: INT64

unit: morphology specific id


### morpho_section_distance_pre
description: distance from the betweenthe beginning of the associated section on the pre-synaptic neuron
and the touch. Distance in micro-meter.


This property is optional


dataset size: 1xS

datatype: FLOAT

unit: micro-meters


### morpho_section_id_post
description: section id of the touched segment associated with the post-synaptic neuron.
The ID comes is associated with the morphology model.
Used in detailed model.


This property is optional


dataset size: 1xS

datatype: INT64

unit: morphology specific id


### morpho_section_distance_post
description: distance from the betweenthe beginning of the associated section on the post-synaptic neuron
and the touch. Distance in micro-meter.


This property is optional


dataset size: 1xS

datatype: FLOAT

unit: micro-meters


### position_center_pre
description: Pre-synaptic touch position in the center of the segment. xyz positions

This property is optional


dataset size: 3xS

datatype: DOUBLE


### position_center_post
description: Post-synaptic touch position in the center of the segment. xyz positions

This property is optional


dataset size: 3xS

datatype: DOUBLE


### position_contour_pre
description: Pre-synaptic touch position on the surface of the segment. xyz positions

This property is optional


dataset size: 3xS

datatype: DOUBLE


### position_contour_post
description: Post-synaptic touch position on the surface of the segment. xyz positions

This property is optional


dataset size: 3xS

datatype: DOUBLE


### nrn_line
description: Synapse position in corresponding postsynaptic GID dataset in nrn.h5.
Used for retro-compatibility with NRN; would be dropped eventually.

This property is optional


dataset size: 1xS

datatype: INT64


## Indexes

### connected_neurons_{pre,post} indexes
description: Index groups used for Neuron to Synapse mapping
in O(1)-O(log(S)) maximum time complexity.

The layout of the index allows an indexation from pre-synatic and post-synaptic neuron
perspective.

The SYN2 connected_neurons index allows a complete flexibility in the storage
of the synapse properties and allow to optimise the storage of properties for specific queries:
 * pre-synaptic queries
 * post-synaptic queries
 * out-of-order writing





### connected_neurons_pre/neuron_id_to_range
description: Index for pre-synaptic neuron to synapse association.
This index provide the range of *connected_neurons_pre/range_to_synapse_id* rows
associated with each neuron

    * row: neuron_id
    * column[0]: beginning of the range in index connected_neurons_pre/range_to_synapse_id
    * column[1]: non-inclusive end of the range in the index connected_neurons_pre/range_to_synapse_id


SHOULD be provided.


dataset size: 2xN

datatype: INT64

constrains: column[0] < column[1]. A negative value in column 0 indicates that no synapse is associated to a given neuron


### connected_neurons_pre/range_to_synapse_id
description: Index for pre-synaptic neuron to synapse association.
This index provides a range of *properties* per index row

    * column[0]: beginning of the range of synapses *properties*
    * column[1]: non-inclusive end of synapses *properties*


SHOULD be provided.


dataset size: 2x O(S)

datatype: INT64

constrains: column[0] < column[1]


### connected_neurons_post/neuron_id_to_range
description: same than *connected_neurons/neuron_to_range_pre*
but for post-synaptic neurons


### connected_neurons_post/range_to_synapse_id
description: same than *connected_neurons/range_to_synapse_pre*
but for post-synaptic neurons


## Attributes

### Version

SYN2 files MUST define two attributes for file format versionning

"/synapses#version"

    * associated elent : "/synapses"
    * key : "version_major"
    * datatype : "INT8"
    * dataset size : 2x1
    * value "[ 1, 0 ]"




# SCALE CONSIDERATIONS

Each dataset stored in a SYN2 HDF5 file MIGHT be compressed using the DEFLATE compression algorithm.

Each dataset stored in a SYN2 HDF5 file MIGHT use the HDF5 external reference feature to split the layout into several
HDF5 files. If doing so, every separated file part of the layout SHOULD have the following extension .syn2.part


# PERFORMANCE CONSIDERATIONS


Even if not mandatory, We recommend to store the synapses *properties* double-sorted by pre-synaptic neuron and
post-synaptic neuron to optimise the size of the associated index.


# EXTENSIBILITY

Additional properties can be defined with a new {property_name} dataset.



# SEE ALSO

 - [MVD3 File fornat](https://bbpteam.epfl.ch/project/spaces/display/BBPWFA/MVD+version+3+-+Draft+0.0.1)
 - [NRN file format](https://bbpteam.epfl.ch/project/spaces/pages/viewpage.action?pageId=10919530)
 - [NRN position file format](https://bbpteam.epfl.ch/project/spaces/display/BBPHPC/Synapses+-+nrn_positions.h5)
 - [NRN extra](https://bbpteam.epfl.ch/project/spaces/display/BBPHPC/Synapses+-+nrn_extra.h5)



