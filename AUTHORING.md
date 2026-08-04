# Sonari Packs

One repo serves every downloadable Sonari pack: `index.json` on main is the
catalog the app fetches at runtime; release assets hold the bytes. Publishing
here reaches every installed app without an app update.

## Layout
- `index.json` — the catalog. Schema: `{schemaVersion, packs: [...]}`.
  - Sound entry: `id, kind:"sound", name, tagline, description, content[],
    order, license, soundAsset{url, sha256, sizeMB}`, optional additive
    `soundAssets[{container, minAppVersion, url, sha256, sizeMB}]`, and
    optional `voices[]`
    (`name, caption, symbolName, category, program, tags`). Entries whose id
    matches a pack the app already knows keep their built-in voice lists, so
    lean entries are fine for those.
  - Visual entry: same header plus legacy `coverURL` and
    `visualAssets[{abi, url, sha256, sizeMB}]`. The app picks the asset whose
    `abi` matches its `StagePackABI`; other assets are ignored.
  - Either kind may carry `artwork{thumbnail,banner}`. A rendition describes
    optional `path, url, sha256, pixelWidth, pixelHeight, byteCount` fields.
    Standalone preview bytes and the file inside the package are identical.
- Releases — one tag per content wave (e.g. `visual-packs-v1`). Sound assets
  may live in other repos (the original `sonari-sound-packs` URLs still work);
  visual and sound `.sonaripack.zip` files plus artwork previews live here.

## Releasing visual packs
From the Sonari repo:

```
scripts/release-visual-packs.sh
```

That script builds the pack dylibs and metallibs, slices presets and
manifests out of `content/stage/StagePresets-full.json`, copies optimized
artwork into `Artwork/`, signs, zips, uploads byte-identical previews,
regenerates `index.json`, and pushes it here. Visual marketing metadata lives
in `content/packs/visual-packs.json`; a release must merge built facts into it,
never replace it with a thinner record.

## Releasing sound packs
The raw `.sf2` release asset is immutable compatibility data. Do not repoint
`soundAsset`: old apps save that URL directly as `<id>.sf2`. Current apps can
use `scripts/release-sound-packs.sh` to build a self-contained container and
prepend an additive `soundAssets` record while preserving the raw entry.

For the first catalog-wide artwork rollout, publish one class with
`--defer-index` and publish the companion class normally. The first command
lands immutable assets and durable URLs without treating the intentionally
incomplete cross-class index as releasable; the second emits the single
complete index. `scripts/publish-pack-index.sh <index.json>` can retry only the
catalog push after a network failure. It validates all 25 packages and both
release-ready artwork descriptors per package before touching `main`.

## Artwork

Keep lossless masters under `content/packs/artwork-sources/<id>/` in the
Sonari source repository. Optimized delivery files are
`content/packs/artwork/<id>/{thumbnail,banner}.jpg`: currently 800×800 and
1600×900 progressive JPEG. Artwork contains no product title, Sonari logo,
interface, or screenshot. Set `SONARI_REQUIRE_PACK_ARTWORK=1` in release CI;
local build-only runs warn and continue while another pack's art is in flight.
After optimizing a batch, run `scripts/update-pack-artwork-metadata.py` to
write its dimensions, byte counts, and hashes into the durable catalogs, then
regenerate the app fallback with `scripts/generate-pack-catalog-fallback.py`.

## Rules
- Never edit an already-published asset; upload a new tag/version instead.
- `sha256` pins are law: the app refuses mismatched bytes.
- `soundAsset` raw SF2 URLs and hashes are permanent compatibility fields.
- Visual pack `abi` must match the app generation it was built against;
  rebuild and add a new `visualAssets` entry when the app's ABI bumps.
