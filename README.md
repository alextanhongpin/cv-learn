# cv-learn
Things related to Computer Vision

## Preprocessing Pipeline

There are several pipeline operations that can be done to improve the extraction pipeline:

- contrast/brightness correction
- rotate/resize
- blur
- grayscale
- separate foreground from background
- deskew
- remove noise (long horizontal/vertical lines, unrelated subject)
- contour selection

There are no right and wrong steps/order. But to optimize it, we can run several preprocessing pipelines simulataneously. For images that are already clean (scanned documents), applying too many preprocessing might remove the subject of interest. For images that are not clean (photo taken with poor camera), there might be too many noise that requires heavy preprocessing to correct.

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


Best way to extract content from unstructured documents

- https://www.quora.com/What-is-the-best-methodology-for-extracting-data-from-unstructured-documents
- https://github.com/tesseract-ocr/tesseract/wiki/ImproveQuality
- https://gist.github.com/endolith/334196bac1cac45a4893#


## Standard iso A4 paper dimensions
```
210 x 297
2480 x 3508
30pt = 40px
```
## Preprocessing pipeline
- transformation - scaling, rotation, resize
- feature reduction - gray, denoise, tophat
- feature extraction
- feature scaling

each pipeline is configurable, and skippable. One way is to run the pipeline multiple times and get the values and repeat until it is reaches the optimum.

## Features
- target features are the text. we can be more specific on the font by specifying an initial fontsize, say 12pt and rescaling them back to px. This can be used to infer the row height.
- since it is a document, we can assume there’s a pattern to how the texts are aligned. One way is to get all the rows and recompute the height.
- size of A4 document. If we know that we are scanning a rectangle A4 document, that is a good hint itself. We can assume that the image scanned will be portrait, and the ratio should be within the range.
- color of documents. white. increase the contrast
- field names. Since it is a form, we can infer specific field names
- number of columns. The form can have two columns, multi columns etc
- company name or logo. we can deduce the type of document and which company it belongs to if we have the logo to do feature matching.

## Noise
- background. We want to eliminate the background from the foreground document
- lines, especially long horizontal/vertical lines can be a noise, since it will make boundaries to be captured wrongly. One way to remove them is to find the boundary of such lines, and compute the width and height respectively. If they are long (to 0.8 of width or height) and have a thickness less than the scaled font size, we can safely remove them. We can mask them first, and redo the preprocessing steps to get better results.
- watermark? 
- different color documents.
- margin padding of document. We can make this optional to remove the first 2cm padding top left right bottom of the document to improve preprocessing.

## OCR on documents

Results may vary a lot. So creating an optimized pipeline on one solution will fail miserably on another. The best option is to create multiple pipelines, e.g.
- pipeline for scanned documents. Since they have less noise, we do not need to perform much operation which could degrade the image instead of making it better
- pipeline for documents taken with photos. This would be a more noisy image and could require heavy preprocessing. We could make it easier to process the image by allowing users to provide hints such as:
    - photo taken with dark background - allow users to crop that section
    - noise removal. allow users to select which part to exclude, or clean up noises
    - area selection. allow user to select the region or interest by panning the image and cropping that part
    - skew correction - allow users to select the four corners of the document
    - contrast adjustment - allow users to adjust the contrast of the image instead
    - text correction. allow users to correct the recommended text
- A more constraint way is to provide a template to match. When users upload the template and the document associated with the template, feature matching can be performed to identify the region of interests, as well as the area of the dynamic fields to capture.

- other possible things to do with OCR - license plate detector (LPD)
- identity card/passport detector
- sudoku solver


## OpenCV solution to clear border
- if the border is known. Do a flood fill on the top left/top right/bottom right/bottom left of the image.
- find the largest contour (approximate the area > 90% of the original image). Draw a black border of 1px. Iterate until the borders disappear. 
bluring without affecting edges 
- if the edges are important (such as that of document/text), using gaussian blur might cause some loss of information. use median filter that preserves the edges or bilateral filtering (which is slower).
corner detection:
- first, we need to check if the image really has a corner…We can find the first top 4 edges [100 x 100px] or 20% of the image width/height respectively. Then find the min and max color. If the difference is more than, say 25.5 (10%), then we can try to do a floodfill operation to erase the corner (and deskew it). There are four possible corners for floodfill, we should try all of them and find the mean of the color. If there are drastic changes, then it means that the whole image has been modified, which is not what we want. We can try flood filling one corner first…If all produce negative results, skip this step.
- angle correction - we can first attempt to perform a long dilation to extract all horizontal lines. then, from this horizontal sections, we compute the rectangle box and find the mean angle of rotation as well as the changes of width/height. We can do this by finding the contours, and sorting them from top to bottom, we assume it fills the whole width left to right. Then grab all the outleft values and compute the gradient. do the same for the outerright values. draw the intersection line between the first and last row to get the actual rectangle
Font size:
	- to find the font size, we can just approximate the height of each rows. If there are no large changes to the fontsize, we will get the average.
Check if the contents are in the same row:
	- dilate horizontally with a large horizontal kernel. if the width is greater than the height by a factor of 5 (this depends on what you are doing), dilate the text in this row, find the contour, and scan the texts individually. We can use the left anchor as the initial y position for the row, perform skew correction to align the items back to 0.

some potential problems
- multiple columns documents. If a document has multiple columns in a single row, then scanning them line by line might not make sense, since we do not know the exact conditions. Best is to be specific - let users choose the option whether it’s single or multiple column or has min or max columns. Else assume it’s only a single column.
- unknown border - we should not be concerning ourselves with the border unless we want to really clean the document border. Make it an option whether to detect the border or not. If the user opt for it, then perform it. If the user wants to capture the whole thing nicely, they should have scanned the document well enough to separate the background from the foreground. Else, perform double scanning - with and without the border if you have enough computation power. 
- problematic borders - borders on the left, right, cut borders …. and maybe borders within the image and text to be scanned can prove challenging when you set the algorithm to detect borders. the best way is to ignore it. Capture only the edges of the text and find the contours - then work on the small individual images - rotate, skep, transform etc. This way, we can also parallelize the operation (think map reduce) by sending part of the image to different machines to operate).
- multiple column, with text break. Another challenging one. Imagine you have a page with two columns, and the first column has a text that breaks into multiple line  They are still classified as a single line.
- Varying heights and columns. 
- Other interesting things to do is create an image editor that allows user to manipulate the image (crop borders, enhance contrast etc). Or if it is possible, get the device sensor orientation and angle to correct tilted images)
- landscape vs portrait. If the orientation is wrong, it is not possible to perform ocr. also, different language have different orientation
- languge detection. It is not possible to detect and convert to multiple languages at the moment. The only way is to scan the image twice, and merge the two copies later.

detecting coins
	- watershed transformation
detecting image…in an image. requires one to find the bouding box. This first requires. 

## TODO

- look at offscreen canvas for webworker operations

## How to get rid of outliers? 

Find the median (mean are sensitive to outliers), then take the values less than the min / more than the mean, find the majority. 

https://docs.opencv.org/master/d8/d4b/tutorial_py_knn_opencv.html

## Information Extraction

https://towardsdatascience.com/deep-learning-for-specific-information-extraction-from-unstructured-texts-12c5b9dceada
https://blogs.itemis.com/en/deep-learning-for-information-extraction
