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

## Classes are labelled as owl:NamedIndividuals

In [MLS_UFO_ONTOUML.ttl](Protege_Ontologies/MLS_UFO_ONTOUML.ttl) has statements like

```turtle
:Run rdf:type owl:Class, gufo:EventType, owl:NamedIndividual;
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
in the same directory could have referenced the definitions there using a relative url such as `<MLS_UFO#:Run1>`. Adding a symbolic link from `<MLS_UFO>` to [MLS_UFO.owl.ttl](Protege_Ontologies/MLS_UFO.owl.ttl) would make those links locally dereferenceable too, for software that knew how to follow links.

This would also make it easy to seperate the instance data
from the ontology itself, by simply placing them in different fils and having the instance data refer to the ontology. For example instead of defining `Run` in `<MLS_UFO>` it could have used the `Run` defined in [MLS_UFO_ONTOUML.ttl](Protege_Ontologies/MLS_UFO_ONTOUML.ttl).

### No Links to the data

The repository helfpil comes with a [data and Code](Data%20and%20Code/) directory, but there is no link to those files such as [dataset.zip](Data%20and%20Code/dataset.zip) or files contained in that archive such as `covid.csv`.

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






