# Java2PlantUML
This maven plugin allows you to inspect compile time relations on classes 
within the class path of your projects, and its dependencies. And renders a
Class Diagram src for [PlantUML](http://plantuml.com/) in cli output as 

```
[INFO] Following is the PlantUML src: 
@startuml
' Created by juanmf@gmail.com

' Using left to right direction to try a better layout feel free to edit
left to right direction
' Participants 

class com.github.juanmf.java2plant.render.filters.PredicateFilter {
#  predicate : Predicate
--
+  satisfy()  : boolean

}
class...

' Relations

com.github.juanmf.java2plant.render.filters.Filters "1"  o-left-  "1" com.github.juanmf.java2plant.render.filters.RelationFieldsFilter  : FILTER_RELATION_FORBID_TO_BASE
com...
@enduml

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 3.295 s
[INFO] Finished at: 2016-08-02T08:19:29-03:00
[INFO] Final Memory: 17M/212M
[INFO] ------------------------------------------------------------------------

```
It also writes output diagram to j2puml<time>.txt at pom's directory.

Installation
============

To use it as a plugin, 1st you need install it in local repo as follows. Step 
at this project's pom Directory and run

```
mvn clean install
```

Then add the plugin gropu to your ~/.m2/settings.xml

```xml
  <pluginGroups>
    <pluginGroup>com.github.juanmf</pluginGroup>
  </pluginGroups>
```

Usage
=====

Step at your project's pom directory and then play around with CLI options

```
 mvn   -Dparse.thePackage="com.my.package" clean compile java2PlantUML:parse
 mvn   -Dparse.thePackage="com.my.package, com.other.package" clean compile java2PlantUML:parse
 mvn   -Dparse.thePackage="com.my.package, com.other.Class" clean compile java2PlantUML:parse
```


Usage of Filters 
----------------

Filters control the output by providing a configurable way to limit noise.
These filters are applied in key rendering points by `PlantRenderer#render()`
 
There are of three kinds:

* Classes: prevent printing details of Class<?> objects that match some criteria.
* Relation:  prevent printing Relations whose from or To side objects matches some criteria.
* RelationTypes: prevent printing relations by their type, this is a coarse approach.

### Chain filters

To customize output further more, youc an use Chain filters of each of the three
filter types, achieving a complex behavior by combination of simpler filters.
e.g. `Filters.FILTER_CHAIN_RELATION_STANDARD`

### PredicateFilters
You can define filters of any of the three types, that will apply a custom logic to
the given object. e.g. {@link #FILTER_RELATION_FORBID_AGGREGATION_FROM_PRIVATE}

Default Filters
---------------

If no Filtering options are given following Filters will be used

```
  "parse.relationTypeFilter" = "FILTER_CHAIN_RELATION_TYPE_STANDARD"
  "parse.classesFilter" = "FILTER_CHAIN_CLASSES_STANDARD"
  "parse.relationsFilter" = "FILTER_CHAIN_RELATION_STANDARD"
 ```

 To change them use:
```
 mvn   -Dparse.thePackage="p1" -Dparse.classesFilter="FILTER_FORBID_ANONYMOUS" clean compile java2PlantUML:parse
 mvn   -Dparse.thePackage="p1" -Dparse.relationTypeFilter="FILTER_FORBID_USES" clean compile java2PlantUML:parse
 mvn   -Dparse.thePackage="p1" -Dparse.relationsFilter="FILTER_RELATION_FORBID_TO_PRIMITIVE" clean compile java2PlantUML:parse
```
 
 Available filter names and types are defined in `com.github.juanmf.java2plant.render.filters.Filters#FILTERS` Map.
 
 Read `com.github.juanmf.java2plant.render.filters.Filters`'s javadoc and src to understand the three types of filters
 you can use.
 
 Custom filters usage
 --------------------
 
 You can use a custom chain filter for each of the three types of filters, combining any of the existing filters
 without the need to code.

``` 
  mvn -Dparse.thePackage="p1" \
      -Dparse.classesFilter="FILTER_CHAIN_CLASSES_CUSTOM" \
      -Dparse.customClassesFilter="FILTER_FORBID_ANONYMOUS,FILTER_FORBID_PRIMITIVES" \
      clean compile java2PlantUML:parse

  mvn -Dparse.thePackage="p1" \
      -Dparse.relationTypeFilter="FILTER_CHAIN_RELATION_TYPE_CUSTOM" \
      -Dparse.customRelationTypeFilters="FILTER_FORBID_USES,FILTER_FORBID_AGGREGATION" \
      clean compile java2PlantUML:parse

  mvn -Dparse.thePackage="p1" \
      -Dparse.relationsFilter="FILTER_CHAIN_RELATION_TYPE_CUSTOM" \
      -Dparse.customRelationsFilter="FILTER_RELATION_FORBID_TO_PRIMITIVE,FILTER_RELATION_FORBID_FROM_ANONYMOUS" \
      clean compile java2PlantUML:parse
```

 Of course you can use all three custom chain filters in a single run.

`parse.thePackage` is the root (or roots if a comma separated list of packages is given) 
from where class scanning will get the main classes of your Diagram, it can be a package or a FQCN.

Results
=======

The end result is a file named "j2puml+now+.puml" that you can process with [PlantUML online Render](http://plantuml.com/plantuml) 
in order to get the UML diagram rendered by PlantUML as per the instructions in the generated script. A run over this project renders:

![java2Plant diagram should appear here..](/doc/java2Plant.png?raw=true "Java2Plant Collaboration")

Note: the command used was: 
```
mvn   -Dparse.thePackage="com.github.juanmf.java2plant,\
    com.github.juanmf.java2plant.goal.Parse, \
    com.github.juanmf.java2plant.Parser, \
    com.github.juanmf.java2plant.render.filters.Filters, \
    com.github.juanmf.java2plant.render.PlantRenderer" \
    clean compile java2PlantUML:parse
```
By including specific Class names, even if they are already included in specified packages, you get a note with 
"Relevant Class" linked to it in the diagram

TODO
====
 * Render several diagrams, making focus on classes inside given packages.
 * Add some indicator in super classes that the methods are overridden
 * Optionally Add a hiararchy top to bottom.
 * fix Map binding in method params.