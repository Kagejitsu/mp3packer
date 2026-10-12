# mp3packer — Linux / OCaml 5 fork

[mp3packer](mp3packer.html) by Reed "Omion" Wilson losslessly repacks MP3 files:
it moves the bytes of a CBR file into the smallest valid VBR layout (and, with
`-z`, re-chooses the Huffman tables for each frame) without touching the audio
data itself. This is a fork of [hmage/mp3packer](https://github.com/hmage/mp3packer),
which itself carried the original 2.04 source with first-pass Linux build fixes.

Branch `linux-ocaml5` (the default here) adds:

- **Builds with OCaml 5.x.** The original targets OCaml 4 and its mutable strings.
  All `String.create` / in-place string writes are ported to `Bytes`, the C stubs
  get a small compatibility block for the renamed runtime calls and dropped
  typedefs, and the makefile uses the split stdlib paths.
- **Fixes corrupt `-z` output on Linux.** The portable branch of the bit reader in
  `mp3framehuffman-c.c` never shifted the fetched word by the current bit offset,
  unlike the Windows branch. Every Linux build of the upstream source therefore
  decoded large quantised values wrongly and `-z` produced badly wrong audio.
- **Enables the SSE4.1 search on GCC/Clang x86.** It was guarded Windows-only; the
  code is plain `smmintrin.h`. Output is byte-identical to the base path and about
  twice as fast per worker. `--process base` still selects the scalar search.
- A clean clone builds in one `make` (`.depend` is now a real target).

## Building

Requires an OCaml 5 native compiler (`ocamlopt.opt`, `ocamldep`) with the bundled
`unix`, `str` and `threads` libraries, and a C compiler. On Arch: `pacman -S ocaml`.

```sh
make            # produces ./mp3packer
strip mp3packer
```

The binary depends only on libc and libm.

## Verifying a build

Repacked files must decode to exactly the same samples as the input. Compare raw
PCM rather than `ffmpeg -f md5`, because the md5 muxer also hashes embedded cover art:

```sh
ffmpeg -i in.mp3  -f s16le - | md5sum
./mp3packer -z -f in.mp3 out.mp3
ffmpeg -i out.mp3 -f s16le - | md5sum   # must match
```

## Licence

GPL v2 or later, as upstream. See `gpl.txt`.
