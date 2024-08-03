# Data Visualization using Plotly and Matplotlib

In this project we're going to analyse a dataset on the past winners of the Nobel Prize  and see what patterns we can uncover in the past Nobel laureates and what can we learn about the Nobel prize and our world more generally.

### Table of Contents:
 - [00. Project Overview](#project-overview)
 - [01. Data Exploration and Cleaning](#data-exploration-and-cleaning)
   - [01-1. Preliminary Data Exploration](#preliminary-data-exploration)
   - [01-2. NaN Values ](#nan-values)
   - [01-3. Type Conversions](#type-conversions)
 - [02. Data Visualization using Plotly](#data-visualization-using-plotly)
   - [02-1. Creating a Donut Chart with Plotly](#creating-a-donut-chart-with-plotly)
   - [02-2. Creating a Plotly Bar Chart](#creating-a-plotly-bar-chart)
   - [02-3. Creating a Choropleth Map with Plotly](#creating-a-choropleth-map-with-plotly)
   - [02-4. Creating a Line Graph with Plotly](#creating-a-line-graph-with-plotly)
   - [02-4. Creating a Sunburst Chart with Plotly](#creating-a-sunburst-chart-with-plotly)
 - [03. Data Visualization Using Matplotlib](#data-visualization-using-matplotlib)
 - [04. Analyzing results and Next steps](#analyzing-results-and-next-steps)
---

### Project Overview
On November 27, 1895, Alfred Nobel signed his last will in Paris. When it was opened after his death, the will caused a lot of controversy, as Nobel had left much of his wealth for the establishment of a prize. Alfred Nobel dictates that his entire remaining estate should be used to endow “prizes to those who, during the preceding year, have conferred the greatest benefit to humankind”. Every year the Nobel Prize is given to scientists and scholars in the categories chemistry, literature, physics, physiology or medicine, economics, and peace.

This project will use data visualization tools and a dataset of noble prize winners to visualize the data and find patterns.

---

### Data Exploration and Cleaning
#### Preliminary Data Exploration
When we run `df_data.shape`, `df_data.tail()` and `df_data.head()`, we see that there are 962 rows and 16 columns. The first Nobel prizes were awarded in 1901 and the data goes up to 2020.

![image](https://github.com/user-attachments/assets/ce417131-f8bc-4a6e-b30e-bc66865e08e8)
![image](https://github.com/user-attachments/assets/837446e1-c61c-4ef3-bf49-6a8270ce3508)

We notice that the columns contain the following information:

- ***birth_date:*** date in string format
- ***motivation:*** description of what the prize is for
- ***prize_share:*** given as a fraction
- ***laureate_type:*** individual or organisation
- ***birth_country:*** has countries that no longer exist
- ***birth_country_current:*** current name of the country where the birth city is located
- ***ISO:*** three-letter international country code
- ***organization_name:*** research institution where the discovery was made
- ***organization_city:*** location of the institution

#### NaN Values
Checking for duplicates and NaN values we discover that there is no duplicates but there are a lot of NaN values in dataframe.

![image](https://github.com/user-attachments/assets/a53f0b52-ada8-428b-b32b-6e6e7e954076)

After filtering the NaN values we see that most of the NaN values for birth dates is organizations which explains why they do not have birth dates or organizations that they belong to.

![image](https://github.com/user-attachments/assets/fcdcd4b7-2535-42ea-8b1a-0649738e88a2)

#### Type Conversions
We can use pandas to convert the birth_date to a Datetime object with a single line:
```Python
df_data.birth_date = pd.to_datetime(df_data.birth_date)
```

Adding a column that contains the percentage share as first requires that we split the string on the forward slash. Then we can convert to a number. And finally, we can do the division.
```Python
separated_values = df_data.prize_share.str.split('/', expand=True)
numerator = pd.to_numeric(separated_values[0])
denomenator = pd.to_numeric(separated_values[1])
df_data['share_pct'] = numerator / denomenator
```
----

### Data Visualization using Plotly
#### Creating a Donut Chart with Plotly
We created a donut chart using plotly which shows how many prizes went to men compared to how many prizes went to women.

```Python
biology = df_data.sex.value_counts()
fig = px.pie(labels=biology.index, 
             values=biology.values,
             title="Percentage of Male vs. Female Winners",
             names=biology.index,
             hole=0.4,)
 
fig.update_traces(textposition='inside', textfont_size=15, textinfo='percent')
 
fig.show()
```
![image](https://github.com/user-attachments/assets/1eb6d41c-5a2e-4939-bb40-37aa1301f9ca)

#### Creating a Plotly Bar Chart
We created a Plotly Bar Chart with the number of prizes awarded by category.

```Python
prizes_per_category = df_data.category.value_counts()
v_bar = px.bar(
        x = prizes_per_category.index,
        y = prizes_per_category.values,
        color = prizes_per_category.values,
        color_continuous_scale='Aggrnyl',
        title='Number of Prizes Awarded per Category')

v_bar.update_layout(xaxis_title='Nobel Prize Category', 
                    coloraxis_showscale=False,
                    yaxis_title='Number of Prizes')
v_bar.show()
```
![image](https://github.com/user-attachments/assets/1dd65787-4016-45cd-a089-9a1441b04392)

#### Creating a Choropleth Map with Plotly
To show the ranking of the countries on a colour coded map, we need to make use of the ISO codes.
```Python
df_countries = df_data.groupby(['birth_country_current', 'ISO'], 
                               as_index=False).agg({'prize': pd.Series.count})
df_countries.sort_values('prize', ascending=False)
```
This means we can use the ISO country codes for the locations parameter on the choropleth.
```Python
world_map = px.choropleth(df_countries,
                          locations='ISO',
                          color='prize', 
                          hover_name='birth_country_current', 
                          color_continuous_scale=px.colors.sequential.matter)
 
world_map.update_layout(coloraxis_showscale=True,)
 
world_map.show()
```
![image](https://github.com/user-attachments/assets/e8ec3d1a-16cf-4f99-bbec-95035213fa30)

We can notice that plotly also lets us zoom to the specific location thta we want to see in the map

#### Creating a Line Graph with Plotly
To see how the prize was awarded over time, we can count the number of prizes by country by year.
```Python
prize_by_year = df_data.groupby(by=['birth_country_current', 'year'], as_index=False).count()
prize_by_year = prize_by_year.sort_values('year')[['year', 'birth_country_current', 'prize']]
```
Then we can create a series that has the cumulative sum for the number of prizes won.
```Python
cumulative_prizes = prize_by_year.groupby(by=['birth_country_current',
                                              'year']).sum().groupby(level=[0]).cumsum()
cumulative_prizes.reset_index(inplace=True)
```
Using this, we can create a chart, using the current birth country as the color:
```Python
l_chart = px.line(cumulative_prizes,
                  x='year', 
                  y='prize',
                  color='birth_country_current',
                  hover_name='birth_country_current')
 
l_chart.update_layout(xaxis_title='Year',
                      yaxis_title='Number of Prizes')
 
l_chart.show()
```
![image](https://github.com/user-attachments/assets/db077630-47a0-482a-bb19-d3eb9f6e24b5)

#### Creating a Sunburst Chart with Plotly
Each country has a number of cities, which contain a number of cities, which in turn contain the research organisations. The sunburst chart is perfect for representing this relationship. It will give us an idea of how geographically concentrated scientific discoveries are!

```Python
burst = px.sunburst(country_city_org, 
                    path=['organization_country', 'organization_city', 'organization_name'], 
                    values='prize',
                    title='Where do Discoveries Take Place?',
                   )
 
burst.update_layout(xaxis_title='Number of Prizes', 
                    yaxis_title='City',
                    coloraxis_showscale=False)
 
burst.show()
```
![image](https://github.com/user-attachments/assets/3a49a075-598c-499f-b340-3ad813aeb8c2)


---

### Data Visualization Using Matplotlib
We are going to use Matplotlib to visualize trend of prize awarded over time.

First we visualize number of Prizes Awarded over Time in a graph.
```Python
plt.figure(figsize=(16,8), dpi=200)
plt.title('Number of Nobel Prizes Awarded per Year', fontsize=18)
plt.yticks(fontsize=14)
plt.xticks(ticks=np.arange(1900, 2021, step=5), 
           fontsize=14, 
           rotation=45)

ax = plt.gca()
ax.set_xlim(1900, 2020)

ax.scatter(x=prize_per_year.index, 
           y=prize_per_year.values, 
           c='dodgerblue',
           alpha=0.7,
           s=100,)

ax.plot(prize_per_year.index, 
        moving_average.values, 
        c='crimson', 
        linewidth=3,)

plt.show()
```
![image](https://github.com/user-attachments/assets/30d9ca87-c75a-42b4-b630-591e0e4eff08)

Now we can work out the rolling average of the percentage share of the prize. If more prizes are given out, perhaps it is because the prize is split between more people.
```Python
plt.figure(figsize=(16,8), dpi=200)
plt.title('Number of Nobel Prizes Awarded per Year', fontsize=18)
plt.yticks(fontsize=14)
plt.xticks(ticks=np.arange(1900, 2021, step=5), 
           fontsize=14, 
           rotation=45)

ax1 = plt.gca()
ax2 = ax1.twinx() # create second y-axis
ax1.set_xlim(1900, 2020)

ax1.scatter(x=prize_per_year.index, 
           y=prize_per_year.values, 
           c='dodgerblue',
           alpha=0.7,
           s=100,)

ax1.plot(prize_per_year.index, 
        moving_average.values, 
        c='crimson', 
        linewidth=3,)

# Adding prize share plot on second axis
ax2.plot(prize_per_year.index, 
        share_moving_average.values, 
        c='grey', 
        linewidth=3,)

plt.show()
```
![image](https://github.com/user-attachments/assets/2b8caa76-387c-4064-b944-2137dc3bdd9d)

To see the relationship between the number of prizes and the laureate share even more clearly we can invert the second y-axis.

```Python
plt.figure(figsize=(16,8), dpi=200)
plt.title('Number of Nobel Prizes Awarded per Year', fontsize=18)
plt.yticks(fontsize=14)
plt.xticks(ticks=np.arange(1900, 2021, step=5), 
           fontsize=14, 
           rotation=45)
 
ax1 = plt.gca()
ax2 = ax1.twinx()
ax1.set_xlim(1900, 2020)
 
# Can invert axis
ax2.invert_yaxis()
 
ax1.scatter(x=prize_per_year.index, 
           y=prize_per_year.values, 
           c='dodgerblue',
           alpha=0.7,
           s=100,)
 
ax1.plot(prize_per_year.index, 
        moving_average.values, 
        c='crimson', 
        linewidth=3,)
 
ax2.plot(prize_per_year.index, 
        share_moving_average.values, 
        c='grey', 
        linewidth=3,)
 
plt.show()
```
![image](https://github.com/user-attachments/assets/cbca01dc-73b4-48c3-adb6-b26bd208b1f7)

There is clearly an upward trend in the number of prizes being given out as more and more prizes are shared.

---

### Analyzing results and Next steps

In this project we used Data Visualization tools to visualize information about Nobel prize winners, see trends of change and find patterns.

Further steps to take in this project can be:
01. To analyze more detailed information like motivation, organizaions and etc.
02. To group winners according to their organizations
03. To analyze the trend of winning of organizations compared to individuals
