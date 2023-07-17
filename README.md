<img src="https://raw.githubusercontent.com/mccoy/mac-dev-playbook/master/files/Mac-Dev-Playbook-Logo.png" width="250" height="156" alt="Mac Dev Playbook Logo" />

# Mac Development Ansible Playbook

[![CI][badge-gh-actions]][link-gh-actions]

Fork of [Jeff Geerling's Mac Dev Playbook](https://github.com/geerlingguy/mac-dev-playbook)

This playbook installs and configures most of the software I use on my Mac. There are still a few manual steps needed but for the most part this playbook should be able to automate most of the basic admin of my Mac. If you want a full breakdown of what is happening and how this works please see the README.md in Jeff's original playbook.  I have trimmed out a lot of overview that I thought was unnecessary and have adeed a few bits to explain things I added to the playbook.

## Installation

  1. Check that preliminary steps have been completed:
     1. Perform basic setup including creating an account, icloud login, enable filevault, and set hostname
     1. Ensure Apple's command line tools are installed (`xcode-select --install` to launch the installer)
     1. For Apple silicon, ensure Rosetta is installed (`/usr/sbin/softwareupdate --install-rosetta --agree-to-license`)
  1. [Install Ansible](https://docs.ansible.com/ansible/latest/installation_guide/index.html):

     1. Run the following command to add Python 3 to your $PATH: `export PATH="$HOME/Library/Python/3.8/bin:/opt/homebrew/bin:$PATH"`
     1. Upgrade Pip: `sudo pip3 install --upgrade pip`
     1. If pip fails with a PackageFinder error then the command line tools need to be updated
       1. From the terminal run: softwareupdate --list which produces a list of available updates.
       1. "Command Line Tools for Xcode-<SOME_VERSION>" should be on that list
       1. Run: softwareupdate -i "Command Line Tools for Xcode-<SOME_VERSION>" from previous output. This will then install the updates and the fix for python3. It may take a long time to run.
     1. Install Ansible insto system python: `sudo pip3 install ansible`

  1. Install homebrew: `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`
  1. Install git: `brew install git`
  1. Clone this repository to your local drive.
  1. Run `ansible-galaxy install -r requirements.yml` inside this directory to install required Ansible roles.
  1. Run `ansible-playbook main.yml --ask-become-pass` inside this directory. Enter your macOS account password when prompted for the 'BECOME' password.

> Note: If some Homebrew commands fail, you might need to agree to Xcode's license or fix some other Brew issue. Run `brew doctor` to see if this is the case.

> Note: You can set the hostname using System Preferences > Sharing but a more complete name config can be performed with the following commands:
> ```
> $ sudo scutil --set ComputerName "name"
> $ sudo scutil --set LocalHostName "name"
> $ sudo scutil --set HostName "name"
> ```
>
> The LocalHostName is for Rendezvous/Bonjour and cannot have spaces and is all alphanumeric and HostName is the unix hostname so must follow the standard rules for such names.
>
> This will eventually become a part of the playbook but for now I am just noting it as a manual step to perform.

### Use with a remote Mac

You can use this playbook to manage other Macs as well; the playbook doesn't even need to be run from a Mac at all! If you want to manage a remote Mac, either another Mac on your network, or a hosted Mac like the ones from [MacStadium](https://www.macstadium.com), you just need to make sure you can connect to it with SSH:

  1. (On the Mac you want to connect to:) Go to System Preferences > Sharing.
  2. Enable 'Remote Login'.
  3. Push a local ssh pubkey into the .authorized_keys file of the remote system

> You can also enable remote login on the command line:
>
>     sudo systemsetup -setremotelogin on
>
> This requires 'Full Disk Access' permission to be granted to Terminal in order to set via
> CLI.  This permission cannot be granted via CLI and you must use System Prefernces > Security 
> & Privacy > Privacy and add Terminal to the list of applicaitons that have Full Disk Access.

Then edit the `inventory` file in this repository and change the line that starts with `127.0.0.1` to:

```
[ip address or hostname of mac]  ansible_user=[mac ssh username]
```

If you need to supply an SSH password (if you don't use SSH keys), make sure to pass the `--ask-pass` parameter to the `ansible-playbook` command.

### Running a specific set of tagged tasks

You can filter which part of the provisioning process to run by specifying a set of tags using `ansible-playbook`'s `--tags` flag. The tags available are `dotfiles`, `homebrew`, `mas`, `extra-packages` and `osx`.

    ansible-playbook main.yml -K --tags "dotfiles,homebrew"

## Overriding Defaults

Not everyone's development environment and preferred software configuration is the same.

You can override any of the defaults configured in `default.config.yml` by creating a `config.yml` file and setting the overrides in that file. For example, you can customize the installed packages and apps with something like:

```yaml
homebrew_installed_packages:
  - cowsay
  - git
  - go

mas_installed_apps:
  - { id: 443987910, name: "1Password" }
  - { id: 498486288, name: "Quick Resizer" }
  - { id: 557168941, name: "Tweetbot" }
  - { id: 497799835, name: "Xcode" }

composer_packages:
  - name: hirak/prestissimo
  - name: drush/drush
    version: '^8.1'

gem_packages:
  - name: bundler
    state: latest

npm_packages:
  - name: webpack

pip_packages:
  - name: mkdocs

configure_dock: true
dockitems_remove:
  - Launchpad
  - TV
dockitems_persist:
  - name: "Sublime Text"
    path: "/Applications/Sublime Text.app/"
    pos: 5
```

Any variable can be overridden in `config.yml`; see the supporting roles' documentation for a complete list of available variables. JIm's version of this playbook also supports a `config.yml.username` file where the username var is the current unix username. When the main playbook is run it will first check for a username-specific config and use that if found, then it will check for `config.yml` and only if both of those are missing will it use `config.yml.default` as its source for config vars.

The dotfiles playbook is also run by this playbook to install various configuration dotfiles via a [dotfiles](https://github.com/geerlingguy/dotfiles) repo, including the `.osx` dotfile for configuring many aspects of macOS for better performance and ease of use. You can disable dotfiles management by setting `configure_dotfiles: no` in your configuration. To use your own dotfiles just clone a suitable repo and then add the repo url into the playbook config file.

## Included Applications / Configuration (Default)

Applications (installed with Homebrew Cask):

  - [ChromeDriver](https://sites.google.com/chromium.org/driver/)
  - [Docker](https://www.docker.com/)
  - [Dropbox](https://www.dropbox.com/)
  - [Firefox](https://www.mozilla.org/en-US/firefox/new/)
  - [Google Chrome](https://www.google.com/chrome/)
  - [Handbrake](https://handbrake.fr/)
  - [Homebrew](http://brew.sh/)
  - [LICEcap](http://www.cockos.com/licecap/)
  - [nvALT](http://brettterpstra.com/projects/nvalt/)
  - [Sequel Ace](https://sequel-ace.com) (MySQL client)
  - [Slack](https://slack.com/)

Packages (installed with Homebrew):

  - autoconf
  - bash-completion
  - doxygen
  - gettext
  - gifsicle
  - git
  - github/gh/gh
  - go
  - gpg
  - httpie
  - iperf
  - libevent
  - sqlite
  - mcrypt
  - nmap
  - node
  - nvm
  - php
  - ssh-copy-id
  - cowsay
  - readline
  - openssl
  - pv
  - wget
  - wrk
  - zsh-history-substring-search

Finally, there are a few other preferences and settings added on for various apps and services.

## Full / From-scratch setup guide

Since I've used this playbook to set up something like 20 different Macs, I decided to write up a full 100% from-scratch install for my own reference (everyone's particular install will be slightly different).

It's my hope that I can get the rest of these things wrapped up into Ansible playbooks soon, but for now, these steps need to be completed manually (assuming you already have Xcode and Ansible installed, and have run this playbook).

  1. Set a default iTerm2 theme.
  3. Install all the apps that aren't yet in this setup (see below).
  4. Remap Caps Lock to Escape (requires macOS Sierra 10.12.1+), can also be done using tools like Karabiner-Elements
  5. Set trackpad tracking rate.
  6. Set mouse tracking rate.
  7. Configure extra Mail and/or Calendar accounts (e.g. Google, Exchange, etc.).

You can see my full from-scratch setup document here: [full-mac-setup.md](full-mac-setup.md).

## Testing the Playbook

Jeff has posted instructions for how he builds a [Mac OS X VirtualBox VM](https://github.com/geerlingguy/mac-osx-virtualbox-vm), on which he can continually run and re-run this playbook to test changes and make sure things work correctly.

## Author

This project was created by [Jeff Geerling](https://www.jeffgeerling.com/) (originally inspired by [MWGriffin/ansible-playbooks](https://github.com/MWGriffin/ansible-playbooks)) and updated by [Jim McCoy](https://github.com/mccoy)

[badge-gh-actions]: https://github.com/geerlingguy/mac-dev-playbook/workflows/CI/badge.svg?event=push
[link-gh-actions]: https://github.com/geerlingguy/mac-dev-playbook/actions?query=workflow%3ACI
