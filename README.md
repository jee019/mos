# MOS Evaluation Form Generator

This repository contains static HTML forms for subjective speech evaluation.
It generates NMOS and SMOS evaluation pages from CSV filelists using Jinja2
templates.

- **NMOS**: Naturalness Mean Opinion Score. Each question contains one audio
  sample and asks the listener to rate naturalness from 1 to 5.
- **SMOS**: Speaker Similarity MOS. Each question contains two audio samples
  and asks the listener to rate speaker similarity from 1 to 5.

Submitted answers are sent to the Google Apps Script URL configured in each
render script.

## Repository Layout

```text
.
├── filelist/          # CSV files that define the evaluation items
├── templates/         # Jinja2 HTML templates for NMOS/SMOS forms
├── wav/               # Audio files referenced by the generated HTML files
├── utils.py           # CSV readers that convert filelists into questions
├── render_test.py     # Generates a combined ENG + KOR NMOS/SMOS form
├── render_test_eng.py # Generates an English-only NMOS/SMOS form
├── render_test_kor.py # Generates a Korean-only NMOS/SMOS form
├── render_nmos.py     # Legacy NMOS-only renderer
└── app*.gs            # Google Apps Script examples for saving responses
```

Generated HTML files, such as `test.html`, `test_eng.html`, and
`test_kor.html`, are static pages that can be hosted directly.

## Setup

Install Python dependencies:

```bash
pip install jinja2
```

## Preparing Audio Files

Place the audio files under `wav/` so that the generated HTML page can load
them.

The file paths in the CSV are inserted directly into the HTML `<audio src>`
attribute. Because the generated HTML files are written to the repository root,
the paths should usually start with `wav/`.

Example:

```text
wav/eval_data/eng/mimi/7127-75947-0015.wav
```

If you move the generated HTML file to another directory, update the CSV paths
or keep the same relative `wav/` directory structure next to the HTML file.

## Preparing Filelists

Filelists are CSV files stored under `filelist/`.

For NMOS, the renderer requires a `filename` column:

```csv
order,filename,transcript
1,wav/eval_data/eng/mimi/sample.wav,example transcript
```

For SMOS, the renderer requires `filename1` and `filename2` columns:

```csv
order,filename1,filename2,transcript
1,wav/eval_data/eng/system/sample.wav,wav/eval_data/eng/speakers/reference.wav,example transcript
```

The current renderers only read the audio path columns. Other columns, such as
`order` and `transcript`, can be kept as metadata for tracking the evaluation
items.

## Generating HTML

Run a renderer from the repository root.

Generate a combined English + Korean evaluation form:

```bash
python render_test.py
```

This reads:

- `filelist/NMOS_eng.csv`
- `filelist/SMOS_eng.csv`
- `filelist/NMOS_kor.csv`
- `filelist/SMOS_kor.csv`

and writes:

```text
test.html
```

Generate an English-only evaluation form:

```bash
python render_test_eng.py
```

This reads `filelist/NMOS_eng_15.csv` and `filelist/SMOS_eng_15.csv`, then
writes `test_eng.html`.

Generate a Korean-only evaluation form:

```bash
python render_test_kor.py
```

This reads `filelist/NMOS_kor_15.csv` and `filelist/SMOS_kor_15.csv`, then
writes `test_kor.html`.

## Customizing an Evaluation

1. Add the target audio files under `wav/`.
2. Create or edit the matching CSV files in `filelist/`.
3. If needed, update the CSV filenames, page title, language labels, or
   `form_url` in the render script.
4. Run the render script.
5. Open the generated HTML file and confirm that all audio files play.
6. Host the generated HTML file together with the referenced `wav/` files.

## Response Collection

Each HTML form redirects submissions to the `form_url` value set in the render
script. The provided `app*.gs` files are Google Apps Script examples that write
responses to a Google Spreadsheet.

### Google Sheets integration

1. Create a Google Sheet for the evaluation results.
2. Copy the spreadsheet ID from the sheet URL. In a URL like
   `https://docs.google.com/spreadsheets/d/SPREADSHEET_ID/edit`, the
   `SPREADSHEET_ID` part is the ID.
3. Open the Apps Script editor from the sheet, usually through
   **Extensions > Apps Script**.
4. Paste the matching Apps Script code:
   - Use `app.gs` for the combined ENG + KOR form.
   - Use `app_eng.gs` for the English-only form.
   - Use `app_kor.gs` for the Korean-only form.
5. Replace the spreadsheet ID in `SpreadsheetApp.openById(...)` with your
   result sheet ID.
6. Save the Apps Script project.
7. Deploy it as a web app with **Deploy > New deployment > Web app**.
   Recommended settings:
   - **Execute as**: `Me`, so submissions are written with the script owner's
     permission.
   - **Who has access**: choose the access level that matches your evaluation
     setup. Use `Anyone` only if respondents should submit without signing in.
8. Copy the deployed web app URL. Use the `/exec` deployment URL for real
   evaluations, not the `/dev` test URL.
9. Paste that URL into the `form_url` value in the render script.
10. Regenerate the HTML and submit one test response before sharing the form.

The Apps Script writes responses to the first sheet tab returned by
`getSheets()[0]`. Add header rows manually if you want labeled columns.

For language-specific forms, also check the spreadsheet ID in the matching
`app_eng.gs` or `app_kor.gs` file before deployment.

If you update the Apps Script after deploying it, create a new version and edit
the existing deployment to use that version. This keeps the same web app URL.

## Notes

- `utils.py` defines the CSV readers used by the render scripts.
- `templates/` contains the HTML templates. Edit these files to change the
  instructions, layout, or rating labels.
- `render_nmos.py` is a legacy NMOS-only renderer. It expects
  `filelist/NMOS.csv`; update that path before using it with the current
  filelists.
