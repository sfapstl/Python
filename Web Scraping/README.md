# Python Web Scraping Project

In this project, I scraped data from a webpage containing a list of the 100 largest companies in the United States.

First, I imported the BeautifulSoup and Requests libraries for web scraping.

```python
from bs4 import BeautifulSoup
import requests

page = requests.get('https://en.wikipedia.org/wiki/List_of_largest_companies_in_the_United_States_by_revenue')
soup = BeautifulSoup(page.text, 'html')

print(soup)
```

After printing `soup`, I notice that the table I need is enclosed in a `<table>` tag. I then locate the table in the html containing the needed data.

```python
# Locate table containing needed data

soup.find_all('table')[1]
```
```html
<table class="wikitable sortable">
<caption>
</caption>
<tbody><tr>
<th>Rank
</th>
<th>Name
</th>
<th>Industry
</th>
<th>Revenue <br/>(USD millions)
</th>
<th>Revenue growth
</th>
<th>Employees
</th>
<th>Headquarters
</th></tr>
.
.
.
```
The table is in the first `[1]` position using find_all. I turn this result into the `table` variable.

```python
# Isolate the table containing the needed data

table = soup.find_all('table')[1]

print(table)
```
In the HTML file, the column names are enclosed in `<th>` tags
```python
# Locate the column names

table.find_all('th')
```
```html
[<th>Rank
 </th>,
 <th>Name
 </th>,
 <th>Industry
 </th>,
 <th>Revenue <br/>(USD millions)
 </th>,
 <th>Revenue growth
 </th>,
 <th>Employees
 </th>,
 <th>Headquarters
 </th>]
```
Isolating the column names in the `world_table ` variable and inserting them into the `world_table_titles` list:
```python
world_table = table.find_all('th')

# Insert the column titles into a list

world_table_titles = [title.text.strip() for title in world_table]

print(world_table_titles)

['Rank', 'Name', 'Industry', 'Revenue (USD millions)', 'Revenue growth', 'Employees', 'Headquarters']
```
Then I import the pandas library to assign the column names to a DataFrame
```python
# Import pandas to assign column names to a DataFrame

import pandas as pd

# Assign the column titles in the list as column titles in the DataFrame

df = pd.DataFrame(columns = world_table_titles)

df

| Rank | Name | Industry | Revenue (USD millions) | Revenue growth | Employees | Headquarters |
|------|------|----------|------------------------|----------------|-----------|--------------|

```
Now, locate the values for the columns. In the HTML, the column values are contained in the `tr` tag.

```python
# Locating the values for the columns

table.find_all('tr')
```
```html
[<tr>
 <th>Rank
 </th>
 <th>Name
 </th>
 <th>Industry
 </th>
 <th>Revenue <br/>(USD millions)
 </th>
 <th>Revenue growth
 </th>
 <th>Employees
 </th>
 <th>Headquarters
 </th></tr>,
 <tr>
 <td>1
 </td>
 <td><a href="/wiki/Walmart" title="Walmart">Walmart</a>
 </td>
 <td><a href="/wiki/Retail" title="Retail">Retail</a>
 </td>
 <td style="text-align:center;">611,289
 </td>
 <td style="text-align:center;"><span typeof="mw:File"><span title="Increase"><img alt="Increase" class="mw-file-element" data-file-height="300" data-file-width="300" decoding="async" height="11" src="//upload.wikimedia.org/wikipedia/commons/thumb/b/b0/Increase2.svg/11px-Increase2.svg.png" srcset="//upload.wikimedia.org/wikipedia/commons/thumb/b/b0/Increase2.svg/17px-Increase2.svg.png 1.5x, //upload.wikimedia.org/wikipedia/commons/thumb/b/b0/Increase2.svg/22px-Increase2.svg.png 2x" width="11"/></span></span> <span data-sort-value="7000300000000000000♠" style="display:none"></span> 6.7%
 </td>
 <td style="text-align:center;">2,100,000
 </td>
 <td><a href="/wiki/Bentonville,_Arkansas" title="Bentonville, Arkansas">Bentonville, Arkansas</a>
 </td></tr>
```
Assigning the columns values to the `column_data` variable:
```python
column_data = table.find_all('tr')

column_data
```
Now I transfer the column values into the DataFrame using a loop.
```python
# Using a loop to transfer data to new rows in the DataFrame

for row in column_data[1:]: # Exclude the column names inside <td></td> by specifying position
    row_data = row.find_all('td')
    individual_row_data = [data.text.strip() for data in row_data]
    if row.find_all('span', title = 'Decrease'): # The table contains colored triangle images to indicate whether revenue growth is negative (red inverted triangle) or positive (green triangle)
        individual_row_data[4] = '-' + individual_row_data[4] # If the revenue growth is negative, add a minus sign  

    length = len(df)
    df.loc[length] = individual_row_data
```
```python
# Checking whether the data was transferred to the DataFrame successfully

df.head()
```
| Rank | Name               | Industry               | Revenue (USD millions) | Revenue growth | Employees | Headquarters            |
|------|--------------------|------------------------|------------------------|----------------|-----------|-------------------------|
| 1    | Walmart            | Retail                 | 611,289                | 6.7%           | 2,100,000 | Bentonville, Arkansas   |
| 2    | Amazon             | Retail and cloud computing | 513,983            | 9.4%           | 1,540,000 | Seattle, Washington     |
| 3    | ExxonMobil         | Petroleum industry     | 413,680                | 44.8%          | 62,000    | Spring, Texas           |
| 4    | Apple              | Electronics industry   | 394,328                | 7.8%           | 164,000   | Cupertino, California   |
| 5    | UnitedHealth Group | Healthcare             | 324,162                | 12.7%          | 400,000   | Minnetonka, Minnesota   |

```python
# Checking whether the loop to add a minus sign for negative revenue growth worked correctly

# Revenue growth is -10.7%, loop worked successfully

df.iloc[26]

Rank                                            27
Name                      Walgreens Boots Alliance
Industry                   Pharmaceutical industry
Revenue (USD millions)                     132,703
Revenue growth                              -10.7%
Employees                                  262,500
Headquarters                   Deerfield, Illinois
Name: 26, dtype: object
```
The data seems to have been inputted into the DataFrame successfully.

## Exploratory Data Cleaning

The data still needs to be cleaned and prepared for analysis and visualization later in Power BI. For example, the numerical values contain special characters.
```python
# Remove special characters from columns with numeric values

df['Revenue (USD millions)'] = df['Revenue (USD millions)'].str.replace(',', '')
df['Revenue growth'] = df['Revenue growth'].str.replace('%', '')
df['Employees'] = df['Employees'].str.replace(',', '')
df['Employees'] = df['Employees'].str.replace('[2]', '')

df.head()
```
| Rank | Name                | Industry                    | Revenue (USD millions) | Revenue growth | Employees | Headquarters           |
|------|---------------------|-----------------------------|------------------------|----------------|-----------|------------------------|
| 1    | Walmart             | Retail                      | 611,289                | 6.7            | 2,100,000 | Bentonville, Arkansas |
| 2    | Amazon              | Retail and cloud computing | 513,983                | 9.4            | 1,540,000 | Seattle, Washington   |
| 3    | ExxonMobil          | Petroleum industry          | 413,680                | 44.8           | 62,000    | Spring, Texas          |
| 4    | Apple               | Electronics industry        | 394,328                | 7.8            | 164,000   | Cupertino, California |
| 5    | UnitedHealth Group  | Healthcare                  | 324,162                | 12.7           | 400,000   | Minnetonka, Minnesota |

Now, checking the column data types.
```python
# Checking column data types

print(df.dtypes)

Rank                      object
Name                      object
Industry                  object
Revenue (USD millions)    object
Revenue growth            object
Employees                 object
Headquarters              object
dtype: object
```
Data types need to be changed for analysis using pandas.
```python
# Change data types

df = df.astype({'Rank': int, 
                'Name': str, 
                'Industry': str, 
                'Revenue (USD millions)': int, 
                'Revenue growth': float, 
                'Employees': int, 
                'Headquarters': str})

print(df.dtypes)

Rank                        int32
Name                       object
Industry                   object
Revenue (USD millions)      int32
Revenue growth            float64
Employees                   int32
Headquarters               object
dtype: object
```
Further data cleaning below:
```python
# Convert 'Revenue growth' column from percentage to decimal

df['Revenue growth'] = df['Revenue growth'] / 100

df.head()
```
| Rank | Name                | Industry                    | Revenue (USD millions) | Revenue growth | Employees | Headquarters           |
|------|---------------------|-----------------------------|------------------------|----------------|-----------|------------------------|
| 1    | Walmart             | Retail                      | 611,289                | 6.7%           | 2,100,000 | Bentonville, Arkansas |
| 2    | Amazon              | Retail and cloud computing | 513,983                | 9.4%           | 1,540,000 | Seattle, Washington   |
| 3    | ExxonMobil          | Petroleum industry          | 413,680                | 44.8%          | 62,000    | Spring, Texas          |
| 4    | Apple               | Electronics industry        | 394,328                | 7.8%           | 164,000   | Cupertino, California |
| 5    | UnitedHealth Group  | Healthcare                  | 324,162                | 12.7%          | 400,000   | Minnetonka, Minnesota |

```python
# Drop the 'Rank' column, unnecessary for analysis

df.drop('Rank', axis=1, inplace=True)

df.head()
```
| Name                | Industry                    | Revenue (USD millions) | Revenue growth | Employees | Headquarters           |
|---------------------|-----------------------------|------------------------|----------------|-----------|------------------------|
| Walmart             | Retail                      | 611,289                | 6.7%           | 2,100,000 | Bentonville, Arkansas |
| Amazon              | Retail and cloud computing | 513,983                | 9.4%           | 1,540,000 | Seattle, Washington   |
| ExxonMobil          | Petroleum industry          | 413,680                | 44.8%          | 62,000    | Spring, Texas          |
| Apple               | Electronics industry        | 394,328                | 7.8%           | 164,000   | Cupertino, California |
| UnitedHealth Group | Healthcare                  | 324,162                | 12.7%          | 400,000   | Minnetonka, Minnesota |

```python
# Extract the states only from the 'Headquarters' column

df['Headquarters'] = df['Headquarters'].str.split(',').str.get(-1)

df.head()
```
| Name                | Industry                    | Revenue (USD millions) | Revenue growth | Employees | Headquarters |
|---------------------|-----------------------------|------------------------|----------------|-----------|--------------|
| Walmart             | Retail                      | 611,289                | 6.7%           | 2,100,000 | Arkansas     |
| Amazon              | Retail and cloud computing | 513,983                | 9.4%           | 1,540,000 | Washington   |
| ExxonMobil          | Petroleum industry          | 413,680                | 44.8%          | 62,000    | Texas        |
| Apple               | Electronics industry        | 394,328                | 7.8%           | 164,000   | California   |
| UnitedHealth Group | Healthcare                  | 324,162                | 12.7%          | 400,000   | Minnesota    |

```python
# Add new column 'Revenue per employee'

df['Revenue per employee'] = round(df['Revenue (USD millions)'] / df['Employees'], 2)

df.head()
```
| Name                | Industry                    | Revenue (USD millions) | Revenue growth | Employees | Headquarters | Revenue per employee |
|---------------------|-----------------------------|------------------------|----------------|-----------|--------------|----------------------|
| Walmart             | Retail                      | 611,289                | 6.7%           | 2,100,000 | Arkansas     | 0.29                 |
| Amazon              | Retail and cloud computing | 513,983                | 9.4%           | 1,540,000 | Washington   | 0.33                 |
| ExxonMobil          | Petroleum industry          | 413,680                | 44.8%          | 62,000    | Texas        | 6.67                 |
| Apple               | Electronics industry        | 394,328                | 7.8%           | 164,000   | California   | 2.40                 |
| UnitedHealth Group | Healthcare                  | 324,162                | 12.7%          | 400,000   | Minnesota    | 0.81                 |

## Exploratory Data Analysis

I conduct exploratory data analysis to preliminarily investigate the data, spotting patterns and relationships, before visualizing in Power BI.
```python
# Getting the company with the most employees

df_sorted = df.sort_values(by = 'Employees', ascending = False, ignore_index = True)

df_sorted.head(1)
```
| Name    | Industry | Revenue (USD millions) | Revenue growth | Employees | Headquarters | Revenue per employee |
|---------|----------|------------------------|----------------|-----------|--------------|----------------------|
| Walmart | Retail   | 611,289                | 6.7%           | 2,100,000 | Arkansas     | 0.29                 |

```python
# Getting the company with the least employees

df_sorted = df.sort_values(by = 'Employees', ascending = True, ignore_index = True)

df_sorted.head(1)
```
| Name       | Industry           | Revenue (USD millions) | Revenue growth | Employees | Headquarters | Revenue per employee |
|------------|--------------------|------------------------|----------------|-----------|--------------|----------------------|
| PBF Energy | Petroleum industry | 46,830                 | 0.718          | 3,616     | New Jersey   | 12.95                |

```python
# Getting the average revenue

df['Revenue (USD millions)'].median(axis = 0)

80824.5
```
```python
# Calculating the average revenue per industy

avg_revenue = round(df.groupby(['Industry'])['Revenue (USD millions)'].mean(), 2)

print(avg_revenue)

Industry
Aerospace and defense                  66296.00
Agriculture cooperative                47194.00
Agriculture manufacturing              52577.00
Airline                                48169.33
Apparel                                46710.00
Automotive and energy                  81462.00
Automotive industry                   157396.00
Beverage                               86859.00
Chemical industry                      56902.00
Conglomerate                          148572.67
Conglomerate and telecomunications    120741.00
Consumer products manufacturing        80187.00
Electronics industry                  394328.00
Financial                              53537.50
Financial services                    154792.00
Financials                             84296.55
Food industry                          84394.00
Food processing                        50238.00
Food service                           68636.00
Health                                276711.00
Health insurance                      136693.00
Healthcare                            198228.00
Infotech                               62344.00
Insurance                              53172.20
Laboratory instruments                 44915.00
Logistics                              78620.00
Machinery                              59427.00
Media                                  82722.00
Petroleum industry                    152122.50
Petroleum industry and logistics       59043.00
Pharmaceutical industry                81912.00
Pharmacy wholesale                    238587.00
Retail                                157890.90
Retail and cloud computing            513983.00
Technology                             77829.40
Technology and cloud computing        180545.33
Telecom hardware manufacturing         51557.00
Telecommunications                    104094.67
Transportation                         96925.00
Name: Revenue (USD millions), dtype: float64
```
```python
# Checking for correlations

df.corr(numeric_only = True)
```
|                   | Revenue (USD millions) | Revenue growth | Employees | Revenue per employee |
|-------------------|------------------------|----------------|-----------|----------------------|
| Revenue (USD millions) | 1.000000               | -0.054699      | 0.641994  | 0.044186             |
| Revenue growth    | -0.054699              | 1.000000       | -0.205546 | 0.510497             |
| Employees         | 0.641994               | -0.205546      | 1.000000  | -0.291540            |
| Revenue per employee | 0.044186               | 0.510497       | -0.291540 | 1.000000             |

```python
df.corr(method = 'kendall', numeric_only = True)
```
|                          | Revenue (USD millions) | Revenue growth | Employees | Revenue per employee |
|--------------------------|------------------------|----------------|-----------|----------------------|
| Revenue (USD millions)   | 1.000000               | -0.004245      | 0.236459  | 0.084211             |
| Revenue growth           | -0.004245              | 1.000000       | -0.262259 | 0.283342             |
| Employees                | 0.236459               | -0.262259      | 1.000000  | -0.681110            |
| Revenue per employee     | 0.084211               | 0.283342       | -0.681110 | 1.000000             |

```python
df.corr(method = 'pearson', numeric_only = True)
```
|                          | Revenue (USD millions) | Revenue growth | Employees | Revenue per employee |
|--------------------------|------------------------|----------------|-----------|----------------------|
| Revenue (USD millions)   | 1.000000               | -0.054699      | 0.641994  | 0.044186             |
| Revenue growth           | -0.054699              | 1.000000       | -0.205546 | 0.510497             |
| Employees                | 0.641994               | -0.205546      | 1.000000  | -0.291540            |
| Revenue per employee     | 0.044186               | 0.510497       | -0.291540 | 1.000000             |

```python
df.corr(method = 'spearman', numeric_only = True)
```
|                          | Revenue (USD millions) | Revenue growth | Employees | Revenue per employee |
|--------------------------|------------------------|----------------|-----------|----------------------|
| Revenue (USD millions)   | 1.000000               | 0.000642       | 0.336566  | 0.131765             |
| Revenue growth           | 0.000642               | 1.000000       | -0.383911 | 0.406741             |
| Employees                | 0.336566               | -0.383911      | 1.000000  | -0.857523            |
| Revenue per employee     | 0.131765               | 0.406741       | -0.857523 | 1.000000             |

### Exploratory Visualizations

Importing matplotlib for visualization
```python
# Importing matplotlib for visualization

import matplotlib.pyplot as plt
```
```python
# Revenue vs No. of Employees

df.plot(kind = 'scatter', x = 'Revenue (USD millions)', y = 'Employees')

plt.show()
```

```python
# Revenue growth vs No. of Employees

df.plot(kind = 'scatter', x = 'Revenue growth', y = 'Employees')

plt.show()
```

Finally, outputting the DataFrame into a .CSV file:

```python
# Import DataFrame to .CSV

df.to_csv(r'C:\Users\Nicolas\Documents\Work\SQL\Data Cleaning\Companies.csv', index = False)
```



