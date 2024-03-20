## Camera recognition

* ```lookupNameFor```, ```lookupPassioAttributesForName``` and ```lookupAllDescendantsFor``` have been removed since these function were querying the local database
* ```lookupPassioAttributesFor``` has been replaced with ```fetchFoodItemForPassioID``` and now returns a *PassioFoodItem* result
* ```fetchPassioIDAttributesForBarcode``` and ```fetchPassioIDAttributesForPackagedFood``` have been replaced with ```fetchFoodItemForProductCode``` that now returns a *PassioFoodItem* result
* Alternative results in the form of *parents*, *siblings* and *children* have been replaced with two types of alternatives:
a) Every detected candidate will have a list of ```alternatives: List<DetectedCandidate>```. These alternatives represent contextually similar foods. Example: If the DetectadeCandidate would be "milk", than the list of alternatives would include items like "soy milk", "almond milk", etc. 
b) Also, the interface ```FoodRecognitionListener``` might return multiple detected candidates, ordered by confidence. These multiple candidates represent the top result that our recognition system is predicting, but also other results that are visually similar to the top result. Example: If the first result in the list of ```detectedCandidates``` is "coffee", there might be more results in the list that are visually simillar to coffee like "coke", "black tea", "chocolate milk", etc.
* ```lookupIconFor``` has been deprecated, the correct function to use is ```lookupIconsFor```

## Search
* ```searchForFood``` now returns *PassioSearchResult* and a list of search options. The PassioSearchResult represent a specific food item associated with the search term, while search options provide a list of possible suggested search terms connected to the input term.

```kotlin
data class PassioSearchResult(
    val foodName: String,
    val brandName: String,
    val iconID: PassioID,
    val score: Double,
    val scoredName: String,

    val labelId: String,
    val type: String,
    val resultId: String,

    val nutritionPreview: PassioSearchNutritionPreview
)

data class PassioSearchNutritionPreview(
    val calories: Int,
    val servingUnit: String,
    val servingQuantity: Double,
    val servingWeight: String
)
```
* To Fetch a food item for a specific search result use ```fetchSearchResult```

## Data models

**PassioIDAttributes** as a top level representation of a food item has been replaced with ```PassioFoodItem```. **PassioFoodItemData** and **PassioFoodRecipe** have also been deprecated.

```kotlin
data class PassioFoodItem(
    val id: String,
    val name: String,
    val details: String,
    val iconId: String,
    val amount: PassioFoodAmount,
    val ingredients: List<PassioIngredient>,
)

data class PassioIngredient(
    val id: String,
    val name: String,
    val iconId: String,
    val amount: PassioFoodAmount,
    val referenceNutrients: PassioNutrients,
    val metadata: PassioFoodMetadata,
) 

data class PassioFoodAmount(
    val servingSizes: List<PassioServingSize>,
    val servingUnits: List<PassioServingUnit>
)

data class PassioNutrients(
    val weight: UnitMass
) {
    fun fat(): UnitMass? 
    fun calories(): UnitEnergy? 
    fun protein(): UnitMass? 
    fun carbs(): UnitMass?
    fun satFat(): UnitMass? 
    fun monounsaturatedFat(): UnitMass? 
    fun polyunsaturatedFat(): UnitMass? 
    fun cholesterol(): UnitMass? 
    fun sodium(): UnitMass?
    fun fibers(): UnitMass?
    fun transFat(): UnitMass? 
    fun sugars(): UnitMass?
    fun sugarsAdded(): UnitMass? 
    fun alcohol(): UnitMass? 
    fun iron(): UnitMass?
    fun vitaminC(): UnitMass?
    fun vitaminA(): Double?
    fun vitaminD(): UnitMass? 
    fun vitaminB6(): UnitMass? 
    fun vitaminB12(): UnitMass? 
    fun calcium(): UnitMass?
    fun potassium(): UnitMass? 
    fun magnesium(): UnitMass? 
    fun phosphorus(): UnitMass? 
    fun sugarAlcohol(): UnitMass? 
    fun vitaminB12Added(): UnitMass? 
    fun vitaminE(): UnitMass? 
    fun vitaminEAdded(): UnitMass? 
    fun iodine(): UnitMass? 
}
```

* To migrate from the old data structure to the new one this snipped of code will be 

```kotlin
private fun attrsToFoodItem(attrs: PassioIDAttributes): PassioFoodItem {
    val ingredients = mutableListOf<PassioIngredient>()
    if (attrs.passioFoodItemData != null) {
        val ingredient = foodDataToIngredient(attrs.passioFoodItemData!!)
        ingredients.add(ingredient)
    } else {
        val recipeIngredients = attrs.passioFoodRecipe!!.foodItems.map {
            foodDataToIngredient(it)
        }
        ingredients.addAll(recipeIngredients)
    }
    val amount = if (attrs.passioFoodItemData != null) {
        PassioFoodAmount(
            attrs.passioFoodItemData!!.servingSizes,
            attrs.passioFoodItemData!!.servingUnits
        ).apply {
            selectedUnit = attrs.passioFoodItemData!!.selectedUnit
            selectedQuantity = attrs.passioFoodItemData!!.selectedQuantity
        }
    } else {
        PassioFoodAmount(
            attrs.passioFoodRecipe!!.servingSizes,
            attrs.passioFoodRecipe!!.servingUnits
        ).apply {
            selectedUnit = attrs.passioFoodRecipe!!.selectedUnit
            selectedQuantity = attrs.passioFoodRecipe!!.selectedQuantity
        }
    }
    return PassioFoodItem(
        attrs.passioID,
        attrs.name,
        "",
        attrs.passioID,
        amount,
        ingredients
    )
}
private fun foodDataToIngredient(data: PassioFoodItemData): PassioIngredient {
    val amount = PassioFoodAmount(
        data.servingSizes,
        data.servingUnits
    ).apply {
        selectedUnit = data.selectedUnit
        selectedQuantity = data.selectedQuantity
    }
    data.selectedUnit = "gram"
    data.selectedQuantity = 100.0
    val nutrients = PassioNutrients(
        data.totalFat(),
        data.totalSatFat(),
        data.totalMonounsaturatedFat(),
        data.totalPolyunsaturatedFat(),
        data.totalProtein(),
        data.totalCarbs(),
        data.totalCalories(),
        data.totalCholesterol(),
        data.totalSodium(),
        data.totalFibers(),
        data.totalTransFat(),
        data.totalSugars(),
        data.totalSugarsAdded(),
        data.totalAlcohol(),
        data.totalIron(),
        data.totalVitaminC(),
        data.totalVitaminD(),
        data.totalVitaminB6(),
        data.totalVitaminB12(),
        data.totalVitaminB12Added(),
        data.totalVitaminE(),
        data.totalVitaminEAdded(),
        data.totalIodine(),
        data.totalCalcium(),
        data.totalPotassium(),
        data.totalMagnesium(),
        data.totalPhosphorus(),
        data.totalSugarAlcohol(),
        data.totalVitaminA()
    )
    val metadata = PassioFoodMetadata(
        data.foodOrigins,
        data.barcode,
        data.ingredientsDescription,
        data.tags
    )
    return PassioIngredient(
        data.passioID,
        data.name,
        data.passioID,
        amount,
        nutrients,
        metadata
    )
}
```

<sup>Copyright 2023 Passio Inc</sup>
