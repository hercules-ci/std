# Hello Moon

_A slightly more complete [hello world][hello-world] tutorial._

This tutorial implements a very typical `automation` Cell and its Organelles for a somewhat bigger project.
It also makes use of more advanced functions of `std`.
Namely:

- `std.growOn` instead of `std.grow`
- `std.harvest` to provide compatibility layers of "soil"
- non-default Organelle definitions
- the input debug facility

The terms _"Clade"_, _"Cell"_, _"Organelle"_, _"Target"_ and _"Action"_ have special meaning within the context of `std`.
With these clear definitions, we navigate and communicate the code structure much more easily.
In order to familiarize yourself with them, please have a quick glance at the [glossary][glossary].

## File Layout

Let's start again with a flake:

#### `./flake.nix`

```nix
{{#include ./flake.nix}}
```

This time we specified `cellsFrom = ./nix;`.
This is gentle so that our colleagues know immediately which files to either look or never look at depending on where they stand.

We also used `std.growOn` instead of `std.grow` so that we can add compatibility layers of "soil".

Furthermore, we only defined two Organelles: `nixago` & `devshells`. More on them follows...

#### `./nix/automation/*`

Next, we define a `automation` cell.
Each project will have some amount of automation.
This can be repository automation, such as code generation.
Or it can be a CI/CD specification.
In here, we wire up two tools from the Nix ecosystem: [`numtide/devshell`][devshell] & [`nix-community/nixago`][nixago].

> _Please refer to these links to get yourself a quick overview before continuing this tutorial, in case you don't know them, yet._
>
> A _very_ short refresher:
>
> - **Nixago**: Template & render repository (dot-)files with nix. [Why nix?][why-nix]
> - **Devshell**: Friendly & reproducible development shells &mdash; the original ™.

> **Some semantic background:**
>
> Both, Nixago & Devshell are _Component Tools_.
>
> _(Vertical) Component Tools_ are distinct from _(Horizontal) Integration Tools_ &mdash; such as `std` &mdash; in that they provide a specific capability in a minimal linux style: _"Do one thing and do it well."_
>
> Integration Tools however combine them into a polished user story and experience.
>
> The Nix ecosystem is very rich in _component tools_, however only few _integration tools_ exist at the time of writing.

#### `./nix/automation/devshells.nix`

Let's start with the `cell.devshells` Organelle and work our way backwards to the `cell.nixago` Organelle below.

> **More semantic background:**
>
> I could also reference them as `inputs.cells.automation.devshells` & `inputs.cells.automation.nixago`.
>
> But, because we are sticking with the local Cell context, we don't want to confuse the future code reader.
> Instead, we gently hint at the locality by just referring them via the `cell` context.

```nix
{{#include ./nix/automation/devshells.nix}}
```

The `nixago = [];` option in this definition is a special integration provided by the [Standard's `devshell`-wrapper (`std.lib.mkShell`)][devshell-wrapper].

_This is how `std` delivers on its promise of being a (horizontal) integration tool that wraps (vertical) component tools into a polished user story and experience._

Because we made use of `std.harvest` in the flake, you now can actually test out the devshell via the Nix CLI compat layer by just running `nix develop -c "$SHELL"` in the directory of the flake.
For a more elegant method of entering a development shell read on the [direnv][direnv-sec] section below.

#### `./nix/automation/nixago.nix`

As we have seen above, the `nixago` option in the `cell.devshells` Organelle references Targets from both `std.nixago` _and_ `cell.nixago`.
While you can explore `std.nixago` [here][std-nixago], let's now have a closer look at `cell.nixago`:

```nix
{{#include ./nix/automation/nixago.nix}}
```

In this Organelle, we have been modifying some built-in convenience `std.nixago.*` pebbles.
The way `configData` is merged upon the existing pebble is via a simple left-hand-side/right-hand-site `data-merge` (`std.dmerge`).

> **Background on array merge strategies:**
>
> If you know how a plain data-merge (does not magically) deal with array merge semantics, you noticed:
> We didn't have to annotate our right-hand-side arrays in this example because we where not actually amending or modifying any left-hand-site array type data structure.
>
> Would we have done so, we would have had to annotate:
>
> - either with `std.dmerge.append [/* ... */]`;
> - or with `std.dmerge.update [ idx ] [/* ... */]`.
>
> But lucky us (this time)!

## Command Line Synthesis

With this configuration in place, you have a couple of options on the command line.
Note, that you can accessor any `std` cli invocation also via the `std` TUI by just typing `std`.
Just in case you forgot exactly how to accessor one of these repository capabilities.

> **Debug Facility:**
>
> Since the debug facility is enabled, you will see some trace output while running these commands.
> To switch this off, just comment the `debug = [ /* ... */ ];` attribute in the flake.
>
> It looks something like this:
>
> ```nix
> trace: inputs on x86_64-linux
> trace: {
>   cells = {…};
>   nixpkgs = {…};
>   self = {…};
>   std = {…};
> }
> ```

**Invoke devshell via `nix`**

```bash
nix develop -c "$SHELL"
```

By quirks of the Nix CLI, if you don't specify `-c "$SHELL"`, you'll be thrown into an unfamiliar bare `bash` interactive shell.
That's not what you want.

**Invoke the devshell via `std`**

In this case, invoking `$SHELL` correctly is taken care for you by the Clade's `enter` Action.

```bash
# fetch `std`
$ nix shell github:divnix/std
$ std //automation/devshells/default:enter
```

Since we have declared the devshell Organelle as a `clades.devshells`, `std` auments it's Targets with the Clade Actions.

See [`clades.devshells`][clades-devshells] for more details on the available Actions and their implementation.

Thanks to the `cell.devshells`' `nixago` option, entering the devshell will also automatically reconcile the repository files under Nixago's management.

**Explore a Nixago Pebble via `std`**

You can also explore the nixago configuration via the Nixago Clade's `explore`-Action.

```bash
# fetch `std`
$ nix shell github:divnix/std
$ std //automation/nixago/treefmt:explore
```

See [`clades.nixago`][clades-nixago] for more details on the available Actions and their implementation.

## direnv

Manually entering the devshell is boring.
How about a daemon always does that automatically & efficiently when you `cd` into a project directory?
Enter [`direnv`][direnv] &mdash; the original (again; and even from the same author) 😊.

Before you continue, first install direnv according to it's [install instructions][direnv-install].
It's super simple & super useful ™ and you should do it _right now_ if you haven't yet.

Please learn how to enable `direnv` in this project by following the [direnv how-to][direnv-how-to].

In this case, you would adapt the relevant line to: **`use std nix //automation/devshells:default`**.

Now, you can simply `cd` into that directory, and the devshells is being loaded.
The MOTD will be shown, too.

The first time, you need to teach the `direnv` daemon to trust the `.envrc` file via `direnv allow`.
If you want to reload the devshell (e.g. to reconcile Nixago Pepples), you can just run `direnv reload`.

Because I use these commands so often, I've set: `alias d="direnv"` in my shell's RC file.

---

[why-nix]: ../../explain/why-nix.md
[direnv]: https://direnv.net
[direnv-sec]: #direnv
[direnv-install]: https://direnv.net/docs/installation.html
[direnv-how-to]: ../../guides/envrc.md
[clades-devshells]: ../../reference/clades/devshells-clade.md
[clades-nixago]: ../../reference/clades/nixago-clade.md
[std-nixago]: https://github.com/divnix/std/blob/main/cells/std/nixago.nix
[direnv]: #direnv
[hello-world]: ../hello-world
[devshell-wrapper]: ../../reference/std/lib/mkShell.md
[devshell]: https://github.com/numtide/devshell
[nixago]: https://github.com/nix-community/nixago
[glossary]: ../../glossary.md