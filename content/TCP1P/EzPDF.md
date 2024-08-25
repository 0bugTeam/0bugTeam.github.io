---
title: Ez PDF
date: 2023-12-10T13:37:49+02:00
description: Writeup for Ez PDF [TCP1P]
author: 1byteBoy
tags:
  - Forensics
draft: false
---

**Author**: `daffainfo`

I just downloaded this PDF file from a strange site on the internet....

-----

*For this challenge i was having some issues with my system, odd one, since this flag was divided into 3 parts, i didn't get the first first for the problem, so my fried `@soham` got it. I was imply by running string on the pdf file or using exiftool*

```bash
strings TCP1P-CTF.pdf
or
exiftool TCP1P-CTF.pdf
```

which gave us the following encoded string

```
SW4gdGhpcyBxdWVzdGlvbiwgdGhlIGZsYWcgaGFzIGJlZW4gZGl2aWRlZCBpbnRvIDMgcGFydHMuIFlvdSBoYXZlIGZvdW5kIHRoZSBmaXJzdCBwYXJ0IG9mIHRoZSBmbGFnISEgVENQMVB7RDAxbjlfRjAyM241MUM1
```

since it looks like a base64 encode strings we can easily decode it by running 

```bash
echo "SW4gdGhpcyBxdWVzdGlvbiwgdGhlIGZsYWcgaGFzIGJlZW4gZGl2aWRlZCBpbnRvIDMgcGFydHMuIFlvdSBoYXZlIGZvdW5kIHRoZSBmaXJzdCBwYXJ0IG9mIHRoZSBmbGFnISEgVENQMVB7RDAxbjlfRjAyM241MUM1" | base64 -d
```

which gave us the first part of the flag

```
In this question, the flag has been divided into 3 parts. You have found the first part of the flag!! TCP1P{D01n9_F023n51C5
```

the second part which i got was by using a tool named `pdftohtml` or `pdfimages` , which extracted a hidden image from the second layer of the PDF document

![part2](https://cdn.discordapp.com/attachments/1104280989410279539/1162497558703243456/TCP1P-CTF-1_1.png?ex=653c2738&is=6529b238&hm=cdf2be648e83ce89191512af216b7af02b78fb1aa4f665d56a68115513641358&)

For the last part it was an obfuscated JavaScript strings in the PDF file, i copied the code or string and used a online tool to first prettify or format the code https://beautifier.io/

Next i used another online tool to de-obfuscate the code 
https://deobfuscate.relative.im/ 

which finally gave me the following

```js
if (whutisthis === 1) {
	this.print({
	bUI: true,
	bSilent: false,
	bShrinkToFit: true,
	})
} else {
	function pdf() {
	a = '_15N7_17'
	b = 'i293m1d}'
	c = '_l3jaf9c'
	console.log(a + c + b)
	}
	pdf()
}
```

Now adding all of them together and setting it up with our other parts gives us the full flag to submit

```
TCP1P{D01n9_F023n51C5_0N_pdf_f1L35_15_345y_15N7_17_l3jaf9ci293m1d}
```