# Orthophoto Extension Specification

- **Title:** Orthophoto
- **Identifier:** <https://stac-extensions.github.io/ortho/v1.0.0/schema.json>
- **Field Name Prefix:** ortho
- **Scope:** Collection, Item
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @m-mohr

This document explains the Orthophoto Extension to the [SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.

Orthophotos (orthoimages) are aerial or satellite images that were rectified to a map projection,
so that they have a uniform scale and can be used like a map.
Two properties matter to users who search for orthophotos, and no other extension covers them:

- The elevation model used for the rectification:
  A *true orthophoto* is rectified on a surface model that includes buildings and vegetation,
  so roofs lie exactly on their footprints and buildings don't lean over streets.
  A *classic orthophoto* is rectified on a terrain model (bare earth), so elevated objects lean away from the image centre.
- The leaf state of the vegetation at the time of the acquisition:
  Leaf-off images show the ground below deciduous trees, leaf-on images show the vegetation.

The fields correspond to the per-tile metadata of national orthophoto programmes,
e.g. `Bezugsflaeche` and `Belaubungszustand` in the German AdV product standard for digital orthophotos (DOP).

- Examples:
  - [Item example](examples/item.json): A true orthophoto tile, acquired leaf-off
  - [Collection example](examples/collection.json): A collection of orthophotos with summaries
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Fields

The fields in the table below can be used in these parts of STAC documents:

- [ ] Catalogs
- [x] Collections
- [x] Item Properties (incl. Summaries in Collections)
- [x] Assets (for both Collections and Items, incl. Item Asset Definitions in Collections)
- [ ] Links

| Field Name            | Type       | Description |
| --------------------- | ---------- | ----------- |
| ortho:elevation_model | \[string] | The types of elevation model that the image was rectified on: `surface` (true orthophoto) and/or `terrain` (classic orthophoto). |
| ortho:leaf_state      | \[string] | The leaf states of the vegetation at the time of the acquisition: `leaf-off`, `partial` and/or `leaf-on`, or `unknown`. |

Both fields are arrays with at least one value and no duplicates.
They usually have one value.
They have several values if the image combines parts of different kinds,
e.g. a mosaic of tiles of which some are true and some are classic orthophotos.
Arrays are used even for a single value, so that clients and queries can rely on one data type,
e.g. `a_contains(ortho:elevation_model, 'surface')` in [CQL2](https://docs.ogc.org/is/21-065r2/21-065r2.html).

The fields are usually provided in the Item Properties.
They can be provided in Assets if an Item has, for example, a true and a classic orthophoto as separate assets.

### Additional Field Information

#### ortho:elevation_model

| Value     | Description |
| --------- | ----------- |
| `surface` | Rectified on a digital surface model (DSM) that includes buildings and vegetation, e.g. an image-based surface model. Also known as *true orthophoto*. |
| `terrain` | Rectified on a digital terrain model (DTM) of the bare earth. Also known as *classic orthophoto*. |

If the elevation model is not known, the field is omitted. It is never an empty array.
The elevation model itself can be linked with the relation types `surface-model` and `elevation-model`
of the [Accuracy extension](https://github.com/stac-extensions/accuracy).

#### ortho:leaf_state

| Value      | Description |
| ---------- | ----------- |
| `leaf-off` | Deciduous vegetation without leaves, usually acquired in early spring. |
| `partial`  | Deciduous vegetation partially in leaf. |
| `leaf-on`  | Deciduous vegetation in full leaf, usually acquired in summer. |
| `unknown`  | The leaf state was not recorded. |

The value `unknown` makes the absence of the information explicit,
so that clients can tell it apart from data that doesn't use this field.
It can't be combined with other values: `["unknown"]` is valid, `["leaf-off", "unknown"]` is not.

## Best Practices

Other information that is commonly recorded for orthophotos is provided in existing fields:

| Information                 | Field |
| --------------------------- | ----- |
| Acquisition (flight) date   | `datetime`, `start_datetime` and `end_datetime` ([Common Metadata](https://github.com/radiantearth/stac-spec/blob/master/commons/common-metadata.md#date-and-time)) |
| Flight campaign or project  | `mission` ([Common Metadata](https://github.com/radiantearth/stac-spec/blob/master/commons/common-metadata.md#instrument)), e.g. the flight number of the campaign |
| Camera                      | `instruments` ([Common Metadata](https://github.com/radiantearth/stac-spec/blob/master/commons/common-metadata.md#instrument)) |
| Ground sample distance      | `gsd` ([Common Metadata](https://github.com/radiantearth/stac-spec/blob/master/commons/common-metadata.md#instrument)) |
| Positional accuracy         | `accuracy:geometric_x_stddev` and `accuracy:geometric_y_stddev` ([Accuracy extension](https://github.com/stac-extensions/accuracy)) |
| Spectral bands              | `bands` with `eo:common_name`, e.g. `red`, `green`, `blue` and `nir` ([EO extension](https://github.com/stac-extensions/eo)) |

## Contributing

All contributions are subject to the
[STAC Specification Code of Conduct](https://github.com/radiantearth/stac-spec/blob/master/CODE_OF_CONDUCT.md).
For contributions, please follow the
[STAC specification contributing guide](https://github.com/radiantearth/stac-spec/blob/master/CONTRIBUTING.md) Instructions
for running tests are copied here for convenience.

### Running tests

The same checks that run as checks on PRs are part of the repository and can be run locally to verify that changes are valid.
To run tests locally, you'll need `npm`, which is a standard part of any [node.js installation](https://nodejs.org/en/download/).

First you'll need to install everything with npm once. Just navigate to the root of this repository and on
your command line run:

```bash
npm install
```

Then to check markdown formatting and test the examples against the JSON schema, you can run:

```bash
npm test
```

This will spit out the same texts that you see online, and you can then go and fix your markdown or examples.

If the tests reveal formatting problems with the examples, you can fix them with:

```bash
npm run format-examples
```
