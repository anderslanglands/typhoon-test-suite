# USD Lux regression suite

This suite is copied from the active `usdlux/` fixtures in
`usdlux-test-suite`, except `_assets/settings.usda` renders at 64x64 and 16 spp.

The 328 active frames sweep light type, intensity, exposure, color temperature,
normalization, shape dimensions, transforms, shaping cone/focus controls, IES
profiles, and primary-ray visibility. `ty:maxBounces = 0` isolates direct-light
sampling from transport.

The `physlight/` half is not imported. Its `PhysicalLightIlluminantAPI` and
`PhotometricAreaLightAPI` schemas are not in the main OpenUSD checkout, so the
authored photometric attributes would otherwise be ignored while tests
misleadingly passed.

Run with:

```sh
pixi run pytest usdlux
```
