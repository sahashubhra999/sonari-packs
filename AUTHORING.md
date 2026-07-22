# Sonari Packs

One repo serves every downloadable Sonari pack: `index.json` on main is the
catalog the app fetches at runtime; release assets hold the bytes. Publishing
here reaches every installed app without an app update.

## Layout
- `index.json` — the catalog. Schema: `{schemaVersion, packs: [...]}`.
  - Sound entry: `id, kind:"sound", name, tagline, description, content[],
    order, license, soundAsset{url, sha256, sizeMB}`, optional `voices[]`
    (`name, caption, symbolName, category, program, tags`). Entries whose id
    matches a pack the app already knows keep their built-in voice lists, so
    lean entries are fine for those.
  - Visual entry: same header plus `coverURL` and
    `visualAssets[{abi, url, sha256, sizeMB}]`. The app picks the asset whose
    `abi` matches its `StagePackABI`; other assets are ignored.
- Releases — one tag per content wave (e.g. `visual-packs-v1`). Sound assets
  may live in other repos (the original `sonari-sound-packs` URLs still work);
  visual `.sonaripack.zip` files and cover PNGs live here.

## Releasing visual packs
From the Sonari repo:

```
scripts/release-visual-packs.sh
```

That script builds the pack dylibs and metallibs, slices presets and
manifests out of `content/stage/StagePresets-full.json`, signs, zips,
uploads the assets, regenerates `index.json`, and pushes it here. A new
pack = a new entry in that script's tables plus a SwiftPM pack product.

## Releasing sound packs
Upload the `.sf2` as a release asset (any repo), then add its entry to
`index.json` with the file's SHA-256 (`shasum -a 256 file.sf2`) and voices.

## Rules
- Never edit an already-published asset; upload a new tag/version instead.
- `sha256` pins are law: the app refuses mismatched bytes.
- Visual pack `abi` must match the app generation it was built against;
  rebuild and add a new `visualAssets` entry when the app's ABI bumps.
