# Contributing to Brick

This document is a set of guidelines and resources for contributing to the Brick effort.
Improvements and changes to this document are always welcome via [Pull Request](https://github.com/BrickSchema/brick-owl-dl/pulls)

## Asking Questions

If you have a question about Brick or its related tools, it is recommended to make a post in the [Brick User Forum](https://groups.google.com/forum/#!forum/brickschema) or under the [Brick GitHub Issue Tracker](https://github.com/BrickSchema/brick-owl-dl/issues). If you have a question about the website, please file the question either in the User Forum or on the [Brick Website Issue Tracker](https://github.com/BrickSchema/brick-owl-dl/issues).

Please conduct a brief search to see if someone has asked your question already; if they have, feel free to jump into the conversation. Otherwise, please file a new issue or make a new post on the forum.

## Reporting Bugs and Issues

Reporting of bugs and issues should be done on the [Brick GitHub Issue Tracker](https://github.com/BrickSchema/brick-owl-dl/issues). The purview of "bugs and issues" includes (but is not limited to):

- missing, incomplete or incorrect definitions of Brick classes
- errors, mistakes or inconsistencies in the Brick ontology definition

Bug reports are most helpful when they fully explain the problem and include as many details as possible.
Some suggestions:

- **Use a clear and descriptive title** for the issue that identifies the problem
- **Include as many details as possible** about the problem, including any relevant Brick/SPARQL queries, RDF triples, segments of Turtle files, Python code, etc
- **Describe the observed and expected behavior**: for example, what query did you run, what were the results, and what did you expect the results to be? What definition exists and what definition would you expect?
- **Describe the exact steps to reproducing the problem** where it is appropriate: did you execute a query and
- Make sure you are using the most recent version of the Brick repository

## Proposing Changes to Brick

The content, structure and extent of Brick is determined by its community, so suggestions for how to improve Brick are always welcome and will be taken under consideration.
Proposed changes to Brick are tracked on the [Brick GitHub Issue Tracker](https://github.com/BrickSchema/brick-owl-dl/issues).

Effective proposals should fully explain the motivation and scope of the proposed changes, and should have at least an initial impression of the nature of the implementation.
The more detail, the better!

## Submitting Changes to Brick

Changes to Brick are performed through [Pull Requests](https://github.com/BrickSchema/brick-owl-dl/pulls).
It is recommended that you become familiar with how to [fork a repository](https://help.github.com/en/articles/fork-a-repo) and [create a pull request](https://help.github.com/en/articles/creating-a-pull-request-from-a-fork).

### Extending the Class Hierarchy

The Brick class hierarchy is defined across several files, named according to the Brick class that roots the hierarchy defined in the file.

Brick point class definitions should be placed in one of the following files:
- `command.py` for subclasses of the Brick `Command` class
- `sensor.py` for subclasses of the Brick `Sensor` class
- `setpoint.py` for subclasses of the Brick `Setpoint` class
- `status.py` for subclasses of the Brick `Status` class

Brick `Equipment` definitions should be placed in the `equipment.py` file.

Brick `Location` definitions should be placed in the `location.py` file.

Brick `Parameter` definitions should be placed in the `parameter.py` file.

Brick `Quantity` definitions should be placed in the `quantities.py` file.

Brick `Substance` definitions should be placed in the `substances.py` file.

Brick class definitions are written using a nested Python dictionary structure.
Observe the example below from `sensor.py`:

```python
{
    "Sensor": {
        "tags": [ TAG.Sensor ],
        "subclasses": {
            "Air_Grains_Sensor": {
                "tags": [ TAG.Sensor, TAG.Air, TAG.Grains ],
                "substances": [ [ BRICK.measures, BRICK.Air ], [ BRICK.measures, BRICK.Grains ], ],
                "subclasses": {
                    "Outside_Air_Grains_Sensor": {
                        "tags": [ TAG.Outside, TAG.Air, TAG.Grains, TAG.Sensor ],
                    },
                    "Return_Air_Grains_Sensor": {
                        "tags": [ TAG.Return, TAG.Air, TAG.Grains, TAG.Sensor ],
                    }
                }
            }
        }
    }
}
```

The core class definition structure is a dictionary whose keys are Brick class names and whose values are dictionaries containing the class properties.
Class names should be alpha-numeric strings consisting of capitalized words separated by the `_` character; class names can not contain spaces.
The class property dictionary can have the following keys:

- `tags`: a list of Brick tags; an entity who has all of these tags will be inferred as an instance of the class
- `subclasses`: a dictionary whose keys+values are class names and definitions; this recursively follows the same structure
- `substances`: a nested list of Brick Substance classes. Each list item should be of the form `[BRICK.measures, BRICK.<substance name>]` where `<substance name>` is replaced with the substance that is measured. Substances are defined in `substances.py`
- `SKOS.definition`: contains the definition of the class. The value must use the `Literal` constructor from RDFlib:
    ```python
    # example
    from rdflib import Literal
    "Thermostat": {
        SKOS.definition: Literal("An automatic control device used to maintain temperature at a fixed or adjustable setpoint."),
    }
    ```

### Managing Tags

Tags provide an alternative way of instantiating classes; Brick can infer classifications from the set of tags applied to an entity with the `brick.hasTag` relationship.
Each subclass's tags should contain *at least* the tags of its parent class; currently, the set of tags for a class must be explicitly annotated.
Tags can be explicitly defined in `tags.py`.

The set of tags for a class should be unique to the hierarchy rooted at that class.
For consistency, if the set of tags `T1` for a class `C1` is a subset of the set of tags `T2` for a class `C2`, then `C1` **must** be a superclass of `C2`.

### Defining Brick Relationships

Brick relationships are defined in `properties.py`.
The `properties` dictionary contains these definitions and follows a similar structure to the Brick class definitions: keys are property names; values are dictionaries of properties of the relationships.

The following keys are expected for each relationship:

- `A`: value is a list of OWL property classes defining how this relationships behaves. `OWL.AsymmetricProperty` and `OWL.IrreflexiveProperty` are the most common
- `SKOS.definition`: value is an RDFlib Literal containing a textual definition of the relationship
- `OWL.inverseOf` (optional): value is a string representing the name of the inverse relationship. If a relationship has a defined inverse, the inverse relationship should also appear as a top-level relationship in the `properties` dictionary.
- `RDFS.range` (optional): value is a Brick class whose instances can be object of this relationship
- `RDFS.domain` (optional): value is a Brick class whose instances can be subject of this relationship

*Subproperties are under development; this document will be updated with instructions for defining subproperties when development has finished*

### Development Environment

The Brick schema is generated by a set of Python files located in this repository.
To set up the development environment

0. Make sure you have Python3 installed on your development system
1. Fork the [Brick repository](https://github.com/BrickSchema/brick-owl-dl) and clone it to your development system
2. [Set up a virtual environment](https://docs.python.org/3/library/venv.html) in the repository folder
    ```bash
    # making the virtual environment
    python3 -m venv brickvenv
    ```
3. Install the Python3 dependencies using pip:
    ```bash
    # enter the virtual environment
    . brickvenv/bin/activate
    pip install -r requirements.txt
    ```
4. Make the desired changes **to the Python files**. Do not edit the `Brick.ttl` or `Brick_expanded.ttl` files directly.
5. Compile the `Brick.ttl` and `Brick_expanded.ttl` files by executing the `generate_brick.py` script using Python
    ```bash
    python generate_brick.py
    ```
6. Commit your changes and push to your fork
7. Submit the pull request
