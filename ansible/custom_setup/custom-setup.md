## Intro

I found a lot of Ansible articles out there and also a bunch related to molecule testing.
Most of these approaches didn't quite meet my requirements - so here is my approach.
Just as a heads-up, this article is by no means a setup guide for beginners rather a bit of documentation (also for myself) about how and why I built this testing setup the way it works now.

## Requirements

* Develop Ansible roles locally
* Test for one or two distributions while developing
* Run extended tests on GitHub
* Keep it fast and convenient for both cases
* UI pup up when VM starts
## Structure

@@TODO Show directory structure

## Encountered issues - solved in molecule.yml file

I choose Ansible Molecule as development and testing package to enable quick iterations and easy resets. Even tho restarting virtualbox VMs still takes some time.
For this article I will just focus on the default scenario which I used for local testing.
After a lot of trial and error I came up with this molecule.yml config file

```yaml
---

dependency:
  name: galaxy
driver:
  name: vagrant
  provider:
    name: virtualbox
    type: virtualbox
lint: |
  set -e
  yamllint
  ansible-lint
platforms:
  - name: Alma9
    box: philnewm/alma9.4-gnome
    box_version: "0.1.11"
    memory: 4096
    cpus: 2
    provider_options:
      gui: true
    interfaces:
      - auto_config: true
        network_name: private_network
        type: "static"
        ip: "192.168.56.10"
	provider_raw_config_args:
```

Due to limited documentation (or my inability to find it) I ended up figuring out most of the config options by checking the source code on how molecule creates the Vagrantfile

* Research what I need - or figure while developing
## Notes on VM Spec requirements

* Certain desktop applications may require some more RAM so 4096MB(4GB) seems reasonable so far (Firefox was not able to open up with just 2048MB(2GB) of RAM 
## Current State
* What do I use
* Why do I use it
* Other options available
* Different molecule scenarios for different purposes
* Explain custom scenario files
* Explain step by step what happens
* Tests in Ansible allow utilizing the roles data structures without any additional parsing


## Auto generate readme

* started implementing a [python package](https://github.com/philnewm/ansible-readme)
* Automatically generate file structure tree using `tree` or `lsd` in bash
* Needs equivalent solution for windows power-shell
* example code
```bash
tree --charset utf-8
alias tree='tree | sed "s/^|/📂 /g"'
tree | sed -e "s/\.txt$/ 📄/g" -e "s/\// 📂/g"
lsd --tree // requires NERD FONT
tree | sed 's/^\([│ ]*\) \([^│ ]*\)/\1 📁 \2/'
```

## Future Ideas

## Wrapping Up

@@ TODO
* explain key options in use