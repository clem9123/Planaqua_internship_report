README
================

# Ecotron Lake Experiment

## Hello Fishes

Here is a little introduction !  
You will find in this documents every information you need on those data
and the script and analyses going with it.  
I suggest you read everything to understand what we are talking about
and to be aware of the choices and hypothesis I made.  
But as I know it’s often easier to read and discuss about something we
can see and play with, you can also begin by reading about the
experiment, the list of files and jump directly to the hand on section
to get directly into this data.

## Experiment

Information on the experimantal lake : [Aquacosm.pdf](Aquacosm.pdf)

## List and quick explanation of the different files

### Dataset in /data

#### Data bases listing the Tag_ids and the related information

See section [Tag data bases]() for more precision, else a quick summary
:

-   **BDD_original complet.xslx**: row dataset input from the
    experiment  
    see explanation : [BDD_original_complet.xlsx explanation]()
-   **data_final_ws_norm.xlsx** : cleaned version of BDD_original with
    unification of notation and elimination of incoherent data probably
    coming from experipmental input error (such as tag_id found in wrong
    lake)  
    data_final with additional weight and size for juveniles and 2022 by
    using a normal distribution  
    this is the data used in my following analyses, see hypothesis and
    precision on this data : [data_final_ws_norm.xlsx explanation]()

#### Other data

-   **Lake_treatment.xlsx** : a table containing the experimental
    characteristics of each lake : Nutrients enrichment, presence of
    perch (predator species), and a Treatment column summarizing the two
    see precision : [Lake_tretment.xlsx explanation]())

### R Scripts

-   **construction_BDD.Rmd**
-   **importation_data.R**
-   **setup.R**
-   **Jags_CJS_model**
-   **MODEL_visualisation_function.R**

## Diving deeper in the data

### Data bases

These data bases present the result of the fisheries that happened in
the lake during the 5 years experiments.  
You will find a line for each fish observed, with the different
information collected (Tag_id,date, size, weight, Lake_capture,
Lake_released), and information about the experimental characteristics
of this fishery (Method, Passage, Session).

  

#### BDD_original_complet.xlsx explanation

Simply the first data base I was given.  
Not necessary for a simple data usage. Need data cleaning before
analyses.

  

#### data_final_ws_norm.xlsx explanation

Cleaned version of BDD_original_complet.xlsx, see the
[construction_BDD.Rmd](construction_BDD.Rmd) file to precisely see the
modification that where made. (Not necessary for a simple usage of the
data)  

Specific hypothesis on Weight and size :  

**For juveniles** : weight and size where added using a normal
distribution coming from the measured and weighted juveniles from the
same Lake and Year  
This was done for two reasons :

-   It’s closer to the biologic reality
-   It diminishes the bias toward one over-represented value \\

**For jan 2022** :  
The hypothesis is that fishes weight and size didn’t change between nov
2021 and jan 2022, because of the time gap (4 months only) and the fact
that winter is a slow growth period  
This can be verified because some fishes where weighted at both
occasion. This difference didn’t seem significant, I then accepted the
first hypothesis  
Weight and size where added in 2022 by taking the associated weight and
size from nov 2021

  

##### Columns description

| Columns name        | Description                                                                                                                                                           |
|---------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Index**           | Number of this observation                                                                                                                                            |
| **Date**            | Date of observation (-dd/mm/yyyy-)                                                                                                                                    |
| **Obs_status**      | Action done with this fish -introduction- -capture- -recapture-                                                                                                       |
| **Lake_capture**    | The lake in which the fish was extracted, -NA- if it is a new introduction                                                                                            |
| **Method_capture**  | How was the fish extracted ? -trawl- (in october or november each year), -creel- (for the perch during summer), or -draining- (for 2022, when the lake where emptied) |
| **Session_capture** | In 2019 a problem of efficiency made 2 fisheries necessary (one normal -A- and a second with brushing -B-)                                                            |
| **Passage_net**     | Every fishery had 3 net passage -1- -2- and -3-                                                                                                                       |
| **Tag_id**          | Tag identifying individually each fish bigger than 8g                                                                                                                 |
| **Species**         | The species of the fish (-pike-, -roach-, -perch-, -gudgeon-, -frog-)                                                                                                 |
| **Weight**          | Weight (in g)                                                                                                                                                         |
| **Size**            | Size of the fish from head to the fork of the tail (in mm)                                                                                                            |
| **Lake_released**   | The lake in which the fish was released, -NA- if it wasn’t released                                                                                                   |
| **Comment**         | Any additional information                                                                                                                                            |

Let’s see how it looks  
  

``` r
library(readxl)
BDD_f <- read_excel("data/data_final_ws_norm.xlsx", 
                    col_types = c("numeric","date","text","text","text","text","text",
                                  "text","text","numeric","numeric","text","text"), na = "")
knitr::kable(head(BDD_f))
```

| Index | Date       | Obs_status   | Lake_capture | Method_capture | Session_capture | Passage_net | Tag_id    | Species | Weight | Size | Lake_released | Comment_obs |
|------:|:-----------|:-------------|:-------------|:---------------|:----------------|:------------|:----------|:--------|-------:|-----:|:--------------|:------------|
|     1 | 2016-12-06 | introduction | NA           | NA             | NA              | NA          | 403274142 | roach   |     31 |  129 | 16            | NA          |
|     2 | 2016-12-06 | introduction | NA           | NA             | NA              | NA          | 403274145 | roach   |     11 |   97 | 16            | NA          |
|     3 | 2016-12-06 | introduction | NA           | NA             | NA              | NA          | 403274148 | roach   |     11 |   98 | 1             | NA          |
|     4 | 2016-12-06 | introduction | NA           | NA             | NA              | NA          | 403274149 | roach   |     11 |  100 | 14            | NA          |
|     5 | 2016-12-06 | introduction | NA           | NA             | NA              | NA          | 403274150 | roach   |     13 |   98 | 1             | NA          |
|     6 | 2016-12-06 | introduction | NA           | NA             | NA              | NA          | 403274151 | roach   |     10 |   88 | 3             | NA          |

  

### Other useful data

#### Lake_treatment.xlsx explanation

A short table giving the characteristics of each Lake :

-   **Nutrients** : TRUE if enriched
-   **Perch** : TRUE if there is no fisheries pressure on perches,
    i.e. presence of predators
-   **Treatment** :
    -   1 for Nutrients and no Perch
    -   2 for Nutrients and Perch
    -   3 for no Nutrients and no Perch
    -   4 for no Nutrients and Perch

``` r
Lake_treatment <- read_excel("data/Lake_treatment.xlsx", col_types =c("text","logical","logical","text"))
knitr::kable(head(Lake_treatment))
```

| Lake | Nutrients | Perch | Treatment |
|:-----|:----------|:------|:----------|
| 1    | TRUE      | FALSE | 1         |
| 2    | FALSE     | FALSE | 3         |
| 3    | TRUE      | TRUE  | 2         |
| 4    | FALSE     | TRUE  | 4         |
| 5    | FALSE     | TRUE  | 4         |
| 6    | TRUE      | TRUE  | 2         |

  

### Script description

#### construction_BDD.Rmd

Cleaning of the data take BDD_original complet.xslx as input and create
data_final_ws_norm.xlsx  

#### importation_data.R

Take data_final_ws_norm.xlsx and import the data in different data frame
: - BDD_f : all the data - BDD_g : every roaches in trawl capture
event - BDD_a : every adult (i.e. tagged) roaches in trawl capture
event - BDD_p : every perches in trawl capture event \\

#### setup.R

**Should be used first** Import data, all the necessary library, and
create jags.data (data for the model)

#### Jags_CJS_model

Model description and everything to run it **Can simply be ran as such,
everything is alredy imported**  

#### MODEL_visualisation_function.R

Create function to extract and visualize the output model  

## Hands on the data, getting started, quick guide

This is a little explanation of how to get started with this data  
It’s often complicated to dive into such analyses so I wanted to give
anyone who wants to use in a way in.  
There are probably other ways to look at the data but if you are in a
hurry, or want to wrap your head around all this to get a better
understanding, here is what you can do :

To be continued… (and where to put some data visualization + numbers of
line and column for every data.frame to have an idea of what we are
dealing with)
