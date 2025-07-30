import pandas as pd 
 
# Load dataset 
df = pd.read_csv("netflix_titles.csv") 
 
# Clean duration column 
df['duration_clean'] = pd.to_numeric(df['duration'].str.replace(' min', ''), errors='coerce') 
 
# Parse date_added 
df['date_added'] = pd.to_datetime(df['date_added'], errors='coerce') 
df['year_added'] = df['date_added'].dt.year 
 
# Extract genres 
df['genres'] = df['listed_in'].str.split(', ') 

Building Dashboard Components 

import panel as pn 
import hvplot.pandas 
pn.extension() 
 
# Content type selector 
type_selector = pn.widgets.RadioButtonGroup( 
    name='Type', options=['Movie', 'TV Show'], button_type='success') 
 
@pn.depends(type_selector) 
def filter_data(content_type): 
    return df[df['type'] == content_type] 

1. Rating Distribution 

@pn.depends(type_selector) 
def rating_plot(content_type): 
    data = filter_data(content_type) 
    return data['rating'].value_counts().sort_values().hvplot.barh( 
        title='🔖 Rating Distribution', xlabel='Count', ylabel='Rating', 
        height=300, width=400, color='tomato') 

2. Titles Released Over Time 

@pn.depends(type_selector) 
def yearly_plot(content_type): 
    data = filter_data(content_type) 
    return data['release_year'].value_counts().sort_index().hvplot.line( 
        title='📅 Titles Released Over Time', xlabel='Year', ylabel='Number of Titles', 
        line_width=3, color='dodgerblue', height=300, width=500) 

3. Top Genres 

@pn.depends(type_selector) 
def genre_plot(content_type): 
    data = filter_data(content_type) 
    return data['genres'].explode().value_counts().head(10).sort_values().hvplot.barh( 
        title='🎬 Top Genres', xlabel='Count', ylabel='Genre', 
        color='mediumseagreen', height=300, width=400) 

4. Duration vs. Date Added 

@pn.depends(type_selector) 
def scatter_plot(content_type): 
    data = filter_data(content_type) 
    return data.hvplot.scatter( 
        x='date_added', y='duration_clean', c='release_year', cmap='plasma', 
        title='⏱️ Duration vs. Date Added', xlabel='Date Added', ylabel='Duration', 
        height=300, width=500, size=5, alpha=0.6) 

Layout 

dashboard = pn.Column( 
    "# 🎥 Netflix Content Explorer", 
    "Use the toggle to explore visualizations by content type.", 
    type_selector, 
    pn.Row(rating_plot, genre_plot), 
    pn.Row(yearly_plot, scatter_plot), 
) 
dashboard.servable() 
