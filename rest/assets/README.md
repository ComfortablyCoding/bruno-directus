# assets

Retrieve and transform file assets served by Directus. For images, Directus can resize, crop, re-encode, and apply Sharp transformations on the fly, caching the generated variants.

## Operations

- Get an asset by ID (with optional image transformations)
- Download a folder's files as a ZIP archive
- Download a set of files (by ID) as a ZIP archive

## Transformations

Pass query parameters to transform images on the fly (`width`, `height`, `fit`, `quality`, `format`, `withoutEnlargement`, `transforms`), or use a preconfigured preset via `key`. Add `download` to force a file download.

## Docs

https://directus.com/docs/api/assets
