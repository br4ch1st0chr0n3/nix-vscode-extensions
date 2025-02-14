# Nix expressions for VS Code Extensions

At the time of writing this, `nixpkgs` contains **271** `VS Code` extensions. This is a small fraction of the more than **40,000** extensions in the `VS Code Marketplace`! In addition, many of the extensions in `nixpkgs` are significantly out-of-date.

This flake provides Nix expressions for the majority of available extensions from [Open VSX](https://open-vsx.org/) and [VS Code Marketplace](https://marketplace.visualstudio.com/vscode). A `GitHub Action` updates the extensions daily.

That said, you can now use a different set of extensions for `VS Code`/`VSCodium` in each of your projects. Moreover, you can share your flakes and cache them so that other people don't need to install these extensions manually!

## Note

> [!NOTE]
> Check the NixOS wiki [page](https://wiki.nixos.org/wiki/Visual_Studio_Code) about VS Code.

- Check [nix4vscode](https://github.com/nix-community/nix4vscode) (and contribute!) if you need a more individual approach to extensions.
- Extension publishers and names are lowercased only in Nix.
  - They're not lowercased in `.json` cache files such as [data/cache/open-vsx-latest.json](./data/cache/open-vsx-latest.json).
- Access an extension in the format `<attrset>.<publisher>.<name>`, where `<attrset>` is `vscode-marketplace`, `open-vsx`, etc. (see [Explore](#explore)).
- If an extension publisher or name aren't valid Nix identifiers, quote them like `<attrset>."4"."2"`.
- We have a permission from MS to use a crawler on their API in this case (see the [discussion](https://github.com/NixOS/nixpkgs/issues/208456)). Please, don't abuse this flake!
- Some previously available extensions may be unavailable in newer versions of this flake.
  - An extension is missing if it doesn't appear during a particular workflow run in a `VS Code Marketplace` or an `Open VSX` response about the full set of available extensions.
  - We let missing extensions remain in cache files (see [data/cache](./data/cache)) at most `maxMissingTimes` (specified in [.github/config.yaml](.github/config.yaml)).
- We don't automatically handle extension packs. You should look up extensions in a pack and explicitly write all necessary extensions.

## Template

This repository has a flake [template](template/flake.nix).

This template provides a [VSCodium](https://github.com/VSCodium/vscodium) with a couple of extensions.

1. Create a flake from the template (see [nix flake new](https://nixos.org/manual/nix/stable/command-ref/new-cli/nix3-flake-new.html)).

    ```console
    nix flake new vscodium-project -t github:nix-community/nix-vscode-extensions
    cd vscodium-project
    git init && git add .
    ```

1. Run `VSCodium`.

    ```console
    nix run .# .
    ```

1. Alternatively, start a devShell and run `VSCodium`. A `shellHook` will print extensions available in the `VSCodium`.

    ```console
    nix develop
    codium .
    ```

In case of problems see [Troubleshooting](#troubleshooting).

## Example

[flake.nix](./flake.nix) provides a default package using [vscode-with-extensions](https://github.com/NixOS/nixpkgs/blob/81b9a5f9d1f7f87619df26a4eaf48bf6dec8c82c/pkgs/applications/editors/vscode/with-extensions.nix) from `nixpkgs`.

This package is `VSCodium` with a couple of extensions.

Run `VSCodium` and list installed extensions.

```console
nix run github:nix-community/nix-vscode-extensions# -- --list-extensions
```

## Usage

> [!NOTE]
> Check the NixOS wiki [page](https://wiki.nixos.org/wiki/Visual_Studio_Code) about VS Code.

### Extensions

We provide extensions attrsets that contain both universal and platform-specific extensions.
We use a reasonable mapping between the sites target platforms and Nix-supported platforms (see the [issue](https://github.com/nix-community/nix-vscode-extensions/issues/20) and `systemPlatform` in [flake.nix](./flake.nix)).

There are several attrsets:

- `vscode-marketplace` and `open-vsx` contain the latest versions of extensions, including pre-release ones. Such pre-release versions expire in some time. That's why, there are `-release` attrsets.
- `vscode-marketplace-release` and `open-vsx-release` contain the release versions of extensions (see [Release extensions](#release-extensions)).
- `forVSCodeVersion "4.228.1"` allows to leave only the extensions [compatible](https://code.visualstudio.com/api/working-with-extensions/publishing-extension#visual-studio-code-compatibility) with the `"4.228.1"` version of `VS Code`.
  - You may supply the actual version of your `VS Code` instead of `"4.228.1"`.

> [!NOTE]
> In `with A; with B;`, the attributes of `B` shadow the attributes of `A`.
> Keep in mind this property of `with` when writing `with vscode-marketplace; with vscode-marketplace-release;`.
> See [With-expressions](https://nix.dev/manual/nix/latest/language/syntax#with-expressions).

### With flakes

See [Template](#template).

Add the following to your `flake.nix` (see [Flakes](https://wiki.nixos.org/wiki/Flakes)).

```nix
inputs.nix-vscode-extensions.url = "github:nix-community/nix-vscode-extensions";
```

### Without flakes

> [!NOTE]
> The values of `url`, `ref`, `rev` and  in the `fetchGit` argument are for demonstration purposes.
> The value `cb0aee6840fb29b70439880656ca6a313a6af101` is the full SHA-256 hash of a commit in this repository.
> Replace it with the hash of the commit you need.

```nix
let
  system = builtins.currentSystem;
  extensions =
    (import (builtins.fetchGit {
      url = "https://github.com/nix-community/nix-vscode-extensions";
      ref = "refs/heads/master";
      rev = "cb0aee6840fb29b70439880656ca6a313a6af101";
    })).extensions.${system};
  extensionsList = with extensions.vscode-marketplace; [
      rust-lang.rust-analyzer
  ];
in ...
```

## History

You can search for an extension in the repository history:

- Get commits containing the extension: `git log -S '"copilot"' --oneline data/cache/vscode-marketplace-latest.json`
- Select a commit, e.g.: `0910d1e`
- Search in that commit: `git grep '"copilot"' 0910d1e -- data/cache/vscode-marketplace-latest.json`

## Explore

Explore extensions via `nix repl`.

> [!NOTE]
> Press the `Tab` button (denoted as `<TAB>` below) to see attrset attributes.

### Get your system

```console
nix-instantiate --eval --expr 'builtins.currentSystem'
```

Output on my machine:

```console
"x86_64-linux"
```

### Get the `extensions` attrset

#### Get `extensions` with flakes

```console
$ nix repl

nix-repl> :lf github:nix-community/nix-vscode-extensions/cb0aee6840fb29b70439880656ca6a313a6af101
Added 10 variables.

nix-repl> t = extensions.<TAB>
extensions.aarch64-darwin  extensions.aarch64-linux   extensions.x86_64-darwin   extensions.x86_64-linux

nix-repl> t = extensions.x86_64-linux

nix-repl> t.<TAB>
t.forVSCodeVersion            t.open-vsx-release            t.vscode-marketplace-release
t.open-vsx                    t.vscode-marketplace
```

#### Get `extensions` without flakes

> [!NOTE]
> The values of `url`, `ref`, `rev` and  in the `fetchGit` argument are for demonstration purposes.
> The value `cb0aee6840fb29b70439880656ca6a313a6af101` is the full SHA-256 hash of a commit in this repository.
> Replace it with the hash of the commit you need.

```console
$ nix repl

nix-repl> t1 = (import (builtins.fetchGit {
                url = "https://github.com/nix-community/nix-vscode-extensions";
                ref = "refs/heads/master";
                rev = "cb0aee6840fb29b70439880656ca6a313a6af101";
              }))

nix-repl> t = t1.extensions.<TAB>
t1.extensions.aarch64-darwin  t1.extensions.aarch64-linux   t1.extensions.x86_64-darwin   t1.extensions.x86_64-linux

nix-repl> t = t1.extensions.x86_64-linux

nix-repl> t.<TAB>
t.forVSCodeVersion            t.open-vsx-release            t.vscode-marketplace-release
t.open-vsx                    t.vscode-marketplace
```

### Pre-release versions

```console
nix-repl> t.vscode-marketplace.rust-lang.rust-analyzer
«derivation /nix/store/jyzab0pdcgj4q9l73zsnyvc1k7qpb381-vscode-extension-rust-lang-rust-analyzer-0.4.1582.drv»
```

### Release versions

```console
nix-repl> t.vscode-marketplace-release.rust-lang.rust-analyzer
«derivation /nix/store/qjlr7iqgqrf2hd2z21xz96nmblxy680m-vscode-extension-rust-lang-rust-analyzer-0.3.1583.drv»
```

### Pre-release versions compatible with a given version of VS Code

```console
nix-repl> (t.forVSCodeVersion "1.78.2").vscode-marketplace.rust-lang.rust-analyzer
«derivation /nix/store/jyzab0pdcgj4q9l73zsnyvc1k7qpb381-vscode-extension-rust-lang-rust-analyzer-0.4.1582.drv»
```

### Removed extensions

Some extensions are hard to handle correctly ([example](https://github.com/nix-community/nix-vscode-extensions/issues/69)), so they have been removed via [removed.nix](./removed.nix).

They may be available in `nixpkgs`, in `pkgs.vscode-extensions` ([link](https://search.nixos.org/packages?channel=unstable&from=0&size=50&sort=relevance&type=packages&query=vscode-extensions)).

### Overlay

See [Overlays](https://wiki.nixos.org/wiki/Overlays#Using_overlays).

#### Get an overlay with flakes

> [!NOTE]
> The value `cb0aee6840fb29b70439880656ca6a313a6af101` is the full SHA-256 hash of a commit in this repository.
> Replace it with the hash of the commit you need.

```console
nix-repl> :lf github:nix-community/nix-vscode-extensions/cb0aee6840fb29b70439880656ca6a313a6af101
Added 14 variables.

nix-repl> pkgs = (legacyPackages.x86_64-linux.extend overlays.default)

nix-repl> pkgs.vscode-marketplace-release.rust-lang.rust-analyzer
«derivation /nix/store/midv6wrnpxfm3in3miilyx914zzck4d7-vscode-extension-rust-lang-rust-analyzer-0.3.1575.drv»
```

#### Get an overlay without flakes

> [!NOTE]
> The value `cb0aee6840fb29b70439880656ca6a313a6af101` is the full SHA-256 hash of a commit in this repository.
> Replace it with the hash of the commit you need.

```console
nix-repl> t1 = (import (builtins.fetchGit {
                url = "https://github.com/nix-community/nix-vscode-extensions";
                ref = "refs/heads/master";
                rev = "cb0aee6840fb29b70439880656ca6a313a6af101";
              }))

nix-repl> pkgs = import <nixpkgs> { overlays = [ t1.overlays.default ]; system = builtins.currentSystem; }

nix-repl> pkgs.vscode-marketplace-release.rust-lang.rust-analyzer
«derivation /nix/store/a701wlb8ckidpikr57bff16mmvsf3jir-vscode-extension-rust-lang-rust-analyzer-0.3.1575.drv»
```

## Contribute

### Issues

Resolve [issues](https://github.com/nix-community/nix-vscode-extensions/issues).

### README

- Fix links.
- Write new sections.
- Update commit SHA used in examples if they're too old.
- Enhance text.

### Release extensions

The [config](.github/config.yaml) contains several extensions.
We cache the information about the latest **release** versions of these extensions (see [Extensions](#extensions)).

You can add new extensions to the config and make a Pull Request.
Use the original extension publisher and name, e.g. `GitHub` and `copilot`.

### Extra extensions

The [extra-extensions.toml](extra-extensions.toml) file contains a list of extensions to be fetched from sites other than `VS Code Marketplace` and `Open VSX`.
These extensions replace ones fetched from `VS Code Marketplace` and `Open VSX`.
Add necessary extensions there, preferrably, for all supported platforms (see [Extensions](#extensions)).
[nvfetcher](https://github.com/berberman/nvfetcher) will fetch the latest release versions of these extensions and write configs to [generated.nix](data/extra-extensions/generated.nix).

### Special extensions

Certain extensions require special treatment.

Provide functions to modify such extension in [overrides.nix](overrides.nix).

Optionally, create and link there issues explaining chosen override functions.

Overrides apply to a whole attrset of extensions, including [Extra extensions](#extra-extensions).

Each override is applied to an argument of the `buildVscodeMarketplaceExtension` function.

#### Build problems

- Extension with multiple extensions in a zipfile ([issue](https://github.com/nix-community/nix-vscode-extensions/issues/31))
- Platform-specific extensions ([comment](https://github.com/nix-community/nix-vscode-extensions/issues/20#issuecomment-1543679655))

### Main flake

1. (Optionally) Install [direnv](https://direnv.net/), e.g., via `nix profile install nixpkgs#direnv`.

1. Run a devshell. When prompted about `extra-trusted-substituters` answer `y`. This is to use binary caches.

    ```console
    nix develop nix-dev/
    ```

1. (Optionally) Start `VSCodium` with necessary extensions and tools.

    ```console
    nix run nix-dev/#writeSettings
    nix run nix-dev/#codium .
    ```

### Haskell script

1. See the [README](./haskell/README.md).

1. Set the environment.

    ```console
    set -a
    source .env
    ```

1. Run the script.

    ```console
    nix run haskell/#updateExtensions
    ```

### Pull requests

Pull requests are welcome!

## Troubleshooting

- If `Nix`-provided `VSCodium` doesn't pick up the extensions:
  - Close other instance of `Nix`-provided `VSCodium`.
  - Try to reboot your computer and start `VSCodium` again.
- See [troubleshooting](https://github.com/deemp/flakes/blob/main/README/Troubleshooting.md).
