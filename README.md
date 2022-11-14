# govcookiecutter-dance
Guidance for using (or reminding yourself how to use) govcookiecutter

**Last Updated:** 13.11.2022. 

**Reason:** Update changes since adoption of PEP 517 & 518 standard

## Deps

```
Cookiecutter 2.1.1
Govcookiecutter 1.3.1
pre-commit 2.20.0

```

## Intro

If it’s your first time using, I suggest reading through the cookie cutter repo readme: https://github.com/best-practice-and-impact/govcookiecutter

And also this super useful [YouTube video](https://www.youtube.com/watch?v=N7_d3k3uQ_M). 

If it’s not your first time setting this up and you setup new repos infrequently enough to forget everything (like me), the rest of this guide may be of use.


**Note - simpler when following this guide to make your repo locally and push it up to GitHub once finished.**


1. In terminal (insert cli of choice), change directory to the parent dir where you would like your new repo to live.
2. Run `cookiecutter https://github.com/best-practice-and-impact/govcookiecutter.git` and follow the prompts.
3. Create virtual env
4. Activate virtual env
5. `git init`
6. `pre-commit install`
7. Make your first commit
8. Does pre-commit complain about unicode_fun? Go to [Troubleshooting](#unicode_fun)
9. For packaging purposes, using the new PEP-517 & 518 standard with this [python.org guidance](https://packaging.python.org/en/latest/tutorials/packaging-projects/):


<details>
  <summary>Adjust repo structure <strong>click to expand</strong></summary>

- Adjust cookie cutter repo structure, package location to `src/package_name`.

Example:
```
src
├── README.md
└── package_name
    ├── __init__.py
    ├── make_data
    │   └── __init__.py
    ├── make_features
    │   ├── __init__.py
    │   └── get_buildings.py
    ├── make_models
    │   └── __init__.py
    ├── make_visualisations
    │   └── __init__.py
    └── utils
        └── __init__.py
        
```
</details>
<details>
<summary>Add a pyproject.toml <strong>click to expand</strong></summary>

Example:
```
[project]
name = "SomeAwesomeName"
version = "0.0.1"
authors = [
  { name="firstname surname", email="someone@somewhere.co.uk" },
]
description = "An informative description"
readme = "README.md"
requires-python = ">=3.9.7"
classifiers = [
    "Programming Language :: Python :: 3",
    "License :: OSI Approved :: MIT License",
    "Operating System :: OS Independent",
]

[project.urls]
"Homepage" = "Probably a GitHub repo url"
"Bug Tracker" = "Likely to be the repo issues url"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

# `coverage` configurations
[tool.coverage.run]
source = [
    "./src"
]

[tool.coverage.report]
exclude_lines = [
    "if __name__ == .__main__.:"
]
omit = ["src/**/__init__.py"]

# `isort` configurations
[tool.isort]
profile = "black"

# `pytest` configurations
[tool.pytest.ini_options]
addopts = [
    "-vv",
    "--doctest-modules"
]
doctest_optionflags = "NORMALIZE_WHITESPACE"
testpaths = [
    "./tests"
]
```
</details>

10. Finally, install your package in editable mode: `Pip install -e .`

## Troubleshooting


### unicode_fun

Increment the black version within the pre-commit-config.yaml. See the refs within the hooks included with govcookiecutter template below:

  ``` 
  - repo: https://github.com/psf/black
    rev: 22.8.0 # HERE
    hooks:
      - id: black
        name: black - consistent Python code formatting (auto-fixes)
        language_version: python # Should be a command that runs python3.6+     
  - repo: https://github.com/nbQA-dev/nbQA
    rev: 0.12.0
    hooks:
      - id: nbqa-isort
        name: nbqa-isort - Sort Python imports (notebooks; auto-fixes)
        args: [ --nbqa-mutate ]
        additional_dependencies: [ isort==5.8.0 ]
      - id: nbqa-black
        name: nbqa-black - consistent Python code formatting (notebooks; auto-fixes)
        args: [ --nbqa-mutate ]
        additional_dependencies: [ black==22.8.0 ] # HERE
        
```

### Importing src to Jupyter

In order for local package updates to feed through to your notebooks, you will need to restart the kernel every time you change your package contents. This is a bit inconvenient. Instead, you can use the jupyter autoreload extension to avoid those restarts, place the magic commands above your import call within the code cell:

```
%load_ext autoreload
%autoreload 2
```

### Pre-commit Flake 8 E402

Now that you used the autoreload extension (see above) flake8 complains about imports not at top of file (E402)? You can have flake 8 ignore this specific error for specified lines by adding `# noqa: E402` to the end of the line:

```
%load_ext autoreload
%autoreload 2
import os.path  # noqa: E402

from pyprojroot import here  # noqa: E402
```

### Pre-commit detect secrets complains about notebook hashes:

**One-time fix:**
Open the notebook in a text editor and delete the cell hashes that are triggering the detect secrets hook. - Thanks to @MartinWoodONS for this tip. You can right click on a notebook in VS Code and select ‘Open With’ to select a text editor.

Have detect secrets ignore common false-positives in notebooks. Thanks to @SStock1 for this snippet. Update the .pre-commit-config.yaml with:

```
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.0.3
    hooks:
      - id: detect-secrets
        name: detect-secrets - Detect secrets in staged code
        args: [ "--baseline", ".secrets.baseline", '--exclude-files', '.*\.ipynb$',  ]
        exclude: .*/tests/.*|^\.cruft\.json$
      - id: detect-secrets
        name: 'detect-secrets-jupyter'
        args: ['--exclude-files', '.*[^i][^p][^y][^n][^b]$', '--exclude-lines', '"(hash|id|image/\w+)":.*', ]
  ```


