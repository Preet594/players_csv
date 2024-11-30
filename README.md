
# Final Project Report: Using Age and Level of Experience to Predict Individual Play Time on PlaiCraft Server

By Prisha Budhiraja, Yiming Chen, Preet Dhillon,  Jaime Fraser, and Stephanie Tamkee

## Introduction

The data for this project is collected from a UBC research project. A MineCraft server called PlaiCraft was set up where players actions are recorded as they navigate through the world. This data consists of two files: players.csv and sessions.csv. The question we want to answer is, "How do age and level of experience affect an individual's playing time on the PlaiCraft server?" To answer this, question, only the players.csv dataset will be used because it contains data about the players' age, experience level, and playing time.

### The Dataset: players.csv
This players.csv file contains a list of all the unique players, including the following data about each player:
 - experience (identifies as Beginner, Amateur, Regular, Pro, or Veteran)
 - subscribe (whether or not they have subscribed to PlaiCraft)
 - hashedEmail (personal identifier)
 - played_hours (number of hours played on the server)
 - name (name of player)
 - gender
 - age
"individualId" and "organizationName" (logical data type) are also column headings in the data set, however, there is no data contained in these columns. Excluding these variables, there are 7 variables (Note: "hashedEmail" and "name" are unique to each individual and are not dependent variables) and 196 observations.


The following table describes each variable by its name, type of variable, any issues observed in the data, and any other potential issues:
   
|     Name     |    Type   |  Visible Issues  |  Potential Issues  |
| ------------ | --------- | ---------------- | ------------------ |
|  experience  | character | N/A              | It is unclear what each category represents in terms of experience |
|  subscribe   | logical   | N/A              | N/A                |
| hashedEamil  | character | Value is not readable              | Not usable as predictor                |
| played_hours | double    | Many values are 0, indicating many players have not played | N/A |
|     name     | character | N/A| Not usable as predictor                |
|    gender    | character | N/A              | Gender is not binary; if this data is to be used, this must be considered |
|      age     | double    | N/A              | The idea that age cannot go to infinity may be considered depending on the method used in this project |

Note: N/A indicates that no issue is observed from looking at data set, however, more issues may be detected or resolved later on.

_figure 1_

## Methods & Results

### Importing Libraries

library(tidyverse)
library(repr)
library(tidymodels)
library(ggplot2)
options(repr.matrix.max.rows = 6)

### Loading the Dataset

url <- "https://drive.google.com/uc?id=1Mw9vW0hjTJwRWx0bDXiSpYsO3gKogaPz"
players <- read_csv(url)
players

### Wrangling and Cleaning the Data

##To wrangle and clean our data we selected only the columns we will be using for our data analysis, filtered the played hours column so that our data set only includes observations from people that have played on the server, and mutated the experience column from a character variable to a numeric variable with Beginner = 1, Amateur = 2, Regular = 3, Pro = 4, and Veteran = 5.

select_data <- players |> 
    filter(played_hours > 0) |>
    select(played_hours, age, experience)
select_data

clean_data <- select_data |>
  filter(played_hours > 0) |>  
  mutate(experience = ifelse(experience == "Beginner", 1,
                              ifelse(experience == "Amateur", 2,
                              ifelse(experience == "Regular", 3,
                              ifelse(experience == "Pro", 4,
                              ifelse(experience == "Veteran", 5, NA))))))
clean_data

### Visualizing the Data Separately

### Visualizing the Relationship between Age and Played Hours 
age_plot <- clean_data |>
                ggplot(aes(x=age, y=played_hours))+
                geom_bar(stat='identity')+
                xlab("Age of Player (in years)")+
                ylab("Total Time Spent Playing (in hours)") +
                 theme(text= element_text(size=12))
age_plot

### Visualizing the Relationship between Experience Level and Played Hours 
experience_plot <- clean_data |>
                    ggplot(aes(x=experience, y=played_hours))+
                    geom_bar(stat='identity')+
                    xlab("Experience Level of Player(Beginner-Veteran)") +
                    ylab("Total Time Spent Playing (in hours)") +
                    theme(text= element_text(size=12))

experience_plot  

### Splitting the Data into Training and Testing Sets

clean_data_split <- initial_split(clean_data, prop = 0.75, strata = played_hours)
clean_data_training <- training(clean_data_split)
clean_data_testing <- testing(clean_data_split)

clean_data_recipe<-recipe(played_hours ~ experience + age, data = clean_data_training ) |>
  step_scale(all_predictors()) |>
  step_center(all_predictors())

clean_data_spec <- nearest_neighbor(weight_func = "rectangular",
                              neighbors = tune()) |>
  set_engine("kknn") |>
  set_mode("regression")

clean_data_vfold <- vfold_cv(clean_data_training , v = 5, strata = played_hours)

clean_data_wkflw <- workflow() |>
  add_recipe(clean_data_recipe) |>
  add_model(clean_data_spec)
  
clean_data_wkflw

gridvals <- tibble(neighbors = seq(from = 1, to = 200, by = 1))

clean_data_results <- clean_data_wkflw |>
  tune_grid(resamples = clean_data_vfold, grid = gridvals) |>
  collect_metrics() |>
  filter(.metric == "rmse")
clean_data_results

clean_data_min <- clean_data_results |>
  filter(mean == min(mean))
  
clean_data_min

#The minimum RMSPE value corresponds to the K value that should be used for testing the data. However, the minimum K value, 64, found from the results of the training data corresponds to a dot on the following graph that does not appear to follow the general pattern of the rest of the RMSPE values in relation to K-value. We chose this K value because it was the minimum, but we are unsure if this is the correct choice as this K value appears out of place in the plot.

### Visualizing RMSE Across K Values

rmspe_plot <- clean_data_results |>
    ggplot(aes(x=neighbors, y = mean))+
    geom_point() +
    xlab("Neighbors")+
    ylab("RMSPE")
    
rmspe_plot

### Finalizing and Testing the Model
#moving onto testing: 
kmin <- clean_data_min |> pull(neighbors)

clean_data_spec <- nearest_neighbor(weight_func = "rectangular", neighbors = kmin) |>
  set_engine("kknn") |>
  set_mode("regression")

clean_data_fit <- workflow() |>
  add_recipe(clean_data_recipe) |>
  add_model(clean_data_spec) |>
  fit(data = clean_data_training)

clean_data_summary <- clean_data_fit |>
  predict(clean_data_testing) |>
  bind_cols(clean_data_testing)
  
clean_data_summary


knn_mult_mets <- clean_data_summary|> 
  metrics(truth = played_hours, estimate = .pred) |>
  filter(.metric == 'rmse')
knn_mult_mets

### 3D Visualization of Data Relationships
library(plotly)
plot_3d <- plot_ly(clean_data, 
                   x = ~age, 
                   y = ~experience, 
                   z = ~played_hours, 
                   marker = list(size = 5, 
                                 color = ~experience, 
                                 colorscale = "RColorBrewer", 
                                 showscale = TRUE)) %>%
  add_markers() %>%
  layout(scene = list(
    xaxis = list(title = "Age of Players"),
    yaxis = list(title = "Experience Level of Players"),
    zaxis = list(title = "Total Played Hours")),
    title = "3D Plot: Relationship between Age, Experience Level, and Total Played Hours of Players")
plot_3d

## Discussion

From visualizing the individual players' total played hours versus their age and experience level, we found that the primary players of PlaiCraft are younger people aged 15-30. The players that play the most hours also fall within this age range and tend to have an experience level of "Amateur" or "Regular". This shows that the total played hours of an individual may be affected by their age and experience level as younger audiences of lower to medium experience levels are primarily the individuals playing the most hours on the server. As age increases, there are fewer individuals altogether on the server. There appears to be a relatively even amount of players within each experience level (fewer individuals identifying as a "Pro" or "Veteran"), however, the players that played the most total hours were most often of "Amateur" or Regular" experience level.

We expected that the data would show younger people and those with a higher experience level ("Pro" or "Veteran") would have the greatest playing time. We correctly hypothesized that younger people would play more often and for longer. However, we incorrectly predicted that players with a higher experience level would have a greater playing time than those with a lower experience level. Instead, our data showed that the players who identified as a "Pro" or "Veteran" often played very few hours on the server. 

Our findings can help with game design and marketing by assisting game developers in understanding who is playing on the server the most. Because most players are younger in age, the game developers should develop features of the game that appeal to younger audiences and create new features that can appeal to older audiences to engage this part of the population more. Overall, with our research question answered, our data should impact the decisions that game developers make as they can further engage the primary players of PlaiCraft and look to involve other populations that are playing less. 

These findings could lead us to ask...
1. What factors influence the level of engagement for the age groups 15 -30, which have the highest total played hours?
2. What causes older players or more experienced players (Vetern) to play less frequently or stop playing all together?
3. How does the playing hours time distribution vary throughout the day for the age group 15-30, which has the highest total played hours?

## References

plotly. (2015, July 30). 3D Scatter Plots in R. Plotly.com. https://plotly.com/r/3d-scatter-plots/
