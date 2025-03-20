# Cummins ZPL Tool

At the Cummins turbo manufacturing plant, we use a lot of Zebra printers. These
are printers to print labels for items being shipped out. The format of these
labels are determined by a .zpl file.

The purpose of the ZPL tool is to automate the uploads/updates of ZPL files to
the Zebra printers.

Previously, to update a ZPL file, you had two options:

1. Go physically to printers with a thumb drive of ZPL files
2. Go to each printer's web portal via its IP address, which has an interface
   that allows you to upload/update one ZPL at a time.

There are ~50 printers located across the plant and in a couple of different
buildings miles away. Both options were very time consuming. If we were updating
let's say 10 ZPL file through the printer's web portal, that's already 500
operations, which could take an hour or two.

With the ZPL tool, you can select the ZPLs you'd like to upload and printers to
be updated, and the tool would broadcast the requests to all the printers.

![zpl tool screenshot](/assets/cummins-zpl-tool/screenshot.png)

## Tech

- Frontend: Web-client with React.js
- Backend: Proxy server with FastAPI (Python)

## Why a proxy server

The Zebra printers implemented some CORS logic to reject requests from certain
origins. When trying to interact with the printers through the web client
directly, the requests would be rejected.

Since we can't adjust CORS on the Zebra printers directly, we need a simple
proxy server to forward our requests to by pass the CORS issue.

## Web Client

The web client needs to make multiple API calls. It's best to make them all at
once rather than sequentially. To do so we can utilize `Promise.all` or
`Promise.allSettled`.

Instead of doing something like:

```js
for (const printer of printerIPs) {
  await someAPICallFunction(printer);
}
```

You can do:

```js
const requests = printerIPs.map((printer) => someAPICallFunction(printer));
const results = await Promise.allSettled(requests);
```

Doing it like this makes the API calls concurrent and we don't have to wait on
the result of one call to make the next.

The issue arises when we need to update the state of the API calls.

How do we let the user know the progress of the upload like?
![zpl upload progress](/assets/cummins-zpl-tool/upload_progress.png)

To do so, we need to pass a callback to our API call function, so it can update
the progress of our uploads.

```js
// pseudo code

function updateCallback(status) {
  setUploadProgress(status);
}

const requests = printerIPs.map((printer) =>
  someAPICallFunction(printer, updateCallBack),
);
const results = await Promise.allSettled(requests);

async function someAPICallFunction(printer, updateCallBack) {
  for (const file of files) {
    updateCallBack("loading ", file);
    const response = someAPILogic(printer, file);
  }
  updateCallBack("success");
}
```
