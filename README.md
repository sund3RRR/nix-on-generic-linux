# Introduction
Using `nix` on systems other than NixOS is slightly different from using it on NixOS. Without these tricks, you may not see app icons and there will be no 3D acceleration in GUI apps. The `home-manager` package solves these problems and helps make your system more *reproducible* and *declarative*. Having configured everything *once*, you can save the config and use it next time, which saves a lot of time.
# Install nix package manager
First of all you need to install ***nix*** package manager.
### Disable SELinux
***Nix*** only works with ***SELinux*** disabled.

You can check if you are using ***SELinux*** by executing
```
sestatus
```
If you got `SELinux status: disabled` or `bash: sestatus: command not found` then you can proceed to **Installing**.
Otherwise you need to edit `/etc/sysconfig/selinux` file and disable ***SELinux*** policy.
```bash
sudo sed -i '/SELINUX=enforcing\|SELINUX=permissive/s/.*/SELINUX=disabled/' /etc/selinux/config
```
Reboot your PC.
### Install
Just execute install command from official website and follow the instructions of the installer.
```bash
sh <(curl -L https://nixos.org/nix/install) --daemon
```
Now you can check version of your nix package manager.
```bash
nix-env --version
```
# Set up home-manager
### Install nix-channels
First of all you need to add and install ***nix-channels*** which ***home-manager*** requires.

```bash
nix-channel --add https://nixos.org/channels/nixos-23.05 nixpkgs
nix-channel --add https://nixos.org/channels/nixpkgs-unstable unstable
nix-channel --add https://github.com/nix-community/home-manager/archive/release-23.05.tar.gz home-manager
nix-channel --add https://github.com/guibou/nixGL/archive/main.tar.gz nixgl
nix-channel --update
```
You can verify installation by running
```bash
nix-channel --list
```
### Install home-manager
```bash
nix-env -iA nixpkgs.home-manager
```
You can verify installation by running
```bash
home-manager --version
```
### Configure home-manager
Create `home-manager` directory under `~/.config`
```bash
mkdir ~/.config/home-manager
```
Then you need to create file `home.nix` inside this directory.
```bash
touch ~/.config/home-manager/home.nix
```
And edit
```bash
nano ~/.config/home-manager/home.nix
```
by adding this lines:
```nix
{config, pkgs, lib, ...}:
let
  unstable = import <unstable> { config = { allowUnfree = true; }; };

  nixgl = import <nixgl> {} ;
  nixGLWrap = pkg: pkgs.runCommand "${pkg.name}-nixgl-wrapper" {} ''
    mkdir $out
    ln -s ${pkg}/* $out
    rm $out/bin
    mkdir $out/bin
    for bin in ${pkg}/bin/*; do
     wrapped_bin=$out/bin/$(basename $bin)
     echo "exec ${lib.getExe nixgl.auto.nixGLDefault} $bin \$@" > $wrapped_bin
     chmod +x $wrapped_bin
    done
  '';
in {
  # enable quirks (e.g. set $XDG_DATA_DIRS environment variable) for non NixOS operating systems 
  targets.genericLinux.enable = true;

  # enable unfree proprietary software
  nixpkgs.config.allowUnfree = true;

  # install software
  home.packages = with pkgs; [
    nixgl.auto.nixGLDefault
    (nixGLWrap fragments)
    (nixGLWrap amberol)
    (nixGLWrap drawing)
    (nixGLWrap unstable.collision)
   ];


  home.username = "<USER>";
  home.homeDirectory = "/home/<USER>";
  home.stateVersion = "23.05";
}

```
replace `<USER>` with your username.

Now you can install nix software by adding package name in `home.nix` and then rebuild config.
You can search packages on official website https://search.nixos.org/packages. Then put your package inside `home.packages` variable like this.
```nix
home.packages = with pkgs; [
    nixgl.auto.nixGLDefault # don't delete this
    (nixGLWrap <YOUR_PACKAGE>) # if you want 3d acceleration
    <YOUR_PACKAGE> # if you don't want 3d acceleration
    unstable.<YOUR_PACKAGE> # if you want to install package from unstable channel
   ];
```
Don't forget to rebuild configuration, otherwise the changes will not be applied!
```bash
home-manager switch
```
Or install software manually using `nix-env` (no 3d acceleration).
```bash
nix-env -iA nixpkgs.<YOUR_PACKAGE>
```

If there is no installed application icon on the desktop, just log out and log in again.
