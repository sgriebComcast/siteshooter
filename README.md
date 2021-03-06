# Siteshooter 
<img src="https://cdn.rawgit.com/devopsgroup-io/siteshooter/master/siteshooter.svg" alt="Siteshooter" width="100" >

[![NPM version](https://img.shields.io/npm/v/siteshooter.svg)](https://www.npmjs.com/package/siteshooter) [![Build Status](https://img.shields.io/travis/devopsgroup-io/siteshooter.svg?branch=master)](https://travis-ci.org/devopsgroup-io/siteshooter)
[![dependencies](https://david-dm.org/devopsgroup-io/siteshooter.svg)](https://david-dm.org/devopsgroup-io/siteshooter#info=dependencies&view=tables)

> Automate full website screen shots and PDF generation with multiple view ports

### Features

* Crawls specified host and generates a `sitemap.xml` on the fly
* Generates entire website screen shots based on `sitemap.xml`
* Define multiple view ports
* Automated PDF generation
* Includes crawled meta data in generated PDF
* Reports on broken website links (404 http response)
* Supports [HTTP basic authentication](https://en.wikipedia.org/wiki/Basic_access_authentication)
* Supports [Salesforce Visualforce](https://developer.salesforce.com/page/Visualforce) 3 step authentication
* Supports site maps with HTTP, HTTPS, and FTP protocol URLs
* Follows HTTP 301 redirects
* [Custom JavaScript inject file](#custom-javascript-inject-file) - injects into page prior to screen shooting
* Trigger page events by passing querystring values to custom inject.js file


**In This Documentation**

1. [Getting Started](#getting-started)
2. [Siteshooter Configuration File](#create-a-siteshooter-configuration-file)
2. [CLI Options](#cli-options)
3. [Tests](#tests)
4. [Troubleshooting & FAQ](#troubleshooting-and-faq)

## Getting Started ##

#### Dependencies

Install the following prerequisite on your development machine:

* [Node.js - **version >= 6.0.0**](http://nodejs.org)

#### NPM Libraries

Libraries **Siteshooter** depends on:

* [PDFKit](https://github.com/devongovett/pdfkit)
* [PhantomJS](https://github.com/ariya/phantomjs)
* [Simple web crawler](https://github.com/cgiffard/node-simplecrawler)


### Quick Start
```
$ npm install siteshooter --global
```
If siteshooter is installed, make sure you have the latest version by running:
```
$ npm update siteshooter --global
```
* You may need to run these commands with elevated privileges, e.g. `sudo`, you will be prompted to do so if needed.
* Installing with the `--global` flag affords you the `siteshooter` command on your machine's command line at any path.
* Read more about the `--global` flag [here](https://docs.npmjs.com/files/folders).

### Create a Siteshooter Configuration File ###
```
$ siteshooter --init
```

### Update Siteshooter Configuration File

[View the full siteshooter.yml example](https://github.com/devopsgroup-io/siteshooter/tree/master/siteshooter.yml)

Inside `siteshooter.yml`, add additional options. 

* All [Simple Web Crawler options](https://github.com/cgiffard/node-simplecrawler#configuration) can be added to `sitecrawler_options` and will pass through to the crawler process

```yml
domain:
  name: https://www.devopsgroup.io
  auth:
    user:
    pwd:

pdf_options:
 excludeMeta: true


sitecrawler_options:
  exclude:
   - "pdf"
  stripQuerystring: false
  ignoreInvalidSSL: true

viewports:
 - viewport: desktop-large
   width: 1600
   height: 1200
 - viewport: iPhone6
   width: 375
   height: 667

```

### Custom JavaScript Inject File
 
 To manipulate the DOM, prior to the screen shot process, add a `inject.js` file in the same working directory as the `siteshooter.yml`. 

 **Example:** inject.js file
 ```js
/**
 * @file:            inject.js
 * @description:     used to inject custom JavaScript into a web page prior to a screen shot. 
 */

console.log('JavaScript injected into page.');

if ( typeof(jQuery) !== "undefined" ) {

    jQuery(document).ready(function() {
        console.log('jQuery loaded.');
    });
}
 ```

#### Trigger JavaScript Events

When using the optional `inject.js` file, events can be triggered based on the following querystring parameter - **pevent**

```js
 // Add URL with pevent querystring parameter in the generated sitemap.xml
<url>
    <loc>https://www.devopsgroup.io?pevent=open-privacy-overlay</loc>
    <changefreq>weekly</changefreq>
</url>
```

**Example:** Event detection & triggering
 ```js
/**
 * @file:            inject.js
 * @description:     used to inject custom JavaScript into a web page prior to a screen shot. 
 */


function getQueryVariable(variable) {
    var query = window.location.search.substring(1);
    var vars = query.split('&');
    for (var i = 0; i < vars.length; i++) {
        var pair = vars[i].split('=');
        if (decodeURIComponent(pair[0]) == variable) {
            return decodeURIComponent(pair[1]);
        }
    }
}

if ( typeof(jQuery) !== "undefined" ) {

    jQuery(document).ready(function() {
        var pageName = window.location.pathname.replace('/', ''),
            pageEvent = getQueryVariable('pevent');

        console.log('document ready.');
        console.log('userAgent', navigator.userAgent);
        console.log('Page: ', pageName);
        console.log('Event: ', pageEvent);

        switch (pageName) {

            // home
            case '':

                switch (pageEvent) {
                    case 'open-privacy-overlay':

                        jQuery('a[data-target~="#modal-privacy"]').trigger('click');
                        break;
                }

                break;
        }

    });
}
 ```



## CLI Options

```bash

$ siteshooter --help

Usage: siteshooter [options]

OPTIONS
_______________________________________________________________________________________
-c --config            Show configuration
-e --debug             Output exceptions
-h --help              Print this help
-i --init              Create siteshooter.yml template file
-p --pdf               Generate PDFs, by defined view ports, based on screen shots created via Siteshooter
-s --screenshots       Generate screen shots, by view ports, based on sitemap.xml file
-S --sitemap           Crawl domain name specified in siteshooter.yml file and generate a local sitemap.xml file
-v --version           Print version number
-V --verbose           Verbose output
-w --website           Report on website information based on Siteshooter crawled results
```

When running a `siteshooter` command without any options, the following will run by default (if a **siteshooter.yml** file exists in working directory):

* `--pdf`
* `--screenshots`
* `--sitemap`

## Tests

Tests are written with [Mocha](https://github.com/mochajs/mocha) and can be run with `npm test`.

## Troubleshooting

If you're having issues with Siteshooter, [submit a GitHub Issue](https://github.com/devopsgroup-io/siteshooter/issues/new).

* Make sure you have a `siteshooter.yml` file in your working directory and it's well formatted

## Code of Conduct

Take a moment to read or [Code of Conduct](CODE_OF_CONDUCT.md)

## Contributing to the project

We are always looking for quality contributions! Please check the [CONTRIBUTING.md](CONTRIBUTING.md) for contribution guidelines.



