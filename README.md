# SKYLARK_DRONES_ASSIGNMENT2



****
&#x20;**GCP Marker Detection and Shape Classification******



This project detects Ground Control Point (GCP) markers in aerial images and classifies their shape. The task consists of two stages:



1\. Detect the marker center in the image

2\. Classify the marker shape (Cross, Square, L-Shape)



The system uses a computer vision based detection method followed by a deep learning classifier (ResNet18) trained on cropped marker patches. The implementation is available in the notebook: 







&#x20;1. Approach



The pipeline consists of two main components:



&#x20;**Step 1: Marker Center Detection**



Since GCP markers are bright white markers, the marker location is detected using classical computer vision:



1\. Convert image to grayscale.

2\. Apply **thresholding** to isolate bright regions.

3\. Extract contours of bright regions.

4\. Select the largest contour.

5\. Compute its centroid using image moments.



This gives the estimated marker center `(x, y)`.



This approach was chosen because:



&#x20;GCP markers are visually distinct (white targets)

&#x20;Classical CV is faster and requires no training

&#x20;It generalizes well across images



The centroid of the detected region is used as the **marker coordinate**.







&#x20;**Step 2: Patch Extraction**



Once the marker center is detected:



&#x20;A 224 × 224 patch centered at `(x, y)` is extracted.

&#x20;The patch is resized if necessary.

&#x20;The patch becomes the input for the classifier.



Patch extraction helps because:



&#x20;It removes irrelevant background

&#x20;It focuses the model only on the marker







**Step 3: Shape Classification**



A ResNet18 CNN is used to classify marker shapes.



Architecture:



&#x20;Backbone: ResNet18 (pretrained on ImageNet)

&#x20;Final layer modified to output 3 classes



Classes:



0 → Cross

1 → Square

2 → L-Shape





ResNet18 was chosen because:



\* Lightweight and fast

\* Good performance on small image datasets

\* Transfer learning improves convergence







&#x20;2. Training Strategy



&#x20;Dataset Preparation



Training labels were loaded from:





**gcp\_marks.json**





Each entry contains:





{

&#x20; "mark": { "x": ..., "y": ... },

&#x20; "verified\_shape": "Cross / Square / L-Shape"

}





For each training sample:



1\. Load image

2\. Crop patch centered at (x, y)

3\. Resize to 224 × 224

4\. Normalize to \[0,1]

5\. Convert to tensor







&#x20;**Data Split**



Dataset split





80% training

20% validation





Implemented using **random\_split**.





**Model Configuration**



Model:





**ResNet18**

**Final layer → Linear(512 → 3)**



Loss function:





**CrossEntropyLoss**



Optimizer:





Adam

Learning rate = 0.0003





Training parameters:





Batch size = 32

Epochs = 5





The model with the lowest validation loss is saved as:





best\_model.pth









**Data Augmentation**



Minimal augmentation was used in the baseline pipeline.



Implicit augmentation occurs due to:



&#x20;varying aerial viewpoints

&#x20;varying lighting conditions

&#x20;different environments



Possible improvements include:



&#x20;random rotations

&#x20;color jitter

&#x20;horizontal flips







&#x20;3. Dataset Characteristics



Observations from dataset analysis:



&#x20;Images are high resolution aerial images

&#x20;Markers appear as bright white objects

&#x20;Marker shapes include:





Cross

Square

L-Shape





The marker location is provided in the labels.



The dataset also contains:



&#x20;multiple folders

&#x20;nested directory structure







&#x20;4. Challenges and Mitigation









&#x20;Challenge 1: Large Image Size



Original images are very large.



Processing full images directly with CNNs would be inefficient.



Solution:



&#x20;Extract small patches around markers

&#x20;Train CNN only on the patch



This drastically reduces computation.







Challenge 2: Marker Localization



Instead of training a detection model, the marker was located using image thresholding.



Advantages:



&#x20;No additional training required

&#x20;Fast and robust for bright markers







&#x20;5. Inference Pipeline



The inference pipeline performs the following steps for each test image:



&#x20;Step 1: Load Image





cv2.imread("image\_path")









&#x20;Step 2: Detect Marker Center





detect\_marker\_center(image)





Procedure:



1\. Convert image to grayscale

2\. Threshold bright pixels

3\. Find contours

4\. Select largest contour

5\. Compute centroid



Output:





(cx, cy)









&#x20;Step 3: Extract Patch



patch = crop\_patch(image, cx, cy, size=224)









&#x20;Step 4: Predict Shape





shape = predict\_shape(patch)





Using trained \*\*ResNet18 model\*\*.







&#x20;Step 5: Save Prediction



Predictions are stored in:





predictions.json





Format:





{

&#x20; "relative\_image\_path": {

&#x20;     "mark": {

&#x20;         "x": float,

&#x20;         "y": float

&#x20;     },

&#x20;     "verified\_shape": "Cross | Square | L-Shaped"

&#x20; }

}









&#x20;**6. How to Run Inference**



&#x20;Step 1: Download Model Weights



Place the trained model file:





best\_model.pth

https://drive.google.com/file/d/14s\_OiAZy6HgebtBNsK0fg6uyFrNsEUpi/view?usp=sharing



in the working directory.







&#x20;Step 2: Prepare Test Dataset



Unzip the dataset:





test\_dataset/

```







&#x20;Step 3: Run Inference Script



Execute the notebook or script:





python inference.py





(or run the Colab notebook)



The script will:



1\. Load the trained model

2\. Iterate through all test images

3\. Detect marker center

4\. Predict marker shape

5\. Save predictions







&#x20;Step 4: Output File



The output will be generated as:





predictions.json









&#x20;**7. Possible Improvements**



Future improvements could include:



&#x20;training a marker detection model (YOLO / Faster R-CNN)

&#x20;stronger data augmentation

&#x20;deeper models such as ResNet50

&#x20;using multi-scale patch extraction





