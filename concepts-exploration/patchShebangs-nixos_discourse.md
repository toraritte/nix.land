source: https://discourse.nixos.org/t/what-is-the-patchshebangs-command-in-nix-build-expressions/12656

I think I figured it out (see below) but would you comment on what I did get wrong and maybe answer the parts that I'm unsure of? Thanks in advance.

UPDATE: Amended this post after @jtojnar 's extensive review, and included comments from @jonringer . Tried to archive the original post but Internet Archive played tricks on me...

## 0. Introduction

`patchShebangs` is indirectly mentioned in the [Nixpkgs manual][1] when describing [the phases of the generic builder][2] of the [Nixpkgs standard environment][3], stating that the [fixup phase][4] at one point

> rewrites the interpreter paths of shell scripts to paths found in `PATH`. E.g., `/usr/bin/perl` will be rewritten to `/nix/store/some-perl/bin/perl` found in `PATH`.
[->][5]

**It is important to note that** (paraphrasing [@jonringer's comment][14]), "_the `patchShebangs` command is only available during the build if you source the [`$stdenv/setup`][15] setup hook_" (more on that below) "_provided by `stdenv`'s (the [Nixpkgs standard environment][3]'s) default builder (you get this by default when using `stdenv.mkDerivation`), which is why the starting point of almost all nix expressions is `import <nixpkgs> {}, stdenv.mkDerivation`, or something similar._"

## 1. Where is `patchShebangs` defined

The file [`patch-shebangs.sh`][6] in the [Nixpkgs repo][7] (also documented at [6.7.4. patch-shebangs.sh][8]) defines the [`patchShebangs` function][9], which in turn is used to implement [`patchShebangsAuto`][10], the [setup hook][11] that is [registered to run][12] during the [fixup phase][4].

## 2. Why are shebang rewrites needed when building Nix packages?

According to the comment at the top of [`patch-shebangs.sh`][6]:

```text
# This setup hook causes the fixup phase to rewrite all script
# interpreter file names (`#!  /path') to paths found in $PATH.  E.g.,
# /bin/sh will be rewritten to /nix/store/<hash>-some-bash/bin/sh.
# /usr/bin/env gets special treatment so that ".../bin/env python" is
# rewritten to /nix/store/<hash>/bin/python.  Interpreters that are
# already in the store are left untouched.
# A script file must be marked as executable, otherwise it will not be
# considered.
```

> **IMPORTANT NOTE**: The criterion above that the "_**script file must be marked as executable, otherwise it will not be considered**_" is an important one.

The line in a shell script starting with `#!` is called [shebang][16] (among others), and it is an interpreter directive to the executing shell as for what program to use to decipher the text below; the characters after `#!` [has to consitute an absolute path that points to this executable][17]. For example, `#!/usr/bin/python3` will expect to find the `python3` program there to carry out the commands in the body of the shell script written in the Python programming language.

Using shell scripts during package build phases becomes problematic though because

> When Nix runs a builder[,][18] it initially completely clears the environment (except for the attributes declared in the derivation). For instance, the `PATH` variable is empty. This is done to prevent undeclared inputs from being used in the build process. If for example the PATH contained `/usr/bin`, then you might accidentally use `/usr/bin/gcc`.
[->][19]

The quote above is [from the Nix manual][19] but the builder, that is shown there as an example, uses [`$stdenv/setup`][15] - a shell script that sets up a pristine sandbox environment for the build process, unsetting most (all?) environment variables from the calling shell, and [only including a small number of utilities][20]. (This is done to make builds reproducible, as much as possible.)<sup>1</sup>

<sup>[`$stdenv/setup`][15] is usually called implicitly when using [`stdenv.mkDerivation`][21] with [the generic builder][22] (i.e., when the `builder` attribute is left undeclared) but [one can write their own builders and invoke it explicitly during the build process][23].</sup>

<sup>TIP: [This answer][24] shows one way to find where a certain Nix function is defined (although [it is not infallible][25]).</sup>

As a corollary, the programs pointed to by the shebang directives won't be at those locations (or unavailable to reach from the sandbox), but they are actually around (or will be) in the [Nix store][26] so the paths will need to be re-pointed to their location in there. 

> NOTE: The [generic builder][22] populates PATH from **inputs** of the derivation so one must make sure that these are included as a dependency.

## 3. How to use

### 3.1 Implicitly

As mentioned above,[`patchShebangs`][9] is automatically invoked by the [`patchShebangsAuto`][10] [setup hook][11] during the [fixup phase][4] whenever a package is built - unless one opts out of this by setting the [`dontPatchShebangs` variable][27] (or the `dontFixup` variable for that matter) (see [Variables controlling the fixup phase][28] in the [Nixpkgs manual][1]).

<sup>Reminder to self: [6.4 Bash Conditional Expressions][29].</sup>

**3.1.0 What scripts is `patchShebangs` used on when invoked automatically?**

Usually on scripts installed by packages (for example to `$out/bin`).

<sup>Or the ones provided default by the Nixpkgs standard library? I presume that these have to be generic enough to run on different platforms so that (1) the template is built, and (2) scripts shebangs are patched in the end. ([@jtojnar confirmed this conjecture][30], but this section needs references, hence the small case.)</sup>

**3.1.1 How to use the variables controlling a build phase?**

Pass it to `mkDerivation` like any other variable controlling the builder.

```nix
stdenv.mkDerivation {
  #...
  dontPatchShebangs = true;
  #...
}
```

### 3.2 Explicitly

<sup>**Historical note**: Originally, `patchShebangs` was not externally callable, but [it was later extracted][13] to make its functionality re-usable in other [build phases][2] as well.</sup>

Again, from the comments in [the implementation][6]:

```shell
# Run patch shebangs on a directory or file.
# Can take multiple paths as arguments.
# patchShebangs [--build | --host] PATH...

# Flags:
# --build : Lookup commands available at build-time
# --host  : Lookup commands available at runtime

# Example use cases,
# $ patchShebangs --host /nix/store/...-hello-1.0/bin
# $ patchShebangs --build configure
```

It needs to be run on scripts that are to be executed directly (shell scripts included) during build time. These may be
1. coming from the source of what is being packaged
2. written by one to be used as helpers during the build process<sup>2</sup>

Specific examples from around the web:

+ [In Nix, how can I build a package that has a Python post-install script?][31] (Unix & Linux Stackexchange)
+ [hard-coded bin path and NixOS][32] (Stackoverflow)
+ [\[QUESTION\] Alias and symlinks in NixOS derivations][33] (Reddit)
+ [This systemd-specific issue on IRC][34]  
  ... and [quoting @jtojnar][35]:
  > That is exactly the use case for the explicit `patchShebangs` call. Meson build system expects to run `src/shared/generate-syscall-list.py` so it calls it. But that fails because `/usr/bin/env` does not exist in the build sandbox. And it only gets confusing because kernel/libc/something else reports that the script does not exist, even though it was the interpreter from the shebang which does not exist.

---

## Footnotes

\[1]: TODO: Find out more about how the sandbox(es) are built exactly and what are barred and what are allowed. [Quoting @jtojnar][36] to bring one example:
  > `/usr/bin/env`, which is not available in sandbox either. (NixOS only has that in user space for convenience but that [does not carry over to Nix sandbox.][37].

\[2]: [@jtojnar's comment][38]: "_Right, you will not need to use it explicitly for scripts that are only executed at run time, since those will be handled by the implicit call._" 

---

<sup>All links in this thread have (hopefully) been saved to the [Internet Archive][39]. (The soundtrack of the thread is [this gem][40].)</sup>


  [1]: https://nixos.org/manual/nixpkgs
  [2]: https://nixos.org/manual/nixpkgs/stable/#sec-stdenv-phases
  [3]: https://nixos.org/manual/nixpkgs/stable/#chap-stdenv
  [4]: https://nixos.org/manual/nixpkgs/stable/#ssec-fixup-phase
  [5]: https://nixos.org/manual/nixpkgs/stable/#:~:text=It%20rewrites%20the%20interpreter%20paths%20of%20shell%20scripts
  [6]: https://github.com/NixOS/nixpkgs/blob/ca156a66b75999153d746f57a11a19222fd5cdfb/pkgs/build-support/setup-hooks/patch-shebangs.sh
  [7]: https://github.com/NixOS/nixpkgs
  [8]: https://nixos.org/manual/nixpkgs/unstable/#patch-shebangs.sh
  [9]: https://github.com/NixOS/nixpkgs/blob/ca156a66b75999153d746f57a11a19222fd5cdfb/pkgs/build-support/setup-hooks/patch-shebangs.sh#L24
  [10]: https://github.com/NixOS/nixpkgs/blob/ca156a66b75999153d746f57a11a19222fd5cdfb/pkgs/build-support/setup-hooks/patch-shebangs.sh#L107
  [11]: https://nixos.org/manual/nixpkgs/unstable/#ssec-setup-hooks
  [12]: https://github.com/NixOS/nixpkgs/blob/ca156a66b75999153d746f57a11a19222fd5cdfb/pkgs/build-support/setup-hooks/patch-shebangs.sh#L10
  [13]: https://github.com/NixOS/nixpkgs/commit/daa66b8b1cb2ea5359f9914418350f63f0a53d7e
  [14]: https://discourse.nixos.org/t/what-is-the-patchshebangs-command-in-nix-build-expressions/12656/5
  [15]: https://github.com/NixOS/nixpkgs/blob/6ba632c2a442082f353bf2d7028fda11a888d099/pkgs/stdenv/generic/setup.sh
  [16]: https://en.wikipedia.org/wiki/Shebang_(Unix)
  [17]: https://en.wikipedia.org/wiki/Shebang_(Unix)#Syntax
  [18]: https://www.youtube.com/watch?v=KwPxhWU1koE
  [19]: https://nixos.org/manual/nix/stable/#ex-hello-builder-co-1
  [20]: https://nixos.org/manual/nixpkgs/stable/#sec-tools-of-stdenv
  [21]: https://github.com/NixOS/nixpkgs/blob/ca156a66b75999153d746f57a11a19222fd5cdfb/pkgs/stdenv/generic/make-derivation.nix
  [22]: https://github.com/NixOS/nixpkgs/blob/6ba632c2a442082f353bf2d7028fda11a888d099/pkgs/stdenv/generic/builder.sh
  [23]: https://nixos.org/manual/nixpkgs/stable/#:~:text=you%20can%20still%20supply%20your%20own%20build%20script
  [24]: https://stackoverflow.com/a/56124590/1498178
  [25]: https://discourse.nixos.org/t/where-is-callpackage-defined-exactly-part-2/12524
  [26]: https://nixos.org/manual/nix/stable/#chap-introduction
  [27]: https://github.com/NixOS/nixpkgs/blob/ca156a66b75999153d746f57a11a19222fd5cdfb/pkgs/build-support/setup-hooks/patch-shebangs.sh#L108
  [28]: https://nixos.org/manual/nixpkgs/stable/#:~:text=Variables%20controlling%20the%20fixup%20phase
  [29]: https://www.gnu.org/software/bash/manual/html_node/Bash-Conditional-Expressions.html
  [30]: https://discourse.nixos.org/t/what-is-the-patchshebangs-command-in-nix-build-expressions/12656#quote-id-12656-1-8
  [31]: https://unix.stackexchange.com/questions/555802/in-nix-how-can-i-build-a-package-that-has-a-python-post-install-script
  [32]: https://stackoverflow.com/questions/35724883/hard-coded-bin-path-and-nixos
  [33]: https://www.reddit.com/r/NixOS/comments/enbbw2/question_alias_and_symlinks_in_nixos_derivations/
  [34]: https://logs.nix.samueldr.com/nixos-dev/2020-11-15
  [35]: https://discourse.nixos.org/t/what-is-the-patchshebangs-command-in-nix-build-expressions/12656#quote-id-12656-1-11
  [36]: https://discourse.nixos.org/t/what-is-the-patchshebangs-command-in-nix-build-expressions/12656#quote-id-12656-1-2
  [37]: https://github.com/NixOS/nix/issues/1205
  [38]: https://discourse.nixos.org/t/what-is-the-patchshebangs-command-in-nix-build-expressions/12656#quote-id-12656-1-4
  [39]: https://archive.org/
  [40]: https://www.youtube.com/watch?v=JlfSSNkMcxw
