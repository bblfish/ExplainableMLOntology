# Review of ontology

We seem to have a number of problems

## Instances are badly named

in file...

## Is the ExplainableOntology extension to MLS that good?

### gufo:participatedIn replaces mls relations?

In [Protege_Ontologies/MLS_UFO_ONTOUML.ttl](MLS_UFO_ONTOUML.ttl) a `Run` is defined
in another namespace call it `expl` 

```Turtle
expl:Run rdfs:subClassOf [
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




