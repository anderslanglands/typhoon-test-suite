# Typhoon test suite

Goldeneye render tests for Typhoon. The test suite contains standalone scenes copied from OpenUSD Omniverse's MaterialXCpp test data. Scenes without source cameras use cameras authored by their collected wrapper layers.

Run with: pixi run test

The collected scene layers author RenderSettings at 512 pixels wide and use resolutions matched to each selected camera's aperture aspect ratio. Original scenes and their dependencies are stored under test-suite/_assets/.
