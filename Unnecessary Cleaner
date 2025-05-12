import streamlit as st
import pandas as pd
import os
import io
import zipfile

# --- Helper Functions ---
def get_country(phone):
    phone = str(phone).replace(' ', '').replace('-', '')
    for country, prefixes in country_codes.items():
        for prefix in prefixes:
            if phone.startswith(prefix):
                return country
    return 'unknown'

def process_file(uploaded_file):
    # Read file
    if uploaded_file.name.endswith('.csv'):
        df = pd.read_csv(uploaded_file, on_bad_lines='skip')
    else:
        df = pd.read_excel(uploaded_file)

    df.fillna('', inplace=True)
    if 'Website' not in df.columns:
        df['Website'] = ''

    columns_to_keep = [
        "Name", "Address", "Street Address", "City", "State", "ZipCode",
        "Plus Code", "Phone", "Website", "Image URL", "Listing URL"
    ]

    trailing_columns = [
        col for col in df.columns
        if col not in columns_to_keep and df[col].astype(str).str.strip().eq('').all()
    ]
    df = df[[col for col in columns_to_keep if col in df.columns] + trailing_columns]

    if 'Plus Code' in df.columns:
        df['Plus Code'] = df['Plus Code'].str.replace(r',?\s*USA$', '', regex=True)

    phone_only_df = df[(df['Phone'].str.strip() != '') & (df['Website'].str.strip() == '')]
    main_df = df[(df['Website'].str.strip() != '') | (df['Phone'].str.strip() != '')]
    main_df = main_df[~((main_df['Phone'].str.strip() == '') & (main_df['Website'].str.strip() == ''))]

    urls_to_remove = [
        "http://serways.de", "http://mcdonalds.com", "http://tankstelle.aral.de",
        "http://nino-vino.de", "http://jet.de", "http://localchamp.de",
        "http://metro.bar", "http://metro.biz", "http://h-hotels.com",
        "http://hilton.com", "http://kempinski.com", "http://globus.de",
        "http://porta.de", "http://pizzaservice.de", "http://burgerking.de",
        "http://achat-hotels.com", "http://vapiano.de", "http://losteria.net",
        "http://jimdosite.com", "http://m.facebook.com", "http://speisekartenweb.de",
        "https://www.google.com", "http://bit.ly", "http://dominos.de",
        "http://gambrinus-hotel.de", "http://nordsee.com", "http://moments-hotel.de",
        "pizzahut.com", "hilton.com"
    ]
    urls_to_remove = [url.lower() for url in urls_to_remove]
    main_df = main_df[~main_df['Website'].str.lower().isin(urls_to_remove)]

    # Country detection
    df['Country'] = df['Phone'].apply(get_country)

    country_files = {}
    for country in df['Country'].unique():
        df_country = df[df['Country'] == country].copy()
        df_country.drop(columns=['Country'], inplace=True)
        if 'Website' not in df_country.columns:
            df_country['Website'] = ''
        csv = df_country.to_csv(index=False).encode('utf-8')
        country_files[f'country_{country}.csv'] = csv

    # Output files
    files = {
        "filtered_main.csv": main_df.to_csv(index=False).encode('utf-8'),
        "phone_only.csv": phone_only_df.to_csv(index=False).encode('utf-8'),
        **country_files
    }

    # Create zip
    zip_buffer = io.BytesIO()
    with zipfile.ZipFile(zip_buffer, "a", zipfile.ZIP_DEFLATED) as zf:
        for name, content in files.items():
            zf.writestr(name, content)
    zip_buffer.seek(0)
    return zip_buffer


# --- COUNTRY CODES ---
country_codes = {
    'US': ['+1'], 'NG': ['+234'], 'NL': ['+31'], 'BE': ['+32', '32'],
    'FR': ['33'], 'ES': ['34'], 'PT': ['351', '352'], 'IE': ['353'],
    'AL': ['355'], 'MT': ['356'], 'HU': ['361', '36'], 'LT': ['370'],
    'SI': ['386'], 'IT': ['39'], 'CH': ['41'], 'CZ': ['420'],
    'SK': ['421'], 'AT': ['43'], 'UK': ['44'], 'DK': ['45'],
    'PL': ['48'], 'AU': ['61'], 'ID': ['62'], 'PH': ['63'],
    'TH': ['66'], 'JP': ['81'], 'CN': ['86'], 'TR': ['90'],
    'IN': ['91'], 'PK': ['92'], 'LK': ['94'], 'BD': ['95'],
    'JO': ['962'], 'SA': ['966'], 'AE': ['971'], 'BH': ['973'],
    'NP': ['977'], 'DE': ['0151', '0152', '0157', '0160', '0162',
                          '0163', '0170', '0171', '0172', '0174',
                          '0175', '0176', '0177', '0178', '0179', '0']
}

# --- STREAMLIT UI ---
st.set_page_config(page_title="Restaurant Contact Processor", layout="centered")
st.title("📊 Restaurant Contact Processor")
st.markdown("Upload a CSV or Excel file and get filtered & organized contact info.")

uploaded_file = st.file_uploader("Choose a file", type=["csv", "xlsx"])

if uploaded_file:
    if st.button("🚀 Process File"):
        with st.spinner("Processing..."):
            zip_file = process_file(uploaded_file)
        st.success("✅ Done! Download your files below:")
        st.download_button("📦 Download ZIP", data=zip_file, file_name="processed_output.zip", mime="application/zip")
