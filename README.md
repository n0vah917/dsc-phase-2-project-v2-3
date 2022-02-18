# Home Pricing


## Overview


The focus of this project is to provide insight for owners looking to list their homes in the housing market, informing them of the most critical elements in pricing to offer methods to increase valuation. Data regarding homes in the greater King County (Washington) area  will be utilized to compose an informed regression model. The results of this model will help home owners understand what goes into the valuation of a home, and find optimal routes of improvement if they wish to sell.

## Business Problem 

Relative perception of a home's price can be a result of several factors, and home prices may skyrocket/plummet due to gentrification, overpopulation, and overall demographic shifts. On a rudimentary level though, a given property's price is determined by its features. The combination of these features may result in radically different prices. For example, homes with similar living square-footage may be priced completely different based on their location, view, or age. 

In an ever-fluctuating housing market, home owners looking to sell may be disappointed when they realize their property is worth less than they were expecting. Owners looking to increase their property valuation may have little direction/insight into the most optimal routes of home improvement. In some cases, blind financial investment into home modifications may prove to be unprofitable. 

Data from the King County housing dataset will be aggregated/summarized in order to determine the most impactful characteristics of a property's pricing. Isolating these critical features would give prospective home sellers a better idea of the make-up of their home's valuation. If it is within reason, the identified features can be improved upon in order to increase the valuation of a home.


## Data Understanding

Each individual field will be assessed for its contribution to a home's resulting price. Each row in the table represents a unique property and its correlated features. This makes the table easy to summarize as it eliminates the presence of duplicate properties in the dataset. 

Some columns in the dataframe are already represented in some way within other aggregate columns. For example, the 'sqft_basement' is already counted in the aggregate living space, 'sqft_living'. The data dictionary will help inform the decisions to remove any columns in order to prevent multicollinearity in our results.


![datastructure](https://github.com/n0vah917/dsc-phase-2-project-v2-3/blob/main/data/data.png)

### Data Preparation

The 'date' field (date home was sold) is removed, as homes may switch ownership on a relatively frequent basis. A more informative time-based metric to keep is 'yr_built', as the year a home was built can have an influence on the state of the its underlying construction, and the utilities present (e.g. insulation, air conditioning/heating, furnishings). 'yr_renovated' is removed for similar reasoning. While it is assumed that renovations will inherently raise a home's value, there is too much variability within the types/scale of renovations a home can go through. 

'sqft_above', 'sqft_lot', 'sqft_basement', 'sqft_living15', sqft_lot15' are removed, as they are already represented within the 'sqft_living' column, which represents the collective living area within a property. The values in 'sqft_lot' would expectedly be highly correlated with 'sqft_living' when looking at 1 floor homes.

'zipcode','lat', and 'long' are removed due to them being impossible to quantify in the context of the overall dataset. Using external tools/mapping API to infer home pricing based on location may be a potential opportunity to explore in future analyses. 

'id' is used as a unique identifier in the overall dataset, and does not have any correlation with a home's price.

![newstructure](https://github.com/n0vah917/dsc-phase-2-project-v2-3/blob/main/data/newdata.png)

The categorical values, 'condition','grade','view', and 'waterfront' will need to be label encoded in order to better evaluate their columns. Detailed ranking definitions for each column were found through the King County Assessor website, and were used to assign numerical values to the values via the use of a dictionary.

For 'condition', this variable appears to have a 5-point scale from 'Poor' to 'Very Good'. This will be encoded into numerical values spanning 1-5.

Similarly 'grade' seems to have a wider scale with 11 points, 'Poor' being the worst ranking, and 'Mansion' being the highest. It seems like 'grade' is given a numerical value despite having several qualitative features comprising the overall rank, implying that there might be some collinearity with square foot living. This will be encoded with the range 1-11.

'view' seems to have 4 points in its scale. If there is no view of any natural features, that home is given a value of 'NONE'. If there is a view present, the value is assigned as either 'FAIR', 'GOOD', or 'EXCELLENT'. This is encoded on a scale from 0-3 in the new dataset. This column also had null values; it is safe to assume that if a value is not present, the property has no observable view.

Waterfront is a binary variable indicating whether the home lies on the water (1) or not (0). Because this variable has little variation in its points, it might be difficult to determine any sort of correlation with price through this feature.

The existing columns are then mapped according to their inferred dictionary values (below), values being inputted into new columns in the table. The table data then has its null column values filled with zero. Previous columns are then dropped.



The data is now prepped for modeling. For this project, the plan is to use multiple linear regression in order to collectively assess the effect of all relevant variables on sale price. We have multiple predictors that all individually affect the 'price' variable, so the magnitude of each will be crucial to understand. Regression will also determine the validity of the relationships between predictor/outcome variables through coefficients in the model's results.

Before creating the model, scatter plots are shown for each predictor vs. price to visually identify any stark trends.


![graph](https://github.com/n0vah917/dsc-phase-2-project-v2-3/blob/main/data/graphed.png)

Upon first glance, some of the variables show a linear trend when plotted against 'price', particularly 'bedrooms', 'bathrooms', and 'sqft_living'. The charts also discerned variables that were either categorical or continuous, and the distribution of their contents. The more segmented scatter charts are categorical columns.

Initially, it was assumed that the more 'floors' a home had, the higher it would be priced. Looking at the scatter plots show that this is not the case. There is a high amount of variability within the lower floor counts. Some entries within 'floors' column are not whole integers. This implies that some houses might features that aren’t considered floors in a strict sense, like attics or guest houses.

Similarly, the 'bathrooms' column is incremented by .25 in some cases. This implies that homes evaluate bathroom count based on features present. A full bathroom (counted as 1) is likely one that contains a shower, toilet, and sink.

There might be a linear correlation between 'viewed' (view),'conditioned' (condition), and 'graded' (grade), against 'price', but variability is observed in the higher values within each column.

'yr_built' also contains a lot of variability, but the bottom of the spread shows that the minimum price of a newly built home is slightly higher than older homes, expectedly.


## Data Modeling

Now that the relevant columns are cleaned, modeling with a focus on 'price' can begin. First, a heat map is created to show the correlation between all of our variables.


![heatmap](https://github.com/n0vah917/dsc-phase-2-project-v2-3/blob/main/data/heatmap.png)

It seems as if 'sqft_living', 'graded', 'viewed', and 'bathrooms' all have some sort of correlation with 'price' seeing as their values of correlation are at least over .35. 

Within each of the individual predictors, it seems like there might be some columns that might have correlation to each other, which could prove to be a problem in the linearity of our model. In later models we will refine and evaluate these tendencies.

## Iterative Linear Regression Modeling

The first model will be created with variables that had the highest correlation with 'price'.
The model coefficient that used to assess performance is the R-squared coefficient (top-right corner of the summary), which describes what percent of the dependent variable is described by the included predictors.

Our goal with this iterative process will be to maximize this value/keep this value high, while reducing the Jarque-Bera coefficient, Kurtosis (sharpness of a frequency distribution curve). The more variables are added to the model will inherently increase the R2 coefficient, but we don't want the model to be over-explained/over-fitted through the addition of new variables.

In the next model, log transformations are applied to the 'sqft_living' and 'price' variables. As a note, when it comes to our interpreting our eventual results, our resulting outcome metric value for price is actually the power 'e' is raised to. Taking the inverse of the natural log value gives us our actual calculated 'price'. 

Next, 'sqft_living' has a high correlation to the grade and bathroom columns, which is proven to be problematic. Multicollinearity is inherently a problem as performing linear regression assumes that unit changes in each individual predictor has no effect on the others. Since 'sqft_living' is highly correlated with more than one variable (correlation factor over .7), it can be replaced with a variable that has the next highest correlation with 'price', 'bathrooms' (.31 from the initial chart). 

Despite making the decision to remove 'sqft_living' from the model, it is still known that it has a noticably high correlation with pricing. More details regarding this choice will be expanded on in the final model's results.


In order to increase the R-squared value and account for these new issues, next, adjustments can be performed to deal with the outliers within each of the predictors. Jarque-Bera improved slightly with these changes and the residual errors look better now that the outliers were removed. 


Next, in an effort to further improve the R-squared value, the interaction between different variables, outside of an additive means should be accounted for. To start, the set is split into predictors and outcome variable, then the baseline R-squared score is re-calculated using KFold validation. 


Each pair combination of predictors is then assessed to understand if the addition of a interaction factor betweeen the two would increase the baseline R-squared value. 
The top resulting scores from each added potential interaction are then printed. 





With the addition of the interaction variable, the R-squared value increased slightly, and Jarque-Bera decreased slightly.
The next step is assessing the model to ensure overfitting within the model is not occurring. 


### Train/Test Split

Now that the model is essentially complete, a train-test split should be performed in order to understand how the model performs with new data in the mix, in the event that more data gets added to the overall set. 

Thankfully, the cross-fold validation performed on the data lines up similarly to initial train/test results. That being said, the model is not overfit with the training data.


# Regression Results


There are several factors that go into determining a valuation of a home. In order to find the ideal predictors that are critical to the determination of home price, the performed analysis/regression results of the King County home dataset revealed the scale/importance of several of these fields. While the overall factors are helpful in determining the scale and effect of variables on sale price, the model as a whole does not completely reflect the whole dataset. The regression metric of success, R-squared value, would end up being closer to 1. 

That being said, the model still revealed the key factors that have the highest effect on price. In the bottom half of the model summary seen below, the 'coef' column, aligning with each of the final variables, shows the numerical importance of the dependent price variable.

As a note, when evaluating a theoretical price using the model, these coefficients represent the percent increase in price Y with each 'unit' increase of a given X predictor variable. The Intercept value in the coefficient column shows the initial Baseline price estimated that each home has according to the model, stripped of its features. To convert this into a value of interpretation, e^(intercept value), done to inverse the initial log transformation to get a baseline price of. In this case, the baseline home value is $112,307.



## Recommendations/Conclusion


### Note Regarding the 'sqft_living' Field

In initial models, 'sqft_living' was perceived to be the most correlated to 'price', which makes sense as increasing the overall living space would increase the amount a home is worth. Its coefficient seen in the results of linear regression was the highest amongst the other variables as well.

In the end, it was not included in the regression due to its multicollinearity with the other variables, 'grade' and 'bedrooms', which made prior regresion results unreliable. The multicollinearity observed makes sense, as the more space/occupants a home has, the higher amount of bedrooms it would likely have, and the higher grade of construction necessary to build it. 

__That being said, living space, or 'sqft_living' is a major contributing factor in a home's resulting sale price.__ If a realtor's objective is to increase sale price to a great degree, they should consider increasing the living space. All homes within the dataset are fully built, so the recommendation would require major foundational, structural, and financial investment. 


### Most Important Variables for increasing Price

__From the final regression model, the variables that were most impactful in resulting sale price were (in order): Grade (represented as graded), View (represented as viewed), and Bathrooms (represented as roundbath). These results are unsurprising as explained below.__

The grade of a home relates to its construction and design. Homes that were designed by renowned artists/architects, or for famous individuals more than likely have special features that drive up the overall valuation. Redesigns/renovations with emphasized focus on the quality of its features will have a positive impact on its resulting price.

Below is a depiction of the initial percieved linear relationship between grade and price.

![grade](https://github.com/n0vah917/dsc-phase-2-project-v2-3/blob/main/data/pricegrade.png)

The view a home has is especially important. Obviously, not much improvement can be done on a built home in terms of its location and view. However, the placement of windows could be optimized, further capitalizing on a home's location/view. Changing the window style, or having floor-to-ceiling glass panes to replace walls may greatly improve a home's price.

Below is a depiction of the initial percieved linear relationship between view and price.

![view](https://github.com/n0vah917/dsc-phase-2-project-v2-3/blob/main/data/priceview.png)


The amount of bathrooms (and similarly bedrooms) a home has is inherently correlated with its square footage. The more rooms/occupants a home accomodates for, in turn, increases the required amount of bathroooms and resulting price. While being a potential improvement from the lens of condition, bathrooms can be renovated to increase the overall quality of utilities and design. 

Below is a depiction of the initial percieved linear relationship between bathrooms and price.

![bathrooms](https://github.com/n0vah917/dsc-phase-2-project-v2-3/blob/main/data/pricebathrooms.png)

While approaching each of these variables for improvement individually would be infeasible, holistic renovations (of varied scales) would encompass all of these categories. There are several home improvement shows on HGTV that make an example of this, such as HGTV ‘s Love It or List It, where current owners have the option to make renovations to a home in order to improve its value. House Hunters, Fixer Upper, Property Brothers, and Flip or Flop, have similar themes of added value.



### Next Steps

Due to the nature of the success metric, the R-squared value, next steps in improving this model would largely involve the addition of variables that help better represent/quantify home pricing.

The latitude and longitude of the home were overlooked within the analysis. the 'view' column, while helpful in emphasizing the location of a home, is quite limited/almost binary in its scope. Next steps would involve mapping the latitude and longitude coordinates via Google Maps API in order to see the home location in relation to geographical features (i.e. Mt. Rainier, Olympics, or the Puget Sound), then applying an adjustment factor based on each landmark for a more nuanced look at home value. 

Similarly, another avenue to explore would be proximity to a certain suburban area or city. With relation to King County, proximity to Seattle would be an inherent upwards driver of cost. Each neighborhood could be represented as predictor variable. This can be analyzed even further through the use of demographics/survey work to identify different social/political communities in the King County area. For example, Brooklyn is known to have a young, progressive, and artsy culture, which may be a draw that can be quantified as an additional variable.

The final point of interest which may be difficult to quantify would be the technological/additional features that a home has. Custom carpentry, unique television displays, in-home surround sound, hot tubs, pools, or outdoor patios could all be additional factors that drive up resulting price. While the grade of the home provides insight into what these features might be, there are no other fields in the dataset that address these additional features. The grade appears to be a catchall variable which is not that descriptive on its own. 


...
├── data                                   #folder containing data used for analysis
├── README.md
├── Home Pricing.pdf    #pdf of presentation
├── jupyternotebook                              #pdf of jupyter notebook
└── Phase 2 Project.ipynb                                #jupyter notebook containing all relevant summaries/analyses
