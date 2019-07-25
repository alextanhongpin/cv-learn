# cv-learn
Things related to Computer Vision

## Information Extraction

Information extraction (IE) is the task of automatically extracting structured information from unstructured and/or semi-structured machine-readable documents.

You took a photo of your document, say your resume with your phone camera. Now you want to extract the text from the image. How do you do that? 

That is a tough question for me to answer initially. If you have heard of OCR, then you would have known it’s possible to perform optical character recognition using tools like Tesseract to extract the text from your resume. If only it were that simple.

Lets deal with reality - while technology has advanced, it is still hard to extract such information from an image accurately - we have to deal with constraints. Having more constraints (if you see it the other way around, it can be thought of business requirements or usecases).

Here are some constraints that we can think of:

- A4 documents only. This allows us to filter images that does not fit the constraint - which can be measured by aspect ratio of the image, or the orientation of the image, which should be portrait
- White paper. Since it is a document, we can assume that the majority of the area should be white. In some rare cases, the document is black and the text itself is white. 
- Font size. This is a little tricky. Even though it’s possible to extract the text from the documents, we can’t distinguish the font style - bold, italic, heading, body, underline etc. In fact, those features may end up making the whole document less scannable. Some font sizes (especially at the footer) are too small to be scanned, and they may be mistakenly thought as a single paragraph, making it harder to extract the whole text line by line.
- font type. Most OCR will have problems detecting uncommon font types. 
- font color. If the font color is not distinguishable from the typical font color (black), this makes it harder to distinguish the text from the background. 
- Columns. Most documents can consist of multiple columns, which can make text extraction hard. Also, some text may overflow to the next line, making it harder to extract entity as a whole. Take for example, the home address. It is normally written on the top right corner, and on the left it may contain other text in the same row too. This makes it harder to group the text together to infer the context. Of course, that is only if we scan the rows line by line. We can instead find the section of the image on the left and on the right, and split them accordingly. The text for each section can the be extracted separately. If we know which region contains which text (say, from all the documents, we can get the probability value of the address being at the section at the top right corner), then we can use naive bayes to predict the section and content of the address.
- Border. Borders, long horizontal/vertical lines can affect the extraction of the text, and also increase computation power (scanning area that does not have text). 
- Image. Some resume may contain images, we may want to extract it too. This can be done by finding the contour of the image, if they have distinguishable border. 
- lighting. The image lighting can affect the text extraction. one solution is to use adaptive threshold to find the mean threshold. It is preferable to run a median filter/gaussian blur to remove noise first before the adaptive threshold, to reduce the number of noise in the image.
- Defining the rules. There are certain rules when it comes to extracting the text. We can find the patterns if we know the domain (the type of documents we are scanning). For resume, we know they will surely be the name, age, gender, work experiences, skills etc. 
- Layouts of the documents. Some documents may have some unconventional layout, such as image and text break in the same column. This can make text extraction complicated, and at best returns partial results. Having to deal with different layouts is what makes the information extraction complicated.
