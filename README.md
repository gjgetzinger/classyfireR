# classyfireR
[![Project Status: Active - The project has reached a stable, usable state and is being actively developed.](http://www.repostatus.org/badges/latest/active.svg)](http://www.repostatus.org/#active)
[![Build Status](https://travis-ci.org/wilsontom/classyfireR.svg?branch=master)](https://travis-ci.org/wilsontom/classyfireR) [![Build status](https://ci.appveyor.com/api/projects/status/ua94fiotdmc0ssq5/branch/master?svg=true)](https://ci.appveyor.com/project/wilsontom/classyfirer/branch/master) [![codecov](https://codecov.io/gh/wilsontom/classyfireR/branch/master/graph/badge.svg)](https://codecov.io/gh/wilsontom/classyfireR) ![License](https://img.shields.io/badge/license-GNU%20GPL%20v3.0-blue.svg "GNU GPL v3.0") [![DOI](https://zenodo.org/badge/118162964.svg)](https://zenodo.org/badge/latestdoi/118162964)

[![CRAN](https://www.r-pkg.org/badges/version/classyfireR)](https://cran.r-project.org/web/packages/classyfireR/index.html) ![total downloads](https://cranlogs.r-pkg.org/badges/grand-total/classyfireR?color=red)
> __R Interface to the [ClassyFire REST API](http://classyfire.wishartlab.com)__ 


### Installation & Usage
`classyfireR` can be installed from CRAN using;

```R
install.packages('classyfireR')
```

Or from GitHub using the `remotes` package

```R
remotes::install_github('wilsontom/classyfireR')
```

* [Entity Classification](#entity-classification)
* [New Classification](#new-classification)
* [Acknowledgements](#acknowledgements)

### Entity Classification

__For retrieval of classifications already available; a InChI Key is supplied to the  `entity_classification` function.__

```R
library(classyfireR)

> inchi_keys <- c('BRMWTNUJHUMWMS-LURJTMIESA-N', 'MDHYEMXUFSJLGV-UHFFFAOYSA-N')

> entity_classification(inchi_keys[1])

✔ classification retrieved
# A tibble: 4 x 3
  Level      Classification                       CHEMONT          
  <chr>      <chr>                                <chr>            
1 kingdom    Organic compounds                    CHEMONTID:0000000
2 superclass Organic acids and derivatives        CHEMONTID:0000264
3 class      Carboxylic acids and derivatives     CHEMONTID:0000265
4 subclass   Amino acids, peptides, and analogues CHEMONTID:0000013
```

__Using the `tidyverse` a vector of InChI Keys can be submitted and easily extracted.__

```R
> library(tidyverse)

> keys <- c(
  'BRMWTNUJHUMWMS-LURJTMIESA-N',
  'XFNJVJPLKCPIBV-UHFFFAOYSA-N',
  'TYEYBOSBBBHJIV-UHFFFAOYSA-N',
  'AFENDNXGAFYKQO-UHFFFAOYSA-N',
  'WHEUWNKSCXYKBU-QPWUGHHJSA-N',
  'WHBMMWSBFZVSSR-GSVOUGTGSA-N'
)

> classification_list <- map(keys, entity_classification)

> classification_tibble <-
  map2(classification_list, keys, ~ {
    add_column(.x, ID = rep(.y))
  }) %>% bind_rows()

# To create a table of just the superclass designation

> superclass <-
  classification_tibble %>% filter(Level == 'superclass') %>% select(-c(CHEMONT))

> superclass
# A tibble: 6 x 3
  Level      Classification                  ID                         
  <chr>      <chr>                           <chr>                      
1 superclass Organic acids and derivatives   BRMWTNUJHUMWMS-LURJTMIESA-N
2 superclass Organic nitrogen compounds      XFNJVJPLKCPIBV-UHFFFAOYSA-N
3 superclass Organic acids and derivatives   TYEYBOSBBBHJIV-UHFFFAOYSA-N
4 superclass Organic acids and derivatives   AFENDNXGAFYKQO-UHFFFAOYSA-N
5 superclass Lipids and lipid-like molecules WHEUWNKSCXYKBU-QPWUGHHJSA-N
6 superclass Organic acids and derivatives   WHBMMWSBFZVSSR-GSVOUGTGSA-N



# To create a data.frame of all classification results

classification_list <- map(classification_list, ~{select(.,-CHEMONT)})

spread_tibble <- purrr:::map(classification_list, ~{
  spread(., Level, Classification)  
}) %>% bind_rows() %>% data.frame()

rownames(spread_tibble) <- keys

classification_df <-  data.frame(InChIKey = rownames(spread_tibble),
    Kingdom = spread_tibble$kingdom,
    SuperClass = spread_tibble$superclass,
    Class = spread_tibble$class,
    SubClass = spread_tibble$subclass)

> classification_df
                     InChIKey           Kingdom                      SuperClass
1 BRMWTNUJHUMWMS-LURJTMIESA-N Organic compounds   Organic acids and derivatives
2 XFNJVJPLKCPIBV-UHFFFAOYSA-N Organic compounds      Organic nitrogen compounds
3 TYEYBOSBBBHJIV-UHFFFAOYSA-N Organic compounds   Organic acids and derivatives
4 AFENDNXGAFYKQO-UHFFFAOYSA-N Organic compounds   Organic acids and derivatives
5 WHEUWNKSCXYKBU-QPWUGHHJSA-N Organic compounds Lipids and lipid-like molecules
6 WHBMMWSBFZVSSR-GSVOUGTGSA-N Organic compounds   Organic acids and derivatives
                             Class                               SubClass
1 Carboxylic acids and derivatives   Amino acids, peptides, and analogues
2         Organonitrogen compounds                                 Amines
3       Keto acids and derivatives Short-chain keto acids and derivatives
4    Hydroxy acids and derivatives    Alpha hydroxy acids and derivatives
5 Steroids and steroid derivatives                       Estrane steroids
6    Hydroxy acids and derivatives     Beta hydroxy acids and derivatives

```


### New Classification

__For compounds which do not already have an available entity classification; a new classification can be generated by submitting the InChI code to the ClassyFire Server using a POST request__.

```R
> new_compound <- 'InChI=1S/C7H11N3O2/c1-10-3-5(9-4-10)2-6(8)7(11)12/h3-4,6H,2,8H2,1H3,(H,11,12)/t6-/m0/s1'

> new_submission <- submit_classification(new_compound, label = 'test', type = 'STRUCTURE')

Classification still in progress

Use `retrieve_classification` once submission is out of queue

> new_submission
query_id
[1] 2815613

$status
[1] "In Queue"


# wait a few seconds

> get_status_code(new_submission$query_id)
$query_id
[1] 2815613

$status
[1] "Done"

# then retrieve the results

> classification_result <- retrieve_classification(new_submission$query_id)

> classification_result

# A tibble: 4 x 3
  Level      Classification                       CHEMONT          
  <chr>      <chr>                                <chr>            
1 kingdom    Organic compounds                    CHEMONTID:0000000
2 superclass Organic acids and derivatives        CHEMONTID:0000264
3 class      Carboxylic acids and derivatives     CHEMONTID:0000265
4 subclass   Amino acids, peptides, and analogues CHEMONTID:0000013
```

### Acknowledgements

If you use `classyfireR` you should cite the [ClassyFire](https://jcheminf.springeropen.com/articles/10.1186/s13321-016-0174-y) publication

> ___Djoumbou Feunang Y, Eisner R, Knox C, Chepelev L, Hastings J, Owen G, Fahy E, Steinbeck C, Subramanian S, Bolton E, Greiner R, and Wishart DS___. ClassyFire: Automated Chemical Classification With A Comprehensive, Computable Taxonomy. Journal of Cheminformatics, 2016, 8:61.

> __DOI:__ [10.1186/s13321-016-0174-y](https://jcheminf.springeropen.com/articles/10.1186/s13321-016-0174-y)
