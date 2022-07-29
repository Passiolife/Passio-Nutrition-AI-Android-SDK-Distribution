# Passio SDK Release Notes

Full project was build with **Kotlin 1.6.10**

## V2.2.5

* No API changes

## V2.2.3

* No API changes

## V2.2.2

* No API changes
* Fixed internal services

## V2.2.1

* Fix issue with alcohol nutrition mapping
* Refactored search function to asynchronously return the list of results. Internally improved the search algorithm to work with small typos

```kotlin
fun searchForFood(
        byText: String,
        callback: (result: List<Pair<PassioID, String>>) -> Unit
)
```

## V2.1.5

* Refactored the lookupIconFor function and split it into two functionalities:
1. Function that does a local lookup of the icon. Returns an actual icon of the food, in which case the second parameter of the result will be true. Or returns the default icon for the food type with the second parameter of the pair being false
```kotlin
fun lookupIconFor(
    context: Context,
    passioID: PassioID,
    iconSize: IconSize = IconSize.PX90,
    type: PassioIDEntityType = PassioIDEntityType.item,
): Pair<Drawable, Boolean>
```
2. Function that fetches the icon from Passio's backend and caches it. The next lookupIconFor call for returned the cached image from the networking call.
```kotlin
fun fetchIconFor(
    context: Context,
    passioID: PassioID,
    iconSize: IconSize = IconSize.PX90,
    callback: (drawable: Drawable?) -> Unit
)
```

* Number of food items recognized via HNN: 4049
* Nutrition database version: passio_nutrition.4049.0.300

## V2.1.3

* PassioFoodItemData nutrient are now provided as UnitMass
* Renamed OCR to PackagedFood
* Refactored PassioStatusListener to encorporate the download listener
* SearchForFood now returns a mapping between the PassioID and the name of the food
* Bug fixed and performance improvements

* Number of food items recognized via HNN: 3907
* Number of logos recognized: 300
* Nutrition database version: passio_nutrition.3907.0.300

## V2.1.1 

* Beta release of the 2.0 version of the SDK

* Number of food items recognized via HNN: 3777
* Number of logos recognized: 300
* Nutrition database version: passio_nutrition.3777.0.300

## V1.4.15

* Number of food items recognized via HNN: 3613
* Number of logos recognized: 300
* Nutrition database version: passio_nutrition.3613.0.300

## V1.4.14

* Number of food items recognized via HNN: 3545
* Number of logos recognized: 300
* Nutrition database version: passio_nutrition.3545.0.300

## V1.4.13

### Overview Models
* Number of food items recognized via HNN: 3439
* Number of logos recognized: 300
* Nutrition database version: passio_nutrition.3439.0.300
## V1.4.12

### Bug fixes
* Fixed issue with ocr vectorize file backwards compatibility
* Added "1 serving" as a default serving size for recipes 

### Overview Models
* Number of food items recognized via HNN: 3328
* Number of logos recognized: 300
* Nutrition database version: passio_nutrition.3329.0.300

## V1.4.11

### Overview Models
* Number of food items recognized via HNN: 3191
* Number of logos recognized: 300
* Nutrition database version: passio_nutrition.3191.0.300

## V1.4.10

### Overview Models
* Number of food items recognized via HNN: 3106
* Number of logos recognized: 300
* Nutrition database version: passio_nutrition.3106.798.300

## V1.4.9

### Overview Models
* Number of food items recognized via HNN: 3056
* Number of logos recognized: 300
* Nutrition database version: passio_nutrition.3056.798.300

## V1.4.8

### Overview Models
* Number of food items recognized via HNN: 3045
* Number of logos recognized: 300
* Nutrition database version: passio_nutrition.3045.798.300

### Api Changes
* autoUpdate flag in the PassioConfiguration class renamed to sdkDownloadsModels

## V1.4.7

### Overview Models
* Number of food items recognized via HNN: 3036
* Number of logos recognized: 300
* Nutrition database version: passio_nutrition.3036.798.300

### Bug fixes
* Caching bug that would cause a PassioIDAttrributes object to change selectedQuantity
* Wrong PassioStatus called in case of autoUpdate

## V1.4.6

### Overview Models
* Number of food items recognized via HNN: 3015
* Number of logos recognized: 300
* Nutrition database version: passio_nutrition.3015.798.300

### Api Changes
* offlineMode added to PassioConfiguration to disable eventual SDK networking calls
* LabelForNutrient enum added to associate macronutrient with a measurement of weight

## V1.4.5

### Overview Models
* Number of food items recognized via HNN: 2998
* Number of logos recognized: 300
* Nutrition database version: passio_nutrition.2998.798.300

### API Changes

* detectFoodIn method can be now used to detect barcode, logo and ocr
```kotlin
fun detectFoodIn(
    bitmap: Bitmap,
    foodDetectionConfiguration: FoodDetectionConfiguration? = null,
    onDetectionCompleted: (candidates: FoodCandidates?) -> Unit
)
```

### API addition

* Added PassioSDKError to the PassioStatus
* Added case FAILED_TO_CONFIGURE to the  PassioStatus.mode
* Added searchForFood(byText: String) method to easily filter food names
* Refactored autoUpdate mode of the SDK configuration process to callback twice if the models are being downloaded and installed for the first time.

## v1.4.4

### Overview Models
* Number of food items recognized via HNN: 2993
* Number of logos recognized: 300
* Nutrition database version: passio_nutrition.2993.798.300

## Bugs fixes
* Fixed issue with the bounding boxes returned by the  ```boundingBoxToViewTransform``` method.

## v1.4.3

### Overview Models
* Number of food items recognized via HNN: 2981
* Number of logos recognized: 300
* Nutrition database version: passio_nutrition.2981.800.300

### Improvements
* Retrieve UPC data from Passio's backend

## Bugs fixes
* Fixed issue with the  ```lookupAllPassioIDs``` method.

### API update 

* Removed  ```viewWidth: Int``` and  ```viewHeight: Int``` parameters from the ```detectFoodCandidatesIn```  method.
* Fixed issue when  ```detectFoodCandidatesIn``` before initializing the camera. 

## v1.4.2.1

### API update 

* Removed  ```viewWidth: Int``` and  ```viewHeight: Int``` parameters from the ```detectFoodCandidatesIn```  method.
* Fixed issue when  ```detectFoodCandidatesIn``` before initializing the camera. 

## v1.4.2

### Update Models
* Number of food items recognized via HNN: 2963
* Number of logos recognized: 300
* Nutrition database version: passio_nutrition.2963.797.300

### API update 

* Removed  ```requireContext()``` method from the ```PassioCameraViewProvider``` 

## v1.4.1

### Added APIs

* Passio SDK's camera can now react to orientation changes. Once the orientation change is detected, pass the displayRotation angle to the ```startCamera``` function.

```kotlin
fun startCamera(
    passioCameraViewProvider: PassioCameraViewProvider,
    displayRotation: Int = 0,
    tapToFocus: Boolean = false
)
```
* Refactored the way the SDK handles bounding boxes. The bounding box returned from the object detection process now has relative coordinates. To fetch the absolute coordinates of the screen this utility method has to be called:

```kotlin
fun boundingBoxToViewTransform(
    boundingBox: RectF,
    viewWidth: Int,
    viewHeight: Int,
    displayAngle: Int = 0,
    barcode: Boolean = false
): RectF
```

## v1.4.0

### Updated architecture

Going forward Passio Android SDK is using v13 models and has deprecated usage of v12 models. Any future model releases will be guaranteed backwards compatibility from this build on. 

### Updated APIs

* The ```data class FoodCandidates``` signature was changed to
```kotlin
data class FoodCandidates(
    val detectedCandidates: List<DetectedCandidate>? = null,
    val logoCandidates: List<ObjectDetectionCandidate>? = null,
    val barcodeCandidates: List<BarcodeCandidate>? = null,
    val ocrCandidates: List<OCRCode>? = null
)
```

* ```VisualCandidate``` was renamed to ```DetectedCandidate``` and its structure was simplified
```kotlin
class DetectedCandidate(
    val passioID: PassioID,
    val confidence: Float,
    val boundingBox: RectF,
    val croppedImage: Bitmap?
)
```

* The ```FoodDetectionListener``` interface was renamed to ```FoodRecognitionListener``` and its signature was changed 
```kotlin
interface FoodRecognitionListener {
    fun onRecognitionResults(
        candidates: FoodCandidates,
        image: Bitmap?,
        nutritionFacts: PassioNutritionFacts?
    )
}
```

* Updated ```PassioMode``` enum entries to
```kotlin
enum class PassioMode {
    NOT_READY,
    IS_BEING_CONFIGURED,
    IS_AUTO_UPDATING,
    IS_READY_FOR_NUTRITION,
    IS_READY_FOR_DETECTION
}
```

* Added ```debugMode``` to ```PassioConfiguration``` class to enable more verbose logs

* The ```startFoodDetection``` signature was changed to
```kotlin
fun startFoodDetection(
    foodRecognitionListener: FoodRecognitionListener,
    detectionConfig: FoodDetectionConfiguration = FoodDetectionConfiguration()
)
```

```DetectionOptions``` were renamed to ```FoodDetectionConfiguration``` and its signature was changed
```kotlin
data class FoodDetectionConfiguration(
    var framesPerSecond: PassioSDK.FramesPerSecond = PassioSDK.FramesPerSecond.TWO,
    var detectFood: Boolean = true,
    var detectLogos: Boolean = false,
    var detectBarcodes: Boolean = false,
    var detectOCR: Boolean = false,
    var detectNutritionFacts: Boolean = false,
    var removeLogoByLogoId: Boolean = false
)
```

* The ```detectFoodIn``` method was renamed to ```detectFoodCandidatesIn```

* The ```PassioDownloadListener``` interface signature was changed to
```kotlin
interface PassioDownloadListener {
    fun onCompletedDownloadingAllFiles(fileUris: List<Uri>)
    fun onCompletedDownloadingFile(fileUri: Uri, filesLeft: Int)
    fun onDownloadError(message: String)
}
```

* Added interface ```PassioStatusListener``` with a method 
```kotlin
fun setStatusListener(statusListener: PassioStatusListener?)
``` 
to enable observing status changes generated by the configure method

## v1.2.14
* Model update

## v1.2.13
* Model update

## v1.2.12
* Model update
* SDK Configure function change for external model configuration:

Create a PassioConfiguration object and pass the application context with the developer key. Point the configuration object to the list of Uris of files that need to be updated.
```Kotlin
val passioConfiguration = PassioConfiguration(
    this.applicationContext,
    "Your developer key here"
).apply {
    localFiles = /* List of file URIs */
}
```
Pass the configuration object to the configure method of the PassioSDK. When the configuration process is complete, a PassioStatus object will be delivered in the callback.
```Kotlin
PassioSDK.instance.configure(passioConfiguration) { passioStatus ->
    when (passioStatus.mode) {
        PassioMode.NOT_READY -> onPassioSDKError(passioStatus.debugMessage)
        PassioMode.READY_FOR_NUTRITION,
        PassioMode.READY_FOR_DETECTION -> onPassioSDKReady()
    }
```
The PassioStatus object contains a mode that return the currently operation configuration of the SDK, missing files if the SDK needs updated files to function optimally, debug message to give more information on the configuration process, and active models version which represents the version number of the models that are currenly running in the SDK.
```Kotlin
class PassioStatus {
    var mode: PassioMode = PassioMode.NOT_READY
    var missingFiles: List<String>? = null
    var debugMessage: String? = null
    var activeModels: Int? = null
}

enum class PassioMode {
    NOT_READY,
    BEING_CONFIGURED,
    READY_FOR_NUTRITION,
    READY_FOR_DETECTION
}
```

## v1.2.11
* Model update

## v1.2.10
* Model update

## v1.2.10
* Model update

## v1.2.8
* Separated food images for the SDK's main .aar file. Images are now optional to include, and can be found in the passioicons.aar file.
* Model update

## v0.12.0
* Removed alternatives from PassioFoodItemData, instead added children, siblings and parents fields
* Model update

## v0.11.0
* Barcode and OCR are enabled through Google Play Services
* Model update

## v0.10.0
* Removed ML Kit Barcode dependecy
* Optimized SDK startup time
* Removed TFLite GPU delegate 
* Model update

## v0.9.0
* Changed OCR dependency to use ML Kit version without the use of google-services.json file
* Model update

## v0.8.0
* Detection API change - all of the different types of detection are now activated through a single method - startFoodDetection. A DetectionOptions object can be passed to control which detection processes to activate or to customize your scanning session.
* Added changes to the model download mechanism
* Fixed initialization without Google Services being enabled
* Model update

## V0.7.0
* Model update

## V0.6.0
* Customisations of the capture session
* Bug fixes and performance improvements
* Model update

## V0.5.0
* Added support for food images
* Bug fixes and performance improvements
* Model update

## V0.4.0
* Stability and bug fix update
* Model update

## V0.3.0
* Voting process takes power from the hierarchical organisation of the food items.
* Classification model is deprecated.
* New HNN model is being used.
* Added OCR to the SDK
* Changes in the nutritional models.
* Added ability to change the resolution of processed images in runtime.

## V0.3.0
* Voting process takes power from the hierarchical organisation of the food items.
* Classification model is deprecated.
* New HNN model is being used.
* Added OCR to the SDK
* Changes in the nutritional models.
* Added ability to change the resolution of processed images in runtime.

## V0.2.0

* Refactored configuration of the SDK to accept files downloaded by the client
* SDK can be configured to download the files itself with auto-update

## V0.1.0

## Overview of the Model Update

* Number of food items recognized: 803 + 1 background class
* Number of logos recognized 121
* Nutrition database version: 0.7.7

## Initial features

* CameraViewProvider for CameraX implementation
* PassioCameraFragment for easy camera implementaion on a Fragment
* Object detection
* Classification
* Object detection with voting layer - combined object detection and classification
* Logo detection
* Nutritional database
* Detection from a Bitmap
