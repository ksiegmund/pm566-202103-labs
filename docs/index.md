lab 7
================
ks
10/08/2021

## Lab week 7

``` r
if (knitr::is_html_output(excludes = "gfm")){
  
}
```

### Question 1: How many sars-cov-2 papers?

Build an automatic counter of sars-cov-2 papers using PubMed. You will
need to apply XPath as we did during the lecture to extract the number
of results returned by PubMed in the following web address:

<https://pubmed.ncbi.nlm.nih.gov/?term=sars-cov-2>

Complete the lines of code:

``` r
# Downloading the website
website <- xml2::read_html("https://pubmed.ncbi.nlm.nih.gov/?term=sars-cov-2")

# Finding the counts
counts <- xml2::xml_find_first(website, "/html/body/main/div[9]/div[2]/div[2]/div[1]/span")

# Turning it into text
counts <- as.character(counts)

# Extracting the data using regex
stringr::str_extract(counts, "[0-9,]+")
```

    ## [1] "114,781"

``` r
#This gives you the same result:
stringr::str_extract(counts, "[[:digit:],]+")
```

    ## [1] "114,781"

### Question 2: Academic publications on COVID19 and Hawaii

You need to query the following The parameters passed to the query are
documented here.

Use the function httr::GET() to make the following query:

Baseline URL:
<https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi>

Query parameters:

db: pubmed  
term: covid19 hawaii  
retmax: 1000

``` r
library(httr)
query_ids <- GET(
  url   = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi",
  query = list(     db = "pubmed",
                  term = "covid19 hawaii",
                retmax = 1000)
)

# Extracting the content of the response of GET
ids <- httr::content(query_ids)
```

Status: 200 means it succeeded.

``` r
query_ids
```

    ## Response [https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?db=pubmed&term=covid19%20hawaii&retmax=1000]
    ##   Date: 2021-10-08 23:11
    ##   Status: 200
    ##   Content-Type: text/xml; charset=UTF-8
    ##   Size: 4.28 kB
    ## <?xml version="1.0" encoding="UTF-8" ?>
    ## <!DOCTYPE eSearchResult PUBLIC "-//NLM//DTD esearch 20060628//EN" "https://eu...
    ## <eSearchResult><Count>150</Count><RetMax>150</RetMax><RetStart>0</RetStart><I...
    ## <Id>34562997</Id>
    ## <Id>34559481</Id>
    ## <Id>34545941</Id>
    ## <Id>34536350</Id>
    ## <Id>34532685</Id>
    ## <Id>34529634</Id>
    ## <Id>34499878</Id>
    ## ...

The query will return an XML object, we can turn it into a character
list to analyze the text directly with as.character(). Another way of
processing the data could be using lists with the function
xml2::as\_list(). We will skip the latter for now.

Take a look at the data, and continue with the next question (don’t
forget to commit and push your results to your GitHub repo!).

``` r
#ids_list <- xml2::as_list(ids)
```

### Question 3: Get details about the articles

The Ids are wrapped around text in the following way: <Id>… id number
…</Id>. we can use a regular expression that extract that information.
Fill out the following lines of code:

``` r
# Turn the result into a character vector
ids <- as.character(ids)

# Find all the ids 
ids <- stringr::str_extract_all(ids, "<Id>[[:digit:]]+</Id>")[[1]]

# Remove all the leading and trailing <Id> </Id>. Make use of "|"
ids <- stringr::str_remove_all(ids, "<Id>|</Id>")
head(ids)
```

    ## [1] "34562997" "34559481" "34545941" "34536350" "34532685" "34529634"

With the ids in hand, we can now try to get the abstracts of the papers.
As before, we will need to coerce the contents (results) to a list
using:

Baseline url:
<https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi>

Query parameters:

db: pubmed  
id: A character with all the ids separated by comma, e.g.,
“1232131,546464,13131”  
retmax: 1000  
rettype: abstract

Pro-tip: If you want GET() to take some element literal, wrap it around
I() (as you would do in a formula in R). For example, the text “123,456”
is replaced with “123%2C456”. If you don’t want that behavior, you would
need to do the following I(“123,456”).

``` r
query_pubs <- GET(
  url   = "https://eutils.ncbi.nlm.nih.gov/",
  path = "entrez/eutils/efetch.fcgi",
  query = list(     db = "pubmed",
                    id = I(paste(ids,collapse=",")),
                retmax = 1000,
                rettype= "abstract")
)
query_pubs
```

    ## Response [https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=pubmed&id=34562997,34559481,34545941,34536350,34532685,34529634,34499878,34491990,34481278,34473201,34448649,34417121,34406840,34391908,34367726,34355196,34352507,34334985,34314211,34308400,34308322,34291832,34287651,34287159,34283939,34254888,34228774,34226774,34210370,34195618,34189029,34183789,34183411,34183191,34180390,34140009,34125658,34108898,34102878,34091576,34062806,33990619,33982008,33980567,33973241,33971389,33966879,33938253,33929934,33926498,33900192,33897904,33894385,33889849,33889848,33859192,33856881,33851191,33826985,33789080,33781762,33781585,33775167,33770003,33769536,33746047,33728687,33718878,33717793,33706209,33661861,33661727,33657176,33655229,33607081,33606666,33606656,33587873,33495523,33482708,33471778,33464637,33442699,33422679,33422626,33417334,33407957,33331197,33316097,33308888,33301024,33276110,33270782,33251328,33244071,33236896,33229999,33216726,33193454,33186704,33176077,33139866,33098971,33096099,33087192,33083826,33043445,33027604,32984015,32969950,32921878,32914097,32914093,32912595,32907823,32907673,32891785,32888905,32881116,32837709,32763956,32763350,32745072,32742897,32692706,32690354,32680824,32666058,32649272,32596689,32592394,32584245,32501143,32486844,32462545,32432219,32432218,32432217,32427288,32420720,32386898,32371624,32371551,32361738,32326959,32323016,32314954,32300051,32259247,32151778&retmax=1000&rettype=abstract]
    ##   Date: 2021-10-08 23:12
    ##   Status: 200
    ##   Content-Type: text/xml; charset=UTF-8
    ##   Size: 1.9 MB
    ## <?xml version="1.0" ?>
    ## <!DOCTYPE PubmedArticleSet PUBLIC "-//NLM//DTD PubMedArticle, 1st January 201...

``` r
# Extracting the content of the response of GET
pubs <- httr::content(query_pubs)
pubs_txt <- as.character(pubs)
```

## Question 4: Distribution of universities, schools, and departments

Using the function stringr::str\_extract\_all() applied on
publications\_txt, capture all the terms of the form:

University of … … Institute of … Write a regular expression that
captures all such instances

The dash gets any names with dash in them.

``` r
institution <- str_extract_all(
  str_to_lower(pubs_txt),
  "university\\s+of\\s+(southern|new|northern|the|hong)?\\s*[[:alpha:]-]+|[[:alpha:]-]+\\s+institute\\s+of\\s+(southern|new|northern|the|marine)?\\s*[[:alpha:]-]+"
  ) 
```

``` r
institution <- unlist(institution)
table(institution)
```

    ## institution
    ##             australian institute of tropical 
    ##                                           13 
    ##            beijing institute of pharmacology 
    ##                                            2 
    ##                   berlin institute of health 
    ##                                            4 
    ##                   broad institute of harvard 
    ##                                            2 
    ##                    cancer institute of emory 
    ##                                            2 
    ##               cancer institute of new jersey 
    ##                                            1 
    ##                genome institute of singapore 
    ##                                            1 
    ##         graduate institute of rehabilitation 
    ##                                            3 
    ##              health institute of montpellier 
    ##                                            1 
    ##                i institute of marine biology 
    ##                                            1 
    ##                 leeds institute of rheumatic 
    ##                                            2 
    ##        massachusetts institute of technology 
    ##                                            1 
    ##               medanta institute of education 
    ##                                            1 
    ##      mediterranean institute of oceanography 
    ##                                            1 
    ##                      mgm institute of health 
    ##                                            1 
    ##            monterrey institute of technology 
    ##                                            1 
    ##                national institute of allergy 
    ##                                            1 
    ##          national institute of environmental 
    ##                                            3 
    ##                 national institute of public 
    ##                                            1 
    ##             national institute of technology 
    ##                                            1 
    ##             nordic institute of chiropractic 
    ##                                            1 
    ##            research institute of new zealand 
    ##                                            4 
    ##                  the institute of biomedical 
    ##                                            1 
    ##                    the institute of medicine 
    ##                                            1 
    ##                        university of alberta 
    ##                                            2 
    ##                        university of applied 
    ##                                            1 
    ##                        university of arizona 
    ##                                            5 
    ##                       university of arkansas 
    ##                                            1 
    ##                          university of basel 
    ##                                            8 
    ##                          university of benin 
    ##                                            1 
    ##                       university of botswana 
    ##                                            1 
    ##                       university of bradford 
    ##                                            1 
    ##                        university of bristol 
    ##                                            4 
    ##                        university of british 
    ##                                            4 
    ##                        university of calgary 
    ##                                            1 
    ##                     university of california 
    ##                                           65 
    ##                        university of chicago 
    ##                                           11 
    ##                     university of cincinnati 
    ##                                            9 
    ##                       university of colorado 
    ##                                            5 
    ##                    university of connecticut 
    ##                                            1 
    ##                     university of copenhagen 
    ##                                            1 
    ##                        university of córdoba 
    ##                                            1 
    ##                      university of education 
    ##                                            1 
    ##                         university of exeter 
    ##                                            1 
    ##                        university of florida 
    ##                                            5 
    ##                        university of granada 
    ##                                            2 
    ##                          university of haifa 
    ##                                            1 
    ##                          university of hawai 
    ##                                           92 
    ##                         university of hawaii 
    ##                                          180 
    ##                   university of hawaii-manoa 
    ##                                            2 
    ##                         university of health 
    ##                                            8 
    ##                      university of hong kong 
    ##                                            1 
    ##                       university of honolulu 
    ##                                            3 
    ##                       university of illinois 
    ##                                            1 
    ##                           university of iowa 
    ##                                            4 
    ##                      university of jerusalem 
    ##                                            1 
    ##                           university of juiz 
    ##                                            4 
    ##                         university of kansas 
    ##                                            1 
    ##                       university of kentucky 
    ##                                            1 
    ##                       university of lausanne 
    ##                                            1 
    ##                          university of leeds 
    ##                                            2 
    ##                     university of louisville 
    ##                                            1 
    ##                         university of malaya 
    ##                                            2 
    ##                       university of maryland 
    ##                                            9 
    ##                       university of medicine 
    ##                                            3 
    ##                      university of melbourne 
    ##                                            1 
    ##                          university of miami 
    ##                                            2 
    ##                       university of michigan 
    ##                                            8 
    ##                      university of minnesota 
    ##                                            4 
    ##                         university of murcia 
    ##                                            1 
    ##                       university of nebraska 
    ##                                            5 
    ##                         university of nevada 
    ##                                            1 
    ##                    university of new england 
    ##                                            1 
    ##                      university of new south 
    ##                                            3 
    ##                       university of new york 
    ##                                            3 
    ##            university of new york-university 
    ##                                            1 
    ##                          university of north 
    ##                                            2 
    ##                        university of ontario 
    ##                                            1 
    ##                           university of oslo 
    ##                                            6 
    ##                         university of ottawa 
    ##                                            1 
    ##                         university of oxford 
    ##                                            9 
    ##                          university of paris 
    ##                                            1 
    ##                   university of pennsylvania 
    ##                                           47 
    ##                     university of pittsburgh 
    ##                                           13 
    ##                          university of porto 
    ##                                            2 
    ##                         university of puerto 
    ##                                            2 
    ##                            university of rio 
    ##                                            1 
    ##                      university of rochester 
    ##                                            4 
    ##                            university of sao 
    ##                                            2 
    ##                        university of science 
    ##                                           13 
    ##                      university of singapore 
    ##                                            1 
    ##                          university of south 
    ##                                            4 
    ##            university of southern california 
    ##                                           21 
    ##               university of southern denmark 
    ##                                            1 
    ##                         university of sydney 
    ##                                            1 
    ##                     university of technology 
    ##                                            3 
    ##                          university of texas 
    ##                                            7 
    ##                     university of the health 
    ##                                           16 
    ##                university of the philippines 
    ##                                            1 
    ##                        university of toronto 
    ##                                            5 
    ##                         university of toulon 
    ##                                            1 
    ##                       university of tübingen 
    ##                                            3 
    ##                           university of utah 
    ##                                            4 
    ##                     university of washington 
    ##                                            6 
    ##                      university of wisconsin 
    ##                                            3 
    ## zoo-prophylactic institute of southern italy 
    ##                                            2

Repeat the exercise and this time focus on schools and departments in
the form of

School of … Department of … And tabulate the results

``` r
#schools_and_deps <- str_extract_all(
#  abstracts_txt,
#  "[YOUR REGULAR EXPRESSION HERE]"
#  )
#table(schools_and_deps)
```

## Question 5: Form a database

We want to build a dataset which includes the title and the abstract of
the paper. The title of all records is enclosed by the HTML tag
ArticleTitle, and the abstract by Abstract.

Before applying the functions to extract text directly, it will help to
process the XML a bit. We will use the xml2::xml\_children() function to
keep one element per id. This way, if a paper is missing the abstract,
or something else, we will be able to properly match PUBMED IDS with
their corresponding records.

``` r
pub_char_list <- xml2::xml_children(pubs)
pub_char_list <- sapply(pub_char_list, as.character)
```

``` r
#cat(pub_char_list[1])
```

Now, extract the abstract and article title for each one of the elements
of pub\_char\_list. You can either use sapply() as we just did, or
simply take advantage of vectorization of stringr::str\_extract

``` r
abstracts <- str_extract(pub_char_list, "<Abstract>[[:print:][:space:]]+</Abstract>")
View(cbind(abstracts))
```

``` r
abstracts <- str_remove_all(abstracts, "</?[[:alnum:]-=\"]+>")
abstracts <- str_replace_all(abstracts, "[[:space:]]+"," ")
```

How many of these don’t have an abstract? Now, the title

``` r
titles <- str_extract(pub_char_list, "<Title>[[:print:][:space:]]+</Title>")
titles <- str_remove_all(titles, "</?[[:alnum:]- =\"]+>")
```

Finally, put everything together into a single data.frame and use
knitr::kable to print the results

``` r
database <- data.frame(
  PubMedId = ids,
  Title    = titles,
  Abstract = abstracts
)
knitr::kable(database[1:20,], caption = "Some papers about Covid19 and Hawaii")
```

| PubMedId | Title                                        | Abstract                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
|:---------|:---------------------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 34562997 | Infectious disease reports                   | <AbstractText Label="INTRODUCTION" NlmCategory="BACKGROUND">Given that the success of vaccines against coronavirus disease 2019 (COVID-19) relies on herd immunity, identifying patients at risk for vaccine hesitancy is imperative-particularly for those at high risk for severe COVID-19 (i.e., minorities and patients with neurological disorders). <AbstractText Label="METHODS" NlmCategory="METHODS">Among patients from a large neuroscience institute in Hawaii, vaccine hesitancy was investigated in relation to over 30 sociodemographic variables and medical comorbidities, via a telephone quality improvement survey conducted between 23 January 2021 and 13 February 2021. <AbstractText Label="RESULTS" NlmCategory="RESULTS">Vaccine willingness (n = 363) was 81.3%. Univariate analysis identified that the odds of vaccine acceptance reduced for patients who do not regard COVID-19 as a severe illness, are of younger age, have a lower Charlson Comorbidity Index, use illicit drugs, or carry Medicaid insurance. Multivariable logistic regression identified the best predictors of vaccine hesitancy to be: social media use to obtain COVID-19 information, concerns regarding vaccine safety, self-perception of a preexisting medical condition contraindicated with vaccination, not having received the annual influenza vaccine, having some high school education only, being a current smoker, and not having a prior cerebrovascular accident. Unique amongst males, a conservative political view strongly predicted vaccine hesitancy. Specifically for Asians, a higher body mass index, while for Native Hawaiians and other Pacific Islanders (NHPI), a positive depression screen, both reduced the odds of vaccine acceptance. <AbstractText Label="CONCLUSION" NlmCategory="CONCLUSIONS">Upon identifying the variables associated with vaccine hesitancy amongst patients with neurological disorders, our clinic is now able to efficiently provide ancillary COVID-19 education to sub-populations at risk for vaccine hesitancy. While our results may be limited to the sub-population of patients with neurological disorders, the findings nonetheless provide valuable insight to understanding vaccine hesitancy. |
| 34559481 | The primary care companion for CNS disorders | NA                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| 34545941 | CA: a cancer journal for clinicians          |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |

Some papers about Covid19 and Hawaii

        CA Cancer J Clin
      
      Cancer statistics for the US Hispanic/Latino population, 2021.
      10.3322/caac.21695
      
        The Hispanic/Latino population is the second largest racial/ethnic group in the continental United States and Hawaii, accounting for 18% (60.6 million) of the total population. An additional 3 million Hispanic Americans live in Puerto Rico. Every 3 years, the American Cancer Society reports on cancer occurrence, risk factors, and screening for Hispanic individuals in the United States using the most recent population-based data. An estimated 176,600 new cancer cases and 46,500 cancer deaths will occur among Hispanic individuals in the continental United States and Hawaii in 2021. Compared to non-Hispanic Whites (NHWs), Hispanic men and women had 25%-30% lower incidence (2014-2018) and mortality (2015-2019) rates for all cancers combined and lower rates for the most common cancers, although this gap is diminishing. For example, the colorectal cancer (CRC) incidence rate ratio for Hispanic compared with NHW individuals narrowed from 0.75 (95% CI, 0.73-0.78) in 1995 to 0.91 (95% CI, 0.89-0.93) in 2018, reflecting delayed declines in CRC rates among Hispanic individuals in part because of slower uptake of screening. In contrast, Hispanic individuals have higher rates of infection-related cancers, including approximately two-fold higher incidence of liver and stomach cancer. Cervical cancer incidence is 32% higher among Hispanic women in the continental US and Hawaii and 78% higher among women in Puerto Rico compared to NHW women, yet is largely preventable through screening. Less access to care may be similarly reflected in the low prevalence of localized-stage breast cancer among Hispanic women, 59% versus 67% among NHW women. Evidence-based strategies for decreasing the cancer burden among the Hispanic population include the use of culturally appropriate lay health advisors and patient navigators and targeted, community-based intervention programs to facilitate access to screening and promote healthy behaviors. In addition, the impact of the COVID-19 pandemic on cancer trends and disparities in the Hispanic population should be closely monitored.
        © 2021 The Authors. CA: A Cancer Journal for Clinicians published by Wiley Periodicals LLC on behalf of American Cancer Society.
      
      
        
          Miller
          Kimberly D
          KD
          https://orcid.org/0000-0002-2609-2260
          
            Surveillance and Health Services Research, American Cancer Society, Atlanta, Georgia.
          
        
        
          Ortiz
          Ana P
          AP
          
            Cancer Control and Population Sciences, University of Puerto Rico Comprehensive Cancer Center, San Juan, Puerto Rico.
          
        
        
          Pinheiro
          Paulo S
          PS
          
            Sylvester Comprehensive Cancer Center, University of Miami Health System, Miami, Florida.
          
        
        
          Bandi
          Priti
          P
          
            Surveillance and Health Services Research, American Cancer Society, Atlanta, Georgia.
          
        
        
          Minihan
          Adair
          A
          
            Surveillance and Health Services Research, American Cancer Society, Atlanta, Georgia.
          
        
        
          Fuchs
          Hannah E
          HE
          
            Surveillance and Health Services Research, American Cancer Society, Atlanta, Georgia.
          
        
        
          Martinez Tyson
          Dinorah
          D
          
            College of Public Health, University of South Florida, Tampa, Florida.
          
        
        
          Tortolero-Luna
          Guillermo
          G
          
            Cancer Control and Population Sciences, University of Puerto Rico Comprehensive Cancer Center, San Juan, Puerto Rico.
          
        
        
          Fedewa
          Stacey A
          SA
          
            Surveillance and Health Services Research, American Cancer Society, Atlanta, Georgia.
          
        
        
          Jemal
          Ahmedin M
          AM
          
            Surveillance and Health Services Research, American Cancer Society, Atlanta, Georgia.
          
        
        
          Siegel
          Rebecca L
          RL
          https://orcid.org/0000-0001-5247-8522
          
            Surveillance and Health Services Research, American Cancer Society, Atlanta, Georgia.
          
        
      
      eng
      
        Journal Article
      
      
        2021
        09
        21
      


      United States
      CA Cancer J Clin
      0370647
      0007-9235

    AIM
    IM

      Hispanics
      Latinos
      statistics
      surveillance





        2021
        08
        04
      
      
        2021
        08
        04
      
      
        2021
        9
        21
        8
        46
      
      
        2021
        9
        22
        6
        0
      
      
        2021
        9
        22
        6
        0
      

    aheadofprint

      34545941
      10.3322/caac.21695


      References |The Hispanic/Latino population is the second largest racial/ethnic group in the continental United States and Hawaii, accounting for 18% (60.6 million) of the total population. An additional 3 million Hispanic Americans live in Puerto Rico. Every 3 years, the American Cancer Society reports on cancer occurrence, risk factors, and screening for Hispanic individuals in the United States using the most recent population-based data. An estimated 176,600 new cancer cases and 46,500 cancer deaths will occur among Hispanic individuals in the continental United States and Hawaii in 2021. Compared to non-Hispanic Whites (NHWs), Hispanic men and women had 25%-30% lower incidence (2014-2018) and mortality (2015-2019) rates for all cancers combined and lower rates for the most common cancers, although this gap is diminishing. For example, the colorectal cancer (CRC) incidence rate ratio for Hispanic compared with NHW individuals narrowed from 0.75 (95% CI, 0.73-0.78) in 1995 to 0.91 (95% CI, 0.89-0.93) in 2018, reflecting delayed declines in CRC rates among Hispanic individuals in part because of slower uptake of screening. In contrast, Hispanic individuals have higher rates of infection-related cancers, including approximately two-fold higher incidence of liver and stomach cancer. Cervical cancer incidence is 32% higher among Hispanic women in the continental US and Hawaii and 78% higher among women in Puerto Rico compared to NHW women, yet is largely preventable through screening. Less access to care may be similarly reflected in the low prevalence of localized-stage breast cancer among Hispanic women, 59% versus 67% among NHW women. Evidence-based strategies for decreasing the cancer burden among the Hispanic population include the use of culturally appropriate lay health advisors and patient navigators and targeted, community-based intervention programs to facilitate access to screening and promote healthy behaviors. In addition, the impact of the COVID-19 pandemic on cancer trends and disparities in the Hispanic population should be closely monitored. © 2021 The Authors. CA: A Cancer Journal for Clinicians published by Wiley Periodicals LLC on behalf of American Cancer Society.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |

\|34536350 \|The Lancet. Infectious diseases \|NA \| \|34532685 \|NAM
perspectives \|NA \| \|34529634 \|MMWR. Morbidity and mortality weekly
report \|Native Hawaiian and Pacific Islander populations have been
disproportionately affected by COVID-19 (1-3). Native Hawaiian, Pacific
Islander, and Asian populations vary in language; cultural practices;
and social, economic, and environmental experiences,† which can affect
health outcomes (4).§ However, data from these populations are often
aggregated in analyses. Although data aggregation is often used as an
approach to increase sample size and statistical power when analyzing
data from smaller population groups, it can limit the understanding of
disparities among diverse Native Hawaiian, Pacific Islander, and Asian
subpopulations¶ (4-7). To assess disparities in COVID-19 outcomes among
Native Hawaiian, Pacific Islander, and Asian populations, a
disaggregated, descriptive analysis, informed by recommendations from
these communities,\*\* was performed using race data from 21,005
COVID-19 cases and 449 COVID-19-associated deaths reported to the Hawaii
State Department of Health (HDOH) during March 1, 2020-February 28,
2021.†† In Hawaii, COVID-19 incidence and mortality rates per 100,000
population were 1,477 and 32, respectively during this period. In
analyses with race categories that were not mutually exclusive,
including persons of one race alone or in combination with one or more
races, Pacific Islander persons, who account for 5% of Hawaii’s
population, represented 22% of COVID-19 cases and deaths (COVID-19
incidence of 7,070 and mortality rate of 150). Native Hawaiian persons
experienced an incidence of 1,181 and a mortality rate of 15. Among
subcategories of Asian populations, the highest incidences were
experienced by Filipino persons (1,247) and Vietnamese persons (1,200).
Disaggregating Native Hawaiian, Pacific Islander, and Asian race data
can aid in identifying racial disparities among specific subpopulations
and highlights the importance of partnering with communities to develop
culturally responsive outreach teams§§ and tailored public health
interventions and vaccination campaigns to more effectively address
health disparities. \| \|34499878 \|Chest
\|<AbstractText Label="BACKGROUND" NlmCategory="BACKGROUND">Following
the publication of 2014 consensus statement regarding mass critical care
during public health emergencies, much has been learned about surge
responses and the care of overwhelming numbers of patients during the
COVID-19 pandemic.1 Gaps in prior pandemic planning were identified and
require modification in the midst of ongoing surge throughout the world.
<AbstractText Label="METHODS" NlmCategory="METHODS">The Task Force for
Mass Critical Care (TFMCC) adopted a modified version of established
rapid guideline methodologies from the World Health Organization2 and
the Guidelines International Network-McMaster Guideline Development
Checklist.3 With a consensus development process incorporating expert
opinion to define important questions and extract evidence, TFMCC
developed relevant pandemic surge suggestions in a structured manner,
incorporating peer-reviewed literature, “gray” evidence from lay media
sources, and anecdotal experiential evidence.
<AbstractText Label="RESULTS" NlmCategory="RESULTS">Ten suggestions were
identified regarding staffing, load-balancing, communication, and
technology. Staffing models are suggested with resilience strategies to
support critical care staff. Intensive care unit (ICU) surge strategies
and strain indicators are suggested to enhance ICU prioritization
tactics to maintain contingency level care and avoid crisis triage, with
early transfer strategies to further load-balance care. We suggest
intensivists and hospitalists be engaged with the incident command
structure to ensure two-way communication, situational awareness, and
the use of technology to support critical care delivery and families of
patients in intensive care units (ICUs).
<AbstractText Label="CONCLUSIONS" NlmCategory="CONCLUSIONS">A
subcommittee from the Task Force for Mass Critical Care offers interim
evidence-informed operational strategies to assist hospitals and
communities to plan for and respond to surge capacity demands from
COVID-19. Copyright © 2021. Published by Elsevier Inc. \| \|34491990
\|PLoS computational biology \|Accurate estimates of infection
prevalence and seroprevalence are essential for evaluating and informing
public health responses and vaccination coverage needed to address the
ongoing spread of COVID-19 in each United States (U.S.) state. However,
reliable, timely data based on representative population sampling are
unavailable, and reported case and test positivity rates are highly
biased. A simple data-driven Bayesian semi-empirical modeling framework
was developed and used to evaluate state-level prevalence and
seroprevalence of COVID-19 using daily reported cases and test
positivity ratios. The model was calibrated to and validated using
published state-wide seroprevalence data, and further compared against
two independent data-driven mathematical models. The prevalence of
undiagnosed COVID-19 infections is found to be well-approximated by a
geometrically weighted average of the positivity rate and the reported
case rate. Our model accurately fits state-level seroprevalence data
from across the U.S. Prevalence estimates of our semi-empirical model
compare favorably to those from two data-driven epidemiological models.
As of December 31, 2020, we estimate nation-wide a prevalence of 1.4%
\[Credible Interval (CrI): 1.0%-1.9%\] and a seroprevalence of 13.2%
\[CrI: 12.3%-14.2%\], with state-level prevalence ranging from 0.2%
\[CrI: 0.1%-0.3%\] in Hawaii to 2.8% \[CrI: 1.8%-4.1%\] in Tennessee,
and seroprevalence from 1.5% \[CrI: 1.2%-2.0%\] in Vermont to 23% \[CrI:
20%-28%\] in New York. Cumulatively, reported cases correspond to only
one third of actual infections. The use of this simple and
easy-to-communicate approach to estimating COVID-19 prevalence and
seroprevalence will improve the ability to make public health decisions
that effectively respond to the ongoing COVID-19 pandemic. \| \|34481278
\|Public health
\|<AbstractText Label="OBJECTIVES" NlmCategory="OBJECTIVE">During the
COVID-19 pandemic, the prevalence of psychological distress rose from
11% in 2019 to more than 40% in 2020. This study aims to examine the
disparities among US adult men and women.
<AbstractText Label="STUDY DESIGN" NlmCategory="METHODS">We used 21
waves of cross-sectional data from the Household Pulse Survey that were
collected between April and December 2020 for the study. The Household
Pulse Survey was developed by the U.S. Census Bureau to document the
social and economic impact of COVID-19.
<AbstractText Label="METHODS" NlmCategory="METHODS">The study population
included four groups of adults: emerging adults (18-24 years); young
adults (25-44 years); middle-aged adults (45-64 years); and older adults
(65-88 years). Psychological distress was measured by their Generalized
Anxiety Disorder score and the Patient Health Questionnaire. The
prevalence of psychological stress was calculated using logistic models
adjusted for socio-demographic variables including race/ethnicity,
education, household income, and household structure. All descriptive
and regression analysis considered survey weights.
<AbstractText Label="RESULTS" NlmCategory="RESULTS">Younger age groups
experienced higher prevalence of psychological distress than older age
groups. Among emerging adults, the prevalence of anxiety (42.6%) and
depression (39.5%) was more than twice as high as older adults who
experienced prevalence of anxiety at 20% and depression at 16.6%. Gender
differences were also more apparent in emerging adults. Women between 18
and 24 years reported higher differential rates of anxiety and
depression than those with men (anxiety: 43.9% vs. 28.3%; depression:
33.3% vs. 24.9%).
<AbstractText Label="CONCLUSION" NlmCategory="CONCLUSIONS">Understanding
the complex dynamics between COVID-19 and psychological distress has
emerged as a public health priority. Mitigating the negative mental
health consequences associated with the COVID-19 pandemic, for younger
generations and females in particular, will require local efforts to
rebuild capacity for social integration and social connection. Copyright
© 2021 The Royal Society for Public Health. Published by Elsevier
Ltd. All rights reserved. \| \|34473201 \|JAMA
\|<AbstractText Label="Importance" NlmCategory="UNASSIGNED">People who
have been infected with or vaccinated against SARS-CoV-2 have reduced
risk of subsequent infection, but the proportion of people in the US
with SARS-CoV-2 antibodies from infection or vaccination is uncertain.
<AbstractText Label="Objective" NlmCategory="UNASSIGNED">To estimate
trends in SARS-CoV-2 seroprevalence related to infection and vaccination
in the US population.
<AbstractText Label="Design, Setting, and Participants" NlmCategory="UNASSIGNED">In
a repeated cross-sectional study conducted each month during July 2020
through May 2021, 17 blood collection organizations with blood donations
from all 50 US states; Washington, DC; and Puerto Rico were organized
into 66 study-specific regions, representing a catchment of 74% of the
US population. For each study region, specimens from a median of
approximately 2000 blood donors were selected and tested each month; a
total of 1 594 363 specimens were initially selected and tested. The
final date of blood donation collection was May 31, 2021.
<AbstractText Label="Exposure" NlmCategory="UNASSIGNED">Calendar time.
<AbstractText Label="Main Outcomes and Measures" NlmCategory="UNASSIGNED">Proportion
of persons with detectable SARS-CoV-2 spike and nucleocapsid antibodies.
Seroprevalence was weighted for demographic differences between the
blood donor sample and general population. Infection-induced
seroprevalence was defined as the prevalence of the population with both
spike and nucleocapsid antibodies. Combined infection- and
vaccination-induced seroprevalence was defined as the prevalence of the
population with spike antibodies. The seroprevalence estimates were
compared with cumulative COVID-19 case report incidence rates.
<AbstractText Label="Results" NlmCategory="UNASSIGNED">Among 1 443 519
specimens included, 733 052 (50.8%) were from women, 174 842 (12.1%)
were from persons aged 16 to 29 years, 292 258 (20.2%) were from persons
aged 65 years and older, 36 654 (2.5%) were from non-Hispanic Black
persons, and 88 773 (6.1%) were from Hispanic persons. The overall
infection-induced SARS-CoV-2 seroprevalence estimate increased from 3.5%
(95% CI, 3.2%-3.8%) in July 2020 to 20.2% (95% CI, 19.9%-20.6%) in May
2021; the combined infection- and vaccination-induced seroprevalence
estimate in May 2021 was 83.3% (95% CI, 82.9%-83.7%). By May 2021, 2.1
SARS-CoV-2 infections (95% CI, 2.0-2.1) per reported COVID-19 case were
estimated to have occurred.
<AbstractText Label="Conclusions and Relevance" NlmCategory="UNASSIGNED">Based
on a sample of blood donations in the US from July 2020 through May
2021, vaccine- and infection-induced SARS-CoV-2 seroprevalence increased
over time and varied by age, race and ethnicity, and geographic region.
Despite weighting to adjust for demographic differences, these findings
from a national sample of blood donors may not be representative of the
entire US population. \| \|34448649 \|Clinical toxicology (Philadelphia,
Pa.) \|<AbstractText Label="OBJECTIVES" NlmCategory="UNASSIGNED">Our six
goals are: 1) describe the relationship between the National Strategy
for the COVID-19 Response and Pandemic Preparedness and the 55 US poison
centers (PCs); 2) detail FDA emergency Use Authorization (EUA) COVID-19
vaccine-related regulatory procedures and associated acronyms; 3) list
availability of specific vaccine clinical information to support PC
staff COVID-19 vaccination and adverse event (AE) data collection; 4)
describe required health care practitioner COVID-19 vaccine AE reporting
to the Vaccine AE Reporting System (VAERS) and PC reporting options; 5)
document public and health care professionals’ use of PCs for COVID-19
vaccine information; and 6) propose strategy to maximize PCs
contribution to the pandemic solution.
<AbstractText Label="METHODS" NlmCategory="UNASSIGNED">We reviewed
13-Feb-2020 through 15-Apr-2021 National Poison Data System (NPDS)
COVID-19 records for changes over time. We examined NPDS cases and VAERS
COVID-19 vaccine reports 1-Nov-2020 through 2-Apr-2021 for vaccine
manufacturer, patient characteristics, state, and clinical effects.
<AbstractText Label="RESULTS" NlmCategory="UNASSIGNED">PCs reported
1,052,174 COVID-19 contacts; maximum (peak) contacts/day (12,163) on
16-Mar-2020. As of 5-Apr-2021 the US reported &gt;167 million
administrations of COVID-19 vaccines (Pfizer-BioNTech, Moderna or
Janssen). US PCs reported 162,052 COVID-19 vaccine contacts. Most
(61.1%) were medical information calls, 34.9% were drug information, and
2.58% were exposures. Over the same period VAERS reported 49,078
COVID-19 vaccine cases reporting 226,205 symptoms - headache most
frequent, ranging from 20% to 40% across the 3 vaccines.
<AbstractText Label="CONCLUSIONS AND RECOMMENDATIONS" NlmCategory="UNASSIGNED">Although
differences exist between the intent and content of the 2 data sets,
NPDS volume is compelling. The PC nationwide 800 number facilitates data
collection and suggests comingling the 2 data streams has merit. PC
professionals received tens of thousands of calls and can: 1) support
fact-based vaccine information; 2) contribute vaccine AE follow-up
information: 3) advocate for best-case coding and reporting, especially
for vaccine adverse experiences. \| \|34417121 \|Paediatric respiratory
reviews \|Mathematical modelling has played a pivotal role in
understanding the epidemiology of and guiding public health responses to
the ongoing coronavirus disease of 2019 (COVID-19) pandemic. Here, we
review the role of epidemiological models in understanding evolving
epidemic characteristics, including the effects of vaccination and
Variants of Concern (VoC). We highlight ways in which models continue to
provide important insights, including (1) calculating the herd immunity
threshold and evaluating its limitations; (2) verifying that nascent
vaccines can prevent severe disease, infection, and transmission but may
be less efficacious against VoC; (3) determining optimal vaccine
allocation strategies under efficacy and supply constraints; and (4)
determining that VoC are more transmissible and lethal than previously
circulating strains, and that immune escape may jeopardize
vaccine-induced herd immunity. Finally, we explore how models can help
us anticipate and prepare for future stages of COVID-19 epidemiology
(and that of other diseases) through forecasts and scenario projections,
given current uncertainties and data limitations. Copyright © 2021.
Published by Elsevier Ltd. \| \|34406840 \|Health affairs (Project Hope)
\|COVID-19 vaccination campaigns continue in the United States, with the
expectation that vaccines will slow transmission of the virus, save
lives, and enable a return to normal life in due course. However, the
extent to which faster vaccine administration has affected
COVID-19-related deaths is unknown. We assessed the association between
US state-level vaccination rates and COVID-19 deaths during the first
five months of vaccine availability. We estimated that by May 9, 2021,
the US vaccination campaign was associated with a reduction of 139,393
COVID-19 deaths. The association varied in different states. In New
York, for example, vaccinations led to an estimated 11.7 fewer COVID-19
deaths per 10,000, whereas Hawaii observed the smallest reduction, with
an estimated 1.1 fewer deaths per 10,000. Overall, our analysis suggests
that the early COVID-19 vaccination campaign was associated with
reductions in COVID-19 deaths. As of May 9, 2021, reductions in COVID-19
deaths associated with vaccines had translated to value of statistical
life benefit ranging between $625 billion and $1.4 trillion. \|
\|34391908 \|International journal of infectious diseases : IJID :
official publication of the International Society for Infectious
Diseases \|<AbstractText Label="OBJECTIVES" NlmCategory="OBJECTIVE">To
evaluate the impact of the World Antimicrobial Awareness Week (WAAW) on
public awareness of antimicrobial resistance using Google Trends
analysis. <AbstractText Label="METHODS" NlmCategory="METHODS">The impact
of WAAW on public awareness of ‘antimicrobial resistance’ (AMR),
‘antibacterial’, and ‘antibiotics’ in Japan, the UK, the United States,
and worldwide from 2015 to 2020 was analyzed, using the relative search
volume (RSV) of Google Trends as a surrogate. A joinpoint regression
analysis was performed to identify a statistically significant time
point of a change in trend.
<AbstractText Label="RESULTS" NlmCategory="RESULTS">No joinpoints around
WAAW were identified in Japan, the United Kingdom, or the United States
from 2015 to 2020 with RSVs of ‘AMR’, whereas increasing RSVs were noted
worldwide in 2017 and 2020. Further, there were decreasing RSVs of
‘antibiotics’ in the first half of 2020, which could be due to the
COVID-19 pandemic. The study results suggest that WAAW did little to
improve public awareness of AMR in the selected countries despite its
contribution worldwide.
<AbstractText Label="CONCLUSIONS" NlmCategory="CONCLUSIONS">This study
implies that we need to develop a more effective method to improve
public awareness to fight against AMR. Copyright © 2021 The Author(s).
Published by Elsevier Ltd.. All rights reserved. \| \|34367726
\|Clinical & experimental pharmacology \|The impact of COVID-19 disease
on health and economy has been global, and the magnitude of devastation
is unparalleled in modern history. Any potential course of action to
manage this complex disease requires the systematic and efficient
analysis of data that can delineate the underlying pathogenesis. We have
developed a mathematical model of disease progression to predict the
clinical outcome, utilizing a set of causal factors known to contribute
to COVID-19 pathology such as age, comorbidities, and certain viral and
immunological parameters. Viral load and selected indicators of a
dysfunctional immune response, such as cytokines IL-6 and IFNα which
contribute to the cytokine storm and fever, parameters of inflammation
D-Dimer and Ferritin, aberrations in lymphocyte number, lymphopenia, and
neutralizing antibodies were included for the analysis. The model
provides a framework to unravel the multi-factorial complexities of the
immune response manifested in SARS-CoV-2 infected individuals. Further,
this model can be valuable to predict clinical outcome at an individual
level, and to develop strategies for allocating appropriate resources to
manage severe cases at a population level. \| \|34355196 \|Hawai’i
journal of health & social welfare \|Native Hawaiian and Pacific
Islander (NHPI) populations suffer from disproportionately higher rates
of chronic conditions, such as type 2 diabetes, that arises from
metabolic dysfunction and are often associated with obesity and
inflammation. In addition, the global coronavirus disease 2019 pandemic
has further compounded the effect of health inequities observed in
Indigenous populations, including NHPI communities. Reversible lifestyle
habits, such as diet, may either be protective of or contribute to the
increasing prevalence of health inequities in these populations via the
immunoepigenetic-microbiome axis. This axis offers insight into the
connection between diet, epigenetics, the microbiome composition, immune
function, and response to viral infection. Epigenetic mechanisms that
regulate inflammatory states associated with metabolic diseases,
including diabetes, are impacted by diet. Furthermore, diet may modulate
the gut microbiome by influencing microbial diversity and richness;
dysbiosis of the microbiome is associated with chronic disease. A high
fiber diet facilitates a favorable microbiome composition and in turn
increases production of intermediate metabolites named short-chain fatty
acids (SCFAs) that act on metabolic and immune pathways. In contrast,
low fiber diets typically associated with a westernized lifestyle
decreases the abundance of microbial derived SCFAs. This decreased
abundance is characteristic of metabolic syndromes and activation of
chronic inflammatory states, having larger implications in disease
pathogenesis of both communicable and non-communicable diseases. Native
Hawaiians and Pacific Islanders that once thrived on healthy traditional
diets may be more sensitive than non-indigenous peoples to the metabolic
perturbation of westernized diets that impinge on the
immunoepigenetic-gut microbiome axis. Recent studies conducted in the
Maunakea lab at the University of Hawai’i at Mānoa John A. Burns School
of Medicine have helped elucidate the connections between diet,
microbiome composition, metabolic syndrome, and epigenetic regulation of
immune function to better understand disease pathogenesis. Potentially,
this research could point to ways to prevent pre-disease conditions
through novel biomarker discovery using community-based approaches.
©Copyright 2021 by University Health Partners of Hawai‘i (UHP Hawai‘i).
\| \|34352507 \|Social science & medicine (1982) \|The United States
experienced three surges of COVID-19 community infection since the World
Health Organization declared the pandemic on March 11, 2020. The
prevalence of psychological distress among U.S. adults increased from 11
% in 2019 to 35.9 % in April 2020 when New York City become the
epicenter of the COVID-19 outbreak. Analyzing 21 waves of the Household
Pulse Survey data collected between April 2020 and December 2020, this
study aimed to examine the distress level in the 15 most populated
metropolitan areas in the U.S. Our study found that, as the pandemic
swept from East to South and soared in the West, 39.9%-52.3 % U.S.
adults living in these 15 metropolitan areas reported symptoms of
psychological distress. The highest distress levels were found within
the Western areas including Riverside-San Bernardino-Ontario (52.3 % in
July 2020, 95 % CI: 44.9%-59.6 %) and Los Angeles-Long Beach-Anaheim
(49.9 % in December 2020, 95 % CI: 44.5%-55.4 %). The lowest distress
level was observed in Washington-Arlington-Alexandria ranging from 29.1
% in May 2020 to 39.9 % in November 2020. COVID-19 and its complex
ecology of social and economic stressors have engaged high levels of
sustained psychological distress. Our findings will support the efforts
of local, state and national leadership to plan interventions by
addressing not only the medical, but also the economic and social
conditions associated with the pandemic. Copyright © 2021 Elsevier
Ltd. All rights reserved. \| \|34334985 \|Seminars in arthroplasty
\|<AbstractText Label="Background" NlmCategory="UNASSIGNED">Although the
COVID-19 pandemic has disrupted elective shoulder arthroplasty
throughput, traumatic shoulder arthroplasty procedures are less apt to
be postponed. We sought to evaluate shoulder arthroplasty utilization
for fracture during the COVID-19 pandemic and California’s associated
shelter-in-place order compared to historical controls.
<AbstractText Label="Methods" NlmCategory="UNASSIGNED">We conducted a
cohort study with historical controls, identifying patients who
underwent shoulder arthroplasty for proximal humerus fracture in
California using our integrated electronic health record. The time
period of interest was following the implementation of the statewide
shelter-in-place order: March 19, 2020-May 31, 2020. This was compared
to three historical periods: January 1, 2020-March 18, 2020, March 18,
2019-May 31, 2019, and January 1, 2019-March 18, 2019. Procedure volume,
patient characteristics, in-hospital length of stay, and 30-day events
(emergency department visit, readmission, infection, pneumonia, and
death) were reported. Changes over time were analyzed using linear
regression adjusted for usual seasonal and yearly changes and age, sex,
comorbidities, and postadmission factors.
<AbstractText Label="Results" NlmCategory="UNASSIGNED">Surgical volume
dropped from an average of 4.4, 5.2, and 2.6 surgeries per week in the
historical time periods, respectively, to 2.4 surgeries per week after
shelter-in-place. While no more than 30% of all shoulder arthroplasty
procedures performed during any given week were for fracture during the
historical time periods, arthroplasties performed for fracture was the
overwhelming primary indication immediately after the shelter-in-place
order. More patients were discharged the day of surgery (+33.2%, P =
.019) after the shelter-in-place order, but we did not observe a change
in any of the corresponding 30-day events.
<AbstractText Label="Conclusions" NlmCategory="UNASSIGNED">The volume of
shoulder arthroplasty for fracture dropped during the time of COVID-19.
The reduction in volume could be due to less shoulder trauma due to
shelter-in-place or a change in the indications for arthroplasty given
the perceived higher risks associated with intubation and surgical care.
We noted more patients undergoing shoulder arthroplasty for fracture
were safely discharged on the day of surgery, suggesting this may be a
safe practice that can be adopted moving forward.
<AbstractText Label="Level of Evidence" NlmCategory="UNASSIGNED">Level
III; Retrospective Case-control Comparative Study. © 2021 American
Shoulder and Elbow Surgeons. Published by Elsevier Inc. All rights
reserved. \| \|34314211 \|American journal of public health \|As of
March 2021, Native Hawaiians and Pacific Islanders (NHPIs) in the United
States have lost more than 800 lives to COVID-19-the highest per capita
death rate in 18 of 20 US states reporting NHPI deaths. However, NHPI
risks are overlooked in policy discussions. We discuss the NHPI COVID-19
Data Policy Lab and dashboard, featuring the disproportionate COVID-19
mortality burden for NHPIs. The Lab democratized NHPI data, developed
community infrastructure and resources, and informed testing site and
outreach policies related to health equity. \| \|34308400 \|The Lancet
regional health. Western Pacific
\|<AbstractText Label="Background" NlmCategory="UNASSIGNED">COVID-19
initially caused less severe outbreaks in many low- and middle-income
countries (LMIC) compared with many high-income countries, possibly
because of differing demographics, socioeconomics, surveillance, and
policy responses. Here, we investigate the role of multiple factors on
COVID-19 dynamics in the Philippines, a LMIC that has had a relatively
severe COVID-19 outbreak.
<AbstractText Label="Methods" NlmCategory="UNASSIGNED">We applied an
age-structured compartmental model that incorporated time-varying
mobility, testing, and personal protective behaviors (through a “Minimum
Health Standards” policy, MHS) to represent the first wave of the
Philippines COVID-19 epidemic nationally and for three highly affected
regions (Calabarzon, Central Visayas, and the National Capital Region).
We estimated effects of control measures, key epidemiological
parameters, and interventions.
<AbstractText Label="Findings" NlmCategory="UNASSIGNED">Population age
structure, contact rates, mobility, testing, and MHS were sufficient to
explain the Philippines epidemic based on the good fit between modelled
and reported cases, hospitalisations, and deaths. The model indicated
that MHS reduced the probability of transmission per contact by 13-27%.
The February 2021 case detection rate was estimated at \~8%, population
recovered at \~9%, and scenario projections indicated high sensitivity
to MHS adherence.
<AbstractText Label="Interpretation" NlmCategory="UNASSIGNED">COVID-19
dynamics in the Philippines are driven by age, contact structure,
mobility, and MHS adherence. Continued compliance with low-cost MHS
should help the Philippines control the epidemic until vaccines are
widely distributed, but disease resurgence may be occurring due to a
combination of low population immunity and detection rates and new
variants of concern. © 2021 Published by Elsevier Ltd. \|

Done! Knit the document, commit, and push.

Final Pro Tip (optional) You can still share the HTML document on
github. You can include a link in your README.md file as the following:

``` r
#View [here](https://ghcdn.rawgit.org/:user/:repo/:tag/:file)
```

For example, if we wanted to add a direct link the HTML page of lecture
7, we could do something like the following:

View
[here](https://ghcdn.rawgit.org/USCbiostats/PM566/master/website/static/slides/07-apis-regex/slides.html)

Knit the document, commit your changes, and Save it on GitHub. git
commit -a -m “Finalizing lab 7
<https://github.com/USCbiostats/PM566/issues/44>”

``` r
sessionInfo()
```

    ## R version 4.1.0 (2021-05-18)
    ## Platform: x86_64-apple-darwin17.0 (64-bit)
    ## Running under: macOS Mojave 10.14.6
    ## 
    ## Matrix products: default
    ## BLAS:   /Library/Frameworks/R.framework/Versions/4.1/Resources/lib/libRblas.dylib
    ## LAPACK: /Library/Frameworks/R.framework/Versions/4.1/Resources/lib/libRlapack.dylib
    ## 
    ## locale:
    ## [1] en_US.UTF-8/en_US.UTF-8/en_US.UTF-8/C/en_US.UTF-8/en_US.UTF-8
    ## 
    ## attached base packages:
    ## [1] stats     graphics  grDevices utils     datasets  methods   base     
    ## 
    ## other attached packages:
    ##  [1] httr_1.4.2      forcats_0.5.1   stringr_1.4.0   dplyr_1.0.7    
    ##  [5] purrr_0.3.4     readr_2.0.1     tidyr_1.1.3     tibble_3.1.4   
    ##  [9] ggplot2_3.3.5   tidyverse_1.3.1
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] tidyselect_1.1.1 xfun_0.25        haven_2.4.3      colorspace_2.0-2
    ##  [5] vctrs_0.3.8      generics_0.1.0   htmltools_0.5.2  yaml_2.2.1      
    ##  [9] utf8_1.2.2       rlang_0.4.11     pillar_1.6.2     glue_1.4.2      
    ## [13] withr_2.4.2      DBI_1.1.1        dbplyr_2.1.1     modelr_0.1.8    
    ## [17] readxl_1.3.1     lifecycle_1.0.0  munsell_0.5.0    gtable_0.3.0    
    ## [21] cellranger_1.1.0 rvest_1.0.1      evaluate_0.14    knitr_1.34      
    ## [25] tzdb_0.1.2       fastmap_1.1.0    curl_4.3.2       fansi_0.5.0     
    ## [29] highr_0.9        broom_0.7.9      Rcpp_1.0.7       scales_1.1.1    
    ## [33] backports_1.2.1  jsonlite_1.7.2   fs_1.5.0         hms_1.1.0       
    ## [37] digest_0.6.27    stringi_1.7.4    grid_4.1.0       cli_3.0.1       
    ## [41] tools_4.1.0      magrittr_2.0.1   crayon_1.4.1     pkgconfig_2.0.3 
    ## [45] ellipsis_0.3.2   xml2_1.3.2       reprex_2.0.1     lubridate_1.7.10
    ## [49] rstudioapi_0.13  assertthat_0.2.1 rmarkdown_2.10   R6_2.5.1        
    ## [53] compiler_4.1.0
