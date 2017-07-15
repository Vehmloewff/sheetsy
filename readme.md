Use Google Sheets as a data source for your webapps.

1. Create a spreadsheet
2. Create a webapp that points at that spreadsheet
3. Update the spreadsheet, watch your app update automatically

Why write a CMS for small projects when you could just edit a spreadsheet?

Works in node and the browser.  The ES5 browser build is ~1188 bytes minified/gzipped.

# Relationship to Tabletop.js

This library owes a lot to @jsoma and [Tabletop.js](https://github.com/jsoma/tabletop).

Tabletop's implementation pointed me at the endpoints to hit, and saved me a lot of investigation into the specifics of how to fetch the data from Google Sheets.

Sheetsy is much simpler than Tabletop, by virtue of doing less, avoiding legacy IE and HTTP support, and being composed of pure functions, which allows this library to be more thoroughly tested.

# How to set up your Google Sheet

1. Create a [Google Sheet](https://docs.google.com/spreadsheets/)
2. Navigate to the "File" menu
3. Click "Publish to the Web"
	- It should default to "Entire Document" + "Web page", which is what you want.
	- The defaults should be fine for "Published content & settings" as well - for maximum easiness, you'll want to "Automatically republish when changes are made".
5. Click "Publish"
6. Copy the URL

# Using a published Google Sheet as a data source

This is what you'll do:

1. Turn the URL into a key
2. Using the key, fetch the top-level document containing a list of sheets
3. With the original key and a sheet id from the list, fetch one of the sheets to access its rows

# API

```js
const sheetsy = require('sheetsy')
const { urlToKey, getDocument, getSheet } = sheetsy
```

## `key = urlToKey(url)`

Given a `url` string, returns a `key` string.  Throws an error if no key is found.

```js
const key = urlToKey(
	'https://docs.google.com/spreadsheets/d/14uk6kljx-tpGJeObmi22DkAyVRFK5Z1qKmSXy1ewuHs/pubhtml'
) // => '14uk6kljx-tpGJeObmi22DkAyVRFK5Z1qKmSXy1ewuHs'
```

Tabletop.js notes that [some publish URLs don't work for some reason](https://github.com/jsoma/tabletop#if-your-publish-to-web-url-doesnt-work) - if you run into a published document that doesn't work for some reason, please open an issue with the link that you're using and a description of the error, I'd be interested to create some tests for those cases.

## `promise = getDocument(key, [getFunction])`

Given a `key` string, returns a Promise that resolves to a object containing an object describing the document.

```js
getDocument('14uk6kljx-tpGJeObmi22DkAyVRFK5Z1qKmSXy1ewuHs').then(document => {
	document // => {
//		authors: [
//			{
//				name: 'joshduffman',
//				email: 'joshduffman@gmail.com',
//			}
//		],
//		updated: '2017-07-14T04:59:24.123Z',
//		sheets: [
//			{ name: 'Herp', id: 'od6', updated: '2017-07-14T04:59:24.123Z' },
//			{ name: 'Derp', id: 'of6b9b5', updated: '2017-07-14T04:59:24.123Z' }
//		]
//	}
})
```

## `promise = getSheet(key, id, [getFunction])`

Given a `key` string and an `id` string from the document results, returns a Promise that resolves to a sheets object:

```js
getSheet('14uk6kljx-tpGJeObmi22DkAyVRFK5Z1qKmSXy1ewuHs', '0d6').then(sheet => {
	sheet // => {
//		updated: '2017-07-14T04:59:24.123Z',
//		authors: [
//			{
//				name: 'joshduffman',
//				email: 'joshduffman@gmail.com',
//			}
//		],
//		rows: [
//			rowArray({
//				firstheader: `Something's up`,
//				secondheader: `That's "cool"`,
//			}),
//			rowArray({
//				firstheader: 'a3',
//				secondheader: 'b3',
//			}, [ 'a3', 'b3', 'c3' ])
//		]
//	}
})
```

In addition, you can also access values of a row by property name.  In this example, the value in cell A1 is "First header":

```js
getSheet('14uk6kljx-tpGJeObmi22DkAyVRFK5Z1qKmSXy1ewuHs', '0d6').then(sheet => {
	const firstRow = sheet.rows[0]
	firstRow.firstheader // => `Something's up`
	firstRow.secondheader // => `That's "cool"`
})
```
### Column names/property names

The column names are generated by Google Sheets, not by this library.

Whether you format the first row as a header or not, Google Sheets will treat it as if it is the header row of your sheet, and will use the values in those columns to generate the names of all of the column values in all the rows following.

It appears to strip spaces and lowercase your text.

## Optional argument: `getFunction`

The `getSheet` and `getDocument` functions take an optional getter function.  This is automatically given a function backed by `XMLHttpRequest` in the browser, or the [`got`](https://github.com/sindresorhus/got) module in node.

You can pass in your own function if you want to try to get it working from an HTTP site or something.  The function is expected to take the url as a string, make a GET request to that url, and return a promise that resolves to the body of the response as parsed JSON.

# How to use in the browser or node

Your browser bundler will need to respect the `browser` field in `package.json`.  Given that, all you need to do is `import sheetsy from 'sheetsy'` or `const sheetsy = require('sheetsy')`.

# Private sheets support

I'm open to the feature, but don't have need of it yet myself.

If you find yourself needing private sheets, open an issue with full details about your use case, and/or a pull request with tests.

# License

[WTFPL](http://wtfpl2.com)
