---
slug: molecule-gh-action
description: Run Molecule CI in GitHub Actions
tags:
  - ansible
  - molecule
  - GitHub-Actions
authors:
  - philipp
date:
  created: 2025-04-12
  updated: 2025-04-12
categories:
  - infrastructure
ci: https://github.com/philnewm/demo-molecule-example/blob/main/.github/workflows/molecule-ci.yml
devto: true
draft: true
---

# Molecule CI in GitHub Actions

## Intro

---

In the [last article](https://philnewm.github.io/molecule-start/) we took a look at a fully automated CI for an Ansible role using *Molecule*. Running this locally works fine during development but to test contentiously against multiple distributions, we need to scale this up a bit.
A great and free option are GitHub Actions that run on every new push.
This example will stay with testing for AlmaLinux9 and Ubuntu22.04 for the demonstration.

Any files used for this demonstration can be found [here](https://github.com/philnewm/demo-molecule-example)
@@TODO ensure nice preview
<!-- more -->

## Requirements

---

### GitHub

You will need a GitHub account, a repository and a basic understanding of the ways GitHub and git itself work. I will give a brief intro to GitHub Actions and link to the documentation for further details.

### VSCode-Extension

While not an actual requirement I can recommend using VSCode and the official [GitHub Actions](https://marketplace.visualstudio.com/items/?itemName=GitHub.vscode-github-actions) extension to see running actions and their results right in the editor interface. It also provides some level of linting while writing workflows. This way you can spot certain errors before committing, which is always nice to have.
Keep in mind that the linting is quite limited and doesn't seem check content like bash code.

## Overview

---

### Workflow containing all tools

We will keep using VirtualBox, Vagrant and Molecule - just like before. But this time they don't install or run on your local machine they run inside a [GitHub-Hosted-Runner](https://docs.github.com/en/actions/concepts/runners/github-hosted-runners). And since all of that belongs to Microsoft, in the Azure-Cloud, see the [documentation](https://docs.github.com/en/actions/concepts/runners/github-hosted-runners?%7B%7B%3Ccda%3E%7D%7D=#cloud-hosts-used-by-github-hosted-runners) for further details.
We will write a YAML (`.yml`) file called [Workflow](https://docs.github.com/en/actions/how-tos/write-workflows) which will describe the installation of all required tools and python packages. In this case a python virtual environment isn't necessary since the GitHub Actions run inside virtual machines are ephemeral anyway.
### Side node on Virtual-Box

Initially I intended to run [libvirt](https://libvirt.org/) VMs in GitHub runners for a lower overhead of additional tools and install times.

But *Spoiler Alert* this did not work:
Libvirt VMs require nested virtualization which seems to be not enabled for GitHub hosted Runners. Even tho this [article](https://actuated.com/blog/kvm-in-github-actions) mentions other ways to get this going.
Virtual-Box on the other hand works fine for me but if I ever give it another try and maybe get it working I'll add it here.
Currently the only way I see to reduce the install and runtime overhead is using containers instead of VMs wherever possible (molecule supports it anyway) but this is a topic for another day.

## GitHub Actions

This is GitHub's integrated CI/CD and Automation platform, see the [docs](https://docs.github.com/en/actions/about-github-actions/understanding-github-actions) for more detailed information. As operating system version we gonna use Ubuntu 24.04 since this is the latest Linux runner image provided by GitHub at this point. See also this overview of currently available [runners and their specs](https://docs.github.com/en/actions/reference/runners/github-hosted-runners#supported-runners-and-hardware-resources) as well as the [repository](https://github.com/actions/runner-images) for their images.

## Role Adjustments

---

### Per-Distribution Scenarios

To test the Ansible role against multiple distributions at in GitHub actions while keeping every run visible,  we need to add dedicated molecule scenarios.
This simply means to add additional sub-directories in the `moelcule/` directory since this is the place it expects those.
So in this case a `almalinux9` and a `ubuntu2204` sub-directory shall do it.
Now we could just copy over all the `.yaml` files form the default scenario but this would be very inefficient.
Instead we just copy the `molecule.yml` files, the one we actually need to adjust per scenario and symlink the others.

From the repositories root run this neat one liner, which I split here over multiple lines or run the commands step by step.

```bash
mkdir -p molecule/{almalinux9,ubuntu2204} && \
cp molecule/default/molecule.yml molecule/almalinux9/ && \
cp molecule/default/molecule.yml molecule/ubuntu2204/ && \
(cd molecule/almalinux9 && ln -s ../default/{create,converge,verify,destroy}.yml .) && \
(cd molecule/ubuntu2204 && ln -s ../default/{create,converge,verify,destroy}.yml .)
```

After that just change the `name`, `box` and the `box_version` in `molecule/ubuntu2204/molecule.yml` to something like this, based on [vagrant cloud box](https://portal.cloud.hashicorp.com/vagrant/discover/ubuntu/jammy64)

```reference
title: "Worklfow head"
file: https://github.com/philnewm/demo-molecule-example/blob/main/molecule/ubuntu2204/molecule.yml
start: 9
end: "+2"
fold: true
ln: true
```

The result should look something like this

```bash
ls -go /molecule/ubuntu2204
total 4
lrwxrwxrwx. 1  23 Aug 16 11:29 converge.yml -> ../default/converge.yml
lrwxrwxrwx. 1  21 Aug 16 11:29 create.yml -> ../default/create.yml
lrwxrwxrwx. 1  22 Aug 16 11:29 destroy.yml -> ../default/destroy.yml
-rw-r--r--. 1 555 Aug 16 11:35 molecule.yml
lrwxrwxrwx. 1  21 Aug 16 11:29 verify.yml -> ../default/verify.yml
```

### Logic Adjustments

To get this Ansible working for AlmaLinux9 and Ubuntu2204 a like I needed to implement a couple of adjustments, which mainly focus on improving the re-usability of the logic. For example the package installation now looks like this.

```reference
title: "Worklfow head"
file: https://github.com/philnewm/demo-molecule-example/blob/main/tasks/main.yml
start: 12
end: "+4"
fold: true
ln: true
```

```reference
title: "Worklfow head"
file: https://github.com/philnewm/demo-molecule-example/blob/main/defaults/main.yml
start: 1
end: "+4"
fold: true
ln: true
```

Turns out Depending on the OS family (AlmaLinux9 is based on RedHat, while Ubuntu is based on Debian) the package has a different name and writing one task per option is pretty redundant. And this change needed to be applied to the `service ` task and a bunch of other places.

```reference
title: "Worklfow head"
file: https://github.com/philnewm/demo-molecule-example/blob/main/tasks/main.yml
start: 6
end: "+5"
fold: true
ln: true
```

Other than that the Ubuntu vagrant box comes with an outdated or empty apt cache so a update task was required. Those were the critical changes, see this [commit](https://github.com/philnewm/demo-molecule-example/commit/43345557fc010a510b8c94df3a06010b150c9f55) for details

## Setup

---
### Create a Workflow

After creating and cloning the repository (assuming it's a new one), we need to create a `.yml` file inside the repository at this location.

```
<repo-root>/.github/workflows/<workflow-name>.yml
```

You might need to create some of the directories first.
GitHub expects workflow files in that exact location and will call them only this way.
### Workflow-Structure

I will create one workflow per molecule test scenario since this allows for easy workflow badge creating later on for the `README.md` file.
Depending on your use-case you might want a different structure tho.
To keep the start simple we will walk through the first workflow in a quite *hard-coded* way before splitting the main logic into its own file later on to create a [reusable workflow](https://resources.github.com/learn/pathways/automation/intermediate/create-reusable-workflows-in-github-actions/).  

### Workflow Implementation

I'll provide the workflow `.yml` file here in snippets and explain briefly below them what they do. Fortunately the human readable and simple structure of YAML should make understanding them pretty straight forward. Any `steps` using the `run` statement execute bash commands. When using `ubuntu-latest` runner here, these commands get executed like they would in a terminal.

```reference
title: "Worklfow head"
file: https://github.com/philnewm/demo-molecule-example/blob/main/.github/workflows/molecule-ci.yml
start: 1
end: "+11"
fold: true
ln: true
```

- Line 1 - Any YAML starts with  `---` at the first line
- Line 3 - Workflow name, used as reference in several overviews
- Line 5 ff. - Workflow trigger event, "*on push to main*"  is used here, see [other options](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows) 
- Line 8 ff. - Ignore paths used here for `.md` and other files with config/meta information

```reference
title: "Job definition"
file: https://github.com/philnewm/demo-molecule-example/blob/main/.github/workflows/molecule-ci.yml
start: 14
end: "+2"
fold: true
ln: true
```

- Line 14 - The only job we need gets defined (*molecule-ci*) will show up in summary
- Line 16 - GitHub runner type `ubuntu-latest`, other runner images can be found [here](https://docs.github.com/en/actions/reference/runners/github-hosted-runners#supported-runners-and-hardware-resources)

```reference
title: "Install requirements"
file: https://github.com/philnewm/demo-molecule-example/blob/main/.github/workflows/molecule-ci.yml
start: 18
end: "+4"
language: shell
fold: true
ln: true
```

- Line 18 - List of steps starts, these execute the workflow logic
- Line 19 ff. - Repo gets cloned onto the runner, by default, the repo it was started from

```reference
title: "Install requirements"
file: https://github.com/philnewm/demo-molecule-example/blob/main/.github/workflows/molecule-ci.yml
start: 24
end: "+12"
language: shell
fold: true
ln: true
```

- Line 24 ff. - Run [`setup-python`](https://github.com/actions/setup-python) to let the workflow install the requested python version
- Line 29 ff. - Install Python requirements in case a `requirements.txt` file exists
- Line 34 ff. - Install Ansible requirements in case a `requirements.yml` file exists

```reference
title: "Set environment variables for VirtualBox"
file: https://github.com/philnewm/demo-molecule-example/blob/main/.github/workflows/molecule-ci.yml
start: 38
end: "+3"
language: shell
fold: true
ln: true
```

- Line 38 ff. - Environment variables used by *VirtualBox* @@TODO what are they used for?

```reference
title: "Set environment variables for VirtualBox"
file: https://github.com/philnewm/demo-molecule-example/blob/main/.github/workflows/molecule-ci.yml
start: 43
end: "+8"
language: shell
fold: true
ln: true
```

- Line 43 ff. - Setup *VirtualBox* repository and install *VirtualBox* 7.1.X
- Line 50 ff. - Verify *VirtualBox* version after installation

> [!warning]- Note on reliability
> Workflow might fail when *VirtualBox* servers are down or vagrant boxes are temporarly unavailable, already happened a few times.
> Also sometimes the AlmaLinux9 box just fails to create for no particular reason - just restart the job
> ![[re_run_job.png|200]]

```reference
title: "Set environment variables for VirtualBox"
file: https://github.com/philnewm/demo-molecule-example/blob/main/.github/workflows/molecule-ci.yml
start: 53
end: "+9"
language: shell
fold: true
ln: true
```

- Line 53 ff. - Setup *Hashicorp* repository and install latest version of *Vagrant*
- Line 61 ff. - Verify *Vagrant* version after installation

```reference
title: "Set environment variables for VirtualBox"
file: https://github.com/philnewm/demo-molecule-example/blob/main/.github/workflows/molecule-ci.yml
start: 64
end: "+"
language: shell
fold: true
ln: true
```

- Line 64 - Run `molecule test` for chosen scenario 

These commands should be similar or even identical to the ones you ran to install Molecule and VirtualBox on your local system. Implemented in a GitHub Actions Workflow like this they will be executed each time the workflows gets triggered (e.g. on push to main) keeping the setup consistent and repeatable to run in a fully automated way.
And due to the fact that the environment gets constructed from the ground up every time you don't need to worry about any kind of cleanups here.

In case a workflow run fails due to hick-ups that are not related to the current code you can simply rerun it from the workflow from the workflows summary page found under the *Actions* tab of your repository.

## Wrap up

This should now work on any new push to the main branch and run your roles for you on different distributions automatically.

Now imagine you implement a bunch of Ansible roles using this CI setup - wouldn't it make sense to have the testing environment separated and just link it to the role repositories.
So updating the testing setup can be done in one place and all roles receive it.
That's exactly what we gonna do next time using Git Sub modules.

Thanks for reading and stay tuned.