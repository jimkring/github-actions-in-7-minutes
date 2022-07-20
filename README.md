
# GitHub Actions in 7 Minutes
## *Automating Development Tasks in the Cloud*
by [Jim Kring](https://github.com/jimkring) on July 20, 2022 at [GDevCon NA](https://gdevconna.org/) in Golden Colorado, USA

# Overview

## GitHub

- Software project hosting in the cloud.
- Source code control (git), issue tracking, and much more.
- The defacto home for open source projects.
- Purchased by Microsoft in 2018 for 7.5B USD

## GitHub Actions

- Perform automation tasks related to your project.
- You can create your own actions and use a variety of ones created by others.


# Your Questions Answered

## GitHub Actions → What are they?
- They are recipes/scripts for work to be performed
- They (typically) perform work on your project source files

## GitHub Actions → Where do they run?
- They run on servers "in the cloud" (not your local development computer)
- They can run on different platforms (Linux, Mac, Windows)

## GitHub Actions → When do they run?

They run when events occur:
- manually (by button press on github.com project's 'actions' page)
- When code is pushed to repository
- On a pull request
- Periodically daily/hourly/etc.
- When external events "webhooks" occur


# Under the Hood

## Key Terms (Table)

| <div style="width:140px">Term</div> | Details                                                                                                                                                                                                                                 |
| ----------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| GitHub Actions                      | a big bucket of functionality for automating project tasks                                                                                                                                                                              |
| Workflow                            | a YAML file that defines a set of jobs                                                                                                                                                                                                  |
| YAML (files)                        | "Yet Another Markup Language" files are great for hierarchical config data used by build automation systems (e.g. GitHub and GitLab). more on that later..                                                                              |
| Job                                 | a set of steps that will run on a runner                                                                                                                                                                                                |
| Step                                | an atomic unit of work that uses an `action`, has a set of inputs, work, and outputs                                                                                                                                                    |
| Action                              | a local or remote folder with action.yml and support files that does the work of the action                                                                                                                                             |
| Runner                              | a "computer" in the cloud that can do work (actions) for you                                                                                                                                                                            |
| Hosted Runner                       | a runner provided by GitHub                                                                                                                                                                                                             |
| Self-hosted Runner                  | a computer on which you have [installed the github runner software](https://docs.github.com/en/actions/hosting-your-own-runners/adding-self-hosted-runners) and which isregistered with your github repository/project or organization. |
| Runner labels                       | a set of labels used to specify the operating system and installed software configuration of the runner (e.g. `windows-2019`, `ubuntu-latest`, `macos-latest`, `self-hosted`)                                                           |
|                                     |

## Workflow YAML Files

*.yml files in the workflows folder define jobs that will run when certain events happen.

Here's an example:

```yaml
# .github/workflows/build-and-release.yml
name: Build and Release
on:
  workflow_dispatch:
  push: main
  pull_request:

env: # set environment variables
  FOO_VAR: 'bar_value'

jobs:
  build:
    runs-on: windows-latest
    steps:
      # http://github.com/actions/checkout repo tag 'v3'
      - uses: actions/checkout@v3
        with:
          foo: 'bar' # input parameter
      - run: build_exes.ps1
        shell: powershell
  release:
    needs: build # depends on the build job (can't run in parallel)
    runs-on: ubuntu-latest
    steps:
      # http://github.com/actions/checkout repo tag 'v3'
        - uses: actions/upload-artifact@v3
        with:
            name: executable
            path: builds/*.exe
```

## .github Folder


Where github actions looks for stuff.

    .github/
        actions/ (where local actions are kept)
            your_action_name/
                action.yml (defines inputs, steps, & outputs)
                *.* (files your action uses to do work) 
	    workflows/ (put your workflows here)
            your_workflow_name.yml (each YAML file is a workflow)
                jobs: (workflows execute jobs on runners)
                    steps: (steps can call/use "actions")
                        uses: (action name)



# Examples


### Build Python Script into EXE


https://github.com/marketplace/actions/build-exe-from-python-script

![image](https://user-images.githubusercontent.com/381432/180043106-7d9cd5fb-ac9d-4e3a-aec0-1f2b73e74744.png)


## Multi-Platform Builds


To build for a given platform, just choose a runner of the appropriate operating system.  You can even build for multiple platforms in a single workflow using a matrix strategy, as shown below:

Here we see a workflow from the [jimkring/kasa-cli](https://github.com/jimkring/kasa-cli) project.

```yaml
jobs:
  build:
    strategy:
      matrix:
        os: [macos-latest, ubuntu-latest, windows-latest]
      
    runs-on: ${{ matrix.os }}
    
    steps:
      - name: Check-out repository
        uses: actions/checkout@v3

      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10' # Version range or exact version of a Python version to use, using SemVer's version range syntax
          architecture: 'x64' # optional x64 or x86. Defaults to x64 if not specified
          cache: 'pip'
          cache-dependency-path: |
            **/requirements*.txt
            
      - name: Install Dependencies
        run: |
          pip install -r requirements.txt -r requirements-dev.txt
          
      - Name: Build Executable
        uses: jimkring/python-script-to-exe@v0.2.0
        with:
          script-name: kasa_cli
          onefile: true
  
      - name: Upload Artifacts
        uses: actions/upload-artifact@v3
        with:
          name: ${{ runner.os }} Build
          path: |
            build/*.exe
            build/*.bin
            build/*.app/**/*
```

And, here's what a resulting job run looks like:

https://github.com/jimkring/kasa-cli/actions/runs/2682890462

![image](https://user-images.githubusercontent.com/381432/179555752-021fd3d6-3f33-4f5f-bc44-0461491813fc.png)

You can see that executable binaries were created for Mac, Linux, and Windows.

# Using Actions with LabVIEW

