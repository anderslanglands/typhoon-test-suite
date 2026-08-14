# Typhoon test suite

Goldeneye render regression tests for Typhoon.

## Setup

Initialize the pinned shaderball asset submodule after cloning, then download
the published reference images:

```sh
git submodule update --init --depth 1
pixi run download-references
```

Pixi installs the Goldeneye test harness and the packaged `openusd-typhoon`
build pinned in `pixi.lock` when a task is first run.

## Running tests

Run all 474 collected image comparisons with:

```sh
pixi run pytest
```

The `test` task accepts any suite, subtree, or individual USD fixture:

```sh
pixi run pytest materials/pbr
pixi run pytest usdlux
pixi run pytest materials/open_pbr/displacement.usda
```

Use Goldeneye's dry-run mode to inspect the resolved renderer command without
rendering:

```sh
pixi run pytest test-suite/openPbr_simple.usda --goldeneye-dry-run -s
```

### Testing a local Typhoon build

By default, tests use the packaged `openusd-typhoon` build pinned in
`pixi.lock`. To test an OpenUSD checkout built with `pixi run build`, create a
gitignored `goldeneye.local.toml` in this repository's root. Replace
`/path/to/OpenUSD` with the checkout path:

```toml
[renderers.typhoon]
command = [
    "pixi", "run",
    "--manifest-path", "/path/to/OpenUSD/pixi.toml",
    "--clean-env",
    "usdrender",
    "{usd_path}",
    "--outputRoot", "{suite_output_root}",
]
```

Goldeneye loads `goldeneye.local.toml` after the other project configuration
profiles, so this replaces the default `typhoon` renderer command without
changing tracked files.

### Viewing reports

Each run writes an HTML report under `_output/run-NNNN` and updates the runs
index at `_output/index.html`. Start the Goldeneye report server with:

```sh
pixi run view
```

Open <http://127.0.0.1:8000/> in a browser. Keep the command running while
viewing reports and press Ctrl-C to stop the server.

To make a smaller report containing only failures from the latest run, use
`pixi run extract-failures` and then refresh the runs index.

### Reference images

References are gitignored and published as immutable archives described by
`reference-releases.json`. Download the current references or publish local
reference updates with:

```sh
pixi run download-references
pixi run update-references
```

Publishing updates requires an authenticated GitHub CLI session and write
access to the repository that hosts the reference releases.

## Suites

`test-suite/` collects 79 general integration cases: 63 top-level renderer and
material fixtures plus 16 OpenPBR furnace and normal-map fixtures under
`test-suite/furnace/`. Collected scenes and dependencies live under
`test-suite/_assets/scenes`. The cases render at 256 pixels on the longest axis,
preserving each fixture's aspect ratio and existing sample count.

`materials/` contains 67 renderer-focused fixtures imported from
`aousd-materials-test-suite`:

- `pbr/` contains 16 MaterialX PBR closure, layer, subsurface, and volume cases.
- `open_pbr/` contains 18 OpenPBR feature, transmission, volume, and displacement
  cases.
- `standard_surface/` contains 8 Standard Surface feature and transmission
  cases.
- `geometric/` contains 18 geometry-input, primvar, and shading-frame cases.
- `textures/` contains 6 OIIO contract cases covering addressing, UDIMs, PNG
  alpha, blur, and triplanar projection.
- `misc/time.usda` covers frame and time plumbing.

Shaderball fixtures sublayer `materials/_assets/shaderball/scene.usda` and the
pinned `usd-wg/assets` submodule. Plane fixtures sublayer the wrappers under
`materials/_assets/plane/`. Imported textures are byte-identical AOUSD assets
under `materials/_assets/materials/`.

Material imports render at 128x128. Shaderball fixtures use 16 spp, plane
fixtures use 8 spp, and noisy shaderball transmission/volume/SSS fixtures use
128 spp. Displacement renders at 256x256 so adaptive subdivision remains visible.

`usdlux/` expands 8 source fixtures into 328 active frames covering rect,
sphere, disk, cylinder, distant, and dome lights, shaping/IES controls, and
primary-ray-visible light geometry. It renders at 128x128 with 16 spp and
`ty:maxBounces = 0`. See `usdlux/README.md`.

## Scope

The material import deliberately excludes broad MaterialX node evaluation.
Math, adjustment, channel, compositing, conditional, logical, noise, pattern,
and procedural node coverage belongs in MaterialXCpp unit tests.

Still requiring hand-authored renderer fixtures:

- light and shadow linking;
- PointInstancer/native instancing and instance primvars;
- dome-light camera visibility;
- depth, normal, primId, and adaptive-heatmap AOVs;
- wireframe reprs;
- depth of field and exposure;
- render-setting behavior and adaptive-subdivision modes.

Physical-light-unit fixtures are deferred until the physical-light schemas
merge into the main provider. Importing them earlier would silently test that
unknown schema attributes have no effect.
