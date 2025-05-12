# 🤖 Setup Godot

[![Chickensoft Badge][chickensoft-badge]][chickensoft-website] [![Discord][discord-badge]][discord] [![Read the docs][read-the-docs-badge]][docs]

Setup Godot for use with (or without) .NET on macOS, Windows, and Linux CI/CD runners.

- ✅ Installs Godot 4.x
- ✅ Optionally installs export templates.
- ✅ C# supported using .NET version of Godot.
- ✅ Versions **without** .NET are also supported.
- ✅ Installs Godot directly on the CI/CD runner.
- ✅ Caches Godot and export template installation for speedier workflows.
- ✅ Adds environment variables (`GODOT4`, `GODOT`) to the system path.
- ✅ Runs on macOS Github Actions runner.
- ✅ Runs on Windows Github Actions runner.
- ✅ Runs on Ubuntu Github Actions runner.

> **Godot 3.x and below are not supported.**

## Usage

Example workflow:

```yaml
name: 🚥 Status Checks
on: push

jobs:
  tests:
    name: 👀 Evaluate on ${{ matrix.os }}
    runs-on: ${{ matrix.os }}
    strategy:
      # Don't cancel other OS runners if one fails.
      fail-fast: false
      matrix:
        # Put the operating systems you want to run on here.
        os: [ubuntu-latest, macos-latest, windows-latest]
    env:
      DOTNET_CLI_TELEMETRY_OPTOUT: true
      DOTNET_NOLOGO: true
      # Smaller cache size
      NUGET_PACKAGES: ${{ github.workspace }}/.nuget/packages
      DOTNET_SKIP_FIRST_TIME_EXPERIENCE: true
    defaults:
      run:
        # Use bash shells on all platforms.
        shell: bash
    steps:
      - uses: actions/checkout@v4
        name: 🧾 Checkout
        # Disable fetching history as most of the time it is useless
        filter: tree:0
        fetch-depth: 0

      - uses: actions/setup-dotnet@v4
        name: 💽 Setup .NET SDK
        with:
          # Use the .NET SDK from global.json in the root of the repository.
          global-json-file: global.json

      - name: 📦 Restore Dependencies
        run: dotnet restore

      - uses: chickensoft-games/setup-godot@v2
        name: 🤖 Setup Godot
        with:
          # Version must include major, minor, and patch, and be >= 4.0.0
          # Pre-release label is optional.
          version: 4.0.0-beta16 # also valid: 4.0.0.rc1 or 4.0.0 or "global.json", etc
          # Use .NET-enabled version of Godot (the default is also true).
          use-dotnet: true
          # Include the Godot Export Templates (the default is false).
          include-templates: true

      - name: 🔬 Verify Setup
        run: |
          dotnet --version
          godot --version

      - name: 🧑‍🔬 Generate .NET Bindings
        run: godot --headless --build-solutions --quit || exit 0

      - name: 🦺 Build Projects
        run: dotnet build --configuration Release

      # Do whatever you want!
```

## Inputs

See [action.yml][action] for the complete guide to all of the action's inputs.

### Specifying the Godot Version

The Godot version should be specified the same as any [GodotSharp] version string: e.g., `4.0.0-beta1`, `4.0.0-beta.16`, `4.0.0`, etc.

In place of a version, you can specify `global` or `global.json` to use the version of Godot specified by the project's global.json file.

```yaml
  - uses: chickensoft-games/setup-godot@v2
    name: 🤖 Setup Godot
    with:
      version: global.json # use Godot version specified by global.json
```

For that to work, your project must have a `global.json` file in the root directory with contents similar to the following.

```json
{
  "sdk": {
    "version": "6.0.406",
    "rollForward": "latestMinor"
  },
  "msbuild-sdks": {
    "Godot.NET.Sdk": "4.0.0"
  }
}
```

**Important:** If using a global.json file in your project, do *not* specify the version of the Godot.NET.Sdk in your project's `.csproj` file. Note that Godot tends to add this back to your `.csproj` file every time you save the Godot project, so discard those changes before committing to source control.

```xml
<Project Sdk="Godot.NET.Sdk"> <!-- GOOD -->
<Project Sdk="Godot.NET.Sdk/4.0.0"> <!-- BAD -->
```

[chickensoft-badge]: https://chickensoft.games/img/badges/chickensoft_badge.svg
[chickensoft-website]: https://chickensoft.games
[discord-badge]: https://chickensoft.games/img/badges/discord_badge.svg
[discord]: https://discord.gg/gSjaPgMmYW
[read-the-docs-badge]: https://chickensoft.games/img/badges/read_the_docs_badge.svg
[docs]: https://chickensoft.games/docs
[action]: ./action.yml
[GodotSharp]: https://www.nuget.org/packages/GodotSharp#versions-body-tab
