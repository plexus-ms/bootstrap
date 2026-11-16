# Plexus Bootstrap

An Ansible collection of roles that take a fresh Ubuntu machine to dev-ready.

This repository holds the Ansible collection only. Your package lists and
personal settings go in your own playbook, which lives in your dotfiles
repository next to the dotfiles that chezmoi manages.

## Requirements

- a chezmoi-managed dotfiles repo
- Ubuntu 26.04 or later
- A user that may use `sudo`

## Usage

In your dotfiles repository, such as `github.com/<github-user>/dotfiles`, 
create a `bootstrap/` directory, and add a `requirements.yml`:

```yaml
collections:
  - name: https://github.com/plexus-ms/bootstrap
    type: git
    version: main
```

and a playbook that runs against the local machine:

```yaml
- name: Bootstrap this machine
  hosts: localhost
  connection: local
  roles:
    - role: plexus.bootstrap.apt
      vars:
        apt_repositories: [github-cli]
        apt_packages: [git, jq, ripgrep, gh]
    - role: plexus.bootstrap.zsh
    - role: plexus.bootstrap.chezmoi
      vars:
        chezmoi_repo: <github-user>/dotfiles
```

Also add `bootstrap/` to `.chezmoiignore`.

Then, on the fresh machine, clone the repository to chezmoi's source directory
and run the playbook from there:

```bash
sudo apt install git ansible
git clone https://github.com/<github-user>/dotfiles ~/.local/share/chezmoi
cd ~/.local/share/chezmoi
ansible-galaxy collection install -r bootstrap/requirements.yml
ansible-playbook bootstrap/playbook.yml --ask-become-pass
```

Run the playbook as your normal user, not as root. Roles escalate with `sudo` only
for the tasks that need it, and install per-user tools for the user running the play.
