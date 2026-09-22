# What Factors are Associated With the Popularity of Games on Steam?
## Project Overview
This project demonstrates how web scraping can be used to collect and prepare data from the internet for analysis.

Using the Steam Store as a data source, information on `50` games was collected using `requests` and `BeautifulSoup`, then the data was cleaned and preprocessed using pandas before being put into a dataframe.

The collected data was used to explore whether factors such as game price, release date, genre or platform are associated with a game's popularity. In this analysis, the target variable `review_count` was used as a proxy for 'popularity', as it is a direct measure of engagement.

The primary focus of this project is data acquisition and preparation, with the supporting exploratory analysis being used to demonstrate how scraped data can be used for analysis.

## Methodology
The project covers:

* Sending HTTP requests using requests
* Parsing HTML using BeautifulSoup
* Extracting data from Steam search results
* Making additional requests to individual game pages to collect genres
* Cleaning and transforming scraped data using Pandas
* Extracting review information from raw HTML
* Converting variables to appropriate data types
* Exploratory data analysis
* Using log scales to examine highly skewed review counts
* Spearman rank correlation
* Comparing categorical variables using median review counts

## Data Acquisition
The dataset was collected directly from Steam's search results using the `requests` library. It returned HTML which was parsed using `BeautifulSoup`. Information including game title, release date, price, supported platforms and review information was extracted from the search result page. 

Steam's search results did not give genre information, so an additional request had to be made to each game's individual store page. The genre information was then extracted from the HTML of each game's store page. 

The scraped data was stored as dictionaries before being converted into a Pandas dataframe for cleaning and analysis. 

``` python
#initialize soup
soup = BeautifulSoup(response.text, "html.parser")


# find all of the html containers for each game
games = soup.find_all('a', class_="search_result_row")
#print the number of games 
print(len(games))

# create empty list to store the games and their data 
data = []

#loop through games and use soup to extract the variable for each game
for game in games:
    # Title
    title = game.find('span', class_='title').text.strip()
    # Release date
    release_date = game.find('div', class_='search_released').text.strip()
    # URL
    url = game['href']
    # App ID
    app_id = game['data-ds-appid']
    # Price
    price = game.find('div', class_='discount_final_price').text.strip()
    # Platforms
    platform = game.find('div', class_='search_platforms')
    platforms = platform.find_all('span')
    platform_names = []

    for p in platforms:
        classes = p.get('class', [])

        for class_name in classes:
            if class_name in ['win', 'mac', 'linux']:
                platform_names.append(str(class_name))
    # Review information
    review = game.find('span', class_='search_review_summary')
    review_info = review['data-tooltip-html']


    # game url 
    game_url = game['href']
    game_response = requests.get(game_url)
    game_soup = BeautifulSoup(game_response.text, 'html.parser')
    # genre
    genres = game_soup.select('a[href*="/genre/"]') 
    game_genres = []
    for genre in genres:
        game_genres.append(genre.text.strip())



    data.append({
        'title': title,
        'release_date': release_date,
        'url': url,
        'app_id': app_id,
        'price': price,
        'review_info': review_info,
        'platforms': platform_names,
        'genres': game_genres})
# Create a dataframe out of the scraped data

games_df = pd.DataFrame(data)
pd.set_option('display.max_colwidth', None)
games_df.head()
# Join the platforms into one string
games_df['platforms'] = games_df['platforms'].apply(', '.join)
# Join the genres into one string
games_df['genres'] = games_df['genres'].apply(', '.join)
```

The final dataset contains 50 games and includes the following variables:

| Variable | Description |
|---|---|
| `title` | Game title |
| `release_date` | Game release date |
| `price_GBP` | Game price in GBP |
| `platforms` | Supported operating systems |
| `genres` | Game genres |
| `review_score` | Steam’s review classification |
| `review_count` | Number of user reviews |

## Key Findings
### Price
There was virtually no association between game price and review count.
* Spearman p = 0.019
* Popular games appeared across a wide range of price listings, including free to play games.

### Release date
There was a weak negative correlation between release data and review count
* Spearman p = -0.298
* p-value = 0.0359

However, older games naturally have more time to accumulate reviews, so this association should not be viewed as evidence that older games are more popular.

### Review score
All 50 games had a review of `Overwhelmingly Positive`. As there was no variation in this variable, it could not be used to distinguish any differences in popularity of games.

### Genre
Differences were seen between the median review count between genres. Among genres that had at least `5` titles, games that we listed as being `strategy` or `free to play` had the highest median review counts.
However, games can belong to multiple genres and a few genres had relatively few listings, which limits the reliability of this interpretation. 

### Platform
Games that supported `Windows, Mac and Linux` had the highest median review count among the platform groups. 

## Limitations
This analysis has several important limitations.

Firstly, the dataset only contained `50` games and was collected frmo the Steam search results that were already ordered by review count. The sample is therefore not randomly selected and should not be considered as being representative of the wider Steam Store. 

Secondly, `review_count` is being used as a proxy for 'popularity'. Not every player leaves a review, and not every review is a positive one. 

Thirdly, older games have had more time to accumulate reviews, which makes it difficiult to separate the effect of the release date from the additional time that older games have had to accumulate reviews.

The categorical variables also have different sample sizes. Given that this is a small dataset overall, categories that contain relatively few games give limited evidence when comparing their averages.

Finally, this analysis only identifies **associations** as opposed to causal relationships. Factors such as marketing, game quality, release timing (in relation to other games) and developer reputation were not included.

## Tools Used
* Python
* Pandas
* NumPy
* Matplotlib
* Seaborn
* Requests
* BeautifulSoup
* SciPy
* Jupyter Notebook
