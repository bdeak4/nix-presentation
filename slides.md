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

# What is Nix really?

- Package manager
- Operating system (NixOS)
- Expression language
- Package repository (Nixpkgs)
- Deployment tool (NixOps)

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
layout: two-cols
---

# Nix package manager

- Cross platform
- Initial release in 2003
- [nesto o storeu i example (ls - symlink u store), TODO]
- Ensures that packages are reproducible and don't have undeclared dependencies
- [Full ripgrep package example &rarr;](https://github.com/NixOS/nixpkgs/blob/master/pkgs/tools/text/ripgrep/default.nix)

<!-- todo -->

::right::

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

# Nix shell

---

# NixOps

<!-- todo -->

---

# Nix Darwin

<!-- todo -->

---

# Idea of Nix(PM/OS/lang/pkgs/shell/Ops)

<!-- todo -->

---
layout: center
---

# Q&A

<!-- todo -->