+++
date = '2025-09-15T09:54:22+02:00'
title = 'OCR using Abbyy Finereader for ScanSnap'
tags = ['Mac']
+++
The other day, I wanted to run some Optical Character Recognition (OCR) on a
document that while it could have been machine-readable, it was a scanned
document.
I first tried to use Adobe Acrobat, but the results were very disappointing.
Since I own a Fujitsu ScanSnap scanner, I have a license for
[Abbyy FineReader PDF](https://en.wikipedia.org/wiki/ABBYY_FineReader).
However, since it is bundled with the ScanSnap device, it is actually
"Abbyy FineReader for ScanSnap".
Opening the PDF using the `Scan To Searchable PDF.app`,
I got an error:

> This file was not created by ScanSnap. ABBYY FineReader for ScanSnap
> can only process PDF files created by ScanSnap.

Now, I could have printed those 40 pages and scanned them again or trick
Abbyy FineReader into believing that this particular document has been
scanned with my scanner.
I went for the latter:

```shell
brew install exiftool
exiftool -Creator="PFU ScanSnap Organizer" myfile.pdf
```

Voila, shortly after I had my searchable document.
