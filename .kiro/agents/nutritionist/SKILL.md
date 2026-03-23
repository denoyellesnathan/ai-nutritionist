# Skill: Edamam Recipe Search API v2

## Overview
Search millions of web recipes via the Edamam Recipe Search API v2. Use this to find new meal options that match the user's CONFIG.md preferences (cook time, protein goals, dietary restrictions, liked foods).

## Authentication
Credentials are stored as environment variables:
- `EDAMAM_APP_ID`
- `EDAMAM_APP_KEY`

Every request requires the header: `Edamam-Account-User: sage`

To load credentials in bash (since they live in `~/.zshrc`):
```bash
export $(grep 'EDAMAM_' ~/.zshrc | xargs)
```

---

## Endpoints

### 1. Recipe Search
```
GET https://api.edamam.com/api/recipes/v2?type=public
```

### 2. Recipe by ID
```
GET https://api.edamam.com/api/recipes/v2/{id}?type=public
```
Look up a specific recipe. The `id` comes from the recipe URI (last segment). Useful for re-fetching a recipe seen earlier.

### 3. Recipes by URI (Batch)
```
GET https://api.edamam.com/api/recipes/v2/by-uri?uri=RECIPE_URI
```
Batch lookup up to 20 recipes by URI. Repeat `uri` param for multiple.

### 4. Shopping List
```
POST https://api.edamam.com/api/shopping-list/v2
```
Generate an aggregated shopping list from recipe URIs. Send a JSON body with recipe entries and servings — returns consolidated ingredient quantities. See Shopping List section below.

---

## Required Parameters (Search)
| Param | Description |
|-------|-------------|
| `type` | Always `public` |
| `app_id` | `$EDAMAM_APP_ID` |
| `app_key` | `$EDAMAM_APP_KEY` |
| `q` | Search query (e.g., `chicken pasta`, `quick tacos`). Required if no other filter is specified. |

## Filters

### Core Filters (use on most searches)
| Param | Values | Notes |
|-------|--------|-------|
| `mealType` | `Breakfast`, `Lunch`, `Dinner`, `Snack`, `Teatime` | Match to user's meal categories |
| `diet` | `balanced`, `high-protein`, `high-fiber`, `low-carb`, `low-fat`, `low-sodium` | Use `high-protein` for this household |
| `health` | `alcohol-free`, `peanut-free`, `gluten-free`, `dairy-free`, `vegan`, `vegetarian`, `paleo`, `keto-friendly`, etc. | Apply any from CONFIG.md restrictions |
| `time` | `1-30` | Cook time in minutes — use `1-30` for this household |
| `excluded` | Food name to exclude (repeat param for multiple) | Use `excluded=broccoli` to respect household preferences |
| `ingr` | `MIN+`, `MIN-MAX`, or `MAX` (integer) | Filter by ingredient count. Use `ingr=3-10` to keep recipes simple |

### Refinement Filters
| Param | Values | Notes |
|-------|--------|-------|
| `dishType` | `Main course`, `Pasta`, `Sandwiches`, `Salad`, `Soup`, `Side dish`, `Biscuits and cookies`, `Bread`, `Cereals`, `Desserts`, `Drinks`, `Pancake`, `Starter`, `Sweets` | Narrow by dish category |
| `cuisineType` | `American`, `Asian`, `British`, `Caribbean`, `Central Europe`, `Chinese`, `Eastern Europe`, `French`, `Greek`, `Indian`, `Italian`, `Japanese`, `Korean`, `Mediterranean`, `Mexican`, `Middle Eastern`, `Nordic`, `South American`, `South East Asian` | Filter by cuisine |
| `calories` | `MIN-MAX` (e.g., `300-700`) | Filter by calorie range per serving |
| `nutrients[PROCNT]` | `MIN+`, `MIN-MAX`, or `MAX` (float) | Protein in grams |
| `nutrients[FAT]` | same format | Fat in grams |
| `nutrients[CHOCDF]` | same format | Carbs in grams |
| `nutrients[FIBTG]` | same format | Fiber in grams |
| `nutrients[SUGAR]` | same format | Sugar in grams |
| `nutrients[NA]` | same format | Sodium in mg |
| `random` | `true` | Returns random selection of 20 results. Good for variety when seeding new weeks |
| `imageSize` | `THUMBNAIL`, `SMALL`, `REGULAR`, `LARGE` | Filter by available image size |
| `co2EmissionsClass` | `A+` through `G` | Carbon footprint filter |

### Field Selection
Use `field` param (repeat for multiple) to limit response fields and reduce payload size:
```
&field=uri&field=label&field=ingredientLines&field=ingredients&field=calories&field=totalNutrients&field=totalTime&field=yield&field=source&field=url&field=mealType&field=dishType&field=cuisineType&field=instructionLines
```

Multiple values for the same param: repeat the param (e.g., `&dishType=Pasta&dishType=Main%20course`).

---

## Example Curl

```bash
export $(grep 'EDAMAM_' ~/.zshrc | xargs); curl -s -H "Edamam-Account-User: sage" \
  "https://api.edamam.com/api/recipes/v2?type=public&q=chicken+tacos&app_id=$EDAMAM_APP_ID&app_key=$EDAMAM_APP_KEY&mealType=Dinner&time=1-30&diet=high-protein&excluded=broccoli&ingr=3-10"
```

---

## Response Structure (Key Fields)

```json
{
  "count": 123,
  "hits": [
    {
      "recipe": {
        "uri": "http://www.edamam.com/ontologies/edamam.owl#recipe_abc123",
        "label": "Recipe Name",
        "source": "Website Name",
        "url": "https://source-recipe-url.com/...",
        "yield": 4,
        "ingredientLines": ["1 lb chicken breast", "2 cups rice"],
        "ingredients": [
          {
            "text": "1 lb chicken breast",
            "food": "chicken breast",
            "foodId": "food_bdrbb5obd58aweb737pnhb3w5",
            "quantity": 1.0,
            "measure": "pound",
            "weight": 453.6,
            "foodCategory": "Poultry"
          }
        ],
        "instructionLines": [
          "Preheat oven to 375°F.",
          "Season chicken with salt and pepper.",
          "..."
        ],
        "totalTime": 25,
        "calories": 1200,
        "totalNutrients": {
          "PROCNT": { "label": "Protein", "quantity": 80.5, "unit": "g" },
          "FAT": { "label": "Fat", "quantity": 30.2, "unit": "g" },
          "CHOCDF": { "label": "Carbs", "quantity": 100.1, "unit": "g" },
          "FIBTG": { "label": "Fiber", "quantity": 8.2, "unit": "g" },
          "SUGAR": { "label": "Sugars", "quantity": 12.0, "unit": "g" },
          "NA": { "label": "Sodium", "quantity": 800.0, "unit": "mg" }
        },
        "totalDaily": {
          "PROCNT": { "label": "Protein", "quantity": 161.0, "unit": "%" }
        },
        "dietLabels": ["HIGH_PROTEIN"],
        "healthLabels": ["PEANUT_FREE", "TREE_NUT_FREE"],
        "glycemicIndex": 45,
        "inflammatoryIndex": -2.5,
        "co2EmissionsClass": "B",
        "mealType": ["lunch/dinner"],
        "dishType": ["main course"],
        "cuisineType": ["mexican"],
        "tags": ["quick", "easy"]
      },
      "_links": {
        "self": { "href": "https://api.edamam.com/api/recipes/v2/abc123?..." }
      }
    }
  ],
  "_links": {
    "next": { "href": "https://api.edamam.com/api/recipes/v2?..." }
  }
}
```

### Key response notes
- `instructionLines` — cooking steps when available. Not all recipes include this; if absent, fall back to source URL or write simplified steps.
- `uri` — unique recipe identifier. Save this when using the Shopping List API.
- `yield` — number of servings. Divide `calories` and `totalNutrients` by `yield` for per-serving values.
- `ingredients[].foodCategory` — use for grocery list section placement (e.g., "Poultry", "Vegetables", "Dairy").
- `dietLabels` / `healthLabels` — confirm the recipe matches user's goals.

---

## Shopping List API

### Endpoint
```
POST https://api.edamam.com/api/shopping-list/v2?app_id=...&app_key=...
```

### Request Body
```json
{
  "entries": [
    {
      "quantity": 2,
      "measure": "http://www.edamam.com/ontologies/edamam.owl#Measure_serving",
      "item": "http://www.edamam.com/ontologies/edamam.owl#recipe_abc123"
    }
  ]
}
```
- `item` — recipe URI from search results
- `quantity` — number of servings (or omit `measure` for full recipe output)
- `measure` — use the serving URI for per-serving scaling, or omit for full recipe quantity

### Response
Returns aggregated ingredient quantities across all recipes, consolidated by food item. Each entry has `foodId`, `food` (label), and `quantities` with measure URIs.

### When to use
- Could help cross-check or generate grocery lists from recipe URIs
- Currently optional — Sage builds grocery lists manually from ingredient data, which gives more control over formatting and section placement

---

## How Sage Uses This

### Default Filters for This Household
Always apply these unless the user overrides:
- `time=1-30` (cook time preference)
- `diet=high-protein` (nutrition goal)
- `excluded=broccoli` (go easy on broccoli)
- `ingr=3-12` (keep recipes simple)

### When to Search
- User asks for new meal ideas or wants to swap a meal
- New week setup when the user wants fresh options
- User requests a specific type of food (e.g., "find me a quick pasta dinner")

### Search Strategy
1. Build the query from the user's request + defaults above
2. Add `mealType` matching the category being filled
3. Parse the top 3–5 results and present them to the user with:
   - Recipe name
   - Cook time
   - Per-serving macros (calories, protein, fat, carbs) — divide totals by `yield`
   - Ingredient count (simple vs complex indicator)
   - Source URL for full instructions
4. Let the user pick, then run the standard **Propose → Confirm → Execute** change flow

### Writing Recipes from Edamam Results
When adding a recipe to Recipes.md:
- List ingredients from `ingredientLines` (human-readable format)
- Scale to 2 servings if `yield` differs (household size from CONFIG.md)
- If `instructionLines` is present, use those as cooking steps
- If `instructionLines` is absent, include the source URL and write simplified steps based on ingredients and dish type
- Include per-serving macros (calories, protein, carbs, fat)

### Grocery List Integration
Use the `ingredients` array (structured data) to add items to the grocery list:
- Use `foodCategory` to place items in the correct grocery list section
- Consolidate duplicates with existing grocery list items
- Follow the standard checkbox format (`- [ ]`)

---

## Rate Limits
Free tier: 10,000 calls/month. Each search is 1 call. Pagination (`_links.next`) counts as additional calls. Keep searches focused to stay well within limits.

## Pagination
To get more results, follow `_links.next.href`. Only paginate if the first page doesn't have good matches — don't auto-paginate.
