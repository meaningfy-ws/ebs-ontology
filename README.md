# model2owl-boilerplate
Boilerplate for running model2owl on new projects

# Getting started
This project will use model2owl to transform a UML model into a formal OWL ontology, a SHACL shape, a conventions report
and glossary  based on established UML conventions.

Main steps:
* Fork this repository or make a new branch from main
* Put your UML model/models export (XML file) in the implementation folder
* Configure model2owl using config folder
* Modules are auto-detected from the implementation folder

> **Note:**  
> If the branching option is used, the branch will not be merged into `master`. 
> It is recommended to delete the branch once the desired output is generated and the work is complete.
# Usage
This section covers the practical steps for setting up a model: follow the [naming conventions](#naming-conventions), create the expected [folder structure](#folder-structure-conventions), [add a UML model](#adding-a-uml-model), and copy and adapt the [model2owl config](#adding-model2owl-config). It also explains the [GitHub Actions](#adjust-github-actions) used to generate OWL, SHACL, glossaries, along with where those outputs are written. The remaining subsections describe the [output layout](#output), [commit-message conventions](#commit-messages-for-automatically-generated-reports), [workflow summary](#workflow-summary), and [CI troubleshooting](#troubleshooting-failed-ci-runs).
## Naming conventions
* The name of the created folders should not contain spaces. It can contain underscores or hyphen if it's strictly necessary 
* The name of the UML model export file should match its folder name (i.e mymodel.xml)
## Folder structure conventions
To add a new UML model follow this folder structure
```
implementation
        |___firstModel
                |___model2owl-config
                |___xmi_conceptual_model
                |___respec_resources
```
## Adding a UML model
* Create a folder under implementation with the name of the UML model following the naming conventions. 
* Create xmi_conceptual_model folder inside the folder created at the previous step
* Put the UML export in the xmi_conceptual_model folder following the naming conventions
## Adding model2owl config
The top-level [`model2owl-config`](./model2owl-config) folder is a **configuration template**: a generic, documented set of config files to copy and adapt. See [`model2owl-config/README.md`](./model2owl-config/README.md) for the purpose of each file, the placeholder values to replace, and the parameter/metadata groups.
* Copy the `model2owl-config` folder into the UML model implementation folder created at the previous step
* Configure model2owl by editing the files inside that copied `model2owl-config` folder (see [`model2owl-config/README.md`](./model2owl-config/README.md) for details)
### Configuration Files

As presented above, this validator will use a maximum of three configuration files, depending on the UML model validation option you have selected (check the **Validator Options** section).

#### Config Parameters File (config-parameters.xsl)

This is an XSLT file that contains a set of variables used by the system during model validation. Some variables can be modified, while others should remain unchanged as they are preconfigured. A boilerplate for the config parameters can be found [here](#). The boilerplate includes comments to guide what can and cannot be changed.

**Sample:**

```xml
<!-- Types of elements and names for attribute types that are acceptable to produce object properties -->
<xsl:variable name="acceptableTypesForObjectProperties" select="('epo:Identifier', 'rdfs:Literal')"/>

<!-- Acceptable stereotypes -->
<xsl:variable name="stereotypeValidOnAttributes" select="()"/>
<!-- ... other variables ... -->

<!-- Allowed characters for a normalized string (characters allowed in a QName) -->
<xsl:variable name="allowedStrings" select="'^[\w\d-_:]+$'"/>
```
There are three types of data you can pass into the variables: lists, strings, or booleans.  
- For **lists**, the variable value should be enclosed in double quotes (`""`), with individual values wrapped in single quotes (`''`) and separated by commas.  
- For **strings** and **booleans**, the value should always be enclosed in double quotes (`""`).

**Example:**
```xml
String variable 
<xsl:variable name="my-variable" select="'my string value'"/>
List variable
<xsl:variable name="stereotypeValidOnDependencies" select="('Disjoint', 'disjoint', 'join')"/>
Boolean variable
<xsl:variable name="enableGenerationOfSkosConcept" select="fn:false()"/>
```
**Notes:**
- **Do not delete variables**: Each variable serves a purpose in the generation process.
- **Maintain variable types**: Do not change the type of a variable (e.g., from a list to a string). If a variable is originally set as a list, it must remain a list.
- If a variable is unnecessary or if you prefer not to impose restrictions, leave the variable with an empty list or string. See the examples below:
  - **Empty string variable**:
  
    ```xml
    <xsl:variable name="my-variable" select="''"/>
    ```

  - **Empty list variable**:

    ```xml
    <xsl:variable name="stereotypeValidOnAttributes" select="()"/>
    ```
    
#### Namespaces File (namespaces.xml)

This file holds the namespace values and names used in the model being processed.
Sample:

```xml

<?xml version="1.0" encoding="UTF-8"?>
<prefixes xmlns="http://publications.europa.eu/ns/">
   <prefix name="" value="http://data.europa.eu/a4g/ontology#"/>
    <prefix name="foaf" value="http://xmlns.com/foaf/0.1/"/>
   <!-- ... other prefixes ... -->
</prefixes>
```
**Notes:**
- If you want any URI from this list to be imported as statements (`owl:imports`) in the generated OWL and SHACL artefacts, define it in the imports.xml file (as described in the [Imported ontologies configuration](https://github.com/meaningfy-ws/model2owl/tree/develop?tab=readme-ov-file#imported-ontologies-configuration)).

#### XSD and RDF Datatypes File (xsdAndRdfDataTypes.xml)

This file contains declarations of XSD and RDF datatypes. 

**Sample:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<datatypes xmlns="http://publications.europa.eu/ns/">
   <datatype namespace="xsd" qname="xsd:anyURI"/>
   <datatype namespace="xsd" qname="xsd:base64Binary"/>
   <datatype namespace="xsd" qname="xsd:boolean"/>
   <datatype namespace="xsd" qname="xsd:byte"/>
   <datatype namespace="xsd" qname="xsd:date"/>
   <datatype namespace="xsd" qname="xsd:dateTime"/>
   <datatype namespace="rdfs" qname="rdfs:Literal"/>
</datatypes>
```
#### UML to XSD DataTypes File (umlToXsdDataTypes.xml)

This file defines the mappings between UML datatypes and XSD datatypes.
If this in not necessary leave the default file.
### Sample:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<mappings xmlns="http://publications.europa.eu/ns/">

    <!-- epo prefixed datatypes -->
    <mapping>
        <from qname="epo:Indicator"/>
        <to qname="xsd:boolean"/>
    </mapping>
    <mapping>
        <from qname="epo:Date"/>
        <to qname="xsd:date"/>
    </mapping>
    <mapping>
        <from qname="epo:DateTime"/>
        <to qname="xsd:dateTime"/>
    </mapping>
</mappings>
```
#### Metadata File (metadata.json)

This file contains metadata information used for generating documentation, convention report, and ReSpec documentation. It defines titles, descriptions, versioning, contributors, and other publication details for the ontology.

**Sample:**

```json
{
    "metadata": {
        "conventionReportAuthor": "Your Organization",
        "conventionReportUMLModelName": "Your Ontology Name",
        "ontologyTitleCore": "Your Ontology Core",
        "feedbackUrl": "https://github.com/your-org/your-ontology/issues",
        "license": "CC BY 4.0",
        "repositoryUrl": "https://github.com/your-org/your-ontology",
        "status": "Draft",
        "title": "Your Ontology Documentation"
    },
    "customMetadata": {
        "metadataSectionProperties": [
            {
                "key": "Related documentation",
                "data": [
                    {
                        "value": "User Guide",
                        "href": "https://your-organization.org/user-guide"
                    }
                ]
            }
        ]
    }
}
```



**Notes:**
- This file is used by all model2owl artefacts
- The custom metadata (`customMetadata`) section allow you to add additional documentation links and information
- The `metadata.projectLocalResources.path` property must specify a path **relative to the module** that contains the configuration file. For instance, in the demo_ontology module, when editing `implementation/demo_ontology/model2owl-config/metadata.json`, the `OWL core resource` is referenced as `owl_ontology/demo_ontology.rdf`. Additional examples can be found in the [metadata.json](./implementation/demo_ontology/model2owl-config/metadata.json) file.


## Adjust GitHub actions
Two GitHub Action scripts are located in the [.github](./.github) directory:
 * [transform_with_model2owl.yml](.github/workflows/transform_with_model2owl.yml)
   is used to transform the UML model/models into set of supported artefacts.
 * **DISABLED** [diff-combined.yml](.github/workflows/diff-combined.yml) is used to compute a
   difference between two versions of RDF artefacts and generate
   machine-readable (JSON) and human-readable (AsciiDoc) reports.

### CI workflow and customization

`transform_with_model2owl.yml` first detects the affected modules, then runs the generation jobs for glossary/conventions report, OWL/SHACL.

To customize the workflow, disable the generation you do not need by removing or guarding the corresponding job or step; the generated files and their paths are listed in [Generated Output](#generated-output). 

When adjusting the workflow to your needs, keep the dependency chain in mind: ReSpec needs SHACL, diff needs the transform outputs, and Pages needs ReSpec.
That means `generate_respec` cannot run unless `transform` has already produced `implementation/*/shacl_shapes/`, because the ReSpec job copies `ontology_shapes.ttl` from that output into the documentation package. The `diff` job is tied to the OWL and SHACL artefacts emitted by `transform`, so disabling those outputs also removes the input that diff uses to compare revisions. Likewise, `build_pages` only publishes the `respec/` artefact, so if you turn off ReSpec generation there is nothing for GitHub Pages to deploy.

### Transform with model2owl

The workflow triggers on changes to any XMI file under the implementation directory:
```yaml
    paths:
      - "implementation/*/xmi_conceptual_model/*.xml"
```
Modules are auto-detected from the changed files. On manual dispatch, all modules are processed.

```
Example:

implementation
        |___modelOne
        |___modelTwo

Both models will be auto-detected and processed by the workflow.
```

## Output
The output is automatically generated by the GitHub action scripts described previously. Each of the scripts will 
do an automatic commit on the branch that was executed from. To see the output executing a git pull after the GitHub 
action ran successfully is mandatory.
The scripts will generate automatically folders and transformation files under a specific structure that is presented
below.

### Output folders structure and content

Glossaries will be stored at the top level of this project outside the implementation folder, and it will 
contain the individual glossaries for the UML model and a unified glossary if there are more than one UML model to
be processed by GitHub action scripts

```
     /
    .github
    glossary
        |__static  -> folder to hold css and js neccesary for the glossary
        |__modelOne_glossary.html
        |__modelOne_glossary.adoc
        |__modelOne_glossary.adoc.html*
        |__modelTwo_glossary.html
        |__modelTwo_glossary.adoc
        |__modelTwo_glossary.adoc.html*
        |__ontologies_combined_glossary.html        -> combined glossary
        |__ontologies_combined_glossary.adoc        -> combined glossary
        |__ontologies_combined_glossary.adoc.html*   -> combined glossary
    implementation
    model2owl-config
```
Note: The `*.html` files relate to the former glossary, while the `*.adoc` and `*.adoc.html` files represent the new glossary. The new version overlaps significantly with the old one but includes minor enhancements, such as deduplicated term definitions in the _definition_ columns.

_* The `*.adoc.html` file is different from `*.html` as it's generated from `*.adoc` sources for demonstration purposes only._

The formal OWL ontology and a SHACL shape will be inside each UML model folder under specific folders as described 
below.
```
    implementation
        |___modelOne
                |__conventions_report
                |       |__static -> folder to hold css and js neccesary for the convention report
                |       |__modelOne-convention-report.html
                |__owl_ontology
                |       |__modelOne_core.rdf
                |       |__modelOne_core.ttl
                |       |__modelOne_restrictions.rdf
                |       |__modelOne_restrictions.ttl
                |__shacl_shapes
                |       |__modelOne_shapes.rdf
                |       |__modelOne_shapes.ttl
                |___model2owl-config
                |___xmi_conceptual_model
        |___modelTwo
```

When a module's model2owl configuration enables the optional consolidated
OWL-full form (`generateOWLFull`), its `owl_ontology` folder instead contains a
single self-contained ontology — `<module>_full.owl`, `<module>_full.rdf` and
`<module>_full.ttl` — and the separate core/restrictions files are not produced.
This is an alternative form of the same ontology; see the model2owl documentation
for details.

Note that regardless of the chosen directory structure for the output, any set
of diffing reports can be accessed by inspecting past Git revisions. Identifying
a report’s sources is made easier by the information encoded in the commit
message.

### Commit messages for automatically generated reports
The workflows commit generated files with messages following conventional commits format:
```
chore(ci): transform 2 module(s) #143 [skip ci]
chore(ci): diff 2 module(s) #143 [skip ci]
```
The message includes the number of modules processed and the workflow run number.
`[skip ci]` prevents the commit from retriggering the workflow.

### Workflow summary
Each workflow run generates a summary visible in the GitHub Actions run page. The summary
shows which modules were processed, with links to the generated diff reports. If a module
fails (e.g., no preexisting files to compare to), it will be noted in the summary.

### Troubleshooting failed CI runs
When a transform/diff/pages run fails, the [`debugging-ci-runs`](.claude/skills/debugging-ci-runs/SKILL.md)
Claude Code skill describes how to inspect the run with `gh`, filter the logs down to the real
error, classify the failure, and fix it iteratively. **Important:** on a (partially) successful
run the workflows commit generated artefacts back to the branch, so always `git pull` before
editing, committing, or rebasing.

### Generated Output

The workflow writes the generated artefacts back into the repository using the same paths that GitHub Actions packages and commits:
- **Glossary**: `glossary/`
- **Conventions report**: `implementation/*/conventions_report/`
- **Formal OWL ontology**: `implementation/*/owl_ontology/`
- **SHACL shapes**: `implementation/*/shacl_shapes/`
