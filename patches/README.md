# XHYN V Mali-G720 DXVK compatibility patch

## Why this exists

The XHYN V PanVK driver is currently detected on the target Mali-G720 MC8, but the bundled DXVK 2.7.1 rejects the Vulkan adapter when `fillModeNonSolid` is not exposed:

```
Found device: Mali-G720 MC8
Skipping: Device does not support required feature 'fillModeNonSolid'
DXVK: No adapters found
Failed to initialize DXVK.
```

`fillModeNonSolid` controls non-solid polygon rasterization in Vulkan. It must **not** be faked in the PanVK driver merely to pass DXVK's adapter check.

## Patch

`dxvk-2.7.1-mali-g720-remove-fillmodes.patch` removes `fillModeNonSolid` from the DXVK D3D9 and D3D11 baseline requirement profiles.

This is a DXVK-side compatibility change, not a Vulkan-driver feature spoof.

## Important limitation

This repository does not contain the Rebase application's bundled DXVK source, so this patch cannot change `com.xhynph.v.rebase` by itself. The Rebase build must apply the patch and rebuild its DXVK DLLs.

Also, this patch only removes the adapter gate. It does not prove that every D3D9/D3D11 workload will work without non-solid polygon mode. Any later workload that genuinely uses wireframe/point polygon rasterization still needs a supported implementation or an appropriate fallback.

## Validation target

After rebuilding the bundled DXVK, the startup log should no longer contain:

```
Skipping: Device does not support required feature 'fillModeNonSolid'
DXVK: No adapters found
Failed to initialize DXVK.
```

Instead, DXVK should proceed to its next Vulkan feature/extension checks. Those next checks must be addressed from real runtime evidence; unsupported Vulkan features must not be advertised as implemented.

Source context: DXVK 2.7+ has a documented Vulkan feature baseline, while the XHYN V runtime evidence shows the Mali-G720 device itself is being enumerated successfully. See the project notes and upstream PanVK sources for the experimental support boundary.
