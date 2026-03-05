# Restaurant Industry Analytics — DSC 10 Midterm Project

## Introduction
The restaurant industry is one of the most competitive and dynamic sectors in the American economy. From billion-dollar fast food empires to beloved neighborhood independents, the factors that drive success — sales growth, location, and customer satisfaction — vary widely across different types of restaurants. To better understand these dynamics, this project investigates three datasets sourced from [Restaurant Business Online](https://www.restaurantbusinessonline.com) (RB), a leading media brand providing business intelligence for the commercial restaurant industry. The data was collected in 2022 and reflects fiscal year 2021 sales figures.

#### Research Questions
- Which restaurant segment categories are growing the fastest, and which represent the best investment opportunities?
- What makes a city a "hot food city," and does population size drive food scene quality?
- Is there a relationship between a restaurant's price point and its Yelp rating?

---

## The Data

This project uses three distinct CSV datasets, each offering a different lens on the U.S. restaurant landscape.

#### Top 250 Restaurant Chains (`Top250.csv`)
The 250 largest restaurant chains in the US by 2020 annual sales, sorted in decreasing order. Sales are reported in millions of dollars.

| Column               | Description                                                                 |
|----------------------|-----------------------------------------------------------------------------|
| `'Rank'`             | Chain's ranking by 2020 sales [int]                                        |
| `'Restaurant'`       | Name of the restaurant chain [object]                                      |
| `'Sales'`            | Annual sales in millions of dollars [float]                                |
| `'YOY_Sales'`        | Year-over-year sales change from 2019 to 2020, as a percentage string [object] |
| `'Units'`            | Number of locations [int]                                                  |
| `'YOY_Units'`        | Year-over-year change in number of units [object]                          |
| `'Segment_Category'` | Food and service style classification (e.g., "Quick Service & Burger") [object] |
| `'Category'`         | Broader category classification [object]                                   |
| `'Segment'`          | Segment classification [object]                                            |

<p></p>

#### Future 50 (`Future50.csv`)
The 50 fastest-growing mid-sized restaurant chains in the US, with annual sales between $25M and $50M. Sorted in decreasing order of 2021 sales.

| Column         | Description                                                                 |
|----------------|-----------------------------------------------------------------------------|
| `'Rank'`       | Chain's ranking by 2021 sales [int]                                        |
| `'Restaurant'` | Name of the restaurant chain [object]                                      |
| `'Sales'`      | Annual sales in millions of dollars [float]                                |
| `'YOY_Sales'`  | Year-over-year sales change, as a percentage string [object]               |
| `'Location'`   | City and state where the chain is headquartered [object]                   |

<p></p>

#### Independents 100 (`Independents100.csv`)
The 100 highest-grossing independent restaurants (fewer than 5 locations) in the US. Sales are reported in dollars (not millions).

| Column           | Description                                                                 |
|------------------|-----------------------------------------------------------------------------|
| `'Rank'`         | Restaurant's ranking by 2021 sales [int]                                   |
| `'Restaurant'`   | Name of the restaurant [object]                                            |
| `'Sales'`        | Annual sales in dollars [float]                                            |
| `'City'`         | City where the restaurant is located [object]                              |
| `'State'`        | State where the restaurant is located [object]                             |
| `'Average Check'`| Average amount spent per customer per visit, in dollars [float]            |
| `'Meals Served'` | Total number of meals served in 2021 [int]                                 |

<p></p>

#### Yelp Ratings (`Yelp.csv`)
Yelp ratings collected in January 2023 for the 100 restaurants in the Independents 100 dataset.

| Column         | Description                                                                 |
|----------------|-----------------------------------------------------------------------------|
| `'Restaurant'` | Name of the restaurant [object]                                            |
| `'City'`       | City where the restaurant is located [object]                              |
| `'Rating'`     | Yelp star rating (3.0 – 5.0, in 0.5 increments) [float]                   |

<p></p>

---

## Part 1 — Chains ⛓️

This section investigates the `Top250` dataset to understand which restaurant segment categories dominate by sales and which showed the strongest growth during the 2019–2020 period.

#### Data Cleaning & Feature Engineering
The `chain_restaurants` DataFrame was first trimmed to only the relevant columns (`Rank`, `Restaurant`, `Sales`, `YOY_Sales`, `Segment_Category`), creating a leaner `chains` DataFrame.

Since `YOY_Sales` values were stored as percentage strings (e.g., `'4.80%'`), a custom function `percent_str_to_prop` was written to convert them into numerical proportions. This enabled proper sorting and mathematical operations on the column.

A `Sales_2019` column was then back-calculated using the formula:

```
Sales_2019 = Sales / (1 + YOY_Sales)
```

Finally, a 5-tier `Growth_Category` classification system was engineered using a custom function applied to the converted `YOY_Sales` column:

| Growth Category | Interpretation  | Year-over-Year Sales      |
|-----------------|-----------------|---------------------------|
| 5               | Rapid increase  | ≥ 10%                     |
| 4               | Steady increase | [2.5%, 10%)               |
| 3               | Stagnant        | [-2.5%, 2.5%)             |
| 2               | Steady decrease | [-10%, -2.5%)             |
| 1               | Rapid decrease  | < -10%                    |

<p></p>

#### Key Findings

**Segment Categories by Average Sales**
After filtering to only segment categories with average sales over $1 billion, Quick Service dominated the top spots:

| Segment_Category              | Average Sales ($M) |
|-------------------------------|--------------------|
| Quick Service & Burger        | 4550.94            |
| Quick Service & Mexican       | 4178.00            |
| Quick Service & Coffee Cafe   | 3243.44            |

The average sales gap between Quick Service and non-Quick Service chains was over **$2,100M**.

**Growth Category Distribution**
Only **~12%** of the Top 250 chains achieved a Growth Category of 5 (rapid increase ≥10% YOY), led by chicken and delivery-friendly concepts. Growth Category 1 (rapid decline) was the least common, suggesting most major chains held relatively steady despite the pandemic year.

**Best Segments to Invest In**
Ranking all 20 segment categories by mean growth category score revealed three clear frontrunners for capital allocation:

1. Quick Service & Chicken
2. Quick Service & Pizza
3. Quick Service & Mexican

---

## Part 2 — Cities 🌆

This section uses the `Future50` and `Independents100` datasets to identify "hot food cities" — cities with a meaningful presence on both lists.

#### Data Cleaning
The `Future50` dataset's `Location` column contained city and state together (e.g., `"New York, NY"`). A custom function `split_state_city` was applied to extract only the city name, creating a new `City` column.

Restaurant names in `independents` sometimes included location information in parentheses (e.g., `"Maple & Ash (Chicago)"`). A `remove_location` function was written to strip these suffixes before merging with Yelp data.

#### Defining "Food Hotness"
A city's **food hotness** score was defined as the total number of its restaurants appearing on both the Independents 100 and Future 50 lists combined. A **hot food city** must appear on both lists.

| City          | Independents 100 | Future 50 | Food Hotness |
|---------------|------------------|-----------|--------------|
| Chicago       | 16               | 1         | 17           |
| New York      | 10               | 2         | 12           |
| Orlando       | 3                | 2         | 5            |
| Atlanta       | 2                | 2         | 4            |
| Houston       | 2                | 1         | 3            |
| Austin        | 2                | 1         | 3            |
| West Hollywood| 1                | 1         | 2            |

<p></p>

#### Key Findings

**Foodiest City Per Capita**
After normalizing food hotness scores against 2021 population data (sourced from `populations.csv`), **West Hollywood** emerged as the foodiest city per capita — a surprising result given that Chicago and New York dominate in raw count.

**Most Expensive Dining Cities**
Filtering to cities with at least 3 independent restaurants and computing the median average check revealed:

| City        | Median Average Check |
|-------------|----------------------|
| Las Vegas   | $129.50              |
| Miami Beach | $129.00              |
| Chicago     | $97.00               |

**Price vs. Volume**
A scatter plot of `Average Check` vs. `Meals Served` across the Independents 100 confirmed a negative correlation: higher-priced restaurants generally serve significantly fewer customers, consistent with the exclusivity premium of fine dining.

---

## Part 3 — Stars ⭐️

This section merges the `Independents100` and `Yelp` datasets to explore the relationship between customer ratings, price, and geography.

#### Data Cleaning & Merging
Before merging, restaurant names with location suffixes in parentheses were cleaned using `remove_location`. Both DataFrames were then merged on both `Restaurant` and `City` to correctly handle duplicate restaurant names across different cities (e.g., two restaurants named "Nobu" in different cities).

#### Key Findings

**Ratings Distribution**
The distribution of Yelp ratings across the Independents 100 was right-skewed toward the higher end, with most restaurants clustered between 4.0 and 4.5 stars. Only 5 restaurants received the minimum rating of 3.0 stars, and these still averaged **$87 per check** — demonstrating that high sales do not guarantee high customer satisfaction.

**Price Correlates with Rating**
A bar chart of median average check by Yelp rating confirmed a clear positive trend:

| Yelp Rating | Median Average Check |
|-------------|----------------------|
| 3.0         | ~$50                 |
| 3.5         | ~$78                 |
| 4.0         | ~$92                 |
| 4.5         | ~$93                 |

**State-Level Quality**
Grouping by state and computing the proportion of restaurants with at least 4 stars revealed significant geographic variation in perceived dining quality — pointing to regional differences in diner expectations and culinary culture independent of raw sales figures.

---

## Methods & Technical Approach

This project was completed entirely without Python loops, leveraging vectorized operations, functional transformations, and relational joins throughout.

| Technique                  | Application                                                                 |
|----------------------------|-----------------------------------------------------------------------------|
| ETL Pipeline               | Loading, trimming, and restructuring three source datasets                 |
| String Parsing             | Converting `YOY_Sales` percentage strings to numerical proportions         |
| Feature Engineering        | Back-calculating `Sales_2019`; creating `Growth_Category` classifier       |
| `.apply()` & lambda        | Applying custom functions element-wise without loops                       |
| GroupBy Aggregation        | Computing mean sales, median check, and growth scores by segment and city  |
| Multi-key DataFrame Join   | Merging `independents` and `ratings` on both `Restaurant` and `City`       |
| Per-Capita Normalization   | Dividing food hotness by 2021 population to find the foodiest city         |
| Density Histograms         | Visualizing the distribution of Yelp ratings with custom bin boundaries    |
| Scatter Analysis           | Exploring the relationship between average check and meals served          |

**Libraries used:** `babypandas`, `numpy`, `matplotlib`

---

## Website
A data visualization site summarizing the project findings is live at:

**[sammyg82.github.io/restaurant-analytics](https://sammyg82.github.io/restaurant-analytics)**