# -Exploring-Population-Growth-Prediction-using-Web-Scraping-
'''In my latest project, I scraped data from Wikipedia and Worldometers using Python's BeautifulSoup and analyzed it to predict future population growth for various countries over the next 10 years. This exercise not only improved my web scraping skills but also helped me implement predictive modeling.

Here's a brief overview of the process:

🌐 Web Scraping: Collected fertility rate and population data from Wikipedia and Worldometers using the requests library and BeautifulSoup for HTML parsing.
📊 Data Analysis: Cleaned and organized the data using pandas for further analysis.
🔮 Population Prediction: Applied growth rates to forecast population for the next decade, visualizing predictions for countries like China, India, and the USA.
📈 Visualization: Plotted the population trends using matplotlib for clear insights.'''

###First we'll scrape Fertility rate data from a webpage and make it ready for analysis.###
import requests
from bs4 import BeautifulSoup
import pandas as pd

# Wikipedia page URL for fertility rate data
url = 'https://en.wikipedia.org/wiki/List_of_countries_by_total_fertility_rate'

# Fetch the page content
response = requests.get(url)

# Check if the request was successful
if response.status_code == 200:
    # Parse the HTML using BeautifulSoup
    soup = BeautifulSoup(response.content, 'html.parser')

    # Find the table containing fertility rate data
    table = soup.find('table', {'class': 'wikitable'})

    # Extract table rows
    rows = table.find_all('tr')

    # Prepare a list to store the data
    data = []

    # Iterate through the rows and extract country names and fertility rates
    for row in rows[1:]:  # Skip the header row
        cols = row.find_all('td')
        if len(cols) > 1:
            # Extract country name and fertility rate correctly
            country = cols[1].get_text(strip=True)  # Country name is in the second column
            fertility_rate = cols[2].get_text(strip=True)  # Fertility rate is in the third column
            data.append([country, fertility_rate])

    # Create a pandas DataFrame
    df = pd.DataFrame(data, columns=['Country', 'Fertility Rate'])

    # Display the first few rows
    print(df.head())

else:
    print(f"Failed to retrieve the webpage. Status code: {response.status_code}")


    ### Now we'll scrape the data about current world population , migration rate and all the relevent information ###
    import requests
from bs4 import BeautifulSoup
import pandas as pd

# URL of the page to scrape
url = 'https://www.worldometers.info/world-population/population-by-country/'

# Send a request to fetch the content of the webpage
response = requests.get(url)

# Check if the request was successful
if response.status_code == 200:
    # Parse the HTML content using BeautifulSoup
    soup = BeautifulSoup(response.content, 'html.parser')
    
    # Find the table containing population data
    table = soup.find('table', {'id': 'example2'})
    
    # Extract the rows of the table
    rows = table.find_all('tr')

    # Prepare an empty list to store the data
    population_data = []

    # Iterate over each row in the table (skipping the header row)
    for row in rows[1:]:
        cols = row.find_all('td')
        country = cols[1].get_text(strip=True)
        population = cols[2].get_text(strip=True).replace(',', '')
        yearly_change = cols[3].get_text(strip=True)
        net_change = cols[4].get_text(strip=True).replace(',', '')
        density = cols[5].get_text(strip=True)
        land_area = cols[6].get_text(strip=True).replace(',', '')

        # Append the extracted data to the list
        population_data.append([country, population, yearly_change, net_change, density, land_area])

    # Create a DataFrame using pandas
    columns = ['Country', 'Population', 'Yearly Change', 'Net Change', 'Density (P/Km²)', 'Land Area (Km²)']
    population_df = pd.DataFrame(population_data, columns=columns)

    # Convert population and land area to numeric for analysis
    population_df['Population'] = pd.to_numeric(population_df['Population'])
    population_df['Land Area (Km²)'] = pd.to_numeric(population_df['Land Area (Km²)'])

    # Display the first few rows of the DataFrame
    print(population_df.head())

else:
    print(f"Failed to retrieve the webpage. Status code: {response.status_code}")

### now we will perform analysis on the collected data and visualizations for clear undarstandings ###
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

# Let's assume population_df is the DataFrame from the previous step
# Convert the 'Yearly Change' to numeric values and remove '%' sign
population_df['Yearly Change'] = population_df['Yearly Change'].str.rstrip('%').astype('float') / 100

# Define the number of years for prediction
years_ahead = 10

# Create a dictionary to store future population predictions
prediction_dict = {'Year': np.arange(2024, 2024 + years_ahead)}

# Predict future population for each country
for index, row in population_df.iterrows():
    current_population = row['Population']
    yearly_growth_rate = row['Yearly Change']
    
    # Predict population for each year ahead
    future_populations = [
        current_population * (1 + yearly_growth_rate) ** year for year in range(1, years_ahead + 1)
    ]
    
    # Add future populations for this country to the dictionary
    prediction_dict[row['Country']] = future_populations

# Convert the dictionary into a DataFrame
prediction_df = pd.DataFrame(prediction_dict)

# Set the 'Year' column as index
prediction_df.set_index('Year', inplace=True)

# Plotting predictions for a few countries
plt.figure(figsize=(10, 6))
for country in ['China', 'India', 'United States', 'Indonesia', 'Brazil']:
    if country in prediction_df.columns:
        plt.plot(prediction_df.index, prediction_df[country], label=country)

plt.title('Predicted Population Growth (2024-2034)')
plt.xlabel('Year')
plt.ylabel('Population')
plt.legend()
plt.grid(True)
plt.show()

###thats it , for more accuracy we can add more parameters like average live expectancy and various other parameters but this code will give you the gist of the approch we can use ###

