# Install Guide

## download

Rather than download archive files, it is highly recommended to use the [git](https://git-scm.com) version control system. The software and plugins are currently available via the following http or ssh URLs:

```
https://github.com/jfennick/workflow_inference_compiler.git
```
```
git@github.com:jfennick/workflow_inference_compiler.git
```

Regular users can use http, but developers should use ssh.

Either way, the commands

```shell
git clone --recursive <URL>
cd workflow_inference_compiler
```

should automatically download both the software and the plugins. Note that if you just use

```shell
git clone <URL>
cd workflow_inference_compiler
```
it will not download the plugins; you will then need to run the following command:

```
git submodule init && git submodule update
```

## docker

Most of the plugins have been packaged up into [docker](https://www.docker.com) containers. If docker is not installed, you can compile workflows and generate DAGs but the workflows will fail at runtime.

## conda

[conda](https://en.wikipedia.org/wiki/Conda_(package_manager)) is an open source, cross platform package management system. It can install both Python dependencies and system binary dependencies, so conda is essentially a replacement for `pip`. The associated package distributions named `anaconda` and `miniconda` provide a database of packages that can be used with the `conda` command. (There is also an open source package distribution called [conda-forge](https://conda-forge.org)) Either one works, so if you already have anaconda installed then great. Otherwise, [miniconda](https://docs.conda.io/en/latest/miniconda.html) is all you need.

After conda is installed, enter the following commands:

```
conda create --name wic
conda activate wic
./conda_devtools.sh
```

Note that if you close your terminal, the next time you open your terminal you will need to re-activate the wic environment:

```
conda activate wic
```

## install

To install into the wic environment, simply use the following command:

```
pip install .
```

You should now have the `wic` executable available in your terminal.

## testing

To test your installation, you can run the example in README.md:

```
wic --yaml examples/gromacs/tutorial.yml --cwl_dir . --cwl_run_local True
```

You can also run the automated test suite. This will run the full set of tests, which takes about 20-30 minutes on a laptop.

```
pytest
```

If you're in a hurry, just run

```
pytest -m fast
```

which tests the compiler only (not the runtime) and only takes about 15 seconds.

### known issues

At runtime, the following warning mesage may be printed to the console repeatedly:

```
WARNING: No ICDs were found. Either,
- Install a conda package providing a OpenCL implementation (pocl, oclgrind, intel-compute-runtime, beignet) or 
- Make your system-wide implementation visible by installing ocl-icd-system conda package.
```

It does not appear to cause any problems, and moreover the suggested solutions don't seem to work, so it appears that this warning can be ignored.

## documentation

The latest documentation is available on [readthedocs](https://workflow-inference-compiler.readthedocs.io/en/latest/) as well as under the `docs/` folder. To build the documentation in html format, use the commands

```
cd docs/
make html
```

Then simply open the file `docs/_build/html/index.html` using any web browser.

## IntelliSense code completion

You can use any text editor to modify the YAML files, but we *highly* recommend [Visual Studio Code](https://code.visualstudio.com). vscode is an extremely popular development environment written by Microsoft, but it is open source. It has a large number of extensions for pretty much everything. Most importantly, vscode provides [IntelliSense](https://code.visualstudio.com/docs/editor/intellisense) code completion. We now support this for our custom YAML file format. Just simultaneously press ctrl-space (the control key and the spacebar) and a list of possible values will be shown specific to each yaml tag (along with in-line documentation via tooltips). We also support completion for gromacs mdp files!

To enable this feature, simply install the YAML vscode extension (by Red Hat). Then add the following line to your `.vscode/settings.json` file (or simply use our settings.json file), and run the following command. Note that the schemas are also re-generated at compile time.

```json
"yaml.schemas": {"autogenerated/schemas/wic.json": "*.yml"},
```

```
wic --generate_schemas_only True --cwl_dir . --yml_dir .
```

After a ~10 second delay, vscode should display "Validating against the Workflow Interence Compiler schema" just above the first line in a \*.yml file.

We also *highly* recommend you add the following line to settings.json, which "Controls whether completions should be computed based on words in the document."

```json
"[yaml]": {
    "editor.suggest.showWords": false
},
```

Using the default value of true, IntelliSense will additionally suggest any text whatsoever that you have previously entered in the current yml file (or other yml files). Of course, even if you select a bad suggestion, IntelliSense will immediately validate it against the schema and prominently highlight any invalid text with a red underline. Moreover, the very first thing the compiler does is use a separate validation implementation to check for errors in the yml files. But in this case it's better to simply disable that particular vscode feature.

### known issues

* Strings which are part of enumerated types (e.g. certain gromacs mdp options) are case-sensitive. Unfortunately, there does not seem to be a simple way to allow case-insensitive strings in the schema. This should not be a problem at runtime (we can ignore case in the compiler), but vscode will complain that 'Value must be ...'


* The following vscode YAML extension issues seem to be invoked by our schemas:

[AutoComplete not showing up in .yaml (but in .json with same json schema file!)](https://github.com/redhat-developer/vscode-yaml/issues/495)

[Autocompletion not showing up with conditional subschemas](https://github.com/redhat-developer/vscode-yaml/issues/222)

Specifically, IntelliSense has trouble parsing the syntax of some of the `config:` tags. Starting from a `config:` tag, if you press enter and then (correctly) indent two spaces to add subtags, you should be able to invoke IntelliSense by pressing ctrl-space. However, that doesn't always work! (The IntelliSense popup box will just contain "No suggestions.") Fortunately there is a simple workaround: just use the explicit JSON braces syntax! Type `config: {}` then, with your cursor between the braces, pressing ctrl-space will reliably work.

Similarly, there also appears to be an IntelliSense issue w.r.t. the `selection:` subtag. Specifically, IntelliSense code completion will not work when there is not enough information for at least one exact match. For example, if you type "MainChai" and press ctrl-space, the result is "No suggestions." But once you type "MainChain", IntelliSense correctly suggests "MainChain", "MainChain+Cb", "MainChain+H". Thus far, the IntelliSense issues appear to be limited to these two tags.

### vscode extensions

These vscode extensions are recommended for users:

* YAML (Red Hat)

These vscode extensions are recommended for developers:

* Python
* MyPy
* Remote Development
* Docker
* CWL (Rabix/Benten)