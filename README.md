# Typhoon test suite

Goldeneye render regression tests for Typhoon.

## Setup and use

Initialize the shaderball asset submodule after cloning:

```sh
git submodule update --init --depth 1
```

Run everything with `pixi run test`. Run one suite or subtree with:

```sh
pixi run pytest materials
pixi run pytest usdlux
```

References are gitignored and published through `reference-releases.json`:

```sh
pixi run goldeneye:download-references
pixi run goldeneye:update-references
```

## Suites

`test-suite/*.usda` contains the original 41 gallery fixtures. Their collected
scenes and dependencies live under `test-suite/_assets/scenes`. They render at
128 pixels on the longest axis, preserving each fixture's aspect ratio and
existing sample count.

`materials/` is a root-level suite containing 67 renderer-focused fixtures imported from
`aousd-materials-test-suite`:

- `pbr/`, `open_pbr/`, and `standard_surface/` cover closure sampling and
  surface/volume transport.
- `geometric/` covers geometry inputs, primvars, and shading frames.
- `textures/` covers OIIO contracts including wrap modes, UDIMs, PNG alpha,
  blur, and triplanar projection.
- `misc/time.usda` covers frame/time plumbing.

Shaderball fixtures sublayer `materials/_assets/shaderball/scene.usda` and the
pinned `usd-wg/assets` submodule. Plane fixtures sublayer the wrappers under
`materials/_assets/plane/`. Imported textures are byte-identical AOUSD assets
under `materials/_assets/materials/`.

Material imports render at 64x64. Shaderball fixtures use 16 spp, plane
fixtures use 8 spp, and noisy shaderball transmission/volume/SSS fixtures use
128 spp. Displacement renders at 128x128 so adaptive subdivision remains visible.

`usdlux/` contains 328 active frames covering rect, sphere, disk, cylinder,
distant, and dome lights, shaping/IES controls, and primary-ray-visible light
geometry. It renders at 64x64 with 16 spp and `ty:maxBounces = 0`. See
`usdlux/README.md`.

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
