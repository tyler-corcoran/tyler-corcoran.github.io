# ML Project Template

> Don't be a pirate, be the Navy.

This project structure template and the following development patterns are based loosely on the philosophy laid out by the [Cookiecutter Data Science](https://drivendata.github.io/cookiecutter-data-science/) project. The motivation for having such a structure is to reduce the cognitive switching costs of understanding and evaluating projects that vary widely on the surface but essentially tackle a small set of similar problems.

## Scope of the Template

### Workflows

The scope of this project template is to encapsulate **both exploratory workflows** that serve the early iterative phases of a project **and engineering workflows** that serve the later, deployment-centric phases of a project.

### Project Code vs Project Assets

An important distinction to make right away is the difference between project code and project assets. Project code includes scripts, functions, queries, database connections, etc. which should be versioned within the project structure on GitHub. Project assets include all of the results of running project code, such as models, images, data, snapshots, graphs, etc. which should be captured using the Azure ML Workspace. **Project assets should never be commited to GitHub**. See below for more on the relationship between *Feature Workflows*.

If you have questions, clarifications, or suggestions, please feel free to post issues via GitHub, or just walk up and talk to someone on the team. This should be a slowly-changing - but changing for the better - template.

## Directory Structure

```plaintext
│   .gitignore       <- Preconfigured for Python/R.
│   .Rprofile        <- (R only) Initialization script with Checkpoint setup.
│   environment.yml  <- (Python only) Conda environment file.
│   README.md        <- Project-level README for end-users and collaborators.
│
├───data             <- Local datastore for intermediate, destructible data and artifacts.
│
├───main             <- Main scripts, pipelines, tests, and other end-user scripts.
|   │
|   └───config       <- .csv, .json, and .yml files used to parametrize main script execution.
│
├───notebooks        <- .ipynb, .rmd, and other files that are "notebook in spirit".
|   │
|   └───config       <- .csv, .json, and .yml files used to parametrize notebook execution.
│
├───references       <- Data dictionaries, manuals, and other explanatory materials.
│
└───src              <- Source code, including functions, modules, etc.
    │
    └───db           <- Queries and Python/R code related to source data.
    │
    └───etc          <- Organize Python/R code as works best for the project.
```

## Launch Checklist

Please review and follow this Launch Checklist each time you start a new project, whether from the ground up or when porting over an existing legacy project.

### 1. Setup a Private DriveTime GitHub Repository

Ask someone from the dev ops team (DL-MIS DevOps <DL-MISDevOps@drivetime.com>) to create a new private repo on the DriveTime corporate GitHub account using this repository [as the creation template](https://help.github.com/en/articles/creating-a-repository-from-a-template).

Naming conventions for all projects created from this template should be `ml-` followed by the dash-delimited project name. For example, `ml-app-score`.

See below for more on restrictions on project names in *Feature Workflows*.

### 2. Create and Maintain an Environment

You should maintain and version the metadata required to reproduce your environment at the beginning and as you go. For Python, this means using `conda` to create an environment and versioning an `environment.yml` file. For R, this means making use of the `Checkpoint` package.

### 3. Create and Maintain a README

After you've cloned from this project template repository and created your project repository, you should overwrite this README with one specific to your project.

The README should sit in the project root directory to allow it to be automatically rendered on GitHub. At a minimum, the README should include header sections for the following:

* Purpose or background information for the entire project
* Instructions on setting up the project environment
* List of all main scripts (see below for more about *Main Scripts*)
  * Name of main script
  * A short description of the purpose or output of main script
  * Name and description of any config files associated with the main script

### 4. Let the Standardized Fun Begin

Now you're ready to work on your project in a reproducible, collaboration-ready fashion. Please review the *Global Best Practices* below!

## Global Best Practices

### Source Data is Immutable

Source data is immutable. If you filter it, process it, or modify it at all, it's no longer source data (see *Intermediate Data* below). From your project's perspective, source data is "a given" and is bestowed upon your project by a separate team (e.g. database or BI) or by a separate project or process. For example, another project might produce intermediate data that your project consumes as source data, so it's all a matter of project scope.

If you have a process outside of your project structure that you need to run in order for your project to access its source data, then it isn't source data, and you have project code living outside of your project. You should refactor the outside process and fold it into your project structure.

Common examples of source data:

* Databases, data warehouses, data lakes, etc. maintained by the database teams
* Locked down BLOB storage
* Azure ML SDK DataReference

### Intermediate Data is Destructible

Intermediate data is the product of project code and source data (or other intermediate data) and is thereby destructible. If intermediate data is deleted, it should be recreateable using project code and source data. All that is lost is processing time.

Common examples of intermediate data:

* Any data files of any format anywhere that are generated by code in your project
* Azure ML SDK PipelineData
* Models can be viewed as intermediate data

### Data Access for local vs remote execution

For local runs and debugging, initial source data must be accessed and downloaded from remote data storage and all steps must initially be run in order to generate all intermediate data objects.  After the initial run, each step can be run independently.

#### Inputs/Outputs for local vs remote execution
#### Local:
* Inputs:
    + source data: DataReference (with subsetting for larger objects)
    + intermediate data: local files (location designated by the previous step's output)
* Outputs:
    + intermediate data: local files (location designated by the previous step's output)
#### Remote:
* Inputs:
    + source data: DataReference
    + intermediate data: PipelineData objects
* Outputs: 
    + intermediate data: PipelineData objects or DataReference (in special cases)

### Environments First

Before anything else, set your project up for reproducibility with an environment. As you include additional packages, you should periodically update and version your environment files. Your README should include instructions on how the environment is set up, even if it's something as simple as `conda env create -f environment.yml`. If your project makes use of an IDE project file (e.g. `.RProj` for RStudio), then your README should include instructions for this as well.

#### TODO: Container-based development using Docker

### Analysis is a DAG

An indispensable abstraction for reproducible analysis is the [Directed Acyclic Graph (DAG)](https://en.wikipedia.org/wiki/Directed_acyclic_graph). Like DAGs, reproducible analyses have the property of being one-directional dependency chains. If prior steps in an analysis have already run and generated intermediate data, then later steps can be developed and rerun iteratively without needing to rerun prior steps. Whenever we talk about pipelines, we're talking about DAGs.

The DAG abstraction lends itself directly to several development patterns in data science, from simple reproducible analyses to deployment-ready ML model development pipelines. Python tools that implement DAGs include [Azure ML Pipelines](https://docs.microsoft.com/en-us/azure/machine-learning/service/concept-ml-pipelines), [Airflow](http://pythonhosted.org/airflow/cli.html), and [Luigi](https://luigi.readthedocs.io/en/stable/index.html).

### Notebooks are for Communication and Exploration

It's very easy to get started with notebooks, and they lend themselves to iterative, exploratory workflows. However, they don't lend themselves well to version control. Use notebooks to your heart's content, but keep in mind that notebooks will ultimately be used only for communication and exploration (where the key advantages are flexibility and speed).

Keep notebooks in the `/notebooks` subfolder in the project structure, and use a naming convention that includes a numbering system (to recall the order in which the notebooks were developed, or to suggest the order in which the notebooks should be referenced), your initials, and a hypen-delimited description of the notebook. For example, a notebook might be called `03.2-si-proof-stepwise-regression-is-lame.ipynb`.

Notebooks don't need to be literal notebook files (e.g. `.rmd` or `.ipynb`), but can be any file format that "in spirit" is a notebook as discussed above.

Ultimately, notebooks are for saving our though processes, and are the least formalized files in the project structure. The only requirements for notebooks are:

* Stick to the naming convention
* Maintain consistency within the project
* Ensure that notebook outputs are rendered on GitHub

### Main Scripts are for End-Users

When your colleague - or often, your six-months-in-the-future self - needs to execute any of the various scripts in your project, where should they start? The answer is main scripts. A main script is the entry point for an end-user (or a developer) to execute one of the various programs embedded in your project structure. One main script should exist for each distinct program or task accomplished by the project. Some common main scripts include:

* Analyses or reports that are periodically rerun with parameters
* ML Pipeline definition scripts
* Code testing scripts
* In early/iterative phases, notebooks can serve as main scripts, but should ultimately be replaced by source code

The project README should enumerate the distinct main scripts in the project, along with instructions on how to use them. Main scripts often make use of parameters, which should be factored out of main scripts and kept in separate config files (see more about *Config Files* below).

It goes almost without saying that if a main script doesn't execute its purpose as intended, then it isn't really a main script yet. Main scripts don't necessarily need to be in some perfect finished state, but they should be work (perform their purpose without errors). By extension, any main script dependencies such as config files, source code, etc. should also work. For more guidance on "things should work", see the section below on *Feature Workflows*.

### Config Files are for Main Scripts

Each time a main script is executed, the parameters of the execution should be taken from config files. This is to prevent parameters that could reasonably change from one execution to another from being hard-coded within scripts (requiring end-users to modify code to adjust the execution).

If you find yourself changing lines inside main scripts to modify the normal execution of a program, consider refactoring these lines as parameters in config files and adding logic to your main script that parses the parameters from the config files into global variables. Common use cases for config files include:

* Dataset metadata, typically stored in `.csv` files, where there is one row per feature in the dataset and several columns where metadata about each feature can be organized (e.g. data type, predictor vs response, allowable ranges, nullable, etc.)
* Database connection information, typically stored in `.yml` or `.json` files
* Global main script parameters such as random seed, etc., typically stored in `.yml` or `.json` files

The project README should have instructions on the config files and how to use them for each distinct main script.

## Feature Workflows

As mentioned above in *Project Code vs Project Assets*, project code and assets should follow a universal naming convention across GitHub and the Azure ML Workspace for a clear separation. Having this convention in place can help us access and load assets programmatically, as well as maintain a _virtual_ gate between **temporary assets** and **deployable assets**.  

In order to harness the power of the GitHub flow to drive our workflows as a team, we depend on the following concepts: 

* A **master branch** represents the deployable state of the code base. It doesn't have to be a complete solution, but it must run without errors. 
* A **feature branch** is where you are free to commit experimental code. Changes you make on a branch won't affect the `master` branch, so you're free to experiment in it until it's ready to be reviewed and merged into `master`. 

You can read more about the GitHub flow and version control best practices [here](https://guides.github.com/introduction/flow/)


## Naming Conventions

> There are only two hard things in Computer Science: cache invalidation and naming things.
> 
> ~ Phil Karlton

For a consistent naming convention across our projects, follow these rules:

### GitHub
- Repositories: `ML-ProjectName` (make sure that `ProjectName` does not exceed 16 characters, `ML-` excluded)
- Branches: `FeatureName` (make sure that `FeatureName` does not exceed 15 characters)  
**Note**: Before you ask for a new repository, make sure that `ProjectName-FeatureName` does not exceed 32 characters in total (the `ML-` in the repository name doesn't count towards the 32 characters). This will unlock the potential to dynamically name your experiments and assets without hard-coding the names.  
- Directories: `all-directories/in-project/`
- Files: `almost-all-files.txt`
- Python modules: `onlypythonmodules.py` (for PEP 8 [module requirements](https://www.python.org/dev/peps/pep-0008/#package-and-module-names))

### Azure ML Services
- Workspaces: {dev ops controls}
- Feature branch assets:  
  * **Dashes** are the universal delimiter for assets, models and images. Use them as a separation between project name & feature name. Before you decide on a feature name, you should make sure it doesn't exceed 15 characters, and doesn't contain underscores `_`, slashes `/`, dots `.` or colons `:`
  * Experiments: `ProjectName-FeatureName`
  * Registered models: `ProjectName-FeatureName`
  * Images: `ProjectName-FeatureName`
  * Published pipelines: `ProjectName-FeatureName`
- Master branch assets:
  * Same as feature branch assets but replace `FeatureName` with `master`
- Tags: consistent within project but flexible
- Secrets: {dev ops controls}

Here is an example of how to dynamically assign the name of your assets:
```python
import re
from pygit2 import Repository

feature_name = Repository('.').head.shorthand
repo = Repository('.').config._get('remote.origin.url')[1].value
project_name = re.split(r'\/ML-(.+).git', repo)[1]

asset_name = project_name+'-'+feature_name 
```
This code snippet will detect the name of your repository, extract `ml-` and then concatenates the `ProjectName` and `FeatureName`. You can also include the following assertions before assigning the asset names to make sure they follow the naming restrictions: 
```python
assert len(asset_name) <= 32, 'asset name is more than 32 characters'
assert re.findall('(_?\/?:?\.?)', asset_name), 'asset name contains a slash(/), underscore(_), dot(.) or colon (:)'
```

### Naming config files

Whenever you are naming your config files (e.g. attributes metadata files), choose a specific name for the metadata file, something that informs what kind of config file (e.g. source attribute metadata).

Make sure to use concise & universal keywords across your project (datatype, role etc.).

#### TODO: Unit test requirements
