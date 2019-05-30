# Testing AWS Textract's ability to correctly extract data tables from a difficult FBI stats report PDF

(not done yet)

> tl;dr: pretty good table structure overall, given the issues with the original PDF. However, there were inexplicable and critical data errors, as if Textract converted the PDF to an image, OCRed it, and then attempted to extract the data tables.
> 



## File manifest

- [fbi-nics-sample-page.pdf](fbi-nics-sample-page.pdf): an example page from the FBI NICS system representing data for November 2015. This PDF file was copied directly from [jsvine/pdfplumber](https://github.com/jsvine/pdfplumber/blob/master/examples/pdfs/background-checks.pdf)'s example,[ background-checks.pdf](https://github.com/jsvine/pdfplumber/blob/master/examples/pdfs/background-checks.pdf)
- [textract-results-fbi-nics.zip](results/textract-results-fbi-nics.zip): the zip file that the Textract demo sends you as a download
    - And, for your convenience, the individual files as extracted from the zip:
     - [apiResponse.json](results/textract-results-fbi-nics-zip/apiResponse.json)
     - [keyValues.csv](results/textract-results-fbi-nics-zip/keyValues.csv) (note: this file is empty)
     - [rawText.txt](results/textract-results-fbi-nics-zip/rawText.txt)
     - [tables.csv](results/textract-results-fbi-nics-zip/tables.csv)
- [abbyy.xlsx](results/abbyy.xlsx): the PDF-to-XLSX conversion produced by ABBYY FineReader 12.1 for MacOS
- [abbyy.csv](results/abbyy.csv): the PDF-to-XLSX conversion via ABBYY FineReader, but saved as CSV. 


## Intro



**Again, fair warning**: for this gist, I'm mostly/only interested in seeing how good Textract is when it comes to extracting data tables from **non-image PDFs**. In other words, I don't really care about Textract's OCR's capability for now, [though you can read Mozilla Source's roundup of OCR options (which didn't include Textract at time of publication)](https://source.opennews.org/articles/so-many-ocr-options/) to get a taste of how complex and painful the OCR problem is on its own. In other other words, we can assume that if data table extraction is hard, then data table extraction on the results of OCRed document is an additional level of of clusterfuck complexity; if Textract can just get data table extraction right, that will be a huge victory for data folks on its ownn



## Why extracting data tables from PDFs is so hard

PDF is a great format for when you need a digital document that – unlike the vast majority of webpages and spreadsheets – will essentially look the same to anyone else who opens it on any computer, and/or wants to print it out on paper. PDFs, like Word documents, can contain not just standard prose, but data tables – such as copy-pasting from an Excel spreadsheet into Word, and then saving as PDF.

But for various technical reasons, extracting the data table from a PDF is often not as easy as copy-pasting from PDF into Excel. In fact, the metadata of the data's layout and structure is usually destroyed and irrecoverable when data tables are saved as PDF documents. It's a hard enough problem that it's one of the only situations in which I've given up on trying to hack it myself and settled for a commercial software package that is non-unscriptable (i.e. have to use by manual point-and-click): $99 for ABBYY FineReader – [which is *still* far-from-perfect but good enough](https://github.com/helloworlddata/white-house-salaries), all things considered.

Note: If you are interested in more technical details about on why extracting data tables from PDFs is so complicated, I highly recommend the following resources:

- [Heart of Nerd Darkness: Why Updating Dollars for Docs Was So Difficult](https://www.propublica.org/nerds/heart-of-nerd-darkness-why-dollars-for-docs-was-so-difficult), by ProPublica's Jeremy Merrill, which is an excellent and detailed overview of the PDF problem and how it applies to a real-world data investigation.
- [Introducing Tabula](https://source.opennews.org/en-US/articles/introducing-tabula/), by the authors of the Knight Mozilla-supported [open source Tabula project](https://source.opennews.org/articles/introducing-tabula/), because PDF-to-CSV is really that huge of a data problem for journalists.
- [Introduction to The Camelot Project](https://camelot-py.readthedocs.io/en/master/user/intro.html), excellent documentation and writeup by another open source PDF-to-CSV extraction library and tool, because PDF-to-CSV is a massive problem for everyone who works with documents and data
- [One of the many, many Hacker News discussions about how awesome it would be if someone could create a good PDF-to-CSV tool](https://news.ycombinator.com/item?id=13729301)




## Why the FBI's report on background checks for firearms is a very annoying data PDF

In any case, rather than try Textract on a real-world-but-simple PDF, I decided to upload one of the most annoying government data-as-PDF examples I've seen: a report from the **FBI's National Instant Criminal Background Check System**, which Jeremy Singer-Vine uses as an example (complete [with Jupyter notebook](https://github.com/jsvine/pdfplumber/blob/master/examples/notebooks/extract-table-nics.ipynb)) to demonstrate the **pdfplumber** library he wrote (yes, yet another open source PDF-to-CSV project, because it really is such a painful and critical problem). You can read Singer-Vine's [helpful Python notebook showing the code and the PDF](https://github.com/jsvine/pdfplumber/blob/master/examples/notebooks/extract-table-nics.ipynb). Or better yet, you can read Singer-Vine's [separate writeup of the FBI data](https://github.com/BuzzFeedNews/nics-firearm-background-checks) (one of the best web-scraping-for-journalism examples I've seen). Or even even better, read a story that Singer-Vine's BuzzFeed News colleague, Peter Aldhous, wrote based on the data: [Under Trump, Gun Sales Did Not Spike After The Las Vegas Shooting](https://www.buzzfeednews.com/article/peteraldhous/gun-sales-after-vegas-shooting).

Suffice to say, the background-checks.pdf supplied by pdfplumber is a very nice example of a data-stuck-in-PDF – it has real-world importance and is actually, at first glance, very readable, but contains enough technical and epistemological issues to be a hard challenge to both humans and automated software when it comes to extracting the data.

## How Textract handles the FBI NICS example PDF

So I uploaded the single-page example [backgrounds-checks.pdf](https://github.com/jsvine/pdfplumber/blob/master/examples/pdfs/background-checks.pdf) to Textract. After about a minute, I was able to download a zip which contained 4 files, 3 of which I've attached to this gist in their raw forms:


- apiResponse.json (3.6MB): contains the data behind Textract's decision at the granular per-word/character/line level, including the exact bounding boxes and confidvalues for each text extraction.
- keyValues.csv (empty): Not sure what this is supposed to be but it was empty
- rawText.txt (5 KB): just the extracted results as a stream of unformatted plaintext
- tables.csv (9 KB): the tabular data that Textract found, in spreadsheet-ready format


**tables.csv** is the thing we obviously care about. You can download it and open it in Excel yourself, but for your convenience, I've also uploaded it to Google Sheets. Or you can just read my quick summations and screenshots below:

Here's a screenshot of the top half of the table, and all of its columns, to show that Textract did quite well in not only getting the right number of columns, but also dealing with the confusing grouped headers

