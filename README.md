# Automatic Number Plate Recognition (ANPR) Pipeline

This project contains a step-by-step progressive Computer Vision pipeline to detect, extract, validate, and log license plates in real-time from a webcam feed.

The system is broken down into modular stages to easily understand how ANPR works, from basic camera integration up to temporal smoothing and logging.

## Requirements
```bash
pip install opencv-python numpy pytesseract
```
*Note: Ensure Tesseract OCR is installed on your system and is added to the system PATH.*

## Pipeline Stages

### 1. Camera Initialization (`camera.py`)
This is a basic sanity check script. It initializes `cv2.VideoCapture(0)` to ensure the webcam is working, capturing, and displaying frames correctly.

### 2. Plate Detection (`detect.py`)
This stage introduces computer vision heuristics to find license plate candidates.
- Uses Grayscale conversion, Gaussian Blur, and Canny Edge Detection.
- Extracts contours and filters them based on `MIN_AREA` (minimum pixel area) and aspect ratios specific to license plates.
- Draws a green bounding box over potential plate regions.

**Result Screenshot:**
*(Insert screenshot of detect.py output showing a green box around a license plate here)*

### 3. Perspective Alignment (`align.py`)
This script takes the exact contour detected in the previous step and applies a Perspective Transform (Homography). 
- It identifies the 4 corners of the plate.
- It extracts and warps that region into a strictly rectangular, flat `450x140` image.

**Result Screenshot:**
*(Insert screenshot of align.py output showing the flattened, top-down view of the cropped plate here)*

### 4. Optical Character Recognition (`ocr.py`)
With the plate cleanly isolated and flattened, this script passes the image to the `pytesseract` OCR engine.
- Binarizes the image using Otsu's Thresholding.
- Extracts raw text using Tesseract's Character Whitelist (`A-Z, 0-9`), trying to read what is written on the plate.

**Result Screenshot:**
*(Insert screenshot of ocr.py output showing raw tesseract string overlaid near the bounding box here)*

### 5. Format Validation (`validate.py`)
OCR isn't perfect and frequently reads background noise or bumper stickers. This stage introduces Regex validation to filter out incorrect detections.
- Ensures the OCR output strictly matches the `[A-Z]{3}[0-9]{3}[A-Z]` format (e.g., `ABC123D`).
- Categorizes outputs clearly as "VALID" or just "OCR" gibberish on the UI.

**Result Screenshot:**
*(Insert screenshot of validate.py showing a successfully validated string in bright green text here)*

### 6. Temporal Smoothing & Logging (`temporal.py`)
The final, robust application. To prevent errors from a single "lucky/unlucky" bad frame, this stage buffers reading results over time.
- Uses a `majority_vote` from the previous 5 frames.
- If the system consistently reads the same valid plate, it is marked as `CONFIRMED`.
- The confirmed plate is appended to a logging CSV file: `data/plates.csv`.
- A 10-second cooldown is implemented to prevent spamming the CSV with identical plate records.

**Result Screenshot:**
*(Insert screenshot of temporal.py showing "CONFIRMED: ABC123D" on screen and logs appended)*

## How to Run
To use the final ANPR application, run the temporal script directly:
```bash
python src/temporal.py
```
Logs will be saved to `data/plates.csv`.
