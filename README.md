# webpack-mpa-builder

An experimental CLI tool that builds a multi-page app using Webpack. Why? To unbloat your project and abstract away build scripts and dev dependencies. It comes with the following features:
1. [babel](https://www.npmjs.com/package/babel-core) + [babel-preset-env](https://www.npmjs.com/package/babel-preset-env)
2. [pug](https://www.npmjs.com/package/pug)
3. [node-sass](https://www.npmjs.com/package/node-sass) + [autoprefixer](https://www.npmjs.com/package/autoprefixer)
4. [eslint](https://www.npmjs.com/package/eslint)
5. [i18n](https://www.npmjs.com/package/i18n)
6. [webpack-dev-middleware](https://www.npmjs.com/package/webpack-dev-middleware) + [webpack-hot-middleware](https://www.npmjs.com/package/webpack-hot-middleware)

## Overview

1. [Usage](#usage)
2. [CLI](#cli)
3. [Configuration](#configuration)
4. [Using Assets](#using-assets)
    1. [Webpack Assets](#webpack-assets)
    2. [Static Assets](#static-assets)
5. [Internationalization Setup](#internationalization-setup)
6. [Example Project Structure](#example-project-structure)

## Usage

```sh
$ npm install webpack-mpa-builder
```

Add `webpack-mpa-builder` commands to your `package.json` scripts, for example:

```js
{
  ...
  "scripts": {
    "clean": "webpack-mpa-builder clean",
    "build": "webpack-mpa-builder build",
    "dev": "webpack-mpa-builder dev",
    "lint": "webpack-mpa-builder lint",
    "lint:fix": "webpack-mpa-builder lint -f"
  },
  ...
}
```

Then you'll have access to the following commands:

```sh
# Run your project on a local dev server with hot reloading enabled
$ npm run dev

# Build your project for production
$ npm run build

# Clean the built files
$ npm run clean

# Lint and/or fix the source directory
$ npm run lint
$ npm run lint:fix
```

To use the linter, you must provide your own `.eslintrc.*` file on your project root. Note that you also can use the alias `wmb` instead of `webpack-mpa-builder`.

## CLI

```sh
  Usage: webpack-mpa-builder [options] <command>
  Alias: wmb [options] <command>

  where <command> is one of:
    build:  builds the project in production
      dev:  runs the project on a local dev server with hot module reloading
    clean:  wipes the built files
     lint:  lints the input directory


  Options:

    -V, --version                output the version number
    -c, --config <config>        the config file relative to project root
    -i, --inputDir <inputDir>    the input directory relative to project root
    -o, --outputDir <outputDir>  the output directory relative to project root
    -a, --analyze                specifies whether the bundle analyzer should run on build
    -f, --fix                    specifies whether the linter should automatically fix issues
    -h, --help                   output usage information
```

## Configuration

You can configure the behavior of `webpack-mpa-builder` by providing a config file. By default, it looks for `<project_root>/config/build.conf.js` or `<project_root>/config/build.conf.json`. You can change this by specifying the `--configFile` flag. The config file you provide will be merged into the default one defined by `webpack-mpa-builder`, overwriting any overlapping configurations.

The default configuration is as follows:

```js
{
  // Default locale for i18n.
  defaultLocale: `en`,

  // Input paths, where all source files to be processed by Webpack are located.
  input: {
    // The base path of the input files RELATIVE to the project root (cwd). All
    // the subseqent paths are RELATIVE to this path.
    baseDir: `app`,

    // The path to assets (i.e. fonts, CSS, Javascripts, images, etc.), RELATIVE
    // to the input base path.
    assetsDir: `assets`,

    // The path to manifest files (i.e. favicons, iOS/Android app icons,
    // manifest.json, etc), RELATIVE to the input base path.
    manifestDir: `manifest`,

    // The path to the entry Javascript files for Webpack. This path is RELATIVE
    // to the input base path. These entry files correspond to every page of
    // your app. For every page template you create, you should have a matching
    // entry file with the same file name as the page template. Note that only
    // the files at the root of this path are taken into account, meaning that
    // directories are ignored.
    entriesDir: `assets`,

    // The path to all the page templates which will be piped into
    // `html-webpack-plugin` individually. This path is RELATIVE to the input
    // base path. If there exists a Javascript entry file having the same name
    // as the template, it will be injected into the generated HTML file. Note
    // that only the files at the root of this path are taken into account,
    // meaning that directories are ignored.
    viewsDir: `views`,

    // Name of the page to be generated as the root `index.html` file.
    viewIndexFile: `index`
  },

  // Output paths, where files generated by Webpack will be saved to.
  output: {
    // The base path of the output files RELATIVE to the project root (cwd). All
    // the subsequent paths are RELATIVE to this path.
    baseDir: `public`,

    // The path where processed assets should be saved. The files include CSS,
    // Javascripts, fonts, images, media, etc. Basically all processed files
    // except for HTML files, manifest files and files that were in the static
    // directory. This path is RELATIVE to the output base path.
    assetsDir: `assets`,

    // The path where unprocessed static files are copied to. These are the
    // files that were originally in the static directory (not part of the
    // inputs). This path is RELATIVE to the output base path.
    staticDir: ``
  },

  // Path to configuration files.
  config: {
    // The base path of the config files, RELATIVE to the project root (cwd).
    // All subsequent paths are RELATIVE to this path.
    baseDir: `config`,

    // The path where all the locale files are stored. This path is RELATIVE to
    // the config base path.
    // @see https://www.npmjs.com/package/i18n
    localesDir: `locales`,

    // The path to app specific config. This file is merely passed to Webpack's
    // `DefinePlugin` as `$config`, so it can be referenced in the source code.
    // This is like optional meta data.
    appConfigFile: `app.conf`
  },

  // Path to static files. These files are outside of the input path and will
  // not be processed by Webpack. Rather, they will be directly copied to the
  // output path.
  static: {
    // The base path of the static files, RELATIVE to the project root (cwd).
    baseDir: `static`,

    // Glob patterns to ignore when copying static files.
    ignore: [`.*`]
  },

  // Config options specific to the `build` task.
  build: {
    // Public path of all loaded assets.
    publicPath: process.env.PUBLIC_PATH || `/`,

    // Specifies whether the linter should run.
    linter: true,

    // Specifies whether assets should be gzipped. You might want to skip this
    // if your web host already does it.
    gzip: false,

    // The extensions of the files to gzip, if enabled.
    gzipExtensions: [`js`, `css`],

    // Specifies whether a bundle analyzer report should be generated at the end
    // of the build process.
    analyzer: false
  },

  // Config options specific to the `dev` task.
  dev: {
    // Public path of all loaded assets.
    publicPath: `/`,

    // Specifies whether the linter should run.
    linter: false,

    // Specifies the port of the dev server.
    port: 8080,

    // Specifies whether the browser should automatically open when the dev
    // server is live.
    autoOpenBrowser: true
  }
}
```

## Using Assets

There are two places where you can put your assets in your project:
1. `<project_root>/app/assets` (default path, can be changed in the config file)
2. `<project_root>/static` (default path, can be changed in the config file)

### Webpack Assets

If you put assets inside `<project_root>/app/assets`, these are **processed by Webpack**. Styles will be piped through `sass-loader` -> `postcss-loader` -> `css-loader`, Javascripts will be piped through `babel-loader`, and everything else (i.e. fonts, images, other media) will be piped through `url-loader`. These assets, when processed, will appear in `public/assets` (default, can be changed in the config file).

### Static Assets

These assets are inside `<project_root>/static`. They are directly copied to the output directory, specified by `output.staticDir` in the config file (defaults to the output directory root). Webpack does not process these assets. To access these in your source code, use the `/<output.staticDir>` base path, for example:

```html
<!-- This loads image.png from the static folder if `output.staticDir` is '' (default) -->
<img src='/image.png'/>
```

## Internationalization Setup

This builder uses the [`i18n`](https://www.npmjs.com/package/i18n) package to localize your app. Simply place the locale files in the locales directory (i.e. `config/locales`) and the builder will automatically crawl through all the files in that directory and infer the supported languages from the file names. The generated HTML pages will have one version for each language, inside a subdirectory by the locale name with the exception of the default locale. The builder passes the functions `i18n.__()` and `i18n.__n()` to templates.

Let's say you have two locale files, `en.json` and `jp.json`, with `en` being the default locale. Generated HTML files will look like this:

```sh
.
└── public/
    ├── jp/
    │   └── index.html          # Japanese version of the home page
    ├── index.html              # English version of the home page
    └── ...
```

## Example Project Structure

The folder structure of your project matters, though you do have the option to change the a lot of the paths (see [Configuration](#configuration)). Here is an overview of an example project structure assuming default config (`*` indicates that the path can be changed via config options):


```
.
├── app/ *
│   ├── assets/ *               # Module assets (processed by webpack)
│   │   ├── fonts/
│   │   ├── images/
│   │   ├── stylesheets/
│   │   ├── index.js *          # Entry file for home page
│   │   ├── 404.js              # Entry file for 404 page
│   │   ├── page2.js            # Entry file for every other page
│   │   └── ...
│   ├── manifest/ *             # Manifest files (i.e. favicon.png, manfiest.json, etc)
│   │   ├── app-icon.png
│   │   ├── favicon.png
│   │   ├── manifest.json
│   │   └── ...
│   ├── views/ *                # Base path for template files
│   │   ├── index.pug *         # Template file per page
│   │   └── ...
├── config/ *
│   ├── locales/ *              # Locale JSON files, every file in here is passed to `i18n`
│   │   ├── en.json
│   │   └── ...
│   ├── app.conf.js[on] *       # App meta data to be accessed by `$config` in source files
│   ├── build.conf.js[on] *     # Custom config for `webpack-mpa-builder`
├── public/ *                   # Output built files, gets generated and wiped by build pipeline
│   └── assets/ *               # Built assets
│   ├── page2/                  # Every other page
│   │   └── index.html
│   ├── app-icon.png            # Manifest files directly copied to output root
│   ├── favicon.png
│   ├── manifest.json
│   ├── index.html              # Home page from `home.pug`
│   ├── 404.html                # 404 page from `404.pug`
│   └── ...
└── static/ *                   # Pure static assets (directly copied)
```

## License

This software is released under the [MIT License](http://opensource.org/licenses/MIT).
