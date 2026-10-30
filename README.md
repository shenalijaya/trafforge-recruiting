# Trafforge recruiting

Tools for running Trafforge's hiring process: a candidate tracker for the recruiting team and a careers page for the public website.

| Folder | What it is |
| --- | --- |
| `tracker/` | The candidate tracker (`index.html`). Single file, no build step. |
| `careers/` | The careers page for trafforge's website, plus `README.md` with the backend spec it needs. |
| `samples/` | Two fictional resumes for testing the tracker's resume reader. |

## Running the tracker locally

Open `tracker/index.html` in a browser, or serve the folder:

```bash
cd tracker
python3 -m http.server 8080
# then visit http://localhost:8080
```

It loads PDF.js and Mammoth (resume reading) from cdnjs and fonts from Google Fonts, so it needs an internet connection.

## Important: where the data lives

The team's live tracker runs on claude.ai, which provides its shared database, resume file storage and CSV export. A copy hosted anywhere else (GitHub Pages, your own server, a local file) falls back automatically to **this-browser-only storage**: it works, but each person sees only what they entered on their own computer, and resumes aren't attached.

To run it as a shared app on Trafforge's own infrastructure, add a storage backend:

- All reads and writes go through one small "store" object in `tracker/index.html` (search for `makeDbStore` and `makeLocalStore`). Each store implements `subscribe(callback)`, `save(candidate)`, `remove(id)` and `saveSettings(settings)`.
- Write a third store against your backend (for example Supabase, Firebase, or your own API) with the same four methods, and use it in the start-up code at the bottom of the file.
- Put it behind your company login. Candidate records contain personal data, so the tracker must never be publicly reachable.
- For resume files, replace the `assetsApi.upload(file)` call with an upload to private storage, and for CSV export, replace `downloadsApi.save(...)` with a normal browser download.

## Careers page

See `careers/README.md` for the form endpoint it posts to, how to link LinkedIn job posts to it, and the CSV export format the tracker imports.

## Data protection

Never commit real candidate data, resumes or exports to this repository. The `samples/` resumes are fictional.
