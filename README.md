
googleapis
----------

R client for interacting with [Google Cloud Natural Language APIs](https://cloud.google.com/natural-language/docs/basics).

This is a work in progress. To date, this package only interacts with the `analyzeSentiment` API. If this project interests you, please let me know.

### Getting started with Google SDK

Google's [documentation is fantastic](https://cloud.google.com/natural-language/docs/getting-started). Click the link below for a quick start guide.

-   <https://cloud.google.com/natural-language/docs/getting-started>

Before you can use Google's NLP APIs, you'll need an API key. For that, I recommend following the quick start guide linked above. It'll basically go something like this:

1.  Setup your Google software development account (***Using the API is free up to a point, but you should be able to get a $300.00 credit for signing up as well***). You may also need to "enable billing," but I promise that doesn't mean you're sinking any money into this thing--they clearly are trying to sell you on how awesome it is rather than tricking you into forking over cash.
2.  Enable the Natural Language API. There should be an option to "create credentials." For use in interactive R sessions, you'll just need an API key (a long string of text), which you should be able to get using the defaults provided by Google.

The first time you use any of the functions from the googleapis package, you'll be asked for your API key. It'll be set as an environment variable and saved for future sessions, so you should only have to do this once (per machine).

### install package

Install `googleapis` using the [devtools](https://github.com/hadley/devtools) package.

``` r
if (!"devtools" %in% installed.packages()) {
  install.packages("devtools")
}
devtools::install_github("mkearney/googleapis")
```

DEMO
----

### analyzeSentiment of Twitter statuses

Download tweets using the `rtweet` package, which can be installed [from CRAN](https://cran.r-project.org/package=rtweet)

``` r
install.package("rtweet")
```

or the [dev version from Github](https://github.com/mkearney/rtweet).

``` r
devtools::install_github("mkearney/rtweet")
```

Use `rtweet` to download 100 non-retweeted Twitter statuses about Trump (for best results specify single language).

``` r
trump <- rtweet::search_tweets("Trump lang:en", include_rts = FALSE)
```

Prep the text by **first** removing URLs, AT (@) mentions, and line breaks

``` r
trump_tweets <- gsub(
  "@\\S{1,}|http\\S{1,}|\\n", "", trump$text
)
```

**Second**, because Google's sentiment analysis API will actually return sentiment scores for every sentence, remove any periods at the start of tweets.

``` r
trump_tweets <- gsub(
  "^\\.\\S{0,}", "", trump_tweets
)
```

**Third**, remove all extra spaces (for the record, I'm not entirely sure if this matters, but if you're going to be analyzing individual words at any point, it's not a bad idea).

``` r
trump_tweets <- trimws(gsub(
  "[ ]{2,}", " ", trump_tweets
))
```

Here's what the tweets look like for me:

``` r
head(trump_tweets)
```

    ## [1] "Warned? Sounds like horror coming Trump is different but didn't need warning ahead of victory"                                           
    ## [2] "Collusion yes Russian hacking yes Trump lying yes obstruction yes Trump collusion =Treason and Traitor obstruction against the law felon"
    ## [3] "And had some mysterious creature captured Trump's mouth and made him rant his birtherism tirades whenever he had an audience?"           
    ## [4] "regardless, the point that she was qualified to be president stands. As does the point that Trump/Breitbart is making journos the enemy" 
    ## [5] "Chief critical of incitement against press &amp; worrying remarks re: minorities"                                                        
    ## [6] "Kathy Griffin Says She Ended Her Friendship with Anderson Cooper After He Waited Four Months to Reach Out in the..."

With the text ready to be analyzed, load `googleapis` if you haven't already.

``` r
library(googleapis)
```

Conduct sentiment analysis on the character vector of tweets (text) using the `analyze_sentiment()` function.

``` r
sa <- analyze_sentiment(trump_tweets)
```

The output is a bit messy, but it's easily converted into a tidy data frame using the `as.data.frame` or `as_tibble` functions.

``` r
tibble::as_tibble(sa)
```

    ## # A tibble: 154 x 6
    ##          id document sentence offset score
    ##       <chr>    <dbl>    <int>  <int> <dbl>
    ##  1 b9VrlU8Z      0.0        1      0   0.0
    ##  2 b9VrlU8Z      0.0        2      8  -0.1
    ##  3 isnYeUGN     -0.2        1      0  -0.2
    ##  4 jaz5GWJ0     -0.1        1      0  -0.1
    ##  5 t354oUkD      0.2        1      0   0.3
    ##  6 t354oUkD      0.2        2     69   0.1
    ##  7 uZSrGOxB     -0.1        1      0  -0.1
    ##  8 9MTGAmyg     -0.1        1      0  -0.1
    ##  9 LtKT33G9     -0.4        1      0  -0.3
    ## 10 LtKT33G9     -0.4        2     83  -0.5
    ## # ... with 144 more rows, and 1 more variables: content <chr>

Each row in the converted data frame represents one sentence of the provided text. For each observation, there are six features (variables):

-   `id` Randomly generated ID (assigned to each tweet)
-   `document` Overall sentiment score of the document---in this case, a document is one whole tweet
-   `sentence` The ordinal position of a given sentence as a sequence within a single tweet (e.g., first sentence of a document, second sentence of a document)
-   `offset` The position, in number of characters, from which the sentence started within a document
-   `score` The sentiment (along positive and negative dimensions) score of the sentence
-   `content` The text of the analyzed sentence
