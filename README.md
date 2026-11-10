# PDF Summarizer

A local Python app that turns a short, text-based PDF into a plain-language overview and up to five key points. The app asks the model to include PDF page numbers so you can check the summary against the source.

## How it works

1. **Upload.** Streamlit accepts a PDF from your browser.
2. **Extract.** pypdf reads each page and labels extracted text with its position in the PDF, starting at 1.
3. **Summarize.** The app sends the extracted text and summary instructions to Ollama at `http://localhost:11434/api/generate`.
4. **Review.** The browser shows the generated summary. Expand **See the text sent to the model** to inspect the source text.

The app uses a downloaded local model and requires no API key. Installing dependencies and downloading the model require internet access. The summarization request itself goes to Ollama on your computer.

## Prerequisites

- Python 3 available as `python` in PowerShell. Check with `python --version`.
- [Ollama for Windows](https://ollama.com/download/windows), installed and running.
- The starter `app.py` from the implementation guide, saved next to this README. The README documents that implementation.
- A short test PDF with selectable text.

## Setup on Windows

Open PowerShell in the folder containing `app.py` and run:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
ollama pull llama3.2:3b
```

The virtual environment keeps the dependencies separate from other Python projects. These commands use its Python executable directly, so you can leave the environment unactivated.

To confirm the model is available, run:

```powershell
ollama list
```

Look for `llama3.2:3b` in the list.

## Run the app

```powershell
.\.venv\Scripts\python.exe -m streamlit run app.py
```

Open the local URL printed by Streamlit. Upload a PDF, click **Summarize**, and wait for the result. The first request can take longer while Ollama loads the model. Press **Ctrl+C** in PowerShell to stop Streamlit.

## Verify your first summary

Create a two-page test PDF with these fictional facts. Insert a page break before the second sentence when you export it.

- **Page 1.** The library will close on June 12 for maintenance.
- **Page 2.** The library will reopen on June 15 at 9:00 a.m.

Upload it and check that the summary preserves both dates, the reopening time, and their page references. Open the extracted-text panel to confirm what the model received. If the extraction is correct but a date or citation is wrong, the error is in the generated summary.

Try two more inputs:

- **An image-only scan.** Expect a message that no readable text was found.
- **A PDF with more than 6,000 extracted characters.** Expect a message asking for a shorter document.

## Settings and limits

- **Model.** `MODEL = "llama3.2:3b"` in `app.py` selects the installed model. If you switch models, download the new tag with `ollama pull` and update this value.
- **Input cap.** `MAX_TEXT_CHARS = 6_000` limits the joined text, including page labels. This is a starter-app limit measured in characters. Model context windows are measured in tokens. Oversized text is rejected before the model call.
- **Timeout.** The HTTP request uses a 180-second timeout. Local inference speed depends on the machine and model.
- **Page coverage.** Pages with no extracted text are skipped. The app reports how many pages yielded text when coverage is partial. Diagrams, scanned pages, and layout-heavy tables may lose important information during extraction.
- **Page citations.** Labels use the PDF's page order. Printed page numbers in a document can differ. Check important claims against the original PDF.

## Troubleshooting

- **PowerShell says Python or Ollama is unrecognized.** Confirm the program is installed and reopen PowerShell so it picks up any PATH changes.
- **The app cannot get a model response.** Open Ollama, confirm `ollama list` shows the exact model tag, and try again. You can also try `ollama run llama3.2:3b` to confirm the model responds outside the app.
- **The PDF appears blank to the app.** Try selecting text in your PDF viewer. If the page is an image, apply optical character recognition (OCR) to create a text layer, then upload the OCR output.
- **Text appears out of order.** Compare the extracted text with the original, especially for multi-column pages and tables. For your first test, use a single-column document.
- **The PDF is encrypted.** The starter rejects encrypted PDFs. Use an unencrypted copy you are authorized to access.
- **The summary misses or invents a detail.** Inspect the extracted text first. If the source fact is present, compare another run or revise the summary instructions and rerun your test PDF.

## Extend it to long PDFs

1. Keep the text and page number for each page.
2. Group consecutive pages into chunks sized for the model's input budget. Split unusually long pages within their text and retain the page number on each piece.
3. Summarize each chunk with page references and leave enough context space for the instructions and the model's answer.
4. Combine the chunk summaries into one final summary. If they exceed the input budget, combine them in batches.
5. Test with a document whose most important fact appears on the last page. Verify that the final result includes the fact and its page reference.

## References

- [Streamlit file uploader](https://docs.streamlit.io/develop/api-reference/widgets/st.file_uploader)
- [pypdf text extraction and OCR limitations](https://github.com/py-pdf/pypdf/blob/main/docs/user/extract-text.md)
- [Ollama generate API](https://docs.ollama.com/api/generate)
- [Llama 3.2 model](https://ollama.com/library/llama3.2)
