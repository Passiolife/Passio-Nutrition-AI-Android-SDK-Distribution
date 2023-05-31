# Passio SDK API

## Version 2.2.11

```kotlin


/**
 * The identifier of a food item located in the nutritional
 * database.
 */
typealias PassioID = String

/**
 * String that represents the code of barcode types FORMAT_EAN_13,
 * FORMAT_EAN_8, FORMAT_UPC_E and FORMAT_UPC_A.
 */
typealias Barcode = String

/**
 * String that represents the barcode of a packaged food that is
 * recognized using the image of the packaging.
 */
typealias PackagedFoodCode = String

internal typealias VotedCandidate = ObjectDetectionCandidate

/**
 * Data class that represents the results of the SDK's detection process.
 */
data class FoodCandidates(
    /**
     * Result of the food detection process.
     */
    val detectedCandidates: List<DetectedCandidate>? = null,
    /**
     * Result of the barcode detection process.
     */
    val barcodeCandidates: List<BarcodeCandidate>? = null,
    /**
     * Results of the ocr detection process.
     */
    val packagedFoodCandidates: List<PackagedFoodCandidate>? = null
)

/**
 * Class that represents the results of the food detection process.
 */
open class DetectedCandidate(
    /**
     * The id used to query the nutritional database.
     */
    val passioID: PassioID,
    /**
     * Percentage of the neural network confidence level. Ranges from 0f - 1f or 0% to 100%.
     */
    val confidence: Float,
    /**
     * Relative coordinates of the object detected on an image. Using
     * [PassioSDK.boundingBoxToViewTransform] will determine the correct coordinates in coordinate
     * system of current view hierarchy.
     */
    val boundingBox: RectF,
    /**
     * Represents a part of the original image that the classification process has been ran on.
     */
    val croppedImage: Bitmap?
)

/**
 * Represents the result of the classification process.
 */
open class ClassificationCandidate(var passioID: PassioID, val confidence: Float)

/**
 * Represents the result of the packaged food detection
 * process.
 */
data class PackagedFoodCandidate(
    val packagedFoodCode: PackagedFoodCode,
    val confidence: Float
)

/**
 * Represents the result of the object detection process.
 */
class ObjectDetectionCandidate(
    passioID: PassioID,
    confidence: Float,
    val boundingBox: RectF
) : ClassificationCandidate(passioID, confidence) {

    override fun toString(): String {
        return "passioID: $passioID, " +
                "confidence: $confidence, " +
                "boundingBox: ${boundingBox.toShortString()}"
    }
}

/**
 * Data class that represents the results of the barcode detection process.
 */
data class BarcodeCandidate(
    /**
     * Barcode string used to fetch nutritional information using the
     * [PassioSDK.fetchPassioIDAttributesForBarcode] method.
     */
    val barcode: Barcode,
    /**
     * The bounding box of the detected barcode, in absolute coordinates.
     */
    val boundingBox: RectF
)

/**
 * Interface that serves as a callback for the [PassioSDK.startFoodDetection] method.
 */
interface FoodRecognitionListener {

    /**
     * Callback method for the food detection process.
     *
     * @param candidates top level object that represents all of the detection results based on the
     *        configuration of the detection session.
     * @param image that represents the camera frame that the recognition was ran on.
     * @param nutritionFacts recognized nutrition facts for the camera frame.
     */
    fun onRecognitionResults(
        candidates: FoodCandidates,
        image: Bitmap?,
        nutritionFacts: PassioNutritionFacts?
    )
}

/**
 * Callback interface used to receive information about
 * the SDK's configuration process.
 */
interface PassioStatusListener {
    /**
     * Every time the SDK's configuration process changes
     * a state, a new event will be emitted with the
     * current [PassioStatus].
     */
    fun onPassioStatusChanged(status: PassioStatus)

    /**
     * Will be called once all of the files are downloaded.
     * This doesn't mean that the SDK will immediately run
     * the newly downloaded files.
     */
    fun onCompletedDownloadingAllFiles(fileUris: List<Uri>)

    /**
     * Signals the completion of the download process for a
     * single files. This method also informs how many files
     * are still left in the download queue.
     */
    fun onCompletedDownloadingFile(fileUri: Uri, filesLeft: Int)

    /**
     * If a certain file cannot be downloaded, [onDownloadError]
     * will be invoked with the download error message attached.
     */
    fun onDownloadError(message: String)
}


private const val MINIMUM_CONFIDENCE_TF_OD_API = 0.33f

/**
 * Main access point of the SDK. Defines all of the SDK's
 * main capabilities. The API of the SDK is designed as a
 * singleton with the [instance] containing the concrete
 * implementation of the interface. The SDK is design to
 * be called from the main thread. The execution of most
 * of its functions will be delegated to an internal
 * thread pool, but the resulting callback will always be
 * invoked on the main thread.
 */
@Keep
interface PassioSDK {

    /**
     * Defines the processing speed of the SDK's camera
     * recognition system.
     * ONE - indicates that one frame will be analyzed
     *       every second. If the analysis time of that
     *       frame exceeds time of a second, the next
     *       frame will be analyzed only after the
     *       previous one has been complete.
     * TWO - indicates that one frame will be analyzed
     *       every 500 milliseconds. This is the default
     *       behaviour of the SDK.
     * MAX - there is no possible waiting period between
     *       the analysis of two frames. As soon as the
     *       recognition system is done with the previous
     *       frame, the analysis of the next one is called.
     *       This mode might is computational heavy
     *       and might take up a lot of processing power
     *       of the system.
     */
    enum class FramesPerSecond {
        ONE,
        TWO,
        MAX
    }

    @Keep
    companion object {
        val BARCODE_PREFIX = PassioIDEntityType.barcode.name
        val PACKAGED_FOOD_PREFIX = PassioIDEntityType.packagedFoodCode.name
        const val BKG_PASSIO_ID = "BKG0001"

        private var internalInstance: PassioSDK? = null

        /**
         * Use this singleton to access the Passio SDK.
         */
        @JvmStatic
        val instance: PassioSDK
            get() {
                if (internalInstance == null) {
                    internalInstance = PassioSDKImpl()
                }
                return internalInstance!!
            }

        /**
         * The value of confidence of recognition results below which results won't be returned
         * to the user. The default value is 0.5.
         */
        fun minimumConfidence() = MINIMUM_CONFIDENCE_TF_OD_API
    }

    /**
     * Initializes the SDK with the given [passioConfiguration]. See [PassioConfiguration] for more
     * information on the different types of SDK configuration. The initialization process includes
     * downloading or reading a cached version of the license and loading the models into the
     * TensorFlowLite runner. This process is being executed on a background thread, but the
     * callback with the result of the configuration process will be called on the main thread. You
     * may call [startCamera] or [startFoodDetection] before this method, there is no need to wait
     * the [onComplete] callback.
     *
     * @param passioConfiguration the input configuration for the SDK.
     * @param onComplete the callback that will be called at the end of the configuration process
     *        with the result in the form of the {@link ai.passio.passiosdk.core.config.PassioStatus}
     */
    fun configure(
        passioConfiguration: PassioConfiguration,
        onComplete: (status: PassioStatus) -> Unit
    )

    /**
     * Initializes the SDK with the given [developerKey] and the correct models from the assets
     * folder. Developer key must be 42 character key generated by Passio Inc. For obtaining this
     * key contact support@passiolife.com. The initialization process includes downloading or
     * reading a cached version of the license, loading the models into the TensorFlowLite runner.
     * This process is being executed on a background thread, but the calls to the error or success
     * lambdas will be called on the main thread. You may call [startCamera] before this method,
     * there is no need to wait the [onSDKReady] call.
     *
     * @param context needed to download and store licenses.
     * @param developerKey 42 long string key that servers as a license.
     * @param onSDKError lambda that will be called if there was an error during the initialization
     *        process of the SDK. The errorMessage will provide detailed information about the
     *        error.
     * @param onSDKReady lambda that will be called if the initialization process completed as
     *        successful.
     */
    fun configure(
        appContext: Context,
        key: String,
        onSDKError: (errorMessage: String) -> Unit,
        onSDKReady: () -> Unit = {}
    )

    /**
     * Checks whether the SDK initialization ran to completion. The initialization process starts
     * with the [configure] function.
     *
     * @return true if the SDK has initialized without errors, false instead.
     */
    fun isSDKReady(): Boolean

    /**
     * Starts the camera preview with the given [PassioCameraViewProvider]. Using CameraX
     * (https://developer.android.com/training/camerax), the camera system will start rendering
     * the frames onto the PreviewView. Also this method binds the camera with the lifecycle
     * provider of the CameraViewProvider. When that lifecycle provider calls onStart() the
     * camera preview will start and when it calls onStop() the camera preview will stop and
     * start the shutdown process. [android.view.Surface.ROTATION_]
     *
     * @param passioCameraViewProvider provides the PreviewView to render the camera frames, the
     *        lifecycle holder which will start and stop the camera, and also the context provider
     *        needed to open the camera.
     * @param displayRotation if the orientation changes are enabled, use this parameter to notify
     *        the camera of the display rotation. Accepted values are: Surface.ROTATION_0,
     *        Surface.ROTATION_90, Surface.ROTATION_180, Surface.ROTATION_270.
     * @param tapToFocus set true to enable a tap listener on the passioCameraViewProvider's
     *        PreviewView that will start a manual focus operation.
     */
    fun startCamera(
        passioCameraViewProvider: PassioCameraViewProvider,
        displayRotation: Int = 0,
        @CameraSelector.LensFacing cameraFacing: Int = CameraSelector.LENS_FACING_BACK,
        tapToFocus: Boolean = false
    )

    fun startCamera(
        viewProvider: PassioCameraViewProvider,
        configurator: PassioCameraConfigurator
    )

    fun stopCamera()

    /**
     * Turns the device's flashlight on or off.
     *
     * @param enabled if enabled is set to true, it will turn the flashlight on. If the enabled
     *        parameter is the to false, it will turn the flashlight off.
     */
    fun enableFlashlight(enabled: Boolean)

    /**
     * Sets the time between each analyzed frame. For example, it the [timeForAnalysis] is set to
     * 500L (0.5 seconds), the camera engine will analyze 2 frames per second. If set to 0L, the
     * camera will analyze frames as fast as possible. If the time is smaller than the time it takes
     * to analyze and process the frame, the SDK will work as if set to 0L. The default is set to
     * 1000L (1 second).
     *
     * @param timeForAnalysis the time passed between two frames that are being analyzed.
     */
    fun runFrameEvery(timeForAnalysis: Long)

    /**
     * Finalizes the PassioSDK instance. Deallocates the memory reserved for TFLite models,
     * transformation matrices and shuts down all the background queues that power the SDK. To run
     * the SDK again, [configure] must be called.
     */
    fun shutDownPassioSDK() {
        internalInstance = null
    }

    /**
     * Registers a listener for the food detection process. The results will be returned at a
     * frequency defined in the [FoodDetectionConfiguration.framesPerSecond] field.
     *
     * @param foodRecognitionListener the interface that serves as a callback for the recognition
     *        results.
     * @param detectionConfig an object that defines what to recognize, how often, and other
     *        recognition properties.
     *
     */
    fun startFoodDetection(
        foodRecognitionListener: FoodRecognitionListener,
        detectionConfig: FoodDetectionConfiguration = FoodDetectionConfiguration()
    ): Boolean

    /**
     * Stops the food detection process. After this method is called no results should be delivered
     * to a previously registered [FoodRecognitionListener].
     */
    fun stopFoodDetection(): Boolean

    /**
     * Runs object detection on a given [Bitmap]. The process will run on the same background
     * thread as object detection process from the [startFoodDetection] method. The results will
     * be delivered on the main thread. [viewWidth] and [viewHeight] are used to calculate the
     * correct bounding boxes of the recognized items.
     *
     * @param bitmap image to analyze
     * @param onDetectionCompleted callback for the results of the detection process.
     */
    fun detectFoodIn(
        bitmap: Bitmap,
        foodDetectionConfiguration: FoodDetectionConfiguration? = null,
        onDetectionCompleted: (candidates: FoodCandidates?) -> Unit
    ): Boolean

    fun lookupIconFor(
        context: Context,
        passioID: PassioID,
        iconSize: IconSize = IconSize.PX90,
        type: PassioIDEntityType = PassioIDEntityType.item,
    ): Pair<Drawable, Boolean>

    /**
     * For a given [PassioID] returns the corresponding image from the SDK's asset folder.
     *
     * @param context used to open assets
     * @param passioID key to find the image
     * @return if the [passioID] is valid returns the [Drawable] from the assets. If not returns null.
     */
    fun fetchIconFor(
        context: Context,
        passioID: PassioID,
        iconSize: IconSize = IconSize.PX90,
        callback: (drawable: Drawable?) -> Unit
    )

    /**
     * For a given [PassioID] returns the name of corresponding food item.
     *
     * @param passioID the ID of the food item
     * @return name of the food item if there is a an object in the nutritional database with the
     *         provided passioID, null otherwise.
     */
    fun lookupNameFor(passioID: PassioID): String?

    /**
     * For a given [PassioID] returns the [PassioIDAttributes] of that food item.
     *
     * @param passioID the ID of the food item
     * @return attributes object that represents all of the nutritional data the SDK has on a food
     *         item corresponding to the passioID, null otherwise.
     */
    fun lookupPassioAttributesFor(passioID: PassioID): PassioIDAttributes?

    /**
     * For a given food item [name] returns the [PassioIDAttributes]
     * of that food item.
     *
     * @param name of the food item
     * @return attributes object that represents all of the nutritional data the SDK
     *         item corresponding to the food item name, null otherwise.
     */
    fun lookupPassioAttributesForName(name: String): PassioIDAttributes?

    /**
     * For a given [passioID] returns a list that contains the subtree of the food hierarchy that
     * has the food item with the given [passioID] as a root.
     *
     * @param passioID the ID of the food item
     * @return list that represents the subtree of the given [passioID], null otherwise.
     */
    fun lookupAllDescendantsFor(passioID: PassioID): List<PassioID>?

    /**
     * Search functions that uses the given [byText] to cross reference
     * the names of the food items in the nutritional database.
     *
     * @param byText the string used to query the database
     * @return list of all of the [PassioID]s with the corresponding
     *         names of the food items.
     */
    fun searchForFood(
        byText: String,
        callback: (result: List<Pair<PassioID, String>>) -> Unit
    )

    /**
     * For a given [Barcode] creates a networking call to Passio's
     * backend that tries to fetch the corresponding [PassioIDAttributes].
     *
     * @param barcode the barcode string of the packaged food.
     * @param onAttributesFetched callback function that will return the
     *                            PassioIDAttributes if that packaged food
     *                            is enlisted on Passio's backend.
     */
    fun fetchPassioIDAttributesForBarcode(
        barcode: Barcode,
        onAttributesFetched: (passioIDAttributes: PassioIDAttributes?) -> Unit
    )

    fun fetchPassioIDAttributesForPackagedFood(
        packagedFoodCode: PackagedFoodCode,
        onAttributesFetched: (passioIDAttributes: PassioIDAttributes?) -> Unit
    )

    /**
     * If not null, the [PassioStatusListener] will provide callbacks
     * when the internal state of the SDK's configuration process changes.
     * Passing null will unregister the listener.
     */
    fun setPassioStatusListener(statusListener: PassioStatusListener?)

    /**
     * Transforms the bounding box of the camera frame to the coordinates
     * of the preview view where it should be displayed.
     *
     * @param boundingBox the bounding box from the ObjectDetectionCandidate.
     * @param viewWidth the width of the camera preview view.
     * @param viewHeight the height of the camera preview view.
     * @param displayAngle the rotation of the camera preview view.
     * @param barcode does the bounding box belong to the barcode candidate.
     * @return the bounding box within the coordinate system of the camera view.
     */
    fun boundingBoxToViewTransform(
        boundingBox: RectF,
        viewWidth: Int,
        viewHeight: Int,
        displayAngle: Int = 0,
        barcode: Boolean = false
    ): RectF
}
```

<sup>Copyright 2022 Passio Inc</sup>
