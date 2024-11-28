# players_csv

library(tidyverse)
library(repr)
library(tidymodels)
library(ggplot2)
source("cleanup.R")
options(repr.matrix.max.rows = 6)

url <- "https://drive.google.com/uc?id=1Mw9vW0hjTJwRWx0bDXiSpYsO3gKogaPz"
players <- read_csv(url)
players

select_data <- players |> 
    filter(played_hours > 0) |>
    select(played_hours, age, experience)
select_data

clean_data <- mutate(select_data, 
    as_factor(experience))
clean_data
