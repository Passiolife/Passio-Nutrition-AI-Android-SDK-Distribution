# Passio SDK Release Notes

Full project was build with **Kotlin 1.6.10**

## V3.1.0

### Introducing the Nutrition AI Advisor!

* To access the API od the Advisor use ```NutritionAdvisor.instance```

```kotlin
fun configure(
    appContext: Context,
    licenseKey: String,
    callback: (result: PassioResult<Any>) -> Unit
)

fun initConversation(callback: (result: PassioResult<Any>) -> Unit)

fun sendMessage(
    message: String,
    callback: (response: PassioResult<PassioAdvisorResponse>) -> Unit
)

fun sendImage(
    bitmap: Bitmap,
    callback: (response: PassioResult<PassioAdvisorResponse>) -> Unit
)

fun fetchIngredients(
    response: PassioAdvisorResponse,
    callback: (response: PassioResult<PassioAdvisorResponse>) -> Unit
)
```

### Refactored APIs

* Nutrition facts scanning has been separated out in different recognition flow. The ```FoodDetectionConfiguration``` object no longer has the *detectNutritionFacts* parameter.
* The scanning now is controled by these new functions

```kotlin
fun startNutritionFactsDetection(listener: NutritionFactsRecognitionListener)

fun stopNutritionFactsDetection()

interface NutritionFactsRecognitionListener {
    fun onRecognitionResult(nutritionFacts: PassioNutritionFacts?, text: String)
}
```

### Added APIs

* Added speech recognition API that retrieves a list of recognized foods from the input text query.

```kotlin
fun recognizeSpeechRemote(text: String, callback: (result: List<PassioSpeechRecognitionModel>) -> Unit)
```

* Added LLM image detection, that works remotely through Passio's backend.

```kotlin
fun recognizeImageRemote(bitmap: Bitmap, callback: (result: List<PassioAdvisorFoodInfo>) -> Unit)
```

* Added a function to retrieve a PassioFoodItem for a *v2 PassioID*

```kotlin
fun fetchFoodItemLegacy(passioID: PassioID, callback: (foodItem: PassioFoodItem?) -> Unit)
```

## V3.0.3

* Added APIs

* Added two APIs to fetch a list of meal plans and fetch a meal plan for a certain day.

```kotlin
fun fetchMealPlans(callback: (result: List<PassioMealPlan>) -> Unit)

fun fetchMealPlanForDay(
    mealPlanLabel: String,
    day: Int,
    callback: (result: List<PassioMealPlanItem>) -> Unit
)
```

* Added values for carbs protein and fat in the ```PassioSearchNutritionPreview``` data class.

* MealTime was renamed to ```PassioMealTime```.

* Refactored APIs

* PassioSearchResult was renamed to ```PassioFoodDataInfo```, fetchFoodItemForSearchResult was renamed to ```fetchFoodItemForDataInfo```.

* fetchFoodItemForSuggestion was removed. Instead ```fetchFoodItemForDataInfo``` is used.

## V3.0.2

## Added APIs

* Added API to fetch suggestion for a certain meal time. To fetch a the full nutritional data for a suggestion item use ```fetchFoodItemForSuggestion```.

```
fun fetchSuggestions(
    mealTime: MealTime,
    callback: (results: List<PassioSearchResult>) -> Unit
)

enum class MealTime(val mealName: String) {
    BREAKFAST("breakfast"),
    LUNCH("lunch"),
    DINNER("dinner"),
    SNACK("snack")
}

fun fetchFoodItemForSuggestion(
    suggestion: PassioSearchResult,
    callback: (foodItem: PassioFoodItem?) -> Unit
)
```

## Refactored APIs

Renamed ```fetchSearchResult``` to ```fetchFoodItemForSearchResult```.

## V3.0.1

## Added APIs

* Reintroduced support for alternatives to visual results. Every detected candidate will have a list of ```alternatives: List<DetectedCandidate>```. These alternatives represent contextually simillar foods. Example: If the DetectadeCandidate would be "milk", than the list of alternatives would include items like "soy milk", "almond milk", etc. 

* Also, the interface ```FoodRecognitionListener``` might return multiple detected candidates, ordered by confidence. These multiple candidates represent the top result that our recognition system is predicting, but also other results that are visually simillar to the top result. Example: If the first result in the list of ```detectedCandidates``` is "coffee", there might be more results in the list that are visually simillar to coffee like "coke", "black tea", "chocolate milk", etc.

* Added *PassioSearchNutritionPreview* to the PassioSearchResult, for the ability to show the default serving sizes in the search functionality. 

```
data class PassioSearchNutritionPreview(
    val calories: Int,
    val servingUnit: String,
    val servingQuantity: Double,
    val servingWeight: String
)
```

## Refactored APIs

* Changed PassioFoodItem *shortName* and *verboseName* to ```name``` and ```details```. Details represent either the food item where the nutritional data comes from or the brand of the food product.

## V3.0.0-alpha

Version 3 of the Passio SDK introduces major changes to the nutritional data class and the search functionality. The SDK no longer supports offline work, there is no more local database.

## Deprecated APIs

* ```lookupNameFor```, ```lookupPassioAttributesFor```, ```lookupPassioAttributesForName``` and ```lookupAllDescendantsFor``` have been removed since these function were querying the local database

* *PassioIDAttributes*, *PassioFoodItemData* and *PassioFoodRecipe* have been removed. The new data model that will handle nutritional data is called *PassioFoodItem*

## Refactored APIs 

* ```searchForFood``` now returns *PassioSearchResult* and a list of search options. The PassioSearchResult represent a specific food item associated with the search term, while search options provide a list of possible suggested search terms connected to the input term.

* ```lookupIconFor``` has been deprecated, the correct function to use is ```lookupIconsFor```

* ```fetchPassioIDAttributesForBarcode``` and ```fetchPassioIDAttributesForPackagedFood``` have been replaced with ```fetchFoodItemForProductCode``` than now returns a *PassioFoodItem* result

* ```DetectedCandidate``` now has an attribute called _foodName_

* ```FoodRecognitionListener``` method ```onRecognitionResults``` can now return nullable FoodCandidates

* ```fetchNutrientsFor``` has been renamed to ```fetchInflammatoryEffectData```, and *PassioNutrient* has been renamed to *InflammatoryEffectData*


## Added APIs

* ```fetchSearchResult``` returns a *PassioFoodItem* object for a given _PassioSearchResult_

* ```fetchFoodItemForPassioID``` returns a *PassioFoodItem* object for a given passioID corresponding to a result from the visual detection

* Added class *PassioSearchResult* that represents a result from the *searchForFood* function

* Added class *PassioFoodItem* that represent a food item from the Passio Nutritional database. It has a list of *PassioIngredient*s, with their respective *PassioFoodAmount*s and *PassioNutrient*s


## V2.3.15

### Deprecated APIs

* Removed support for language packs, they will be available in version 3.0.0. Removed ```enableLanguagePack``` from the ```PassioConfiguration``` object, and the ```setSDKLanguage```.

### Added APIs 

* Fetch a map of nutrients for a 100 grams of a specific food item

```
fun fetchNutrientsFor(passioID: PassioID, onResult: (nutrients: List<PassioNutrient>?) -> Unit)

data class PassioNutrient(
    val name: String,
    val amount: Double,
    val unit: String,
    val inflammatoryEffectScore: Double,
)
```

## V2.3.13

* Add support for German language pack

### Added APIs

* New flag in the ```PassioConfiguration``` object called ```enableLanguagePack```. If set to true, after SDK initializtion, if the phone locale is set to German will apply the German language pack. English is default.

* Added new API to set the language of the food items from the nutritional database. If set to AUTO, the phone locale is chosen.
```
fun setSDKLanguage(
    context: Context,
    language: SDKLanguage,
    result: (languageSet: SDKLanguage?) -> Unit
)

enum class SDKLanguage(val code: String) {
    AUTO("auto"),
    EN("en"),
    DE("de"),
}
```

## V2.3.11

* Fix ```PassioFoodItemData``` caching error.
* Added support for sugars, sugar alcohol, trans fat, saturated fat and cholesterol in the Nutrition Facts label reader.
* The Nutrition Facts label reader now supports *French*.

## V2.3.8

### Deprecated APIs

* ```lookupIconFor``` has been deprecated. Use ```lookupIconsFor``` instead.

### Refactored APIs

* Total amounts of nutrients in ```PassioFoodItemData``` are now *nullable*.
* Camera and camera autofocus features in the manifest are now marked as not required.

### Added APIs

* As a replacement for the deprecated ```lookupIconFor```, ```lookupIconsFor``` now returns two values: The default icon shipped with the SDK and the cached icon, if one is present.
```
fun lookupIconsFor(
    context: Context,
    passioID: PassioID,
    iconSize: IconSize = IconSize.PX90,
    type: PassioIDEntityType = PassioIDEntityType.item,
): Pair<Drawable, Drawable?>
```
* Get the URL to fetch the icon for a food item with
```
fun iconURLFor(context: Context, passioID: PassioID, size: IconSize = IconSize.PX90): String
```

* Fetch the tags of a food item
```
fun fetchTagsFor(passioID: PassioID, onTagsFetched: (tags: List<String>?) -> Unit)
```

## V2.3.3

* Added support for YOLO object detection model

## V2.3.1

* Added ```@CameraSelector.LensFacing cameraFacing: Int``` parameter to ```startCamera```. Gives the ability to choose which camera to open.
* Added new class ```PassioCameraConfigurator``` and a new ```startCamera``` function that uses this as an input parameter. The PassioCameraConfigurator enables custom setup of the camera preview, image analysis, zoom levels and other. 

```kotlin
interface PassioCameraConfigurator {

    @RotationValue
    fun displayRotation(): Int = Surface.ROTATION_0

    @LensFacing
    fun cameraFacing(): Int

    fun preview(): Preview

    fun analyzer(): ImageAnalysis {
        return ImageAnalysis.Builder().apply {
            setTargetAspectRatio(AspectRatio.RATIO_16_9)
        }.build()
    }

    fun setZoomRatio(minRatio: Float, maxRatio: Float): Float = 1f

    fun tapToFocus(): Boolean = true
}
```
* Added function ```stopCamera``` that will release all resources bound to the camera process, including the lifecycle owner and the preview view.
* Added ```useFrontCamera``` flag to the constructor of the ```PassioCameraFragment```.


## V2.3.0

* Removed the support for Kotlin version 1.4.21
* Now publishing only one sdk file named passiolib-release.aar

## V2.2.23

* No API Changes

## V2.2.21

* No API Changes

## V2.2.19

* Added advanced search flag in ```PassioConfiguration``` market as an experimental feature:
```
@ExperimentalAPI
var advancedSearch: Boolean = false
```
If set to ```true``` the SDK will download an additional file that optimizes the search algorithm.

## V2.2.17

### Dependency changes:
* Updated CameraX and MLKit versions (Kotlin version 1.6.10)
```
// CameraX
def camerax_version = "1.3.0-alpha12"
implementation "androidx.camera:camera-core:$camerax_version"
implementation "androidx.camera:camera-camera2:$camerax_version"
implementation "androidx.camera:camera-lifecycle:$camerax_version"
implementation 'androidx.camera:camera-view:1.3.0-alpha02'
implementation 'androidx.camera:camera-extensions:1.3.0-alpha02'

// Barcode and OCR
implementation 'com.google.android.gms:play-services-mlkit-text-recognition:18.0.2'
implementation 'com.google.android.gms:play-services-mlkit-barcode-scanning:18.1.0'
```

## V2.2.13

* No API changes

## V2.2.11

### API changes:
* ```startFoodDetection``` now returns Boolean. The function will return false if the registration of the ```foodRecognitionListener``` failed.
* ```stopFoodDetection``` now returns Boolean. The function will return false if the unregistration of the current ```foodRecognitionListener``` failed.
* ```detectFoodIn``` now returns Boolean. The function will return false if the SDK is not configured and it can't run the detection process of the given Bitmap.


## V2.2.9

* No API changes

## V2.2.7

* Renamed the .aar file to determine which Kotlin version it supports.
* Introduced compatiblity .aar that supports Kotlin version 1.4.21.
* New dependency has to be added to build the project:

```groovy
dependencies {
    ...
    implementation 'org.tensorflow:tensorflow-lite-metadata:0.4.0'
}
```

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
