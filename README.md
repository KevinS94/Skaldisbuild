# SKALDIS BUILD — Supabase catalog integration v1

This package converts the current BUILD configurator from hardcoded assets to the external Supabase asset backend.

## Architecture

- `index.html` contains no GLB files and no hardcoded asset catalog.
- `build-catalog` returns only enabled public catalog metadata.
- `build-model-url` returns a short-lived signed URL only for enabled variants whose family and pack are also enabled.
- Database tables remain closed to anonymous browser access.
- `skaldis-models` remains a private Storage bucket.
- The Asset Manager remains the only read/write admin interface.

## 1. Deploy `build-catalog`

In Supabase Dashboard:

1. Edge Functions
2. Deploy a new function
3. Via Editor
4. Function name: `build-catalog`
5. Replace the example code with `supabase/functions/build-catalog/index.ts`
6. Disable the built-in JWT verification / make this endpoint public.
7. Deploy.

Test with a GET request. It should return JSON containing `packs`, `types`, and `assets`.

## 2. Deploy `build-model-url`

1. Edge Functions
2. Deploy a new function
3. Via Editor
4. Function name: `build-model-url`
5. Replace the example code with `supabase/functions/build-model-url/index.ts`
6. Disable the built-in JWT verification / make this endpoint public.
7. Deploy.

Test with a POST request:

```json
{
  "variantId": "DG_STR_WALL_0101_BASIC"
}
```

When that variant has an uploaded `model_path`, the response should contain a signed URL and `expiresIn: 180`.

## 3. Replace GitHub Pages `index.html`

Only after both functions work, replace the current BUILD `index.html` with the supplied `index.html` and commit it to GitHub Pages.

The BUILD frontend calls:

- `https://zrhryfiwxpgivojunsip.supabase.co/functions/v1/build-catalog`
- `https://zrhryfiwxpgivojunsip.supabase.co/functions/v1/build-model-url`

No publishable or secret key is embedded in BUILD.

## 4. What changed in BUILD

- Removed hardcoded `PACKS`, `PARTS`, and asset-specific `PART_VARIANTS` data.
- Packs and asset families are generated dynamically from Supabase.
- Only enabled packs/families/variants returned by the server are visible.
- Variant selection uses the database variant codes such as `BASIC`, `DOOR`, `WINDOW`, `RUINED`, `PILLAR`.
- New saved designs use stable family IDs and explicit variant codes.
- Model URLs are requested only when a placed model needs to be rendered.
- Signed model URLs are cached briefly in the browser; loaded Three.js models are cached by stable variant ID.
- Existing grid resizing, pan, mobile UI, BOM, library, save/load, undo/redo and 2D/3D selection remain in place.
- Catalog and Library pack sections are now dynamic rather than hardcoded to Dungeon/Castle.

## Important security limitation

Private Storage + signed URLs prevents the GitHub repository from becoming a downloadable model library and prevents permanent public Storage URLs. However, any 3D model rendered in a public browser must eventually be sent to that browser. A technically skilled user can potentially capture those bytes while the model is being rendered. Signed URLs are access control and friction, not DRM.
