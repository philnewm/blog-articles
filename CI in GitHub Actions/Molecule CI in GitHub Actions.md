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
ci: 
devto: true
draft: true
---

# Molecule CI in GitHub Actions

## Intro

---

In the [last article](https://philnewm.github.io/molecule-start/) we took a look at a fully automated CI for an Ansible role using *Molecule*. Running this locally works fine during development but to test contentiously against multiple distros, we need to scale this up a bit.
A great and free option are GitHub Actions that run on every new push.
This example will stay with testing for AlmaLinux9 and Ubuntu22.04 for the demonstration.

@@TODO where to find static files if any
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

I will create one workflow per molecule test scenario since this allows for easy badge creating later on fir the `README.md` file.
Depending on your use-case you might want a different structure tho.
To keep the start simple we will walk through the first workflow in a quite *hard-coded* way before splitting the main logic into its own file later on to create a [reusable workflow](https://resources.github.com/learn/pathways/automation/intermediate/create-reusable-workflows-in-github-actions/).  

### Workflow Implementation

The workflow file start - as every `.yml` file should - with `---` at the first line. I also tend to put `...` at the last line even tho it's not required.
Next up the workflow needs a name @@TODO where is it displayed?

```yaml
---
name: Ubuntu22.04-CI
```

- Explain workflow trigger
- Install requirements

```reference
title: "Install requirements"
file: https://github.com/philnewm/ansible-role-repo-control/blob/main/.github/workflows/molecule-ci.yml
start: 47
end: "+1"
language: shell
fold: true
ln: true
```

- Set environment variables for VirtualBox @@TODO research why they are requested

```reference
title: "Set environment variables for VirtualBox"
file: https://github.com/philnewm/ansible-role-repo-control/blob/main/.github/workflows/molecule-ci.yml
start: 53
end: "+1"
language: shell
fold: true
ln: true
```

- Install Virtualbox 7.1.X
- Might fail when VirtualBox servers are down (already happened a few times to me)

```reference
title: "Set environment variables for VirtualBox"
file: https://github.com/philnewm/ansible-role-repo-control/blob/main/.github/workflows/molecule-ci.yml
start: 58
end: "+3"
language: shell
fold: true
ln: true
```

- Install Vagrant

```reference
title: "Set environment variables for VirtualBox"
file: https://github.com/philnewm/ansible-role-repo-control/blob/main/.github/workflows/molecule-ci.yml
start: 69
end: "+3"
language: shell
fold: true
ln: true
```

- Run test scenario

```reference
title: "Set environment variables for VirtualBox"
file: https://github.com/philnewm/ansible-role-repo-control/blob/main/.github/workflows/molecule-ci.yml
start: 79
end: "+0"
language: shell
fold: true
ln: true
```


These commands should be similar or even identical to the ones you ran to install Molecule and VirtualBox on your local system. Implemented in a GitHub Actions Workflow like this they will be executed each time the workflows gets triggered (e.g. on push to main) keeping the setup consistent and repeatable to run in a fully automated way.
And due to the fact that the environment gets constructed from the ground up every time you don't need to worry about any kind of cleanups here.

## Wrap up

This should now work on any new push to the main branch and run your roles for you on different distributions automatically.

Now imagine you implement a bunch of Ansible roles using this CI setup - wouldn't it make sense to have the testing environment separated and just link it to the role repositories.
So updating the testing setup can be done in one place and all roles receive it.
That's exactly what we gonna do next time using Git Sub modules.

Thanks for reading and stay tuned.

- Name for "state" must be standardized
- Ref role name by checking `meta/main.yml` - so entries need to exist
- Tests should be present in `tasks/tests.yml`

@TODO don't run molecule as privileged user by default