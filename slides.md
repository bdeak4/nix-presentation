---
theme: default
highlighter: shiki
drawings:
  persist: false
css: unocss
---

# Nix

Bartol Deak -- Extension Engine Summer Internship 2022

<!--
Pozdrav svima,
-->

---

# What is Nix really?

- Package manager
- Operating system (NixOS)
- Expression language
- Package repository (Nixpkgs)
- Deployment tool (Nixops)

...

<!--

-->

---

# Nix expression language

- Purely functional DSL
- Used to:
  - Instruct Nix how to build packages
  - Declaratively express system configuration

---

# Nix package manager

<!--
- 2003
-->

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

## Hydra

- Continuous integration service for Nix based projects
- Used to build Nix, NixOS and Nixpkgs

---

# NixOS

- Linux distribution
- NixOS = Linux kernel + Nix package manager for managing software
- Whole system configuration described in Nix expression language

---

# Nix Darwin

---

# Idea of Nix(PM/OS/lang/pkgs/ops)

---

# Q&A
