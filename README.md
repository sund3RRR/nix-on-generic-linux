# Installing nix package manager
First of all you need to install ***nix*** package manager.
## Disable SELinux
***Nix*** only works with ***SELinux*** disabled.

You can check if you are using ***SELinux*** by executing
```
sestatus
```
If you got `SELinux status: disabled` or `bash: sestatus: command not found` then you can proceed to **Installing**.
Otherwise you need to edit `/etc/sysconfig/selinux` file and disable ***SELinux*** policy.
```
sudo sed -i '/SELINUX=enforcing\|SELINUX=permissive/s/.*/SELINUX=disabled/' /etc/sysconfig/selinux
```
Reboot your PC.
## Installing
Just execute install command from official website and follow the instructions of the installer.
```
sh <(curl -L https://nixos.org/nix/install) --daemon
```
Now you can check version of your nix package manager.
```
nix-env --version
```
# Set up home-manager
## Install nix-channels
First of all you need to add and install ***nix-channels*** that required by ***home-manager***.

```
nix-channel --add https://nixos.org/channels/nixos-23.05 nixpkgs
nix-channel --add https://github.com/nix-community/home-manager/archive/release-23.05.tar.gz home-manager
nix-channel --update
```
You can verify installation by executing
```
nix-channel --list
```
## Install home-manager
```
nix-env -iA nixpkgs.home-manager
```
You can verify installation by executing
```
home-manager --version
```
## Configure home-manager
Create `home-manager` directory under `~/.config`
```
mkdir ~/.config/home-manager
```
Then you need to create file `home.nix` inside this directory.
```
touch ~/.config/home-manager/home.nix
```
And edit
```
nano ~/.config/home-manager/home.nix
```
by adding this lines:
```
{ pkgs, ...}: {
  # enable quirks (e.g. set $XDG_DATA_DIRS environment variable) for non NixOS operating systems 
  targets.genericLinux.enable = true;

  # enable unfree proprietary software
  nixpkgs.config.allowUnfree = true;

  # installing software
  home.packages = [ pkgs.google-chrome pkgs.htop];

  home.username = "<USER>";
  home.homeDirectory = "/home/<USER>";
  home.stateVersion = "23.05";
}

```
replace `<USER>` with your username.

Now you can install nix software by adding program name in `home.nix` and then rebuild config.
```
home-manager switch
```
Or install software manually using `nix-env`.
```
nix-env -iA nixpkgs.<YOUR_PACKAGE>
```
