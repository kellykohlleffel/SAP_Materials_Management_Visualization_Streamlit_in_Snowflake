# SAP Material Management Dashboard

## Overview
This Streamlit in Snowflake application provides a visual analytics dashboard for SAP MAKT (Material Description) table data. The dashboard offers interactive visualizations and summary statistics to help analyze material management data across different languages and material types.

## Prerequisites
* Access to a Snowflake account with Streamlit in Snowflake enabled
* Access to the SAP MAKT table in schema: `reliable_sap_snowflake_sapabap1`
* Required table columns: `maktg`, `maktx`, `mandt`, `matnr`, `spras`

## Features

### 1. Materials by Language Visualization
* Bar chart showing distribution of materials across different language codes
* Interactive tooltips with detailed counts
* Color-coded for easy differentiation

### 2. Top Material Types Analysis
* Donut chart displaying top 10 material types
* Based on English language descriptions (SPRAS = 'E')
* Interactive tooltips showing counts and percentages

### 3. Summary Statistics
* Total number of materials in the system
* Number of supported languages
* Quick-view metrics for key performance indicators

### 4. Detailed Data Table
* Sortable data grid showing language-wise material counts
* Interactive sorting capabilities
* Complete overview of material distribution

## Installation Steps
1. Navigate to Snowflake and create a new Streamlit application
2. Select the following context:
   * Database: Your database name
   * Schema: `reliable_sap_snowflake_sapabap1`
3. Copy the application code into the editor
4. Click "Run" to deploy the dashboard

## Streamlit Application Code
The following code creates the complete dashboard. Copy this entire code block into your Streamlit in Snowflake application:

```
# SAP Material Management Dashboard
# A Streamlit in Snowflake application that visualizes SAP MAKT table data
# Features: Language distribution, material type analysis, and summary metrics

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

## Data Architecture
The application queries the following table:
* Table: `makt`
* Key columns used:
  * `SPRAS`: Language code
  * `MATNR`: Material number
  * `MAKTX`: Material description

## Technical Implementation
* Built using Streamlit in Snowflake
* Visualizations created with Altair
* Uses Snowpark session for data access
* Implements efficient data loading with direct SQL queries
* Responsive design that adapts to window size

## Performance Considerations
* Queries are optimized to aggregate data at the database level
* Limited to top 10 material types to ensure quick loading
* Uses container width for responsive visualization sizing

## Maintenance
To update the dashboard:
1. Access the Streamlit app in your Snowflake account
2. Modify the code as needed
3. Click "Run" to apply changes

## Troubleshooting
If you encounter issues:
1. Verify schema access permissions
2. Ensure the MAKT table contains data
3. Check that all required columns are present
4. Review the error message in the Streamlit interface

## Future Enhancements
Potential areas for expansion:
* Add material search functionality
* Implement time-based analysis
* Include additional SAP tables for broader analysis
* Add export capabilities for reports
* Implement custom filtering options

## Security
The application uses Snowflake's native security features and requires appropriate access to:
* The specified schema
* The MAKT table
* Streamlit in Snowflake functionality