# SAP Materials Management Visualization - Snowflake in Streamlit
 This creates a quick but visually appealing Snowflake in Streamlit application visualization using the MAKT table in SAP HANA.

```
import streamlit as st
import pandas as pd
import altair as alt
from snowflake.snowpark.context import get_active_session

# Set page configuration
st.set_page_config(page_title="SAP Material Management Dashboard", layout="wide")

# Title
st.title("SAP Material Management Analysis")

# Get the active Snowflake session
session = get_active_session()

# Load data functions
def load_language_data():
    return session.sql("""
        SELECT 
            SPRAS as Language,
            COUNT(DISTINCT MATNR) as Material_Count,
            COUNT(DISTINCT MAKTX) as Description_Count
        FROM makt
        GROUP BY SPRAS
        ORDER BY Material_Count DESC
    """).to_pandas()

def load_material_types():
    return session.sql("""
        SELECT 
            LEFT(MAKTX, 20) as Material_Type,
            COUNT(*) as Count
        FROM makt
        WHERE SPRAS = 'E'  -- English language
        GROUP BY Material_Type
        ORDER BY Count DESC
        LIMIT 10
    """).to_pandas()

# Main dashboard
try:
    # Load data
    with st.spinner('Loading data...'):
        lang_data = load_language_data()
        material_types = load_material_types()

    # Create two columns
    col1, col2 = st.columns(2)

    with col1:
        st.subheader("Materials by Language")
        
        # Create bar chart with Altair
        chart1 = alt.Chart(lang_data).mark_bar().encode(
            x=alt.X('LANGUAGE:N', title='Language Code'),
            y=alt.Y('MATERIAL_COUNT:Q', title='Number of Materials'),
            color=alt.Color('LANGUAGE:N', legend=None)
        ).properties(
            title='Number of Materials by Language',
            width=400,
            height=300
        )
        
        st.altair_chart(chart1, use_container_width=True)

    with col2:
        st.subheader("Top Material Types")
        
        # Create pie chart using Altair
        # Note: Altair doesn't directly support pie charts, so we'll create a donut chart
        radius = 100
        chart2 = alt.Chart(material_types).mark_arc(innerRadius=50).encode(
            theta=alt.Theta(field='COUNT', type='quantitative'),
            color=alt.Color(field='MATERIAL_TYPE', type='nominal', legend=alt.Legend(title='Material Type')),
            tooltip=['MATERIAL_TYPE', 'COUNT']
        ).properties(
            title='Distribution of Top 10 Material Types',
            width=400,
            height=300
        )
        
        st.altair_chart(chart2, use_container_width=True)

    # Add summary metrics
    st.subheader("Summary Statistics")
    total_materials = lang_data['MATERIAL_COUNT'].sum()
    total_languages = len(lang_data)
    
    metric_col1, metric_col2 = st.columns(2)
    with metric_col1:
        st.metric("Total Materials", f"{total_materials:,}")
    with metric_col2:
        st.metric("Supported Languages", total_languages)

    # Add interactive data table
    st.subheader("Detailed Language Data")
    st.dataframe(lang_data)

except Exception as e:
    st.error(f"An error occurred: {str(e)}")
```