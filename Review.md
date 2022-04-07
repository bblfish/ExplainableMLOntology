# Review of ontology

We seem to have a number of problems

## Messy github repo

The [original github repo](https://github.com/pNakagawa/ExplainableMLOntology) at this time has 5 commits that were all made the same day. So the github repository was not used to work on
the ontology and improve it in an open source manner. 

The key directory [Protege_Ontologies](https://github.com/pNakagawa/ExplainableMLOntology/tree/1a250665723b8ec8f7d047726c25e914d235f794/Protege_Ontologies) contains numbered files which suggest some form of versioning, and an [Old](https://github.com/pNakagawa/ExplainableMLOntology/tree/1a250665723b8ec8f7d047726c25e914d235f794/Protege_Ontologies/Old) directory all of whose files are numbered.

The [README](https://github.com/pNakagawa/ExplainableMLOntology/tree/1a250665723b8ec8f7d047726c25e914d235f794) does not explain how those 
numbers are to be interpreted, leaving it up to the reader to have
to work their way through these files to understand what each is
doing, which is the latest, etc... 

This contrasts to the much simpler layout of the [ML-Schema/core](https://github.com/ML-Schema/core) repository for example.

### Structure of the github repo ontology files

So we are left to have to guess what the files are doing
As I go through the content I will fill in this section.


* [MLOntology_Specific.ttl](Protege_Ontologies/MLOntology_Specific.ttl): this seems to be a file handwritten that contains concepts similar to the MLS ontology (`Run`, `Feature`, `Software`) but it also contains Covid19 concepts such as `Age`, `AgeCorrelatesDiabetes`, `Covid-19`,...)
* [MLOntology_General2.ttl](Protege_Ontologies/MLOntology_General2.ttl) contains the generic classes equivalent it seems to [MLS](https://ml-schema.github.io/documentation/ML%20Schema.html) such as `Task`, `ModelCharacteristic`, `Algorithm`, ... but with new types that seem to be related to the desire to only use upper level `gufo` ontology relations...
* [MLS_UFO.owl.ttl](Protege_Ontologies/MLS_UFO.owl.ttl): is an ontology for the Covid19 examples described in the paper and the thesis defining terms such as `Obesity`, `Sex`, `Covid-19`,... But it also contains definitions of concepts with the same names as those [defined in MLS](https://ml-schema.github.io/documentation/ML%20Schema.html). E.g. `Run`, `Experiment`, `Model`, `Implementation`, `Algorithm`, `Data` and some that are closely related such as `Implements`,`DataCharacteristic`... Why not just link to the terms defined elsewhere? That would immediately make clear what the purpose of this file was.
* [MLS_UFO_ONTOUML.ttl](Protege_Ontologies/MLS_UFO_ONTOUML.ttl): This looks like a rewrite of the [MLS Ontology](https://ml-schema.github.io/documentation/ML%20Schema.html) redefined using only the gufo ontology.


Other files
* [MLSchema.owl.ttl](Protege_Ontologies/MLSchema.owl.ttl) is a copy of the MLS ontology that Henry Story placed here for easy access
* [Run.ttl](Protege_Ontologies/Run.ttl): a merging of a couple of rdf files Henry Story put together (which ones?) 
 
## Classes are labelled as owl:NamedIndividuals

In [MLS_UFO_ONTOUML.ttl](Protege_Ontologies/MLS_UFO_ONTOUML.ttl) has statements like

```turtle
extml:Run rdf:type owl:Class, gufo:EventType, owl:NamedIndividual;
    rdfs:subClassOf gufo:Event;
    rdfs:label "Run"@en.    
```

Here we have objects that are both NamedIndividuals and Classes (as we can see by the `rdfs:subClassOf` relation). But owl semantics does not allow an individual to be a class. [Owl2 Semantics](https://www.w3.org/TR/2012/REC-owl2-direct-semantics-20121211/#Interpretations) specifies
Individual Names and Class Names, and interpretation functions from individual names to members of the Domain Δ  and from Class Names to subsets of Δ. Elements are distinct from sets.

It is possible with `punning` (see [a new feature of owl2](https://www.w3.org/TR/owl2-new-features/#Simple_metamodeling_capabilities)) for the same name to be used in both roles, but the names then refer to different things. So it is not clear how this would help here.

Given the directory layout problem explained previously, it seems
to me that the `owl:NamedIndividual` is likely some mistake that found its way into that file.

But the messy directory layout also means that it is difficult to know if this file is even worth looking at. Is it an old version? Should
another file be looked at? What is its role? A quick analysis of the
ontology files shows that out of 30 of them, 25 use `owl:NamedIndividual`

```bash
$ find . -type f -exec grep -q NamedIndividual {} \; -print
```

So it does not feel as if this construct was well understood.

## Instances are badly named

Note: files in RDF can be transformed to Turtle by using [raptor](https://librdf.org/raptor/),
which can be installed on macos with [brew](https://brew.sh). We can for example translate the
[MLExp_final\ \(6\).owl](Protege_Ontologies/MLExp_final%20(6).owl) file from rdf/xml to the
more readable turtle with:

```zsh
$ rapper MLExp_final\ \(6\).owl -o turtle
```

MLExp probably stands for Machine Learning Experiment, and this contains instances such
as the following.

```Turtle
:ClassifyMortality_exp1
    dc:creator "Patricia"^^xsd:string, """The goal of the experiment 1 is to understand the ML model using the
 RIPPER rule extractor method.
The ML model is used to predict mortality of confirmed cases using Epidemiology dataset of tested people for COVID-19 in Mexico."""^^xsd:string ;
    gufo:participatedIn :Experiment1 ;
    a owl:NamedIndividual, :Goal .
```    

Here `ClassifyMortality_exp1` is an object not a class, and we can see because [gufo:participatedIn](https://nemo-ufes.github.io/gufo/#participatedIn) is defined as a relation that relates a
[gufo:Object](https://nemo-ufes.github.io/gufo/#Object) to a [gufo:Event](https://nemo-ufes.github.io/gufo/#Event). 

So in this case having `:ClassifyMortality_exp1` be an instance of an `owl:NamedIndividual` makes sense, even if stating it does
not add any information since 
 1. the object is clearly named
 2. it is in instance object, since it is to the left of `a` a.k.a. `rdf:type`

Because RDF names everything with URIs, and those don't come
with a syntactic way to distinguish between instances and classes, it makes sense to have a convention. A widely used convention is that class names start with capitals and instances with lower case names.

Finally, the text "The goal of the experiment 1 ..." is not the name of the
creator of the experiment, but rather the [dc:description](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#http://purl.org/dc/elements/1.1/description)

Also we have the 

So we could improve the example by rewriting it like this:

```Turtle
:mortality_classification_exp1 a :Goal;
    dc:creator "Patricia"; 
    dc:annotation """
    The goal of the experiment 1 is to understand the ML model using the RIPPER rule extractor method.
The ML model is used to predict mortality of confirmed cases using Epidemiology dataset of tested people for COVID-19 in Mexico.""";
    gufo:participatedIn :Experiment1 .
```  

That is an improvement but we noticed that the instance and ontology
names are in the same namespace `<https://example.com/>`. 

## No Linked Data patterns

We already saw above that the ontology being worked on does not seem to have a stable namespace. 

The file [MLS_UFO.owl.ttl](Protege_Ontologies/MLS_UFO.owl.ttl), which seems to contain instance data, uses the default namespace
```Turtle
@prefix : <http://www.semanticweb.org/paty_/ontologies/2021/3/untitled-ontology-9#> .
```
That URL is not dereferenceable at present. Much better would have been to use just the relative url `<#>`. Then other files
in the same directory could have referenced the definitions there using a relative url such as `<MLS_UFO#Run1>`. Adding a symbolic link from `<MLS_UFO>` to [MLS_UFO.owl.ttl](Protege_Ontologies/MLS_UFO.owl.ttl) would make those links locally dereferenceable too, for software that knew how to follow links.

This would also make it easy to seperate the instance data
from the ontology itself, by simply placing them in different fils and having the instance data refer to the ontology. For example instead of defining `Run` in `<MLS_UFO>` it could have used the `Run` defined in [MLS_UFO_ONTOUML.ttl](Protege_Ontologies/MLS_UFO_ONTOUML.ttl).

### No Links to the data

The repository helpfully comes with a [data and Code](Data%20and%20Code/) directory, but there is no link to those files such as [dataset.zip](Data%20and%20Code/dataset.zip) or files contained in that archive such as `covid.csv`.

This failure of linking up all the pieces of the puzzle makes the examples more difficult to read. It also fails
to exploit the most important aspect of linked data, namely the ability of software to follow links to reach the data.
The purpose of these ontologies is after all to make it 
possible to reproduce experiments or to evaluate ML Models from the data that was used to train them. If the link to the data is not made or tested then both of those cannot
be realised.

## Is the ExplainableOntology extension to MLS that good?

### gufo:participatedIn replaces mls relations?

In [Protege_Ontologies/MLS_UFO_ONTOUML.ttl](Protege_Ontologies/MLS_UFO_ONTOUML.ttl) a `Run` is
defined in terms of the the [lightweight Unified Foundational Ontology](https://nemo-ufes.github.io/gufo/).

```Turtle
:Run rdfs:subClassOf [
     rdf:type owl:Restriction;
     owl:onProperty [ owl:inverseOf gufo:participatedIn ];
     owl:someValuesFrom :Data
   ], [
     rdf:type owl:Restriction;
     owl:onProperty [ owl:inverseOf gufo:participatedIn ];
     owl:someValuesFrom :HyperParameterSetting
   ], [
     rdf:type owl:Restriction;
     owl:onProperty [ owl:inverseOf gufo:participatedIn ];
     owl:someValuesFrom :Implementation
   ], [
     rdf:type owl:Restriction;
     owl:onProperty [ owl:inverseOf gufo:participatedIn ];
     owl:minQualifiedCardinality "1"^^xsd:nonNegativeInteger;
     owl:onClass :Task
   ].
```

The concept with the similar name in mls is defined as

```Turtle
mls:Run rdfs:subClassOf :Process,
    [ rdf:type owl:Restriction ;
      owl:onProperty mls:hasInput ;
      owl:someValuesFrom mls:Data
    ] ,
    [ rdf:type owl:Restriction ;
       owl:onProperty mls:hasInput ;
       owl:someValuesFrom mls:HyperParameterSetting
    ] ,
    [ rdf:type owl:Restriction ;
      owl:onProperty mls:executes ;
      owl:someValuesFrom mls:Implementation
    ] ,
    [ rdf:type owl:Restriction ;
      owl:onProperty mls:achieves ;
      owl:someValuesFrom mls:Task
    ] .
```

Notice how we have the same constraints with `owl:sameValuesFrom` in
both cases. But the properties in `expl` are always the inverse of 
`gufo:participatedIn` whereas in `mls` they have unique understandable
names. So we loose here by moving from `mls` to `expl`.

Could both work together? Or would it have just been simpler to
declare the precise relations to be subproperties of `participatesIn`
in N3 as follows

```Turtle
[] owl:inverseOf gufo:participatedIn;
   is rdfs:subPropertyOf of mls:hasInput, mls:executes, mls:achieves .
```

That would take care of adding the extra `gufo` triples if needed. 

If there are reasons not to have mls relations be subproperties of the inverse of `gufo:participatedIn` then
it would have been good to have a discussion on this. 
(todo: check: perhaps there is in the masters thesis)

## mls:Task and extml:Task are different

[mls:Task](https://ml-schema.github.io/documentation/ML%20Schema.html#d4e601) is a subclass of [mls:InformationEntity](https://ml-schema.github.io/documentation/ML%20Schema.html#d4e707).

```Turtle
mls:Task rdf:type owl:Class ;
      rdfs:subClassOf mls:InformationEntity ,
                      [ rdf:type owl:Restriction ;
                        owl:onProperty mls:definedOn ;
                        owl:someValuesFrom mls:Data
                      ] ;
      dct:description "Task is a formal description of a process that needs to be completed (e.g. based on inputs and outputs). A Task is any piece of work that needs to be addressed in the data mining process. In ML Schema, it is defined based on data." ;
      rdfs:comment "Task is a formal description of a process that needs to be completed (e.g. based on inputs and outputs). A Task is any piece of work that needs to be addressed in the data mining process. In ML Schema, it is defined based on data." .
```

On the other hand an `extml:Task` is a subclass of [gufo:Event](https://nemo-ufes.github.io/gufo/#Event).

```
extml:Task rdf:type owl:Class ;
      rdfs:subClassOf gufo:Event .
```      

This feels very much like they are incompatible types: an Information Entity (a document) is not an Event. A Task is something that can fail to happen, whereas an Event is something that happens. 

Add that MLS defines an InformationEntity to be disjoint with a Process which is dijoint with a Quality.

```Turtle
:InformationEntity rdf:type owl:Class ;
    owl:disjointWith :Process .
:Process rdf:type owl:Class ;
    owl:disjointWith :Quality .    
```

This is represented in the ML Schema diagram which encodes 
Entities as yellow boxes, Processes as Blue Boxes and Qualities as Green ones.

![MLS Core Vocab](https://ml-schema.github.io/documentation/ML-Schema.png)

So these are thop level distinctions in MLS.

What is the point of making a Task be an information Entity? Well I think it is what allows one to speak indirectly of possible future ways of achieving or
not achieving the task. This cannot be done with an
event without speaking of possible future events, which
leads us into modal logic. There is a simple relation between an Event and a Task and that is that a Run [mls:achieves](https://ml-schema.github.io/documentation/ML%20Schema.html#d4e113) a Task, or the other way a task is achieved by a Run.

The [MLS Paper](https://ui.adsabs.harvard.edu/abs/2018arXiv180705351C/abstract) gives some insight into Task on page 4, in the section on the upper level categories

> Information content entity is defined in the [IAO (Information Artefact Ontology)](https://github.com/information-artifact-ontology/IAO/) as “a generically dependent continuant that is about some thing.” Examples of information entities in our vocabulary include:
task, data, dataset, feature algorithm, implementation,
software, hyper-parameter, hyper-parameter setting,
model, model evaluation, evaluation measure, evaluation specification and evaluation procedure.

It also fits with [prov:Plan](https://www.w3.org/TR/2013/REC-prov-o-20130430/#Plan) that is defined as an Entity for which provenance may need to be determined. We see here that Prov-O sees a Plan as an Entity. It is not much to assume that a Task forms part of a Plan, so that it would also be an Entity.

![Top Level view of Prov Ontology](https://www.w3.org/TR/2013/REC-prov-o-20130430/diagrams/expanded.svg)

The `prov:Plan` class is a superclass of [p-plan:Plan](http://vocab.linkeddata.es/p-plan/index.html#Plan) from the Provenance Plan ontology that allows one to describe Plans and steps of plans.

This shows how the `mls` and `extml` are diverging on the concept of Task. This does not mean that they cannot be
related. But to do that one has to understand [gUFO](https://nemo-ufes.github.io/gufo/#types) the simplified OWL version of `UFO`, whose concepts are described  for example in a recent paper [UFO: Unified Foundational Ontology](https://content.iospress.com/articles/applied-ontology/ao210256) ([pdf](http://www.inf.ufes.br/~gguizzardi/Applied_Ontology__UFO__Unified_Foundational_Ontology.pdf)), which comes with this UML diagram. The concepts are also described in the extML Masters Thesis.

![UFO Taxonomy](https://content.iospress.com/media/ao/2022/17-1/ao-17-1-ao210256/ao-17-ao210256-g001.jpg?width=755)



