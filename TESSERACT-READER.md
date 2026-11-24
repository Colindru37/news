# Tesseract Reader: how to use it

Tesseract Reader is an automation object that pulls the text out of PDFs, including the ones where
normal text extraction gives you nothing or garbage. It uses Tesseract, the OCR (text recognition)
engine that already comes with your automation software. Nothing has to be installed and no DLL is
needed: you import the object and it works.

File: `vbo/BPA Object - Utility - Tesseract Reader.xml`

## What it can do

- **Read scanned PDFs**: pages that are only a picture of paper.
- **Read "garbage" PDFs**: PDFs where copying the text gives weird characters (for example fonts
  named `T3Font_0`). It redraws the letters and reads them like a scan.
- **Read normal PDFs**: typed text is read directly, which is fast and exact.
- **Read image files**: TIFF (all pages), JPEG, PNG, BMP and GIF. Phone photos are turned upright first.
- **Mix all of the above in one file.** It decides page by page and tells you what it did.
- **Read Arabic**, including mixed with English, once the Arabic language file is in place (see below).
- **Keep the layout** if you want: columns stay side by side and tables stay lined up.
- **Write the text to a .txt file** at the same time.
- **Open protected PDFs** when you give the password.

## How to set it up

1. Import `BPA Object - Utility - Tesseract Reader.xml`.
2. Done for English. For Arabic, add the language file first (next section).

It finds Tesseract by itself: it looks in the `Tesseract` folder next to your automation software,
where `tesseract-4.0.0.exe` lives.

## Arabic (or any other language)

Tesseract needs one file per language. English is usually already there. Arabic is not.

1. Download `ara.traineddata` from the `tessdata` repository of the tesseract-ocr project on GitHub
   (the plain `tessdata` one, which matches Tesseract 4.0).
2. Copy it into the `tessdata` folder inside the `Tesseract` folder, next to `eng.traineddata`.

This is copying one file, not installing anything.

If you are not allowed to write into that folder:

1. Make your own folder, for example `C:\Users\you\tessdata`.
2. Put `ara.traineddata` in it, and also copy `eng.traineddata` from the `Tesseract\tessdata` folder.
3. Give that folder in the **Tessdata Folder** input.

Then set **Language** to `ara+eng` for Arabic with some English, or `ara` for Arabic only.

If the language file is missing, the action stops with an error that names the folder it looked in.

## The action: Extract Text

### Inputs

| Input | What to put in | Empty means |
|---|---|---|
| Input File | Full path of the PDF or image, e.g. `C:\in\statement.pdf` | (required) |
| Language | `eng`, `ara`, `ara+eng`, `deu+eng`... | English |
| Password | Only if the PDF is protected | no password |
| Tesseract Path | Full path of `tesseract.exe`, only if it is not found by itself | find it automatically |
| Tessdata Folder | Folder with the `.traineddata` language files | the one next to Tesseract |
| Force OCR | `True` = read every page with OCR, even pages that look fine | `False` |
| Output File | Full path of a text file to write, e.g. `C:\out\statement.txt` | no file |
| Keep Layout | `True` = keep columns and tables lined up | `False` (plain reading order) |

Paths may have quotes around them (Explorer's "Copy as path" works).

### Outputs

| Output | What you get |
|---|---|
| Text | All the text. Pages are separated by a form feed character. |
| Pages | A collection, one row per page: Page, Method, Text, Note |
| Pages Read By OCR | How many pages went through OCR |

**Method** says how each page was read:

- `text`: typed text, read directly
- `ocr`: read by OCR (a scan, or text that was garbage)
- `text+ocr`: a page with both typed text and a scanned picture
- `none`: nothing on the page (blank)
- `failed`: the page could not be read; **Note** says why

**Note** explains anything special, for example "typed text read by OCR: the text layer has no
usable character map".

## Keep Layout: what it looks like

Without Keep Layout, a two-column page comes out one column after the other:

```
Customer number 4711
Opening balance 1250
Branch Frankfurt Main
Currency EUR
```

With Keep Layout = True, it looks like the page:

```
Customer number 4711                     Branch Frankfurt Main
Opening balance 1250                     Currency EUR
```

Tables stay lined up the same way. Use it when you need to know what stood next to what. Leave
it off when you just need the words (for searching, for example).

## The output file

- Plain text (`.txt`), UTF-8, so Arabic shows correctly in Notepad.
- Windows line breaks.
- Missing folders are created. An existing file with the same name is replaced.

## Good to know

- **Speed**: a few seconds per page that needs OCR. Typed pages are instant. Tesseract is started
  once per document, and a whole document may take up to 1 minute plus 30 seconds per page.
- **Quality**: OCR is very good on clean pages at normal size. Blurry, tiny or crooked scans read
  worse. That's Tesseract, not the object.
- **Temporary files**: page pictures go to your temp folder and are deleted at the end.
- **If the text is garbage** and the page says `text`, run again with **Force OCR = True**.

## What it can't do

- Scans saved as JBIG2 or JPEG 2000 (rare): those pages come back as `failed` with a note.
- Handwriting: Tesseract reads print, not handwriting.
- Fonts that are not inside the PDF can't be redrawn. Those pages keep their normal text, and the
  note says so.
- Exact formatting (bold, colours, boxes, real tables): the output is plain text.
