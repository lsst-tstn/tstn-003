..
  Technote content.

  See https://developer.lsst.io/restructuredtext/style.html
  for a guide to reStructuredText writing.

  Do not put the title, authors or other metadata in this document;
  those are automatically added.

  Use the following syntax for sections:

  Sections
  ========

  and

  Subsections
  -----------

  and

  Subsubsections
  ^^^^^^^^^^^^^^

  To add images, add the image file (png, svg or jpeg preferred) to the
  _static/ directory. The reST syntax for adding the image is

  .. figure:: /_static/filename.ext
     :name: fig-label

     Caption text.

   Run: ``make html`` and ``open _build/html/index.html`` to preview your work.
   See the README at https://github.com/lsst-sqre/lsst-technote-bootstrap or
   this repo's README for more info.

   Feel free to delete this instructional comment.

:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::

.. TODO: Delete the note below before merging new content to the master branch.

.. note::

   This document contains a developer guide for conda and Telescope Site Software. 

Procedure
=========

Our conda Docker environment is provided by the following `repository. <https://cloud.docker.com/u/lsstts/repository/docker/lsstts/conda-package-builder>`_
That image is built with the following `Dockerfile. <https://github.com/lsst-ts/ts_Dockerfiles/blob/develop/conda-package-builder/Dockerfile>`_

Run the image with the following ``docker run -it -env {config_package}_DIR=/home/saluser/{config_package} lsstts/conda_package_builder``
Include the recipe for the conda package in a ``conda`` directory inside of the python software package.
This image provides our common conda packages needed for building and using conda packages.
The following is the list of packages installed in the base environment of the image.

* ts-idl
* ts-dds
* ts-xml

Working with Conda
------------------
Conda is a package manager which works by managing environments.
The key thing for conda is that it works for packages besides python packages.
Environments are isolated spaces and work best for managing individual software package requirements.
Our development environment carries alot of commonalities and therefore the base environment is already setup with many of the important dependencies.
If you create a new environment, the packages from the base environment are inherited into the new environment,
however since our stack is intermingled with environment manipulation, stability of our environment becomes a luxury and so try different things at your own risk.

By default when starting the image, conda is already activated and ready for your use.
Installing a package is fairly straightforward, you just type the name of a package after `conda install`.
Removing a package is simple, just type the name of a package after `conda remove`.
Finding packages that you can install is pretty simple, just search the anaconda-cloud `site <https://anaconda.org/>`_ for various packages.
Conda uses channels to divide their package repos.
Like dockerhub, the channels can be individuals or an organization.
This allows for packages that are the same name to exist on the site.
It also means that an organization can provided their own specialized recipes for their purposes.
Telescope and Site software has a channel that hosts some conda packages.
It is located `here <https://anaconda.org/lsstts>`_.
By default there are several channels installed with your conda environment.
To install a package using a particular channel, simply type the following into your prompt.

.. code:: sh

    conda install -c lsstts ts-salobj

-c  stands for channel and the argument is the name of the channel.

You can search the `anaconda-cloud site <https://anaconda.org>`_ for packages to install into your
environment.

.. code:: sh

    conda install {package}
    conda remove {package}
    conda create -n {name_of_environment} [{list of packages to install}]
    conda activate {name_of_environment}
    conda deactivate

You can type the following command to get help for more commands that conda provides
``conda --help``.


Creating Conda packages
-----------------------
Creating a Conda package requires a Conda recipe.
A recipe consists of one or more files.

::

    .
    ├── build.sh(optional)
    └── meta.yaml(required)

The meta.yaml file contains the metadata for the recipe.
This is where you put the name and version of the package, as well as, the location of the source code.
This file is mandatory for making `conda-build` work.
The other files are optional but useful.
The build script file is where the build steps for installing/building a package are located.
The third file is the run_tests.[sh,bat,py,pl] which tells the recipe how to run the tests.

The meta.yaml file is considered the core file for the recipe.
The file consists of sections, each of which detail a particular aspect of the recipe.

* package
* source
* build
* requirements
* test
* outputs
* about
* extra

The package section is where information such as the name and version of the package go.

The source section contains the information to find the source code for the package.
This section requires one of three paths to be considered complete.
The first path is to use a git repository.
The second path is to use a hg repository.
The final path is to use an archive file.
The two VCS paths can be a local or remote repository.
The archive file can either be a local or server file.
Word of caution, conda divides environment variables that it sets into the type of source that is used.
So recipes which grab metadata from the source will have different environment variables set.

The build section contains the information to build the package.
This section has options that control on what OS the package can be built, you can see it in the example below, how it is used to skip building on Windows.
You can also pass environment variables,
which is not recommended by the official documentation,
but until we solve our reliance on environment variables, we'll just break the guidelines.
This is how we work-around the issue until someone comes up with a better solution.
For one step installation mechanisms, you can use the script key to set a command to run that installs the package.

The requirements section specifies the dependencies for both building and running the package.
The host key is a list of conda packages required to build the package that should be installed on the host.
The runtime key is a list of conda packages required to run the package when installed.
This means that python is both a host and runtime dependency and any python packages used should be listed as runtime dependencies.
Any setuptools extensions should be listed as a host dependency.
It is important to note that any pip packages are not considered usable by conda-build, so those packages must be installable as conda packages.

The test section specifies the dependencies for running unit tests for the package.
The dependencies are inherited from the build section as well.

The outputs section outlines the one or more packages that are built from this recipe.
This section allows for greater granularity over the output of package(s).
For instance, this allows for recipes which create more than one package.
This is useful for metadata packages which are packages that group related packages together.

The about section is for specifying metadata for the package.
The extra section is used for information outside of the package such as metadata for repository hosting service.

The build script is either a unix shell script or Windows batch file that contains the necessary steps to install/build the package.
This script can contain any valid syntax and commands for that particular scripting language.

The test script runs during the testing portion of the build and executes any commands found in those scripts.
For more information on this topic, check the official `documentation. <https://docs.conda.io/projects/conda-build/en/latest/resources/define-metadata.html>`_

Once you think you have a working recipe, you can attempt to build it by invoking the following command.

More advanced features include creating pre-install and post install scripts that run before or after a
package is installed.
They are located in the same directory as the recipe.
The files are called

::
  
    .
    |____pre-unlink.sh
    |____pre-link.sh
    |____post-link.sh

The ``post-link.sh`` is used for creating a package like the ts-salpy-test package.
It creates a ``sal.pth`` file located in the conda python site-packages directory which tells conda to look
for python packages in another location.

.. code:: sh

    conda-build {recipe_location}

Conda-build will then run through the process by installing the package and running whatever tests(unit tests and import tests) that you specified in the recipe.

Development Mode
----------------
Development in Conda requires setting up the base Conda environment(already done for you).
This next part becomes a debate on the definition of developing with conda, because you'll technically be using pip to setup up your source code.
In your directory where setup.py is, run the following in your terminal.

.. code::

  pip install -e .

This will install your package as a symlink to the site-packages directory where you can change the source code and have your ide pick up changes.
You may see a command in conda called `conda develop`, this is not recommended by the maintainers of conda, as it does not behave similiar to `pip install -e`.
Building docs for packages is as simple as using `package-docs build` in the docs or doc directory of the package.
This tool is built for LSST documentation needs.
If running unit tests, just run the `pytest` command in the root directory and pytest will automatically find the tests in the tests subfolder.

.. code:: sh

  # install code in editable mode, this creates symlinks to the site-packages directory with the code directory
  # conda develop is not recommended
  pip install -e .
  # Use pytest command to run unit tests
  pytest
  # build docs using package-docs
  package-docs build # may need to be in doc folder


An Example CSC
==============

ts_ATDome is a CSC that has a good working example of a conda package.
Here's a general/specific procedure for porting a package to conda.
The first step that I like to use, is to determine what the dependencies are for the package.
In EUPs, you can find the dependencies through the {name_of_product}.table.
This only lists the high-level EUPs products so there may be unspecified dependencies.
In this case, there are three dependencies listed for ts_ATDome.

* ts_config_attcs
* sconsUtils
* ts_salobj

We don't need sconsUtils anymore because its only purpose was to provide EUPs integration with scons.
ts_salobj is already available as a conda package which means it can be easily listed as a dependency.
So the only dependency we need to deal with is the ts_config_attcs package.
But we'll come back to that problem later.

Now the next step is to determine how to add the package to the python path.
EUPs works by manipulating the environment to add python packages to the PYTHONPATH environment variable.
However, we can leverage the standard python package installation method to handle that for us.
All we need to do is add a setup.py file to the root package directory of `ts_ATDome <https://github.com/lsst-ts/ts_ATDome>`_.

Following the `TSSW gitflow workflow <https://tssw-developer.lsst.io>`_, we create a branch and you know the rest at this point.
Using the `setup.py <https://github.com/lsst-ts/ts_sal/blob/develop/setup.py>`_ in the ts_sal repo as an example, we can just build a simple one.

.. code:: python

    import os
    import sys
    import setuptools
    import pathlib

    install_requires = []
    tests_require = ["pytest", "pytest-cov", "pytest-flake8", "asynctest"]
    dev_requires = install_requires + tests_require + ["documenteer[pipelines]"]
    scm_version_template = """# Generated by setuptools_scm
    __all__ = ["__version__"]
    __version__ = "{version}"
    """
    tools_path = pathlib.PurePosixPath(setuptools.__path__[0])
    base_prefix = pathlib.PurePosixPath(sys.base_prefix)
    data_files_path = tools_path.relative_to(base_prefix).parents[1]

    setuptools.setup(
        name="ts_ATDome",
        description="LSST auxiliary telescope dome controller",
        use_scm_version={"write_to": "python/lsst/ts/ATDome/version.py",
                        "write_to_template": scm_version_template},
        setup_requires=["setuptools_scm", "pytest-runner"],
        install_requires=install_requires,
        package_dir={"": "python"},
        packages=setuptools.find_namespace_packages(where="python"),
        package_data={"": ["*.rst", "*.yaml"]},
        data_files=[(os.path.join(data_files_path, "schema"),
                    ["schema/ATDome.yaml"])],
        scripts=["bin/run_atdome.py"],
        tests_require=tests_require,
        extras_require={"dev": dev_requires},
        license="GPL",
        project_urls={
            "Bug Tracker": "https://jira.lsstcorp.org/secure/Dashboard.jspa",
            "Source Code": "https://github.com/lsst-ts/ts_ATDome",
        }
    )

This file will add the ts_ATDome package to the package-sites directory of the python install, which is included as the default spot to look for python packages.
You can test your file by using `pip install`.
If no errors come up, then you are all good to go.
However, if errors do pop up, then check the following

* typos in the parameters, especially the require fields

Now create a subdirectory called conda.
This directory is where the recipe will go.
Create a meta.yaml file within this directory.

.. code:: yaml

    { % set data=load_setup_py_data() % }

    package:
      name: ts-ATDome
      version: {{ data.get('version') }}

    source:
      path: ../

    build:
      skip: True #[win]
      script: python -m pip install --ignore-installed --no-deps .
      script_env:
        - PATH
        - PYTHONPATH
        - LD_LIBRARY_PATH
        - OSPL_HOME
        - LSST_DDS_DOMAIN
        - PYTHON_BUILD_VERSION
        - PYTHON_BUILD_LOCATION
        - TS_CONFIG_ATTCS_DIR

    requirements:
      host:
        - python
        - pip
        - setuptools_scm
        - setuptools
      run:
        - python
        - setuptools
        - setuptools_scm
        - ts-salobj

    test:
      requires:
        - pytest
        - pytest-flake8
        - pytest-cov
        - asynctest
        - pytest-tornasync
        - numpy
        - astropy
        - jsonschema
        - pyyaml
        - boto3
        - moto
        - ts-dds
        - ts-idl
        - ts-salobj
      source_files:
        - python
        - bin
        - tests
        - setup.cfg
      commands:
        - py.test

This file will get you through the steps of building and testing the conda package.
You can test it and see if you run into any issues.
If you run into an issue of a package not being found such as pytest-flake8, run the following.

.. code:: sh

    conda config --add channels forge

This is how you permanently add channels to your configuration.

Once you have a built package, you can install it by typing in the following

.. code:: sh

    conda install {location_of_package} # this is found in the final line of a successful conda build

Once installed, you can verify to your standards whether the package works.
Once tested to your satisfaction, you can now upload the package to the repository.
TBD, if that's the appropriate solution.
You will need an account on the anaconda-cloud service and to be added to the lsstts channel on there.
You can be added to the channel by giving an admin, your username on anaconda-cloud.
To upload a package, invoke the following in your terminal

.. code::

    anaconda upload -c lsstts {location_of_conda_package} #again found on the last line of a successful conda build

Upon success, your package will now be uploaded to the channel for distribution purposes.

Once you have a successful build, you can start writing a CI file to build the conda package as a release tag.
ts_ATMCS-simulator has a good working example for this.
Create a file called Jenkinsfile in the root directory of your CSC.

.. code::

  pipeline {
    agent any
    environment {
        package_name = "atmcs-simulator"
        package_version = sh(returnStdout: true, script: "git describe --tags --always --dirty").trim()
        dockerImageName = "lsstts/conda_package_builder:latest"
        container_name = "salobj_${BUILD_ID}_${JENKINS_NODE_COOKIE}"
    }

    stages {
        stage("Pull Docker Image") {
            steps {
                script {
                sh """
                docker pull ${dockerImageName}
                """
                }
            }
        }
        stage("Start builder"){
            steps {
                script {
                    sh """
                    docker run --name ${container_name} -di --rm \
                        --env TS_CONFIG_ATTCS_DIR=/home/saluser/ts_config_attcs \
                        --env LSST_DDS_DOMAIN=citest \
                        -v ${WORKSPACE}:/home/saluser/source ${dockerImageName}
                    """
                }
            }
        }
        stage("Clone ts_config_attcs"){
            steps {
                script {
                    sh """
                    docker exec ${container_name} sh -c "git clone https://github.com/lsst-ts/ts_config_attcs.git"
                    """
                }
            }
        }
        stage("Create ATMCS Simulator Conda package") {
            steps {
                script {
                    sh """
                    docker exec ${container_name} sh -c 'cd ~/source/conda && source ~/miniconda3/bin/activate && source "\$OSPL_HOME/release.com" && conda build --prefix-length 100 .'
                    """
                }
            }
        }
        stage("Push ATMCS Simulator package") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'CondaForge', passwordVariable: 'anaconda_pass', usernameVariable: 'anaconda_user')]) {
                    script {
                        sh """
                        docker exec ${container_name} sh -c "source ~/miniconda3/bin/activate && \
                            anaconda login --user ${anaconda_user} --password ${anaconda_pass} && \
                            anaconda upload -u lsstts --force \
                            ~/miniconda3/conda-bld/linux-64/ts-${package_name}*.tar.bz2"
                        """
                    }
                }
            }
        }
    }
    post {
        cleanup {
            sh """
            docker stop ${container_name}
            """
        }
    }
  }


Q and A
=======

What about EUPs's tagging system?
    DM has not established what they are going to do in this situation.
What about applications that integrate with the LSST Science Pipeline(LSP)?
    DM has agreed to support that software becoming conda packages.

Sources
=======
* https://docs.conda.io/projects/conda-build/en/latest/

.. Add content here.
.. Do not include the document title (it's automatically added from metadata.yaml).

.. .. rubric:: References

.. Make in-text citations with: :cite:`bibkey`.

.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :style: lsst_aa
