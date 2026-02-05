# google-playstore-analysis

#TASK-1
import pandas as pd
import plotly.express as px
from datetime import datetime
import pytz
df = pd.read_csv('play_store_data.csv')
ist = pytz.timezone("Asia/Kolkata")
current_time = datetime.now(ist).time()
start_time = datetime.strptime("15:00", "%H:%M").time()
end_time = datetime.strptime("17:00", "%H:%M").time()
def clean_numeric(column):
    """Remove commas, +, M, K and convert to float"""
    return pd.to_numeric(column.astype(str).str.replace('[+,MK]','', regex=True), errors='coerce')

df['Installs'] = clean_numeric(df['Installs'])
df['Reviews'] = clean_numeric(df['Reviews'])
df['Rating'] = clean_numeric(df['Rating'])
df['Size'] = clean_numeric(df['Size'])
df['Last Updated'] = pd.to_datetime(df['Last Updated'], errors='coerce')
df = df.dropna(subset=['Category','Rating','Reviews','Size','Last Updated'])
filtered_df = df[
    (df['Rating'] >= 4.0) &
    (df['Size'] >= 10) &
    (df['Last Updated'].dt.month == 1)  
].copy()
category_stats = filtered_df.groupby('Category').agg({
    'Rating':'mean',
    'Reviews':'sum',
    'Installs':'sum'
}).reset_index()
top_categories = category_stats.sort_values('Installs', ascending=False).head(10)
melted = top_categories.melt(
    id_vars='Category',
    value_vars=['Rating','Reviews'],
    var_name='Metric',
    value_name='Value'
)
if start_time <= current_time <= end_time:
    fig = px.bar(
        melted,
        x='Category',
        y='Value',
        color='Metric',
        barmode='group',
        title='Top 10 App Categories: Average Rating & Total Reviews',
        labels={'Value':'Value','Category':'App Category'}
    )
    fig.update_layout(
        xaxis_title='App Category',
        yaxis_title='Average Rating / Total Reviews'
    )
    fig.show()    
else:
    print("⛔ This graph is visible only between 3 PM IST and 5 PM IST")

#TASK-2
import pandas as pd
import plotly.graph_objects as go
from datetime import datetime
import pytz
import numpy as np
df = pd.read_csv("play_store_data.csv")
ist = pytz.timezone("Asia/Kolkata")
current_time = datetime.now(ist).time()
start_time = datetime.strptime("13:00", "%H:%M").time()
end_time = datetime.strptime("14:00", "%H:%M").time()
df['Installs'] = (
    df['Installs'].astype(str)
    .str.replace('[+,]', '', regex=True)
)
df['Installs'] = pd.to_numeric(df['Installs'], errors='coerce')
df['Reviews'] = pd.to_numeric(df['Reviews'], errors='coerce')
df['Rating'] = pd.to_numeric(df['Rating'], errors='coerce')
df['Price'] = df['Price'].astype(str).str.strip().str.replace('$','', regex=False)
df['Price'] = df['Price'].replace(['Free','free'], 0)
df['Price'] = pd.to_numeric(df['Price'], errors='coerce')
def size_to_mb(x):
    try:
        if 'M' in str(x):
            return float(str(x).replace('M',''))
        elif 'k' in str(x):
            return float(str(x).replace('k','')) / 1024
        else:
            return np.nan
    except:
        return np.nan
df['Size_MB'] = df['Size'].apply(size_to_mb)
df['Android_Version'] = (
    df['Android Ver'].astype(str)
    .str.extract(r'(\d+\.\d+)')
)
df['Android_Version'] = pd.to_numeric(df['Android_Version'], errors='coerce')
filtered_df = df[
    (df['Installs'] >= 10000) &
    (df['Android_Version'] >= 4.0) &
    (df['Size_MB'] > 15) &
    (df['Content Rating'] == 'Everyone') &
    (df['App'].str.len() <= 30)
].copy()
top_categories = (
    filtered_df.groupby('Category')['Installs']
    .sum()
    .sort_values(ascending=False)
    .head(3)
    .index
)
top_df = filtered_df[filtered_df['Category'].isin(top_categories)]
agg_df = top_df.groupby(['Category', 'Type']).agg(
    Avg_Installs=('Installs', 'mean'),
    Avg_Price=('Price', 'mean')
).reset_index()
if start_time <= current_time <= end_time and not agg_df.empty:

    fig = go.Figure()
    for typ in ['Free','Paid']:
        temp = agg_df[agg_df['Type'] == typ]
        fig.add_trace(go.Bar(
            x=temp['Category'],
            y=temp['Avg_Installs'],
            name=f'Avg Installs ({typ})',
            yaxis='y'
        ))
    for typ in ['Free','Paid']:
        temp = agg_df[agg_df['Type'] == typ]
        fig.add_trace(go.Scatter(
            x=temp['Category'],
            y=temp['Avg_Price'],
            name=f'Avg Price ({typ})',
            yaxis='y2',
            mode='lines+markers'
        ))

    fig.update_layout(
        title="Average Installs and Price by Type for Top 3 Categories",
        xaxis_title="App Category",
        yaxis=dict(title="Average Installs"),
        yaxis2=dict(
            title="Average Price (USD)",
            overlaying="y",
            side="right"
        ),
        template="plotly_white",
        height=600,
        barmode='group'
    )
    fig.show()
else:
    print("⛔Dual Chart visible only between 1 PM and 2 PM IST OR no data after filters")


#TASK-3
import pandas as pd
import plotly.express as px
from datetime import datetime, time
import pytz
df = pd.read_csv("play_store_data.csv")
df['Installs'] = (
    df['Installs']
    .astype(str)
    .str.replace('[+,]', '', regex=True)
)
df['Installs'] = pd.to_numeric(df['Installs'], errors='coerce')
df = df.dropna(subset=['Installs', 'Category'])
df['Country'] = 'India'   
df = df[
    ~df['Category'].str.startswith(('A', 'C', 'G', 'S'), na=False)
]
top_categories = (
    df.groupby('Category')['Installs']
    .sum()
    .sort_values(ascending=False)
    .head(5)
    .index
)
df = df[df['Category'].isin(top_categories)]
map_df = (
    df.groupby(['Country', 'Category'], as_index=False)
    .agg(Total_Installs=('Installs', 'sum'))
)
map_df['Highlight'] = map_df['Total_Installs'] > 1_000_000
ist = pytz.timezone('Asia/Kolkata')
current_time = datetime.now(ist).time()
start_time = time(18, 0)  # 6 PM
end_time   = time(20, 0)  # 8 PM
print("Current IST Time:", current_time)
if start_time <= current_time <= end_time:
    fig = px.choropleth(
        map_df,
        locations="Country",
        locationmode="country names",
        color="Total_Installs",
        hover_name="Category",
        hover_data=["Total_Installs"],
        animation_frame="Category",
        color_continuous_scale="Plasma",
        title="Global Installs by Category (Top 5)"
    )
    fig.show()
else:
    print("⏰ Choropleth map visible only between 6 PM and 8 PM IST.")

#TASK-4
import pandas as pd
import plotly.express as px
from datetime import datetime
import pytz
import re
ist = pytz.timezone('Asia/Kolkata')
current_time = datetime.now(ist).time()
start_time = datetime.strptime("16:00", "%H:%M").time()
end_time = datetime.strptime("18:00", "%H:%M").time()
df['Installs'] = df['Installs'].astype(str).str.replace('[+,]', '', regex=True)
df['Installs'] = pd.to_numeric(df['Installs'], errors='coerce')
df['Reviews'] = pd.to_numeric(df['Reviews'], errors='coerce')
df['Rating'] = pd.to_numeric(df['Rating'], errors='coerce')
df['Size'] = df['Size'].astype(str).str.replace('M','', regex=False)
df['Size'] = pd.to_numeric(df['Size'], errors='coerce')
filtered_df = df[
    (df['Rating'] >= 4.2) &
    (df['Reviews'] > 1000) &
    (df['Size'].between(20,80)) &
    (df['Category'].str.startswith(('T','P'))) &
    (~df['App'].str.contains(r'\d'))
].copy()
filtered_df['Last Updated'] = pd.to_datetime(filtered_df['Last Updated'], errors='coerce')
filtered_df['Month'] = filtered_df['Last Updated'].dt.to_period('M').dt.to_timestamp()
translation = {
    'Travel & Local': 'Voyage & Local',     
    'Productivity': 'Productividad',        
    'Photography': '写真'                     
}
filtered_df['Category_translated'] = filtered_df['Category'].replace(translation)
cumulative_df = filtered_df.groupby(['Month', 'Category_translated'])['Installs'].sum().reset_index()
cumulative_df = cumulative_df.sort_values('Month')
cumulative_df['MoM_change'] = cumulative_df.groupby('Category_translated')['Installs'].pct_change()
cumulative_df['Highlight'] = cumulative_df['MoM_change'] > 0.25
if start_time <= current_time <= end_time:
    fig = px.area(
        cumulative_df,
        x='Month',
        y='Installs',
        color='Category_translated',
        line_group='Category_translated',
        color_discrete_sequence=px.colors.qualitative.Set3,
        title="Cumulative Installs Over Time by Category"
    )
    for category in cumulative_df['Category_translated'].unique():
        highlight_df = cumulative_df[(cumulative_df['Category_translated']==category) & (cumulative_df['Highlight'])]
        fig.add_scatter(
            x=highlight_df['Month'],
            y=highlight_df['Installs'],
            mode='markers',
            marker=dict(size=10, color='red'),
            name=f'{category} Growth >25%'
        )
    fig.update_layout(
        xaxis_title='Month',
        yaxis_title='Cumulative Installs',
        legend_title='App Category'
    )
    fig.show()
else:
    print("⛔ Stacked Area Chart is visible only between 4 PM IST and 6 PM IST")


#TASK-5
import pandas as pd
import matplotlib.pyplot as plt
from datetime import datetime, time
import pytz
import numpy as np
import os
try:
    df = pd.read_csv("play_store_data.csv")
except FileNotFoundError:
    print("❌ Error: 'play_store_data.csv' not found.")
    exit()
df['Installs'] = df['Installs'].astype(str).str.replace(r'[+,]', '', regex=True)
df['Installs'] = pd.to_numeric(df['Installs'], errors='coerce').fillna(0)
df['Reviews'] = pd.to_numeric(df['Reviews'], errors='coerce').fillna(0)
df['Rating'] = pd.to_numeric(df['Rating'], errors='coerce').fillna(0)
def convert_size(size):
    size = str(size).lower()
    if 'm' in size:
        return float(size.replace('m', ''))
    if 'k' in size:
        return float(size.replace('k', '')) / 1024
    return np.nan
df['Size_MB'] = df['Size'].apply(convert_size)
df['Category'] = df['Category'].str.upper()
allowed_categories = [
    'GAME', 'BEAUTY', 'BUSINESS', 'COMICS',
    'COMMUNICATION', 'DATING', 'ENTERTAINMENT',
    'SOCIAL', 'EVENTS'
]
filtered_df = df[
    (df['Rating'] > 3.5) &
    (df['Reviews'] > 500) &
    (df['Installs'] > 50000) &
    (df['Category'].isin(allowed_categories))
].copy().dropna(subset=['Size_MB', 'Rating', 'Installs'])
current_time = datetime.now(ist).time()
start_time = time(17, 0)  
end_time = time(19, 0)    
bypass_time_check = False 
print(f"Current IST Time: {current_time.strftime('%H:%M:%S')}")
is_it_time = (start_time <= current_time <= end_time)
if is_it_time or bypass_time_check:
    print("✅ Time Authorized: Generating Bubble Chart...")    
    if filtered_df.empty:
        print("❌ No data matches the filters (Rating > 3.5, Reviews > 500).")
    else:
        plt.figure(figsize=(12, 7))
        max_installs = filtered_df['Installs'].max()
        installs_scaled = (filtered_df['Installs'] / max_installs) * 1000 + 50 if max_installs > 0 else 100
        for category in filtered_df['Category'].unique():
            subset = filtered_df[filtered_df['Category'] == category]
            color = 'hotpink' if category == 'GAME' else None # Auto-color others
            plt.scatter(
                subset['Size_MB'],
                subset['Rating'],
                s=installs_scaled[subset.index],
                alpha=0.6,
                label=category,
                color=color,
                edgecolors='w',
                linewidth=0.5
            )
        plt.xlabel("App Size (MB)")
        plt.ylabel("Average Rating")
        plt.title(f"Play Store: Size vs Rating\n(Authorized Access: {start_time.strftime('%H:%M')} - {end_time.strftime('%H:%M')})")
        plt.legend(title="Category", bbox_to_anchor=(1.05, 1), loc='upper left')
        plt.grid(True, linestyle='--', alpha=0.6)
        plt.tight_layout()
        plt.show()
else:
    print("--------------------------------------------------")
    print("🔒 ACCESS DENIED")
    print(f"This chart is only scheduled for: {start_time.strftime('%I:%M %p')} to {end_time.strftime('%I:%M %p')} IST.")
    print("Please check back during the scheduled window.")
    print("--------------------------------------------------")



#TASK-6
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go
from datetime import datetime
import pytz
import re
ist = pytz.timezone("Asia/Kolkata")
current_time = datetime.now(ist).time()
start_time = datetime.strptime("18:00", "%H:%M").time()
end_time = datetime.strptime("21:00", "%H:%M").time()
df['Installs'] = df['Installs'].astype(str).str.replace('[+,]', '', regex=True)
df['Installs'] = pd.to_numeric(df['Installs'], errors='coerce')
df['Reviews'] = pd.to_numeric(df['Reviews'], errors='coerce')
categories = [c for c in df['Category'].unique() if c.startswith(('E','C','B'))]
filtered_df = df[
    (df['Reviews'] > 500) &
    (~df['App'].str.contains(r'^[xyz]', flags=re.IGNORECASE, na=False)) &  
    (~df['App'].str.contains('S', case=False, na=False)) &
    (df['Category'].isin(categories))
].copy()
translation = {
    'Beauty': 'सौंदर्य',     
    'Business': 'வியாபாரம்', 
    'Dating': 'Dating (DE)'   
}
filtered_df['Category_translated'] = filtered_df['Category'].replace(translation)
filtered_df['Last Updated'] = pd.to_datetime(filtered_df['Last Updated'], errors='coerce')
filtered_df['Month'] = filtered_df['Last Updated'].dt.to_period('M').dt.to_timestamp()
monthly_installs = (
    filtered_df.groupby(['Month', 'Category_translated'])['Installs']
    .sum()
    .reset_index()
)
monthly_installs['MoM_change'] = monthly_installs.groupby('Category_translated')['Installs'].pct_change()
if start_time <= current_time <= end_time:
    fig = go.Figure()
    for category in monthly_installs['Category_translated'].unique():
        cat_df = monthly_installs[monthly_installs['Category_translated'] == category]
        fig.add_trace(go.Scatter(
            x=cat_df['Month'],
            y=cat_df['Installs'],
            mode='lines',
            name=category
        ))
        significant_growth = cat_df[cat_df['MoM_change'] > 0.2]
        fig.add_trace(go.Scatter(
            x=significant_growth['Month'],
            y=significant_growth['Installs'],
            mode='lines',
            line=dict(color='rgba(0,0,0,0)'),  
            fill='tozeroy',
            fillcolor='rgba(255,0,0,0.2)',  
            showlegend=False
        ))
    fig.update_layout(
        title="Total Installs Over Time by App Category",
        xaxis_title="Month",
        yaxis_title="Total Installs",
        legend_title="App Category"
    )
    fig.show()
else:
    print("⛔ Line Chart is visible only between 6 PM IST and 9 PM IST")


#DASHBOARD
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go
import plotly.io as pio
from datetime import datetime
import pytz
import numpy as np
import os
import webbrowser
ist = pytz.timezone("Asia/Kolkata")
now = datetime.now(ist)
current_hour = now.hour
print(f"🕒 Current Time (IST): {now.strftime('%H:%M:%S')}")
try:
    df = pd.read_csv('play_store_data.csv')
    df['Installs'] = pd.to_numeric(df['Installs'].astype(str).str.replace(r'[+,]', '', regex=True), errors='coerce').fillna(0)
    df['Reviews'] = pd.to_numeric(df['Reviews'], errors='coerce').fillna(0)
    df['Rating'] = pd.to_numeric(df['Rating'], errors='coerce').fillna(0)
    df['Price'] = pd.to_numeric(df['Price'].astype(str).str.replace('$', '', regex=False).replace(['Free','free'], 0), errors='coerce').fillna(0)  
    def get_size(x):
        x = str(x).lower()
        if 'm' in x: return float(x.replace('m',''))
        if 'k' in x: return float(x.replace('k','')) / 1024
        return 10.0  
    df['Size_MB'] = df['Size'].apply(get_size)
    df['Last Updated'] = pd.to_datetime(df['Last Updated'], errors='coerce')
except (FileNotFoundError, KeyError):
    print("⚠️ CSV Not Found. Generating Dummy Data for Demo...")
    cats = ['GAME', 'SOCIAL', 'PRODUCTIVITY', 'TOOLS', 'FINANCE', 'PHOTOGRAPHY', 'BUSINESS']
    types = ['Free', 'Paid']
    data = {
        'Category': np.random.choice(cats, 600),
        'Rating': np.random.uniform(2.5, 5.0, 600).round(1),
        'Reviews': np.random.randint(100, 1000000, 600),
        'Installs': np.random.randint(1000, 100000000, 600),
        'Type': np.random.choice(types, 600),
        'Price': np.random.choice([0, 0.99, 4.99], 600),
        'Size_MB': np.random.uniform(10, 150, 600),
        'Last Updated': pd.date_range(start='1/1/2023', periods=600),
        'App': [f'App {i}' for i in range(600)]
    }
    df = pd.DataFrame(data)
    df.loc[df['Type'] == 'Free', 'Price'] = 0
def chart_1_bar():
    # 1. Bar Chart: Rating & Reviews (15:00 - 17:00)
    temp = df[df['Rating'] >= 4.0].groupby('Category').agg({'Rating':'mean', 'Reviews':'sum'}).reset_index().head(10)
    melted = temp.melt(id_vars='Category', value_vars=['Rating','Reviews'], var_name='Metric', value_name='Value')
    fig = px.bar(melted, x='Category', y='Value', color='Metric', barmode='group', template="plotly_dark")
    fig.update_layout(paper_bgcolor='rgba(0,0,0,0)', plot_bgcolor='rgba(0,0,0,0)', title="Category Ratings & Reviews")
    return fig
def chart_2_dual():
    # 2. Dual Axis: Installs vs Price (13:00 - 14:00)
    top3 = df.groupby('Category')['Installs'].sum().nlargest(3).index
    agg = df[df['Category'].isin(top3)].groupby(['Category', 'Type']).agg({'Installs':'mean', 'Price':'mean'}).reset_index()
    fig = go.Figure()
    for t in ['Free','Paid']:
        d = agg[agg['Type']==t]
        fig.add_bar(x=d['Category'], y=d['Installs'], name=f'{t} Installs')
        fig.add_scatter(x=d['Category'], y=d['Price'], yaxis='y2', name=f'{t} Price', mode='lines+markers')
    fig.update_layout(
        yaxis2=dict(overlaying='y', side='right'), 
        template="plotly_dark", 
        paper_bgcolor='rgba(0,0,0,0)', 
        plot_bgcolor='rgba(0,0,0,0)',
        title="Installs vs Price (Dual Axis)"
    )
    return fig
def chart_3_map():
    map_df = df.groupby('Category')['Installs'].sum().nlargest(5).reset_index()
    countries = ['India', 'USA', 'China', 'Brazil', 'Germany']
    map_df['Country'] = [countries[i % 5] for i in range(len(map_df))]
    fig = px.choropleth(map_df, locations="Country", locationmode="country names", color="Installs", 
                        color_continuous_scale="Plasma", template="plotly_dark", title="Global Installs Map")
    fig.update_layout(paper_bgcolor='rgba(0,0,0,0)', plot_bgcolor='rgba(0,0,0,0)')
    return fig
def chart_4_area():
    df['Month'] = df['Last Updated'].dt.to_period('M').dt.to_timestamp()
    trend = df.groupby(['Month', 'Category'])['Installs'].sum().reset_index().sort_values('Month')
    top_cats = df['Category'].value_counts().head(3).index
    fig = px.area(trend[trend['Category'].isin(top_cats)], x='Month', y='Installs', color='Category', 
                  template="plotly_dark", title="Cumulative Growth Trends")
    fig.update_layout(paper_bgcolor='rgba(0,0,0,0)', plot_bgcolor='rgba(0,0,0,0)')
    return fig
def chart_5_bubble():
    temp = df[(df['Installs'] > 10000) & (df['Rating'] > 3.0)].sample(min(300, len(df)))
    fig = px.scatter(temp, x='Size_MB', y='Rating', size='Installs', color='Category',
                     hover_name='App', size_max=40, template="plotly_dark", title="App Size vs Rating")
    fig.update_layout(paper_bgcolor='rgba(0,0,0,0)', plot_bgcolor='rgba(0,0,0,0)')
    return fig
def chart_6_line():
    df['Month'] = df['Last Updated'].dt.to_period('M').dt.to_timestamp()
    monthly = df.groupby(['Month', 'Category'])['Installs'].sum().reset_index()
    top_cats = df['Category'].value_counts().head(4).index
    fig = px.line(monthly[monthly['Category'].isin(top_cats)], x='Month', y='Installs', color='Category', 
                  template="plotly_dark", title="Monthly Market Analysis")
    fig.update_layout(paper_bgcolor='rgba(0,0,0,0)', plot_bgcolor='rgba(0,0,0,0)')
    return fig
config = [
    ("📊 Top Categories (Bar)", chart_1_bar, 15, 17),
    ("💰 Price Strategy (Dual)", chart_2_dual, 13, 14),
    ("🌍 Global Reach (Map)", chart_3_map, 18, 20),
    ("📈 Growth Trends (Area)", chart_4_area, 16, 18),
    ("🔵 Technical Insights (Bubble)", chart_5_bubble, 17, 19),
    ("📉 Market Analysis (Line)", chart_6_line, 18, 21)
]
FORCE_UNLOCK_ALL = False 
css = """
<style>
    @import url('https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;700&display=swap');
    
    body {
        font-family: 'Roboto', sans-serif;
        background-color: #000000;
        color: #e0e0e0;
        margin: 0;
        display: flex;
        min-height: 100vh;
    }
    
    .sidebar {
        width: 250px;
        background: #111111;
        border-right: 1px solid #333;
        padding: 20px;
        position: fixed;
        height: 100%;
        display: flex;
        flex-direction: column;
    }
    .brand {
        color: #00f2fe;
        font-size: 1.5rem;
        font-weight: bold;
        margin-bottom: 30px;
        letter-spacing: 2px;
    }
    .menu-item {
        padding: 15px;
        color: #888;
        border-bottom: 1px solid #222;
        font-size: 0.9rem;
    }
    .menu-item.active { color: white; border-left: 3px solid #00f2fe; }
    
    .main {
        margin-left: 250px;
        padding: 40px;
        width: 100%;
        box-sizing: border-box;
    }
    
    .header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 40px;
        border-bottom: 1px solid #333;
        padding-bottom: 20px;
    }
    
    .grid {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(450px, 1fr));
        gap: 30px;
    }
    
    .card {
        background: #0a0a0a;
        border: 1px solid #333;
        border-radius: 12px;
        padding: 20px;
        position: relative;
        min-height: 400px;
        box-shadow: 0 4px 15px rgba(0,0,0,0.5);
    }
    .card-header {
        display: flex;
        justify-content: space-between;
        margin-bottom: 15px;
        color: #ccc;
        font-weight: bold;
    }
    
    .badge { padding: 4px 10px; border-radius: 20px; font-size: 0.75rem; }
    .badge-live { background: rgba(0, 255, 127, 0.2); color: #00ff7f; border: 1px solid #00ff7f; }
    .badge-locked { background: rgba(255, 69, 58, 0.2); color: #ff453a; border: 1px solid #ff453a; }
    
    .locked-overlay {
        position: absolute;
        top: 0; left: 0; right: 0; bottom: 0;
        background: rgba(10, 10, 10, 0.85);
        backdrop-filter: blur(5px);
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        border-radius: 12px;
        z-index: 10;
        text-align: center;
    }
    .lock-icon { font-size: 4rem; margin-bottom: 15px; }
    .lock-msg { font-size: 1.2rem; color: #ff453a; font-weight: bold; }
    .lock-sub { color: #888; font-size: 0.9rem; margin-top: 5px; }
</style>
"""
content_html = ""

for title, func, s_h, e_h in config:
    is_open = (s_h <= current_hour < e_h) or FORCE_UNLOCK_ALL
    status_class = "badge-live" if is_open else "badge-locked"
    status_text = "LIVE ACCESS" if is_open else "LOCKED"
    time_str = f"{s_h}:00 - {e_h}:00 IST"
    card_content = f"""
    <div class="card">
        <div class="card-header">
            <span>{title}</span>
            <span class="badge {status_class}">{status_text}</span>
        </div>
    """
    
    if is_open:
        try:
            fig = func()
            chart_html = pio.to_html(fig, full_html=False, include_plotlyjs='cdn')
            card_content += chart_html
        except Exception as e:
            card_content += f"<p style='color:red'>Error generating chart: {e}</p>"
    else:
        card_content += f"""
        <div class="locked-overlay">
            <div class="lock-icon">🔒</div>
            <div class="lock-msg">ACCESS DENIED</div>
            <div class="lock-sub">Available only between {time_str}</div>
            <div class="lock-sub">Current Time: {now.strftime('%H:%M')} IST</div>
        </div>
        """
        
    card_content += "</div>"
    content_html += card_content

full_html = f"""
<!DOCTYPE html>
<html>
<head>
    <title>Executive Black Dashboard (6 Charts)</title>
    {css}
</head>
<body>
    <div class="sidebar">
        <div class="brand">NEXUS DATA</div>
        <div class="menu-item active">● Dashboard</div>
        <div class="menu-item">● Analytics</div>
        <div class="menu-item">● Settings</div>
        <div style="margin-top:auto; color:#444; font-size:0.8rem">
            IST: {now.strftime('%H:%M:%S')}<br>
            Loc: Madurai, TN
        </div>
    </div>
    
    <div class="main">
        <div class="header">
            <h1>Executive Overview</h1>
            <div style="text-align:right">
                <span style="color:#888">Status:</span> <span style="color:#00ff7f">Online</span>
            </div>
        </div>
        
        <div class="grid">
            {content_html}
        </div>
    </div>
</body>
</html>
"""
filename = "black_dashboard_6charts.html"
with open(filename, "w", encoding="utf-8") as f:
    f.write(full_html)

print(f"✅ Dashboard generated: {filename}")
webbrowser.open("file://" + os.path.abspath(filename))
