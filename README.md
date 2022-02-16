# Home Pricing


## Overview


The focus of this project is to provide insight for new realtors in the housing market, informing them of the most critical elements in determining home prices. Data regarding homes in the greater King County (Washington) area  will be utilized to compose an informed regression model. The results of this model will help realtors understand what goes into the valuation of a home, determine fair pricing, and find optimal routes of improvement for homes in their portfolio if their owners wish to sell.

## Business Problem 


New realtors in the housing market may be overwhelmed by the inherent sensitivity of property pricing. Relative perception of a home's price can be a result of several factors, and in an ever-changing market, home prices may skyrocket due to gentrification, overpopulation, and overall demographic shifts. On a rudimentary level though, a given property's price is determined by its features. The combination of these features may result in radically different prices. For example, homes with similar living square-footage may be priced completely different based on their location, view, or age. 

Data from the King County housing dataset will be aggregated/summarized in order to determine the most impactful characteristics of a property's pricing. Isolating these critical features would give new realtors/real estate agencies a way to fairly value their assets. If it is within reason, the identified features can be improved upon in order to increase the valuation of a home.

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


### First Model

The first model will be created with variables that had the highest correlation with 'price'.

![model1](https://github.com/n0vah917/dsc-phase-2-project-v2-3/blob/main/data/model1.png)

The initial model has a very high Jarque-Bera coefficient, which implies a lack of normality within our continuous variables, which we can improve on in later iterations.

The model coefficient that used to assess performance is the R-squared coefficient (top-right corner of the summary), which describes what percent of the dependent variable is described by the included predictors.

Our goal with this iterative process will be to maximize this value/keep this value high, while reducing the Jarque-Bera coefficient, Kurtosis (sharpness of a frequency distribution curve). The more variables are added to the model will inherently increase the R2 coefficient, but we don't want the model to be over-explained/over-fitted through the addition of new variables.


### Second Model

In this model, transformations on the continuous variables will be performed in order to get them to better fit the assumptions of normality. 
In this case, log transformations are applied to the 'sqft_living' and 'price' variables. As a note, when it comes to our interpreting our eventual results, our resulting outcome metric value for price is actually the power 'e' is raised to. Taking the inverse of the natural log value gives us our actual calculated 'price'. 


![model2](https://github.com/n0vah917/dsc-phase-2-project-v2-3/blob/main/data/model2.png)


Plotting the residuals out confirms that our continuous variables are now normalized. The heteroscedasticity we observed in the prior model has been reduced, and the spread of residuals across the X axis looks more even. The QQ plots also show the trademark 45 degree straight line now. Further improvement on this model can be achieved by addressing issues that may arise from multicollinearity in our variables.


### Model 3


Prior to creating the first model, correlation between some of the predictors was observed. These issues can be addressed  by replacing variables where necessary.

'sqft_living' has a high correlation to the grade and bathroom columns, which is proven to be problematic. Multicollinearity is inherently a problem as performing linear regression assumes that unit changes in each individual predictor has no effect on the others. Since 'sqft_living' is highly correlated with more than one variable (correlation factor over .7), it can be replaced with a variable that has the next highest correlation with 'price', 'bathrooms' (.31 from the initial chart). 

Despite making the decision to remove 'sqft_living' from the model, it is still known that it has a noticably high correlation with pricing. More details regarding this choice will be expanded on in the final model's results.

![model3](https://github.com/n0vah917/dsc-phase-2-project-v2-3/blob/main/data/model3.png)


Jarque-Bera went down further, and the R-squared value went down inherently, as a highly correlated variable to 'price' was replaced with a variable with a lower correlation. While the R-squared value is the main metric of importance, concern also lies in the validity of the coefficients for each predictor in the regression.  


### Model 4


In order to increase the R-squared value and account for these new issues, next, adjustments can be performed to deal with the outliers within each of the predictors.


![model4](https://github.com/n0vah917/dsc-phase-2-project-v2-3/blob/main/data/model4.png)

Jarque-Bera improved slightly with these changes and the residual errors look better now that the outliers were removed. 


### Accounting for Interactions

In an effort to further improve the R-squared value, the interaction between different variables, outside of an additive means should be accounted for. To start, the set is split into predictors and outcome variable, then the baseline R-squared score is re-calculated using KFold validation. 


Each pair combination of predictors is then assessed to understand if the addition of a interaction factor betweeen the two would increase the baseline R-squared value. 
The top resulting scores from each added potential interaction are then printed. 


![model5](https://github.com/n0vah917/dsc-phase-2-project-v2-3/blob/main/data/model5.png)


With the addition of the interaction variable, the R-squared value increased slightly, and Jarque-Bera decreased slightly.
The next step is assessing the model to ensure overfitting within the model is not occurring. 


### Train/Test Split

Now that the model is essentially complete, a train-test split should be performed in order to understand how the model performs with new data in the mix, in the event that more data gets added to the overall set. 

Thankfully, the cross-fold validation performed on the data lines up similarly to initial train/test results. That being said, the model is not overfit with the training data.


# Regression Results


There are several factors that go into determining a valuation of a home. In order to find the ideal predictors that are critical to the determination of home price, the performed analysis/regression results of the King County home dataset revealed the scale/importance of several of these fields. While the overall factors are helpful in determining the scale and effect of variables on sale price, the model as a whole does not completely reflect the whole dataset. The regression metric of success, R-squared value, would end up being closer to 1. 

That being said, the model still revealed the key factors that have the highest effect on price. In the bottom half of the model summary seen below, the 'coef' column, aligning with each of the final variables, shows the numerical importance of the dependent price variable.

As a note, when evaluating a theoretical price using the model, these coefficients represent how much Y increases with each 'unit' increase of a given X predictor variable. The Intercept value in the coefficient column shows the initial Baseline price estimated that each home has according to the model, stripped of its features. The resulting sum of all factors+intercepts should then be interpreted as the power that e (irrational number) is raised to, since we took the logarithmic value of the 'price' value in order to normalize the data. 

## Recommendations/Conclusion


### Note Regarding the 'sqft_living' Field

In initial models, 'sqft_living' was perceived to be the most correlated to 'price', which makes sense as increasing the overall living space would increase the amount a home is worth. Its coefficient seen in the results of linear regression was the highest amongst the other variables as well.

In the end, it was not included in the regression due to its multicollinearity with the other variables, 'grade' and 'bedrooms', which made prior regresion results unreliable. The multicollinearity observed makes sense, as the more space/occupants a home has, the higher amount of bedrooms it would likely have, and the higher grade of construction necessary to build it. 

__That being said, living space, or 'sqft_living' is a major contributing factor in a home's resulting sale price.__ If a realtor's objective is to increase sale price to a great degree, they should consider increasing the living space. All homes within the dataset are fully built, so the recommendation would require major foundational, structural, and financial investment. 


### Most Important Variables for increasing Price

__From the final regression model, the variables that were most impactful in resulting sale price were (in order): Grade (represented as graded), View (represented as viewed), and Bathrooms (represented as roundbath). These results are unsurprising as explained below.__

The grade of a home relates to its construction and design. Homes that were designed by renowned artists/architects, or for famous individuals more than likely have special features that drive up the overall valuation. Redesigns/renovations with emphasized focus on the quality of its features will have a positive impact on its resulting price.

The view a home has is especially important. Obviously, not much improvement can be done on a built home in terms of its location and view. However, the placement of windows could be optimized, further capitalizing on a home's location/view. Changing the window style, or having floor-to-ceiling glass panes to replace walls may greatly improve a home's price.

The amount of bathrooms (and similarly bedrooms) a home has is inherently correlated with its square footage. The more rooms/occupants a home accomodates for, in turn, increases the required amount of bathroooms and resulting price. While being a potential improvement from the lens of condition, bathrooms can be renovated to increase the overall quality of utilities and design. 

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



There are three deliverables for this project:

* A **non-technical presentation**
* A **Jupyter Notebook**
* A **GitHub repository**

The deliverables requirements are almost the same as in the Phase 1 Project, and you can review those extended descriptions [here](https://github.com/learn-co-curriculum/dsc-phase-1-project-v2-3#deliverables). In general, everything is the same except the "Data Visualization" and "Data Analysis" requirements have been replaced by "Modeling" and "Regression Results" requirements.

### Non-Technical Presentation

Recall that the non-technical presentation is a slide deck presenting your analysis to ***business stakeholders***, and should be presented live as well as submitted in PDF form on Canvas.

We recommend that you follow this structure, although the slide titles should be specific to your project:

1. Beginning
    - Overview
    - Business and Data Understanding
2. Middle
    - **Modeling**
    - **Regression Results**
3. End
    - Recommendations
    - Next Steps
    - Thank you

Make sure that your discussion of modeling and regression results is geared towards a non-technical audience! Assume that their prior knowledge of regression modeling is minimal. You don't need to explain how linear regression works, but you should explain why linear regression is useful for the problem context. Make sure you translate any metrics or coefficients into their plain language implications.

The graded elements for the non-technical presentation are the same as in [Phase 1](https://github.com/learn-co-curriculum/dsc-phase-1-project-v2-3#deliverables).

### Jupyter Notebook

Recall that the Jupyter Notebook is a notebook that uses Python and Markdown to present your analysis to a ***data science audience***. You will submit the notebook in PDF format on Canvas as well as in `.ipynb` format in your GitHub repository.

The graded elements for the Jupyter Notebook are:

* Business Understanding
* Data Understanding
* Data Preparation
* **Modeling**
* **Regression Results**
* Code Quality

### GitHub Repository

Recall that the GitHub repository is the cloud-hosted directory containing all of your project files as well as their version history.

The requirements are the same as in [Phase 1](https://github.com/learn-co-curriculum/dsc-phase-1-project-v2-3#github-repository), except for the required sections in the `README.md`.

For this project, the `README.md` file should contain:

* Overview
* Business and Data Understanding
  * Explain your stakeholder audience here
* **Modeling**
* **Regression Results**
* Conclusion

Just like in Phase 1, the `README.md` file should be the bridge between your non technical presentation and the Jupyter Notebook. It should not contain the code used to develop your analysis, but should provide a more in-depth explanation of your methodology and analysis than what is described in your presentation slides.

## Grading

***To pass this project, you must pass each project rubric objective.*** The project rubric objectives for Phase 2 are:

1. Attention to Detail
2. Statistical Communication
3. Data Preparation Fundamentals
4. Linear Modeling

### Attention to Detail

Just like in Phase 1, this rubric objective is based on your completion of checklist items. ***In Phase 2, you need to complete 70% (7 out of 10) or more of the checklist elements in order to pass the Attention to Detail objective.***

**NOTE THAT THE PASSING BAR IS HIGHER IN PHASE 2 THAN IT WAS IN PHASE 1!**

The standard will increase with each Phase, until you will be required to complete all elements to pass Phase 5 (Capstone).

#### Exceeds Objective

80% or more of the project checklist items are complete

#### Meets Objective (Passing Bar)

70% of the project checklist items are complete

#### Approaching Objective

60% of the project checklist items are complete

#### Does Not Meet Objective

50% or fewer of the project checklist items are complete

### Statistical Communication

Recall that communication is one of the key data science "soft skills". In Phase 2, we are specifically focused on Statistical Communication. We define Statistical Communication as:

> Communicating **results of statistical analyses** to diverse audiences via writing and live presentation

Note that this is the same as in Phase 1, except we are replacing "basic data analysis" with "statistical analyses".

High-quality Statistical Communication includes rationale, results, limitations, and recommendations:

* **Rationale:** Explaining why you are using statistical analyses rather than basic data analysis
  * For example, why are you using regression coefficients rather than just a graph?
  * What about the problem or data is suitable for this form of analysis?
  * For a data science audience, this includes your reasoning for the changes you applied while iterating between models.
* **Results:** Describing the overall model metrics and feature coefficients
  * You need at least one overall model metric (e.g. r-squared or RMSE) and at least two feature coefficients.
  * For a business audience, make sure you connect any metrics to real-world implications. You do not need to get into the details of how linear regression works.
  * For a data science audience, you don't need to explain what a metric is, but make sure you explain why you chose that particular one.
* **Limitations:** Identifying the limitations and/or uncertainty present in your analysis
  * This could include p-values/alpha values, confidence intervals, assumptions of linear regression, missing data, etc.
  * In general, this should be more in-depth for a data science audience and more surface-level for a business audience.
* **Recommendations:** Interpreting the model results and limitations in the context of the business problem
  * What should stakeholders _do_ with this information?

#### Exceeds Objective

Communicates the rationale, results, limitations, and specific recommendations of statistical analyses

> See above for extended explanations of these terms.

#### Meets Objective (Passing Bar)

Successfully communicates the results of statistical analyses without any major errors

> The minimum requirement is to communicate the _results_, meaning at least one overall model metric (e.g. r-squared or RMSE) as well as at least two feature coefficients. See the Approaching Objective section for an explanation of what a "major error" means.

#### Approaching Objective

Communicates the results of statistical analyses with at least one major error

> A major error means that some aspect of your explanation is fundamentally incorrect. For example, if a feature coefficient is negative and you say that an increase in that feature results in an increase of the target, that would be a major error. Another example would be if you say that the feature with the highest coefficient is the "most statistically significant" while ignoring the p-value. One more example would be reporting a coefficient that is not statistically significant, rather than saying "no statistically significant linear relationship was found"

> "**If a coefficient's t-statistic is not significant, don't interpret it at all.** You can't be sure that the value of the corresponding parameter in the underlying regression model isn't really zero." _DeVeaux, Velleman, and Bock (2012), Stats: Data and Models, 3rd edition, pg. 801_. Check out [this website](https://web.ma.utexas.edu/users/mks/statmistakes/TOC.html) for extensive additional examples of mistakes using statistics.

> The easiest way to avoid making a major error is to have someone double-check your work. Reach out to peers on Slack and ask them to confirm whether your interpretation makes sense!

#### Does Not Meet Objective

Does not communicate the results of statistical analyses

> It is not sufficient to just display the entire results summary. You need to pull out at least one overall model metric (e.g. r-squared, RMSE) and at least two feature coefficients, and explain what those numbers mean.

### Data Preparation Fundamentals

We define this objective as:

> Applying appropriate **preprocessing** and feature engineering steps to tabular data in preparation for statistical modeling

The two most important components of preprocessing for the Phase 2 project are:

* **Handling Missing Values:** Missing values may be present in the features you want to use, either encoded as `NaN` or as some other value such as `"?"`. Before you can build a linear regression model, make sure you identify and address any missing values using techniques such as dropping or replacing data.
* **Handling Non-Numeric Data:** A linear regression model needs all of the features to be numeric, not categorical. For this project, ***be sure to pick at least one non-numeric feature and try including it in a model.*** You can identify that a feature is currently non-numeric if the type is `object` when you run `.info()` on your dataframe. Once you have identified the non-numeric features, address them using techniques such as ordinal or one-hot (dummy) encoding.

There is no single correct way to handle either of these situations! Use your best judgement to decide what to do, and be sure to explain your rationale in the Markdown of your notebook.

Feature engineering is encouraged but not required for this project.

#### Exceeds Objective

Goes above and beyond with data preparation, such as feature engineering or merging in outside datasets

> One example of feature engineering could be using the `date` feature to create a new feature called `season`, which represents whether the home was sold in Spring, Summer, Fall, or Winter.

> One example of merging in outside datasets could be finding data based on ZIP Code, such as household income or walkability, and joining that data with the provided CSV.

#### Meets Objective (Passing Bar)

Successfully prepares data for modeling, including converting at least one non-numeric feature into ordinal or binary data and handling missing data as needed

> As a reminder, you can identify the non-numeric features by calling `.info()` on the dataframe and looking for type `object`.

> Your final model does not necessarily need to include any features that were originally non-numeric, but you need to demonstrate your ability to handle this type of data.

#### Approaching Objective

Prepares some data successfully, but is unable to utilize non-numeric data

> If you simply subset the dataframe to only columns with type `int64` or `float64`, your model will run, but you will not pass this objective.

#### Does Not Meet Objective

Does not prepare data for modeling

### Linear Modeling

According to [Kaggle's 2020 State of Data Science and Machine Learning Survey](https://www.kaggle.com/kaggle-survey-2020), linear and logistic regression are the most popular machine learning algorithms, used by 83.7% of data scientists. They are small, fast models compared to some of the models you will learn later, but have limitations in the kinds of relationships they are able to learn.

In this project you are required to use linear regression as the primary statistical analysis, although you are free to use additional statistical techniques as appropriate.

#### Exceeds Objective

Goes above and beyond in the modeling process, such as recursive feature selection

#### Meets Objective (Passing Bar)

Successfully builds a baseline model as well as at least one iterated model, and correctly extracts insights from a final model without any major errors

> We are looking for you to (1) create a baseline model, (2) iterate on that model, making adjustments that are supported by regression theory or by descriptive analysis of the data, and (3) select a final model and report on its metrics and coefficients

> Ideally you would include written justifications for each model iteration, but at minimum the iterations must be _justifiable_

> For an explanation of "major errors", see the description below

#### Approaching Objective

Builds multiple models with at least one major error

> The number one major error to avoid is including the target as one of your features. For example, if the target is `price` you should NOT make a "price per square foot" feature, because that feature would not be available if you didn't already know the price.

> Other examples of major errors include: using a target other than `price`, attempting only simple linear regression (not multiple linear regression), dropping multiple one-hot encoded columns without explaining the resulting baseline, or using a unique identifier (`id` in this dataset) as a feature.

#### Does Not Meet Objective

Does not build multiple linear regression models

## Getting Started

Please start by reviewing the contents of this project description. If you have any questions, please ask your instructor ASAP.

Next, you will need to complete the [***Project Proposal***](#project_proposal) which must be reviewed by your instructor before you can continue with the project.

Here are some suggestions for creating your GitHub repository:

1. Fork the [Phase 2 Project Repository](https://github.com/learn-co-curriculum/dsc-phase-2-project-v2-3), clone it locally, and work in the `student.ipynb` file. Make sure to also add and commit a PDF of your presentation to your repository with a file name of `presentation.pdf`.
2. Or, create a new repository from scratch by going to [github.com/new](https://github.com/new) and copying the data files from the Phase 2 Project Repository into your new repository.
   - Recall that you can refer to the [Phase 1 Project Template](https://github.com/learn-co-curriculum/dsc-project-template) as an example structure
   - This option will result in the most professional-looking portfolio repository, but can be more complicated to use. So if you are getting stuck with this option, try forking the project repository instead

## Summary

This is your first modeling project! Take what you have learned in Phase 2 to create a project with a more sophisticated analysis than you completed in Phase 1. You will build on these skills as we move into the predictive machine learning mindset in Phase 3. You've got this!
