# Nobel Prize Analysis

This project explores the historical data of Nobel Prize winners to identify patterns and insights. The analysis covers various aspects such as gender distribution, repeat winners, prizes per category, country-wise distribution, and the age of winners at the time of receiving the prize.

## Output


https://github.com/sarvesh-2109/Nobel-Prize-Analysis/assets/113255836/85307c00-27b4-4fb6-9070-1f652ca37834




## Setup and Context

On November 27, 1895, Alfred Nobel signed his last will in Paris, dictating that his remaining estate should endow prizes for those who confer the greatest benefit to humankind. Every year, the Nobel Prize is awarded in the categories of chemistry, literature, physics, physiology or medicine, economics, and peace.

This project aims to uncover patterns and insights from the data of past Nobel laureates.

## Import Statements

```python
import pandas as pd
import numpy as np
import plotly.express as px
import seaborn as sns
import matplotlib.pyplot as plt
```

## Notebook Presentation

```python
pd.options.display.float_format = '{:,.2f}'.format
```

## Read the Data

```python
df_data = pd.read_csv('/content/nobel_prize_data.csv')
```

## Data Exploration & Cleaning

```python
df_data.info()
print(df_data.shape)
df_data.head()
df_data.tail()
```

### Check for Duplicates

```python
duplicate_rows = df_data[df_data.duplicated()]
print(f"Number of duplicate rows: {len(duplicate_rows)}")
```

### Check for NaN Values

```python
print(f"Are there any null values: {df_data.isna().values.any() }")
df_data.isna().sum()
col_subset = ['year','category', 'laureate_type', 'birth_date','full_name', 'organization_name']
df_data.loc[df_data.birth_date.isna()][col_subset]
col_subset = ['year','category', 'laureate_type','full_name', 'organization_name']
df_data.loc[df_data.organization_name.isna()][col_subset]
```

### Type Conversions

#### Convert Year and Birth Date to Datetime

```python
df_data.birth_date = pd.to_datetime(df_data.birth_date)
```

#### Add a Column with the Prize Share as a Percentage

```python
separated_values = df_data.prize_share.str.split('/', expand=True)
numerator = pd.to_numeric(separated_values[0])
denomenator = pd.to_numeric(separated_values[1])
df_data['share_pct'] = numerator / denomenator
df_data.info()
```

## Plotly Donut Chart: Percentage of Male vs. Female Laureates

```python
gender = df_data['sex'].value_counts()
fig = px.pie(labels=gender.index, values=gender.values, title='Percentage of Male vs. Female Winners', names=gender.index, hole=0.6)
fig.update_traces(textposition='inside', textfont_size=15, textinfo='percent')
fig.show()
```

## Who were the first 3 Women to Win the Nobel Prize?

```python
df_data[df_data.sex == 'Female'].sort_values('year', ascending=True)[:3]
```

## Finding the Repeat Winners

```python
is_winner = df_data.duplicated(subset=['full_name'], keep=False)
multiple_winners = df_data[is_winner]
print(f'There are {multiple_winners.full_name.nunique()} winners who were awarded the prize more than once.')
col_subset = ['year', 'category', 'laureate_type', 'full_name']
multiple_winners[col_subset]
```

## Number of Prizes per Category

```python
df_data.category.nunique()
prize_per_category = df_data.category.value_counts()
v_bar = px.bar(x=prize_per_category.index, y=prize_per_category.values, color=prize_per_category.values, color_continuous_scale='Aggrnyl', title='Number of Prizes Awarded per Category')
v_bar.update_layout(xaxis_title='Nobel Prize Category', coloraxis_showscale=False, yaxis_title='Number of Prizes')
v_bar.show()
```

### First prize in Economics category

```python
df_data[df_data.category == 'Economics'].sort_values('year')[:3]
```

## Male and Female Winners by Category

```python
cat_men_women = df_data.groupby(['category', 'sex'], as_index=False).agg({'prize': pd.Series.count})
cat_men_women.sort_values('prize', ascending=False, inplace=True)
v_bar_split = px.bar(x = cat_men_women.category, y = cat_men_women.prize, color = cat_men_women.sex, title='Number of Prizes Awarded per Category split by Men and Women')
v_bar_split.update_layout(xaxis_title='Nobel Prize Category', yaxis_title='Number of Prizes')
v_bar_split.show()
```

## Number of Prizes Awarded Over Time

```python
prize_per_year = df_data.groupby(by='year').count().prize
moving_average = prize_per_year.rolling(window=5).mean()
plt.figure(figsize=(16,8), dpi=200)
plt.title('Number of Nobel Prizes Awarded per Year', fontsize=18)
plt.yticks(fontsize=14)
plt.xticks(ticks=np.arange(1900, 2021, step=5), fontsize=14, rotation=45)
ax = plt.gca() # get current axis
ax.set_xlim(1900, 2020)
ax.scatter(x=prize_per_year.index, y=prize_per_year.values, c='dodgerblue', alpha=0.7, s=100,)
ax.plot(prize_per_year.index, moving_average.values, c='crimson', linewidth=3,)
plt.show()
```

## Are More Prizes Shared Than Before?

```python
yearly_avg_share = df_data.groupby(by='year').agg({'share_pct': pd.Series.mean})
share_moving_average = yearly_avg_share.rolling(window=5).mean()
plt.figure(figsize=(16,8), dpi=200)
plt.title('Number of Nobel Prizes Awarded per Year', fontsize=18)
plt.yticks(fontsize=14)
plt.xticks(ticks=np.arange(1900, 2021, step=5), fontsize=14, rotation=45)
ax1 = plt.gca()
ax2 = ax1.twinx()
ax1.set_xlim(1900, 2020)
ax2.invert_yaxis()
ax1.scatter(x=prize_per_year.index, y=prize_per_year.values, c='dodgerblue', alpha=0.7, s=100,)
ax1.plot(prize_per_year.index, moving_average.values, c='crimson', linewidth=3,)
ax2.plot(prize_per_year.index, share_moving_average.values, c='grey', linewidth=3,)
plt.show()
```

## The Countries with the Most Nobel Prizes

```python
top_countries = df_data.groupby(['birth_country_current'], as_index=False).agg({'prize': pd.Series.count})
top_countries.sort_values(by='prize', inplace=True)
top20_countries = top_countries[-20:]
h_bar = px.bar(x=top20_countries.prize, y=top20_countries.birth_country_current, orientation='h', color=top20_countries.prize, color_continuous_scale='Viridis', title='Top 20 Countries by Number of Prizes')
h_bar.update_layout(xaxis_title='Number of Prizes', yaxis_title='Country', coloraxis_showscale=False)
h_bar.show()
```

## Use a Choropleth Map to Show the Number of Prizes Won by Country

```python
df_countries = df_data.groupby(['birth_country_current', 'ISO'], as_index=False).agg({'prize': pd.Series.count})
df_countries.sort_values('prize', ascending=False)
world_map = px.choropleth(df_countries, locations='ISO', color='prize', hover_name='birth_country_current', color_continuous_scale=px.colors.sequential.matter)
world_map.update_layout(coloraxis_showscale=True,)
world_map.show()
```

## In Which Categories are the Different Countries Winning Prizes?

```python
cat_country = df_data.groupby(['birth_country_current', 'category'], as_index=False).agg({'prize': pd.Series.count})
cat_country.sort_values(by='prize', ascending=False, inplace=True)
merged_df = pd.merge(cat_country, top20_countries, on='birth_country_current')
merged_df.columns = ['birth_country_current', 'category', 'cat_prize', 'total_prize']
merged_df.sort_values(by='total_prize', inplace=True)
cat_cntry_bar = px.bar(x=merged_df.cat_prize, y=merged_df.birth_country_current, color=merged_df.category, orientation='h', title='Top 20 Countries by Number of Prizes and Category')
cat_cntry_bar.update_layout(xaxis_title='Number of Prizes', yaxis_title='Country')
cat_cntry_bar.show()
```

## Number of Prizes Won by Each Country Over Time

```python
prize_by_year = df_data.groupby(by=['birth_country_current', 'year'], as_index=False).count()
prize_by_year = prize_by_year.sort_values('year')[['year', 'birth_country_current', 'prize']]
cumulative_prizes = prize_by_year.groupby(by=['birth_country_current', 'year']).sum().groupby(level=[0]).cumsum()
cumulative_prizes.reset_index(inplace=True)
l_chart = px.line(cumulative_prizes, x='year', y='prize', color='birth_country_current', hover_name='birth_country_current')
l_chart.update
