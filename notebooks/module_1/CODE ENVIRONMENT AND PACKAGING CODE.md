Programs naturally use other libraries or services. However, this introduces a dependency relationship between your program and the libraries it requires to function.

**INSTALL LIBRARY**
- `pip` is the command line tool that the Python programming language provides for installing packages.
- Executing `pip install requests` produces the following sequence of actions:
		1. Search for requests in the Python Package Index ([PyPI](https://pypi.org/))
		2. Search for the appropriate artifact for the platform we are running under
		3. Resolve dependencies - the library itself depends on other packages, so the installer must find compatible versions of all transitive dependencies and install them beforehand
		4. Download the artifacts, then unpack and copy the files into the right places in our filesystem
- using `uv` instead of `pip` - newer and better ->dramatically reduces the installation time
Introducing dependencies also introduces dependency conflicts. Conflicts happen when programs require incompatible versions of the same dependency. This situation — where multiple packages require mutually incompatible versions of shared dependencies — is commonly referred to as dependency hell. One way to deal with conflicts is to isolate the dependencies of each program into their own environment.

**CREATE ENVIRONMENT  **Artifacts & Packaging****

- Environment - entire standalone version of the language runtime with its own set of installed packages. This virtual environment or `venv` isolates the installed dependencies from the global Python installation. It is a good practice to have a virtual environment for each project, containing the dependencies it requires.
- Source code - Developers write and read source code
- Artifacts - the packaged, distributable outputs produced from that source code - ready to be installed or deployed.
- Packaging a library - producing an artifact that package installers like `pip` or `uv` can use to install the relevant files.
- Python artifacts are called _wheels_ - contain all the necessary information to install a package: the code files, metadata about the package (name, version, dependencies), and instructions for where to place files in the environment.
- Building an artifact - requires that we write a project file (also often known as manifest) detailing the specifics of the project, the required dependencies, the version of the package, and other information. In Python, we use `pyproject.toml` for this purpose.
-   `pyproject.toml` -  Example for a library that also provides a command-line tool:

``` python
[project]
name = "greeting"
version = "0.1.0"
description = "A simple greeting library"
dependencies = ["typer>=0.9"]

[project.scripts]
greet = "greeting:cli"

[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"
```
- We can build the wheel with `uv build
- The `.whl` file is the wheel (a zip archive with a specific structure), and the `.tar.gz` is a source distribution for systems that need to build from source.

**INSTALLING SOFTWARE**
When installing software, there’s an important distinction between installing from source and installing a prebuilt binary:
- Installing from source - means downloading the original code and compiling it on your machine - this requires having a compiler and build tools installed, and can take significant time for large projects.
- Installing a prebuilt binary means downloading an artifact that was already compiled by someone else - faster and simpler, but the binary must match your platform and architecture.

**Releases & Versioning**

To simplify this problem there are conventions on how to version software, and one of the most prevalent is [Semantic Versioning](https://semver.org/) or SemVer. Under Semantic Versioning a version has an identifier of the form MAJOR.MINOR.PATCH where each one of the values takes an integer value. The short version is that upgrading:

- PATCH (e.g., 1.2.3 → 1.2.4) should only contain bug fixes and be fully backwards compatible
- MINOR (e.g., 1.2.3 → 1.3.0) adds new functionality in a backwards-compatible way
- MAJOR (e.g., 1.2.3 → 2.0.0) indicates breaking changes that may require code modifications

In the `pyproject.toml` we have different ways of constraining the ranges of compatible versions of our dependencies:

``` python
[project]
dependencies = [
    "requests==2.32.3",  # Exact version - only this specific version
    "click>=8.0",        # Minimum version - 8.0 or newer
    "numpy>=1.24,<2.0",  # Range - at least 1.24 but less than 2.0
    "pandas~=2.1.0",     # Compatible release - >=2.1.0 and <2.2.0
]
```

**VMs & Containers**

In modern software development the code you write sits atop a significant number of layers of abstraction. This includes things like your programming language runtime, third party libraries, the operating system, or even the hardware itself.... For example, in scientific computing and AI, programs often need specialized libraries and drivers to utilize GPU hardware. Traditionally this wider dependency problem was solved with Virtual Machines (VMs).

- VM - abstract the entire computer and provide a completely isolated environment with its own dedicated operating system. More modern approach is container
- Container - package an application along with its dependencies, libraries, and filesystem, but share the host’s operating system kernel rather than virtualizing an entire computer; They are lighter weight than VMs because they share the kernel, making them faster to start and more efficient to run.
	- Docker (most popular container) - introduced a standardized way to build, distribute, and run containers.
	- ...
	- Container images - ship the entire filesystem of the application as the artifact.
	- An important distinction: a Docker image is the packaged artifact (like a template), while a container is a running instance of that image. You can run multiple containers from the same image.