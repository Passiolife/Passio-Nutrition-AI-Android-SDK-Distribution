Before you continue, please make sure you have a new key for version 2.x. The 1.4.x will not work with 2.x SDK!

## Dependencies
Changes to the build.gradle file:
```groovy
// TensorFlow    
implementation 'org.tensorflow:tensorflow-lite:2.8.0'
implementation 'org.tensorflow:tensorflow-lite-metadata:0.4.0'


// Barcode and OCR
implementation 'com.google.android.gms:play-services-mlkit-text-recognition:18.0.0'
implementation 'com.google.android.gms:play-services-mlkit-barcode-scanning:18.0.0'
```

## API Changes
- PassioMode.IS_AUTO_UPDATING renamed to PassioMode.IS_DOWNLOADING_MODELS
- PassioMode.IS_READY_FOR_NUTRITION was removed.
- PassioSDK.instance.setDownloadListener was removed. Instead, the download process is traced through the PassioStatusListener
- FoodDetectionConfiguration.detectOCR renamed to detectPackagedFood
- FoodDetectionConfiguration.detectLogos was removed.
- PassioSDK.instance.lookupImageForFilename, lookupPassioIDFor, lookupAlternativesFor, lookupAllSiblingsFor, lookupParentFor, lookupAllFoodItemsInDatabase, lookupAllPassioIDs, lookupAllVisuallyRecognizableFoodItems were removed
- PassioSDK.instance.searchForFood now returns a list of Pair<passioID, food name>
- PassioSDK.instance.lookupImageFor was separated into two functions:
    1. Local image lookup: lookupIconFor
    2. Network image fetch: fetchIconFor
- Nutrients in the PassioFoodItemData have been refactored from Double to UnitMass