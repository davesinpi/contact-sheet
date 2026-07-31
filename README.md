# Contact Sheet

**[Open the app](https://davesinpi.github.io/photo-sorter/)** (runs entirely in your browser)

Organise a photo library by **who is in the photos**. Faces are detected and matched
entirely in your browser, grouped into people you can name, and exported as one folder
per person, either as a `.zip` or into Google Drive.

No image and no face data ever leaves your machine. The only network requests are for
loading the models once, and, if you choose to use it, Google Drive with your own
credentials.

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
[davesinpi.github.io/photo-sorter](https://davesinpi.github.io/photo-sorter/), which is
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

## Google Drive setup

Optional. Everything except Drive import and export works without it.

In the [Google Cloud console](https://console.cloud.google.com/):

1. Create a project.
2. Enable the **Google Picker API** and the **Google Drive API**.
3. Under **Google Auth Platform → Audience**, set the user type to **External**, then
   add your own Google account under **Test users**. Leaving it on Internal causes an
   `org_internal` error for personal Gmail accounts.
4. Under **Credentials**, create an **API key**.
5. Under **Credentials**, create an **OAuth client ID** of type **Web application**.
   Add the exact address you serve the page from to its **Authorised JavaScript
   origins**. Add `https://davesinpi.github.io` for the hosted copy, and
   `http://localhost:8000` as well if you also run it locally. Host only, no path.
6. Read your **project number** from **IAM & Admin → Settings**.

Open the app, expand the **Google Drive** panel, and paste in the OAuth client ID, API
key and project number. They are kept in your browser only and are never committed to
this repository.

On first connection Google warns that the app is unverified. That is expected for a
private app you built yourself: choose **Advanced**, then continue.

The app requests only the narrow `drive.file` scope, so it can read the files you pick
in the picker and the files it creates. It cannot see anything else in your Drive.

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
- Google credentials are entered at runtime and stay in your browser.
- Drive export writes into a new folder and leaves your originals in place.

## Built with

- [face-api.js](https://github.com/justadudewhohacks/face-api.js) for in-browser face
  detection, landmarks and recognition descriptors
- [Google Picker API](https://developers.google.com/picker) and the Drive API
- [heic2any](https://github.com/alexcorvi/heic2any) for HEIC conversion
- [JSZip](https://stuk.github.io/jszip/) for the download bundles
- [TensorFlow.js](https://www.tensorflow.org/js) with COCO-SSD, in the two earlier
  headcount sorters
