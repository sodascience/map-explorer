# Map Explorer

[![Project Status: Active – The project has reached a stable, usable state and is being actively developed.](https://www.repostatus.org/badges/latest/active.svg)](https://www.repostatus.org/#active)

Explore the app directly in your browser:  
👉 [Live demo on GitHub Pages](https://sodascience.github.io/map-explorer/)

## Overview

This repository contains a Vue.js web application that renders GeoJSON maps with dynamic region coloring. 

The application imports geographic boundary data in GeoJSON format and applies colors to regions based on external datasets (see `/public`). It can use any GeoJSON as the basis for the map and it can use a dataset to determine the coloring of the region.

Everything runs locally in the browser with [duckdb-wasm](https://github.com/duckdb/duckdb-wasm) as the underlying SQL online analytical processing (OLAP) database.

### Features

- Import custom GeoJSON files for map boundaries
- Upload datasets to determine region coloring
- Browser-based processing with no server required
- Real-time filtering and mapping of very large datasets

## Development

**Prerequisites**

- Node.js and npm installed

**Installation**

1. Clone or fork the repository:

```sh
git clone https://github.com/sodascience/map-explorer.git
```

2. Navigate to the project directory:

```sh
cd map-explorer
```

3. Install dependencies:

```sh
npm install
```

4. Start the development server:

```sh
npm run dev
```

## Use this application for yourself

If you want to use this application you can fork it and add your own files and configuration. 
Your map will be displayed on the Github page of your fork and you can share it with your own audience!

### Configuration Modes

map-explorer can be configure by changing or adding configuration files in `/src/config/map-configs`.

The application currently supports one distinct configuration mode, determined by the `kind` field:

#### 1. GeoJSON + External Data File (`geojson-datafile`)

Use this mode when you have a separate data file (e.g., CSV or parquet file) that needs to be joined with your GeoJSON geometries. The data file should contain a column that holds the region id's in the GeoJSON and a valueColumn that holds numberic values that are used to color the regions. This mode supports categorical columns that can be used as filters. 


**Fields:**

- **`kind`** (required): Must be set to `"geojson-datafile"`
- **`geojsonFileName`** (required, string): Filename of the GeoJSON file. Put the file in the `/public` folder
- **`dataFileName`** (required, string): Filename of the data file. Put the file in the `/public` folder
- **`idColumnGeojson`** (required, string): The property name in the GeoJSON features that serves as the unique identifier
- **`idColumnDataFile`** (required, string): The column name of the column in the data file that holds the same unique id's in the GeoJSON
- **`categoryColumns`** (required, array of strings): Column names that represent categorical/filter dimensions
- **`valueColumn`** (required, string): The column name containing the numeric values to visualize
- **`legendTitle`** (optional, string): Title to display in the map legend
- **`mapColorConfig`** (required, object): Configuration for color mapping (see [Map Color Configuration](#map-color-configuration))
- **`filter`** (optional, object): Initial filter state as key-value pairs where keys are category column names and values are the selected categories

**Example:**

```json
{
  "kind": "geojson-datafile",
  "mapDescription": {
    "title": "Historical disease mention rates",
    "description": "This map contains data gathered via Delpher and processed by the ODISSEI SoDa team. Each municipality gets a \"mention rate\" assigned, which is a proxy for the actual disease pressure in this region. See <a href=\"https://github.com/sodascience/disease_database\" target=\"_blank\" rel=\"noopener noreferrer\" class=\"underline\"> https://github.com/sodascience/disease_database</a>."
  },
  "categoryColumns": ["year", "month", "disease"],
  "valueColumn": "mention_rate",
  "idColumnGeojson": "cbscode",
  "idColumnDataFile": "cbscode",
  "dataFileName": "disease_database_v1.2.parquet",
  "geojsonFileName": "nl1869.geojson",
  "legendTitle": "Mention Rate",
  "mapColorConfig": {
    "minValue": 0,
    "maxValue": 0.4,
    "colorScheme": "viridis",
    "dynamic": false,
    "colorSchemeInverted": false
  },
  "filter": {
    "year": "1918",
    "month": "10",
    "disease": "influenza"
  }
}
```

#### Map description Configuration

The `mapDescription` object controls the map description that will be shown in the information box.

**Fields:**

- **`title`** (required, string): Your map title
- **`description`** (required, string): Your map description, supports html so you can use html formatted urls

#### Map Color Configuration

The `mapColorConfig` object controls how numeric values are mapped to colors on the map.

**Fields:**

- **`minValue`** (required, number): The minimum value for the color scale.
- **`maxValue`** (required, number): The maximum value for the color scale.
- **`numBins`** (optional, positive integer): Number of discrete color bins to use.
- **`colorScheme`** (optional, string): The color scheme to apply. If not specified, defaults to a standard scheme
- **`dynamic`** (optional, boolean): If `true`, the color scale adjusts dynamically based on the current filtered data. If `false` or not specified, uses the fixed `minValue` and `maxValue`
- **`colorSchemeInverted`** (optional, boolean): If `true`, inverts the color scheme (reverses the color order)

##### Available Color Schemes

The following color schemes are available:

- `viridis` - Perceptually uniform, colorblind-friendly (purple to yellow)
- `plasma` - Perceptually uniform (purple to pink to yellow)
- `inferno` - Perceptually uniform (black to purple to yellow)
- `magma` - Perceptually uniform (black to purple to white)
- `cividis` - Colorblind-optimized, blue to yellow
- `turbo` - Rainbow-like, high contrast
- `warm` - Warm colors (dark red to yellow)
- `cool` - Cool colors (cyan to magenta)
- `cubehelix` - Perceptually uniform spiral through color space
- `no colorscheme` - No color scheme applied

#### Multiple maps

You can add multiple maps by adding multiple configuration files in the `/src/config/map-configs` folder. 
If you have multiple configuration files, you will be able to select your different maps through a dialog.


#### Configuration errors

If you misspecified the configurations the app won't load an an error message will be shown in the browser console.

### Embedding 

If you want to embed map-explorer you can do so by including it into an iframe, like so:

```html
<iframe 
  src="https://sodascience.github.io/map-explorer/"
  width="100%" 
  height="800"
  frameborder="0"
  allowfullscreen>
</iframe>
```

## Contact

This is a project by the [ODISSEI Social Data Science (SoDa)](https://odissei-soda.nl) team.

Do you have questions, suggestions, or remarks on the technical implementation? Create an issue in the [issue tracker](https://github.com/sodascience/map-explorer/issues) or feel free to contact [Niek de Schipper](https://github.com/trbknl).

<img src="https://odissei-soda.nl/images/logos/soda_logo.svg" alt="SoDa logo" width="250px"/>
