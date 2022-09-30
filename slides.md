---
theme: default
highlighter: shiki
drawings:
  persist: false
css: unocss
---

# Nix

Bartol Deak -- Extension Engine Summer Internship 2022

---

# What I worked on durning 2nd part of internship

- Infrastructure provisioning with Pulumi
- Server configuration management with Nix(OS)
- Deployment automatization with Github Actions
- Explored alternative ways to do development environments

---

# What is Nix really?

- Package manager
- Operating system (NixOS)
- Expression language
- Package repository (Nixpkgs)

...

---

# Nix expression language

<div class="grid grid-cols-2 gap-4">
<div>

- Purely functional DSL
- Declarative
- Used to:
  - Instruct Nix how to build packages
  - Express system configuration

</div>
<div>

```nix
{ argument, ... }:
let 
  name = "Bartol";
in {
  greeting = {
    message = "Hello, ${name}";
  };

  # shortcut
  greeting.message = "Hello, ${name}";
}
```

</div>
</div>

---

<div class="grid grid-cols-2 gap-4">
<div>

# Nix package manager

- Cross platform
- Initial release in 2003
- Packages are stored in nix store and linked to specific locations on system.
  For example:
  ```
  $ which rg
  /run/current-system/sw/bin/rg

  $ readlink /run/current-system/sw/bin/rg
  /nix/store/w4mshhlbxrm6ydp06li19sp2g8332krl-ripgrep-13.0.0/bin/rg
  ```
- Ensures that packages are reproducible and don't have undeclared dependencies
- [Full ripgrep package example &rarr;](https://github.com/NixOS/nixpkgs/blob/master/pkgs/tools/text/ripgrep/default.nix)

</div>
<div>

```nix
rustPlatform.buildRustPackage rec {
  pname = "ripgrep";
  version = "13.0.0";

  src = fetchFromGitHub {
    owner = "BurntSushi";
    repo = pname;
    rev = version;
    sha256 = "0pdcjzfi0fclbzmmf701fdizb95iw427vy3m1svy6gdn2zwj3ldr";
  };
  cargoSha256 = "1kfdgh8dra4jxgcdb0lln5wwrimz0dpp33bq3h7jgs8ngaq2a9wp";

  installCheckPhase = ''
    file="$(mktemp)"
    echo "abc\nbcd\ncde" > "$file"
    $out/bin/rg -N 'cd' "$file"
    $out/bin/rg -N 'cd' "$file"
  '';

  meta = with lib; {
    description = "A utility that combines the usability of The Silver Searcher with the raw speed of grep";
    license = with licenses; [ unlicense mit ];
    maintainers = with maintainers; [ tailhook globin ma27 zowoq ];
    mainProgram = "rg";
  };
}
```

</div>
</div>

---

# Nixpkgs

- Collection of over 80 000 software packages
- Size of similar repositories:

  ```sh
  # Debian
  $ curl -s 'https://packages.debian.org/unstable/allpackages?format=txt.gz' | gunzip - | tail -n+7 | cut -d' ' -f1 | uniq | grep -vE -- '-(dev|doc|data|dbg(sym)?)$' | wc -l
  97161

  # Arch
  $ curl -s 'https://archlinux.org/packages/?repo=Community&repo=Core&repo=Extra&repo=Multilib' | grep -Eom1 '[0-9]+ matching packages found.' | cut -d' ' -f1
  13109

  # AUR
  $ curl -s 'https://aur.archlinux.org/packages.gz' | gunzip - | tail -n+2 | wc -l
  85199
  ```

---
layout: two-cols
---

# NixOS

- Linux distribution
- NixOS = Linux kernel + Nix package manager
- Whole system configuration described in Nix expression language
- Each update builds new "generation"

::right::

```nix
{ config, pkgs, ... }:
{
  imports = [ <nixpkgs/nixos/modules/virtualisation/amazon-image.nix> ];
  nix.trustedUsers = [ "@wheel" ];

  networking.hostName = "nixos-vm";
  networking.firewall.allowedTCPPorts = [ 80 443 ];

  users.users.rasporedar = {
    isNormalUser = true;
    extraGroups = [ "wheel" "docker" ];
    openssh.authorizedKeys.keys = [ "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAfuAQ43SM0EVulTuivIuAGI0P2RcREUY0nTRtlolZDZ b@bdeak.net" ];
  };
  security.sudo.wheelNeedsPassword = false;
  services.openssh.passwordAuthentication = false;

  services.nginx = {
    enable = true;
    virtualHosts."example.com" = {
      enableACME = true;
      forceSSL= true;
      locations."/".proxyPass = "http://localhost:3001";
    };
  };
  environment.systemPackages = with pkgs; [ vim tmux htop ];
}
```

---

<div class="grid grid-cols-2 gap-4">
<div>

```nix
{ config, pkgs, ... }:
{
  # ...
  services.xserver.enable = true;
  services.xserver.displayManager.gdm.enable = true;
  services.xserver.desktopManager.gnome.enable = true;

  users.users.bd = {
    isNormalUser = true;
    extraGroups = [ "networkmanager" "wheel" "docker" ];
  };
  security.sudo.wheelNeedsPassword = false;

  environment.systemPackages = with pkgs; [
    firefox-wayland
    vim
    git
    ripgrep
  ];

  virtualisation.docker.enable = true;

  services.openssh.enable = true;
  services.fprintd.enable = true;
  # ...
}
```

</div>
<div>

```nix
{ config, pkgs, ... }:
{
  # ...
  home-manager.users.bd = {
    programs.bash = {
      enable = true;
      sessionVariables = {
        EDITOR = "vim";
      };
    };

    programs.vim = {
      enable = true;
      plugins = with pkgs.vimPlugins; [
        vim-nix
      ];
      settings = {
        ignorecase = true;
      };
      extraConfig = ''
        au FileType markdown setl ts=2 sw=2 et
      '';
    };
  };
  # ...
}
```

</div>
</div>

---

# Nix Darwin

- Brings the convenience of a NixOS declarative system configuration to macOS
- Uses Nix package manager, Nix language and Nixpkgs
- Benefit of not having to switch to Linux
- Very similar to NixOS but not 1:1 replica

---

<div class="grid grid-cols-2 gap-4">
<div>

# Nix shell

- provides easy way to create virtual environments and try out packages

```
[bd@pc:~]$ nix-shell -p tree

[nix-shell:~]$ tree
.
├── node_modules
│   ├── acorn
│   │   ├── bin
│   │   │   └── acorn
│   │   ├── CHANGELOG.md
│   │   ├── dist
│   │   │   ├── acorn.d.ts

[nix-shell:~]$ ^D
exit

[bd@pc:~]$ tree
The program 'tree' is not in your PATH. 
```

</div>
<div>

```nix
with import <nixpkgs> { };
stdenv.mkDerivation {
  name = "azure-nixos-image_env";

  nativeBuildInputs = [
    azure-cli
    azure-storage-azcopy
    bash
    jq
    cacert
  ];

  AZURE_CONFIG_DIR="/tmp/azure-cli/.azure";
}
```

- External dependencies can easily be described in `shell.nix` files.
  And applied via:

  ```sh
  $ nix-shell
  ```

</div>
</div>

---

# 3 big ideas of Nix(PM/OS/lang/pkgs/shell)

- **Reproducibility** - If package works on one machine, it must also work on another
- **Declarativeness** - Easier to share and less error prone than imperativeness
- **Reliability** - Able to rollback to last working/clean version of system

---
layout: center
---

# Q&A

---
layout: center
---

# <https://nix-presentation.bdeak.net>
