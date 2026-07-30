# Contact Sheet

Sort a pile of photos by how many people are in each one. Drop them in from your
computer or pull them from Google Drive, and the app files each photo into **Solo**
(one person), **Group** (two or more), or **No people**. You can drag any photo
between trays to fix a guess, download each group as a `.zip`, and save the sorted
sets back into Google Drive.

Person detection runs entirely in your browser with TensorFlow.js, so your photos
never get uploaded to any server. The only network calls are to load the detection
model once, and, if you use it, to Google Drive with your own credentials.

## Two versions

- `photo-sorter.html` — local only. Open it and sort photos from your computer.
- `photo-sorter-drive.html` — adds importing from and saving to Google Drive.

The local version works from a plain file. The Drive version has to be served over
`http://localhost` or `https://`, because Google will not authorise a `file://`
page.

## Run it locally

1. Download the file you want.
2. For the local version, just open it in a browser.
3. For the Drive version, serve the folder and open it through the server:
   ```
   python -m http.server 8000
   ```
   then visit `http://localhost:8000/photo-sorter-drive.html`.

## Google Drive setup (one time)

In the [Google Cloud console](https://console.cloud.google.com/):

1. Create a project.
2. Enable the **Google Picker API** and the **Google Drive API**.
3. Under **Google Auth Platform → Audience**, set the user type to **External** and
   add your own Google account under **Test users**.
4. Under **Credentials**, create an **API key**.
5. Under **Credentials**, create an **OAuth client ID** of type **Web application**.
   Add the address you serve the page from to its **Authorised JavaScript origins**
   (for example `http://localhost:8000`, or your GitHub Pages address).
6. Find your **project number** under **IAM & Admin → Settings**.

Open the app, expand the **Google Drive** panel, and paste in the OAuth client ID,
API key, and project number. They are stored only in your browser, never in this
repository.

The app requests the narrow `drive.file` scope, so it can only touch files you pick
and files it creates. It cannot see the rest of your Drive.

## Privacy

- Photos are analysed in the browser and are never sent to a server.
- Your Google credentials are entered at runtime and kept in the browser only.
- Saving to Drive creates copies inside a new `Sorted photos` folder; originals are
  left untouched.

## Built with

- [TensorFlow.js](https://www.tensorflow.org/js) with the COCO-SSD model for
  in-browser person detection
- [Google Picker API](https://developers.google.com/picker) and Drive API
- [JSZip](https://stuk.github.io/jszip/) for the download bundles
