import streamlit as st
import pandas as pd
import requests
from io import BytesIO
from concurrent.futures import ThreadPoolExecutor

# Function to make a POST request to the GraphQL endpoint
def make_graphql_request(query):
    url = "https://api.saralpatro.com/graphql"
    headers = {"Content-Type": "application/json"}
    response = requests.post(url, headers=headers, json={'query': query})
    if response.status_code == 200:
        return response.json()
    else:
        return None

# Function to convert AD to BS for a batch of dates
def convert_ad_to_bs(year, month, day):
    query = f"""
    {{
        dates(adYear: {year}, adMonth: {month}, adDay: {day}) {{
            bsDay
            bsMonth
            bsYear
        }}
    }}
    """
    result = make_graphql_request(query)
    if result and 'data' in result and 'dates' in result['data']:
        date = result['data']['dates'][0]
        return f"{date['bsYear']}-{date['bsMonth']}-{date['bsDay']}"
    return None

# Function to convert BS to AD for a batch of dates
def convert_bs_to_ad(year, month, day):
    query = f"""
    {{
        dates(bsYear: {year}, bsMonth: {month}, bsDay: {day}) {{
            adDay
            adMonth
            adYear
        }}
    }}
    """
    result = make_graphql_request(query)
    if result and 'data' in result and 'dates' in result['data']:
        date = result['data']['dates'][0]
        return f"{date['adYear']}-{date['adMonth']}-{date['adDay']}"
    return None

# Function to handle parallel conversion using ThreadPoolExecutor
def parallel_conversion(rows, conversion_func):
    results = []
    with ThreadPoolExecutor(max_workers=10) as executor:
        futures = [executor.submit(conversion_func, row['Year'], row['Month'], row['Day']) for _, row in rows.iterrows()]
        for future in futures:
            results.append(future.result())
    return results

# Streamlit interface
st.title("Efficient Date Converter (AD to BS or BS to AD)")

# Upload Excel file
uploaded_file = st.file_uploader("Upload Excel file", type=["xlsx"])

if uploaded_file is not None:
    df = pd.read_excel(uploaded_file)

    # Extract Year, Month, and Day from Date column
    df['Year'] = pd.to_datetime(df['Date']).dt.year
    df['Month'] = pd.to_datetime(df['Date']).dt.month
    df['Day'] = pd.to_datetime(df['Date']).dt.day

    # Choose conversion type
    conversion_choice = st.selectbox("Choose conversion type", ["AD to BS", "BS to AD"])

    # Convert dates based on user choice
    if st.button("Convert"):
        result_df = df[['Date']].copy()  # Only keep the Date column
        if conversion_choice == "AD to BS":
            result_df['Converted Date'] = parallel_conversion(df, convert_ad_to_bs)
        else:
            result_df['Converted Date'] = parallel_conversion(df, convert_bs_to_ad)

        # Convert the result DataFrame to Excel format
        output = BytesIO()
        with pd.ExcelWriter(output, engine='openpyxl') as writer:
            result_df.to_excel(writer, index=False, sheet_name="Converted Dates")

        # Provide a download link for the converted Excel file
        st.download_button(
            label="Download Converted Excel",
            data=output.getvalue(),
            file_name="converted_dates.xlsx",
            mime="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
        )
