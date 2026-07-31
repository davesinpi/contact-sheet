# Contact Sheet

**[Open the app](https://davesinpi.github.io/contact-sheet/)** (runs entirely in your browser)

Organise a photo library by **who is in the photos**. Faces are detected and matched
entirely in your browser, grouped into people you can name, and exported as one folder
per person, either as a `.zip` or into Google Drive.

No image and no face data ever leaves your machine. The only network requests are for
loading the models once, and, if you choose to use it, Google Drive.

## What it does

- Detects every face in every photo and computes a signature for each one locally.
- Groups matching faces into people, each shown with a face thumbnail and a name you
  can edit.
- **Combine menu**: tick two or more people and merge them in one action, with undo.
  Useful when one person has been split across several groups.
- **Split and unmerge** on any person when a group turns out to hold more than one.
- Drag a photo onto a person to say they are in it, or use the `remove` button on a
  photo to take it out of that person only. Photos with several people belong to all
  of them.
- **Filter** by person, with "all of" for photos containing everyone selected and
  "any of" for photos containing at least one.
- **Remembers your work** between visits: names, combines, splits and corrections are
  saved locally, and photos you have already analysed are restored instantly instead
  of being re-analysed.
- Exports one folder per person, plus a `people.csv` listing which people are in which
  file.
- Handles large libraries. Photos are decoded once, downscaled, and released, so
  memory stays flat whether you load ten photos or a thousand.
- Converts iPhone HEIC files where the browser allows it, and puts anything unreadable
  in a **Could not process** tray with a retry button.

## Files

| File | What it is |
| --- | --- |
| `index.html` | The main app. Organises a library by person using face matching. |
| `photo-sorter-drive.html` | Earlier, simpler version. Sorts by headcount into solo, group, and no people, with Drive support. |
| `photo-sorter.html` | The same headcount sorter, local files only, no setup at all. |

`photo-sorter.html` runs straight from a double-click. The other two need to be served
over `http://localhost` or `https://`, because Google will not authorise a `file://`
page for Drive access. That is why the hosted copy exists.

## Running it

Download the file you want, then serve the folder:

```
python -m http.server 8000
```

Open `http://localhost:8000/`.

Or just use the hosted copy at
[davesinpi.github.io/contact-sheet](https://davesinpi.github.io/contact-sheet/), which is
this repository served by GitHub Pages. It is the same single file; your photos are
still read on your own machine and never uploaded.

## Getting good results

The **grouping** slider controls how close two faces must be to count as the same
person. Left is stricter and produces more, smaller groups; right is looser and
produces fewer, larger ones. The live tally beside it shows how many people are
currently found and how big the largest group is, so you can see the effect while you
drag.

Run it **stricter than feels right**. The two kinds of mistake are not equally
expensive: one person split across three groups is fixed in seconds with the combine
menu, while two people quietly sharing one group looks correct until you check. Start
around 0.30 to 0.45, watch the tally settle near the real number of people in your
library, then combine the duplicates.

Name the people who matter as you go. Only people you have named, combined, split or
corrected are remembered; automatic groups are re-derived from the photos each time,
which keeps stale records from accumulating.

If results ever look wrong, untick **use saved people** to ignore the saved data
without deleting it, and compare. **Forget saved people** clears it for good.

## Google Drive

Optional. Everything except Drive import and export works without it.

Expand the **Google Drive** panel and press **Connect Google Drive**. That opens the
normal Google sign-in and consent window; approve it and the panel shows the account
you are connected as. Nothing to create, nothing to paste.

The app requests only the narrow `drive.file` scope, so it can read the files you pick
in the picker and the files it creates. It cannot see anything else in your Drive. It
also asks for `email`, purely so the panel can tell you which account is connected.

The access token lives in the tab, not on disk, so reloading keeps you connected and
closing the tab signs you out. **Disconnect** revokes the token immediately.

### Hosting your own copy

The built-in connection is tied to specific origins, so a copy served from your own
address needs its own Google credentials. Two ways to supply them.

**Bake them in**, so everyone using your copy gets the one-click connection. In the
[Google Cloud console](https://console.cloud.google.com/):

1. Create a project.
2. Enable the **Google Picker API** and the **Google Drive API**.
3. Under **Credentials**, create an **API key**. Restrict it to your origins under
   **Application restrictions → Websites**.
4. Under **Credentials**, create an **OAuth client ID** of type **Web application**.
   Add every address you serve the page from to its **Authorised JavaScript origins** —
   for example `https://you.github.io` and `http://localhost:8000`. Host only, no path.
5. Read your **project number** from **IAM & Admin → Settings**.
6. Under **Google Auth Platform → Audience**, publish the app. `drive.file` and `email`
   are both [non-sensitive scopes](https://developers.google.com/identity/protocols/oauth2/production-readiness/sensitive-scope-verification),
   so no verification review is required and users do not see an unverified warning. If
   you leave it in **Testing** instead, add each Google account under **Test users** and
   expect the unverified-app screen (choose **Advanced**, then continue).

Then fill in the `BUILTIN_DRIVE` block near the top of the Google Drive section in
`index.html` and `photo-sorter-drive.html`:

```js
const BUILTIN_DRIVE={
  clientId:'000000000000-xxxxxxxx.apps.googleusercontent.com',
  apiKey:'AIzaSy…',
  appId:'000000000000'
};
```

These three values are public by design. The OAuth client only works on the origins you
authorised, and the API key should be referrer-restricted to the same ones.

**Or paste them at runtime.** Leave `BUILTIN_DRIVE` blank and each person opens **Use
your own Google Cloud project** inside the Drive panel and enters the three values
themselves. They are kept in that browser only and are never committed here. This is
also the escape hatch if you want to connect through your own project on a copy that
already has a built-in one.

### Export modes

- **Drive shortcuts** (default) stores one real file and links it into each person's
  folder. Photos that came from Drive are not re-uploaded at all; the shortcut points
  at your original. Locally added photos are uploaded once into an `_originals` folder.
- **Full copies** duplicates the file into every folder it belongs to. Self-contained,
  but uses more space.

Either way the originals are left untouched and a `people.csv` manifest is written
alongside.

## Known limitations

- Grouping needs a visible face. Someone photographed from behind, heavily obscured or
  very small in frame lands under **No people**.
- Accuracy varies with the photos. Harsh lighting, sharp angles, sunglasses and young
  children are all harder for face models, so expect to combine some groups by hand.
- Memory is per browser and per device. It is local storage, not an account, so it does
  not follow you to another machine and clearing site data wipes it.
- Photos are processed one at a time, which is what keeps memory flat. A large library
  is steady rather than fast on its first pass, then cached afterwards.
- Models are fetched from a CDN on first load, so the first run needs a connection. If
  they fail to load, photos stay in **Unsorted** and **Re-analyse** retries.
- Adding a photo to a person is drag and drop, which has no keyboard equivalent yet.

## Privacy

- Photos are analysed in the browser and are never uploaded to any server.
- Face signatures and thumbnails are stored only in this browser, for your own reuse.
- Signing in to Drive is a standard Google OAuth flow. The token stays in the tab and is
  revoked when you press Disconnect. No server of ours ever sees it, because there is no
  server of ours.
- Drive export writes into a new folder and leaves your originals in place.

## Built with

- [face-api.js](https://github.com/justadudewhohacks/face-api.js) for in-browser face
  detection, landmarks and recognition descriptors
- [Google Picker API](https://developers.google.com/picker) and the Drive API
- [heic2any](https://github.com/alexcorvi/heic2any) for HEIC conversion
- [JSZip](https://stuk.github.io/jszip/) for the download bundles
- [TensorFlow.js](https://www.tensorflow.org/js) with COCO-SSD, in the two earlier
  headcount sorters
