# RecyclerView Implementation Guide

## 1. Add RecyclerView and CardView dependencies

```kotlin
//app-level gradle

implementation "androidx.recyclerview:recyclerview:1.2.1"
// For control over item selection of both touch and mouse driven selection
implementation "androidx.recyclerview:recyclerview-selection:1.1.0"
```

## 2. Create Model Class for RV Data

```kotlin
data class Recipe(val id: Int, val name: String, val description: String, val imageResId: Int)
```

## 3. Create ViewModel for Data Storage/Processing

### 3.1 - Create ViewModel Class

```kotlin
class RecipeViewModel: ViewModel() {
    
}
```

### 3.2 - Create fake data for starters

```kotlin
//inside viewmodel class
    var recipes = mutableListOf<Recipe>()

    init {
        recipes.add(Recipe(1, "Spaghetti Carbonara",
            "Classic spaghetti carbonara with bacon, eggs and Parmesan cheese",
            R.drawable.spaghetti_carbonara))

        recipes.add(Recipe(2, "Fried Chicken",
            "Tasty and crispy fired chicken with the special chickenyaz sauce",
            R.drawable.fried_chicken),
        )

        recipes.add(Recipe(3, "Beef Stroganoff",
            "Hearty beef stroganoff with mushrooms and sour cream",
            R.drawable.beef_stroganoff))

        recipes.add(Recipe(4, "Tomato Soup",
            "Creamy tomato soup with fresh basil", R.drawable.tomato_soup))

        recipes.add(Recipe(5, "Vegetable Curry",
            "Delicious vegetable curry with chickpeas and rice",
            R.drawable.vegetable_curry))

        recipes.add(Recipe(6, "Cheeseburger",
            "Just a well made cheeseburger", R.drawable.cheeseburger))

        recipes.add(Recipe(7 , "Cheese Pizza",
            "Cheesy cheese pizza extra cheese", R.drawable.pizza))
    }
```

### 3.3 - Add XML used in recipe class

This is given to you in the starter code in that case. Take a look at res->drawable

## 4. Add RecyclerView XML in parent XML file

Note: the constraints here are specific to our livecoding. You generally need to configure your own constraints.

Also note the use of `0dp` for the width and height. `0dp` means: **"fill the entire area that your constraints span"**
```xml
    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recipe_list_rv"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintTop_toBottomOf="@id/add_new_recipe_button"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layoutManager="androidx.recyclerview.widget.LinearLayoutManager"/>
```

## 5. Create the Layout of RV Items

### 5.1 - create `recipe_list_item.xml` 

* Right click on `res/layout`
* Select `New->Layout Resource File`
* Give it a name of `recipe_list_item.xml`

### 5.2 - Add views needed in your model data class

Note: you generally need to set your own styling, but we gave you some basic ones for now.

```xml
<ImageView
        android:id="@+id/recipe_image"
        android:layout_width="57dp"
        android:layout_height="97dp"
        android:layout_marginTop="8dp"
        android:layout_marginLeft="8dp"
        android:contentDescription="@string/recipe_image"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:srcCompat="@drawable/unknown_item" />

    <LinearLayout
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        app:layout_constraintStart_toEndOf="@id/recipe_image"
        app:layout_constraintTop_toTopOf="@id/recipe_image"
        android:gravity="top"
        android:layout_marginStart="12dp"
        app:layout_constraintEnd_toEndOf="parent"
        >

        <TextView
            android:id="@+id/recipe_name"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:fontFamily="@font/roboto"
            android:text="@string/name"
            android:textSize="18sp"
            android:textStyle="bold"

            />

        <TextView
            android:id="@+id/recipe_description"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:fontFamily="@font/roboto"
            android:text="@string/description"
            android:textSize="12sp" />

    </LinearLayout>
```

### 5.3 - Modify ConstraintLayout height

It is recommended to set the ConstraintLayout height to `wrap_content` since you do not want a single recipe item to occupy the entire screen!

In `recipe_list_item.xml`:
```xml
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">
```

## 6. Create the RecyclerView Adapter

### 6.1 - Create adapter class `RecipeAdapter` 

```kotlin
class RecipeAdapter(private val recipes: MutableList<Recipe>): RecyclerView.Adapter<RecipeAdapter.RecipeViewHolder>() {
```
Note that we added the dataset of the recyclerview (recipes list) as an argument to the constructor for our adapter class. That way we can pass that list to the adapter from the `RecipeViewModel` instance defined in `MainActivity.kt`

### 6.2 - Create inner `ViewHolder` class in `RecipeAdapter.kt`

```kotlin
    // Provide a direct reference to each of the views within a data item
    // Used to cache the views within the item layout for fast access
    inner class RecipeViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        // Your holder should contain and initialize a member variable
        // for any view that will be set as you render a row
        val recipeNameText: TextView = itemView.findViewById(R.id.recipe_name)
        val recipeDescriptionText: TextView = itemView.findViewById(R.id.recipe_description)
        val recipeImage: ImageView = itemView.findViewById(R.id.recipe_image)
```
### 6.3 - Override methods in `RecipeAdapter`

Observe the function comments to understand what they do
```kotlin
//RecipeAdapter.kt

    // When the view holder gets created, inflate it with a layout from an XML file using our friendly neighborhood LayoutInflater
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecipeAdapter.RecipeViewHolder {
        val layoutInflater = LayoutInflater.from(parent.context)
        val view = layoutInflater
            .inflate(R.layout.recipe_list_item, parent, false)
        return RecipeViewHolder(view)
    }
```
```kotlin
//RecipeAdapter.kt

    // Inform the adapter of the number of items to pass to RV
    override fun getItemCount(): Int {
        return recipes.size
    }
```
```kotlin
//RecipeAdapter.kt

    // Tell the adapter what to do when binding the viewholder to the item at a certain position in the list
    override fun onBindViewHolder(holder: RecipeViewHolder, position: Int) {
        val recipe = recipes[position]
        val recipeName = holder.recipeNameText
        val recipeDescription = holder.recipeDescriptionText
        val recipeImage = holder.recipeImage

        recipeName.text = recipe.name
        recipeDescription.text = recipe.description
        recipeImage.setImageResource(recipe.imageResId)
    }

```

## 7. Initialize the RecyclerView in Parent Class

In our case, the parent class is `MainActivity.kt`

### 7.1 - Initialize ViewModel

```kotlin
//inside MainActivity.kt

private lateinit var viewModel: RecipeViewModel
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    viewModel = ViewModelProvider(this).get(RecipeViewModel::class.java)
}
```

### 7.2 - Initialize RecyclerView & Adapter

```kotlin

private lateinit var viewModel: RecipeViewModel
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    viewModel = ViewModelProvider(this).get(RecipeViewModel::class.java)
    val recyclerView = findViewById<RecyclerView>(R.id.recipe_list_rv)
    val adapter = RecipeAdapter(viewModel.recipes)
    recyclerView.adapter = adapter

}
```

## 8. Adding an item to RV

### 8.1 - Add a function in ViewModel to create a recipe


```kotlin
//RecipeViewModel.kt
var idCounter = 8

/*
.
. init block
.
*/

fun addNewRecipe() {
    recipes.add(Recipe(idCounter, "Recipe #$idCounter", "Another random recipe", R.drawable.unknown_item))
    idCounter++
}
```

### 8.2 - Add onClickListener in MainActivity

```kotlin
//MainActivity.kt
private lateinit var viewModel: RecipeViewModel
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    viewModel = ViewModelProvider(this).get(RecipeViewModel::class.java)
    val recyclerView = findViewById<RecyclerView>(R.id.recipe_list_rv)
    val adapter = RecipeAdapter(viewModel.recipes)
    recyclerView.adapter = adapter


    val addRecipeButton = findViewById<Button>(R.id.add_new_recipe_button)
    addRecipeButton.setOnClickListener {
        viewModel.addNewRecipe()
    }
}
```
### 8.3 - Use LiveData to catch new data and update adapter

* Add gradle dependencies
```gradle
//app-level gradle
implementation 'androidx.lifecycle:lifecycle-extensions:2.2.0'
```

* Create Boolean LiveData in ViewModel

```kotlin
//RecipeViewModel.kt
val _recipesUpdated = MutableLiveData<Boolean>()
val recipesUpdated : LiveData<Boolean>
    get() = _recipesUpdated
    
init {
    _recipesUpdated.value = false 
    /*
    .
    . rest of init block
    .
    */
}   

fun onListUpdated() {
    _recipesUpdated.value = false
}
```

```kotlin 
//MainActivity.kt
viewModel.recipesUpdated.observe(this, Observer { updated ->
    if (updated) {
        adapter.notifyDataSetChanged()
        viewModel.onListUpdated()
    }
})
```

## 9. Decorating RecyclerView

### 9.1 Add divider between items
```kotlin
//MainActivity.kt
val itemDecoration: RecyclerView.ItemDecoration =
    DividerItemDecoration(this, DividerItemDecoration.VERTICAL)
recyclerView.addItemDecoration(itemDecoration)
```

<!-- ### 9.2 Add Animation

Add gradle dependency
```gradle
//app-level gradle
implementation 'jp.wasabeef:recyclerview-animators:4.0.2'
```

Add slide animation
```kotlin
//MainActivity.kt
recyclerView.itemAnimator = SlideInUpAnimator()
``` -->

## 10. Implementing Item Click

### 10.1 - Creating the clicking interface

* Add the following to `RecipeAdapter.kt`

```kotlin
// Define the listener interface
interface OnItemClickListener {
    fun onItemClick(itemView: View?, position: Int)
}

// Define listener member variable
private lateinit var listener: OnItemClickListener

// Define the method that allows the parent activity or fragment to define the listener
fun setOnItemClickListener(listener: OnItemClickListener) {
    this.listener = listener
}
```

### 10.2 - initialize clicklistener in ViewHolder

* Add the following to `RecipeAdapter.RecipeViewHolder` in `RecipeAdapter.kt`

```kotlin
init {
    // Setup the click listener
    itemView.setOnClickListener {
    // Triggers click upwards to the adapter on click
    val position = absoluteAdapterPosition
    if (position != RecyclerView.NO_POSITION) {
        listener.onItemClick(itemView, position)
        }
    }
}
```

### 10.3 - Attach onClickListener to adapter in `MainActivity.kt`

```kotlin
adapter.setOnItemClickListener(object : RecipeAdapter.OnItemClickListener {
    override fun onItemClick(itemView: View?, position: Int) {
        val name = viewModel.recipes[position].name
        Toast.makeText(this@MainActivity, "$name was clicked!", Toast.LENGTH_SHORT).show()
        }
    })
```



