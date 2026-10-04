# Source and runtime data

Every asset exists in two forms, made for two different jobs.

| | Source data | Runtime data |
|---|---|---|
| Made for | **editing** | **loading** |
| Typical forms | PSD, PNG, EXR; FBX, glTF; WAV; the application's own documents; script source | GPU-compressed textures with mipmaps; packed vertex buffers; compressed or streamable audio; bytecode |
| Qualities | lossless, layered, large, in the tools people already use | compact, laid out for fast reading, specific to one platform |
| Who makes it | people, in their tools | the pipeline, from the source |
| Kept in | version control | caches and builds |

## Why keep both

- **The source is the master copy.** Runtime data is always rebuilt from it, never edited by hand.
- **Settings can change after the fact.** A new compression setting, a smaller texture size for phones, a new target
  platform: everything is regenerated from sources, without anyone re-exporting files from their tools.
- **Each platform gets the best format.** One source texture becomes BC-compressed for desktop GPUs and ASTC for
  phones; one recording becomes Vorbis for one platform and a hardware codec for another.
- **Lossless sources avoid generation loss.** Re-encoding an already compressed file loses quality every time;
  encoding from a lossless master once per target doesn't.

## The intermediate form

Between the two there is often a third: an **editor-ready** form, produced by import ([Importers](03-importers.md)).
It is what the editor shows and edits (a decoded image, a mesh with all its attributes), already validated, but not yet
compressed or stripped for a platform. Cooking ([Cooking](07-cooking.md)) produces the runtime form from it.

## The rule for version control

- **Commit:** source assets and their **sidecar files** (identity and import settings), project settings.
- **Never commit:** the asset database, the derived data cache, cooked builds, thumbnails. They are **generated**, and
  can always be rebuilt or downloaded from a shared cache ([Collaboration](10-collaboration.md)).

Committing generated data causes merge conflicts nobody can resolve, bloats the repository, and lets stale results
survive after their sources change.
