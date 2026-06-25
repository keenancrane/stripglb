# stripglb

`stripglb` is a command-line tool that strips data out of a glTF/GLB file,
keeping only the parts you ask for. By default it throws away everything except
the **mesh geometry** — vertex positions and the index/connectivity that defines
the triangulation — and leaves the rest of the file untouched.

It's a single, dependency-free Python script: just standard library, no install
step, no `node_modules`.

## Quick start

Strip a mesh down to bare geometry, in place:

```
stripglb mesh.glb
```

Write the result to a new file instead of overwriting the input:

```
stripglb mesh.glb out.glb
```

Keep a few things in addition to the geometry — say, normals and texture
coordinates:

```
stripglb --normals --uv mesh.glb out.glb
```

Strip every `.glb`/`.gltf` in a folder into another folder (same filenames):

```
stripglb --batch models/ stripped/
```

That's the whole tool. The rest of this document is reference material.

## What gets removed, what gets kept

Running `stripglb` with no flags removes all non-geometry data: vertex normals,
texture coordinates (UVs), vertex colors, joints/weights, textures and their
images, and cameras. Only vertex positions and the index buffer survive.

Everything *not* in the removable set is **preserved as-is** — the scene graph,
node transforms, mesh structure, materials (minus the textures you stripped),
skins, and animations all come through unchanged.

The flags below let you *keep* specific data in addition to the geometry, which
is always preserved.

## Keeping attributes and textures

| Flag | Keeps |
| --- | --- |
| `--normals` | Vertex normals |
| `--tangents` | Vertex tangents |
| `--uv` | All texture-coordinate sets |
| `--uvN` | The Nth texture-coordinate set (e.g. `--uv0`) |
| `--color` | Vertex colors |
| `--texture` | **All** textures, every channel (albedo, metalness/roughness, etc.) |
| `--albedo` | Albedo (base color) textures |
| `--mr` | Metalness–roughness textures |
| `--occlusion` | Occlusion textures |
| `--emission` | Emissive textures |
| `--normalmaps` | Normal-map (tangent-space normal) textures |
| `--jointsN` | The Nth joint/bone-index set |
| `--weightsN` | The Nth influence-weight set |

Notes:

* **Numbered flags.** Flags ending in an uppercase `N` take a nonnegative
  integer, e.g. `--uv0`. Repeat them to keep several sets: `--uv0 --uv1` keeps
  UV sets 0 and 1. Bare `--joints` / `--weights` keep *all* of those sets.
* **Combine freely.** `--normals --albedo --uv0` keeps normals, base-color
  textures, and the first UV set.
* **Textures imply their UVs.** If you keep a texture, the texture-coordinate
  set it samples is kept automatically, so the material stays valid even if you
  didn't pass the matching `--uv` flag.
* **`--normal`/`--normals`, `--tangent`/`--tangents`, `--color`/`--colors`,
  `--texture`/`--textures`** are accepted as either singular or plural.

## Batch mode

`--batch` applies the same flags to many files at once. There are two forms:

Strip a list of files **in place**:

```
stripglb --batch a.glb b.glb c.glb
stripglb --batch --normals --uv *.glb
```

Mirror an **input directory into an output directory**, reusing each filename:

```
stripglb --batch input_dir/ output_dir/
```

The output directory is created if it doesn't exist, and only `.glb`/`.gltf`
files in the input directory are processed. If one file fails, the others still
run and the tool exits with a non-zero status.

## Behavior and guarantees

* **Geometry is never altered.** Vertex positions, the index buffer, and any
  attributes you choose to keep are copied byte-for-byte. Connectivity,
  triangulation, and vertex order are preserved exactly — `stripglb` only
  removes data, it never re-meshes or reorders anything.
* **GLB and glTF.** Both `.glb` (binary) and `.gltf` (JSON, including embedded
  data-URI buffers) inputs are supported. Output format follows the output
  filename's extension; `.gltf` output is written as self-contained JSON with
  the buffer embedded as a data URI.
* **Valid output.** Orphaned accessors, buffer views, textures, images, and
  samplers are dropped and everything is re-indexed; the binary buffer is
  repacked so no stale bytes are left behind. Extension declarations
  (`extensionsUsed` / `extensionsRequired`) that are no longer referenced are
  removed too.

## Usage

```
stripglb [FLAGS] input.glb [output.glb]
stripglb --batch [FLAGS] file1.glb file2.glb ...   # strip each in place
stripglb --batch [FLAGS] input_dir/ output_dir/    # mirror names into a dir
```

With a single path, the file is edited in place; with two paths, the second is
the output. Run `stripglb --help` for a summary.

## Requirements

Python 3 (standard library only — no third-party packages). Run it directly:

```
python3 stripglb mesh.glb
```

or mark it executable and put it on your `PATH`:

```
chmod +x stripglb
```
