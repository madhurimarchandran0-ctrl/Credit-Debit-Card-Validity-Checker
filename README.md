# Card Expiry Detection (OCR)

A Python project that reads the **expiry date** from a credit/debit card image using OpenCV and Tesseract OCR, then tells you whether the card is still valid, expiring soon, or already expired.

Built and tested in a Jupyter notebook (`Card_Detection.ipynb`) in VS Code on Windows.

## Features

- Loads a card image and preprocesses it for better OCR accuracy
- Locates the expiry date (`MM/YY` format) in the extracted text
- Re-reads just the date region with a digits-only OCR pass to reduce noise
- Validates the month and year to reject unrealistic reads
- Calculates days remaining until expiry and reports the card status
- Draws a red box around the detected date on the original image

## How It Works

1. **Load image**: reads the card image with OpenCV (`cv2.imread`).
2. **Preprocess**: converts to grayscale, upscales 2x (cubic interpolation), and applies Otsu thresholding.
3. **OCR**: runs `pytesseract.image_to_data` to get every word with its bounding box.
4. **Find the date**: a loose regex (`MM/` followed by 2 to 4 digits) locates the date token, tolerating noisy reads such as `09/248`.
5. **Re-read the date**: crops the date area, upscales it 3x, and runs Tesseract again with `--psm 7` and a whitelist of `0123456789/`.
6. **Validate**: extracts a strict `MM/YY` match and checks that the year is within a realistic range (10 years back to 15 years ahead).
7. **Compute validity**: the expiry is taken as the **last day of the expiry month**, and days left are calculated from today's date.
8. **Report**: prints the expiry date and one of three statuses.

| Condition | Output |
|---|---|
| Days left < 0 | `No validity (card expired)` |
| Days left <= 90 | `Validity ends soon (N days left)` |
| Otherwise | `Valid for N more days` |

## Requirements

- Python 3.11
- [Tesseract OCR](https://github.com/UB-Mannheim/tesseract/wiki) installed on your system
- Python packages:
  - `opencv-python`
  - `pytesseract`

```bash
pip install opencv-python pytesseract
```

## Setup

1. Install Tesseract OCR (the default Windows path is `C:\Program Files\Tesseract-OCR\tesseract.exe`).
2. Create and activate a virtual environment, then install the packages above.
3. If Tesseract is installed elsewhere, update this line in the notebook:

```python
pytesseract.pytesseract.tesseract_cmd = r"C:\Program Files\Tesseract-OCR\tesseract.exe"
```

## Usage

1. Place your card images (for example `card1.png`) in the same folder as the notebook.
2. Open `Card_Detection.ipynb` in VS Code or Jupyter.
3. Change the image filename in the final cell if needed:

```python
img = cv2.imread("card1.png")
```

4. Run all cells. A window will open showing the card with the detected date boxed in red. Press any key to close it.

### Example Output

```
Expiry date: 09/2028
Valid for 680 more days
```

## Project Structure

```
Card Detection/
├── Card_Detection.ipynb   # Main notebook
├── card1.png ...          # Sample card images
└── README.md
```

## Limitations

- Only the **first** valid `MM/YY` date found is used, so cards that also show a "valid from" date may be misread.
- Accuracy depends on image quality, lighting, and font style; blurry or low-resolution images may fail.
- Assumes a two-digit year format (`MM/YY`).
- `cv2.imshow` needs a desktop environment, so it will not work on headless servers or some hosted notebooks.

## Possible Improvements

- Detect and crop the card automatically before running OCR
- Support `MM/YYYY` and "valid from / valid thru" layouts
- Extract the card number and cardholder name
- Wrap the logic in a function and build a Streamlit upload interface
- Process a whole folder of card images in one run

## Tech Stack

Python, OpenCV, Tesseract OCR (pytesseract), regular expressions
