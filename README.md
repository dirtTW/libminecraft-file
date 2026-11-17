# libminecraft-file (dirtTW fork)

Header-only C++ library for reading and writing Minecraft region files, chunks, NBT, biomes,
blocks, and related data. This is a fork of
[kbinani/libminecraft-file](https://github.com/kbinani/libminecraft-file), used by
[dirtTW/je2be-core](https://github.com/dirtTW/je2be-core).

The upstream library is MIT licensed (see `LICENSE`). The fork keeps the same headers and
APIs; consumers only need to point their dependency at this repository.

## Why this fork exists

Upstream `master` is still at `6f00d2c` (checked 2026-11-17). It predates Java 26.1/26.3 and
Bedrock 26.x, and a 26.1+ world cannot be read correctly with it:

- 26.3 renamed the chunk palette fields to `id` and `properties`, made properties optional
  for the default state, and allowed string palette entries. The old reader returned
  `nullptr`, which drops the entire 16×16×16 section.
- Unknown biome names (for example `sulfur_caves` and `dappled_forest`) made the old reader
  drop the section as well.
- The block tables stopped at 1.21.10, so the 120 blocks added in 26.1–26.3 resolved to
  `unknown`.

This fork carries those fixes and the missing block data until they land upstream.

## What this fork adds

### Java 26.1/26.3 reader

- Accepts both the old `Name`/`Properties` palette shape and the 26.3 `id`/`properties`
  shape, including optional `properties` and string palette entries.
- Keeps a chunk section when a biome name is unknown instead of returning `nullptr`.
- Reads 26.3 worlds with the existing `Chunk`, `Region`, `Block`, and `Biome` APIs.

### Biomes

- Adds `sulfur_caves` and `dappled_forest` to the biome enum and name tables.
- Maps them to Bedrock biome IDs `194` and `195`, matching Bedrock 26.30+ and 26.50+.

### Java 26.3 blocks

Adds all 120 blocks introduced in 26.1–26.3 to the id, name, from-name, and heightmap
tables:

- the poplar wood set: planks, log, wood, stripped variants, colored leaves, sapling,
  slab, stairs, fence, fence gate, door, trapdoor, button, pressure plate, signs, shelf;
- the cinnabar and sulfur sets: base block, polished/chiseled variants, bricks, slabs,
  stairs, walls;
- colored concrete and wool slabs and stairs;
- golden dandelion, red shrub, shelf mushroom, straw bed, and sulfur spikes.

Opacity, leaf handling, plant-only `WORLD_SURFACE` entries, and the wall/fence/sign/shelf
cases follow the classifications used by their 1.21.10 counterparts. The new cases return
`true` from the four heightmap classifiers instead of falling through to `false`.

### Legacy flattening

Includes upstream PR
[kbinani/libminecraft-file#3](https://github.com/kbinani/libminecraft-file/pull/3) by
[@xiej2520](https://github.com/xiej2520):

- fixes the legacy stair `bottm` and trapdoor `fasle` values;
- flattens wooden stone slabs to `petrified_oak_slab` and seamless double slabs to smooth
  blocks;
- flattens logs with `axis:none` to bark/wood blocks;
- flattens legacy shrubs to dead bushes;
- adds missing item renames and fixes `cooked_cod`.

### Java 26.3 output

`je/block.hpp` writes `id`/`properties` when the target data version is 26.3 or newer, so
the writer and reader use the same palette shape.

## Status

- Base: `6f00d2c` (`kbinani/libminecraft-file` master at fork time).
- Fork branch: `master`. The code commits add the 26.x reader, the 26.3 blocks, and the
  flattening fix; the final commit is the README you are reading.
- Integration test: `je2be-core`'s native doctest suite. It covers 26.3 blocks, chunk
  section parsing, biome fallback, round trips, and the existing converters.
- Upstream status: no tags, so pin a commit hash. The 26.x and block-table changes are not
  upstream yet; PR #3 is open upstream and is included here.

## Using it

With CPM:

```cmake
CPMAddPackage(
  NAME libminecraft-file
  GITHUB_REPOSITORY dirtTW/libminecraft-file
  GIT_TAG <commit>
)
```

Pin a commit from this fork rather than `master` so builds are reproducible.
`je2be-core` does exactly that in its `CMakeLists.txt`.

## Limitations / TODO

- Bedrock output version constants are a `je2be-core` concern; this fork does not raise
  Bedrock's storage/network version.
- Item-only 26.3 additions (boats, etc.) still convert by name. Block items are covered by
  the block tables.
- Validate the 26.3 reader and generator against a dedicated 26.3 client/server, not only
  against this library's reader.
- Rebase or upstream these changes when `kbinani/libminecraft-file` starts accepting 26.x
  changes.
- Add fixtures for 26.1/26.3 chunks and the new block heightmaps.

## Credits

- [kbinani](https://github.com/kbinani) for the original library and the 26.x groundwork.
- [@xiej2520](https://github.com/xiej2520) for the legacy flattening fixes in PR #3.
- [Chunker](https://github.com/HiveGamesOSS/Chunker) by Hive Games (MIT) as the reference
  for the 26.3 block names and states.
- [Minecraft Wiki](https://minecraft.wiki/) for block/biome/protocol details.

Minecraft is a trademark of Mojang Studios and Microsoft. This is an unofficial community
fork.
