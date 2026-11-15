import streamlit as st
import google.generativeai as genai
from PIL import Image
import requests
import pandas as pd
from datetime import datetime

st.set_page_config(page_title="TradeLens AI", layout="wide")
st.title("📈 TradeLens AI - SMC + Live Fundamental")
st.write("Chart + Live News = Poora Plan")

# --- API Key ---
try:
    api_key = st.secrets["GEMINI_API_KEY"]
    genai.configure(api_key=api_key)
    model = genai.GenerativeModel("gemini-1.5-flash")
except:
    st.error("GEMINI_API_KEY secrets me nahi mila!")
    st.stop()

# --- 1. LIVE FUNDAMENTAL DATA ---
st.subheader("📰 Live Fundamental - Aaj ki News")
@st.cache_data(ttl=3600)
def get_news():
    try:
        url = "https://nfs.faireconomy.media/ff_calendar_thisweek.json"
        data = requests.get(url, timeout=10).json()
        df = pd.DataFrame(data)
        # Sirf USD + High Impact
        df = df[df['impact'].isin(['High', 'Medium'])]
        return df[['date','time','currency','impact','event','forecast','previous']]
    except:
        return None

news_df = get_news()
if news_df is not None:
    st.dataframe(news_df, use_container_width=True)
    news_text = news_df.to_string()
else:
    st.warning("News load nahi hui, phir bhi chart analysis ho jayega.")
    news_text = "News data available nahi hai."

# --- 2. ASSET ---
asset = st.selectbox("Asset Select Karo:", ["XAUUSD (Gold)", "BTCUSD (Bitcoin)", "EURUSD", "GBPUSD", "USDJPY"])

# --- 3. CHARTS ---
st.divider()
st.subheader("Charts Upload Karo")
col1, col2 = st.columns(2)
with col1:
    h4_file = st.file_uploader("H4 Chart", type=["png","jpg","jpeg"])
    m15_file = st.file_uploader("M15 Chart", type=["png","jpg","jpeg"])
with col2:
    h1_file = st.file_uploader("H1 Chart", type=["png","jpg","jpeg"])
    m5_file = st.file_uploader("M5 Chart", type=["png","jpg","jpeg"])

if st.button("🔍 Analyze Karo - Chart + News", type="primary"):
    if not all([h4_file, h1_file, m15_file, m5_file]):
        st.warning("4ro charts upload karo pehle!")
    else:
        with st.spinner("AI Chart + News mila kar dekh raha hai..."):
            images = [Image.open(f) for f in [h4_file, h1_file, m15_file, m5_file]]
            
            prompt = f"""
            Tum expert SMC + Fundamental trader ho. Asset: {asset}

            LIVE NEWS DATA:
            {news_text}

            4 CHARTS: H4, H1, M15, M5

            Kaam:
            1. SMC: BOS / CHoCH, Order Block, FVG, Liquidity Sweep
            2. Levels: Exact Support / Resistance
            3. FUNDAMENTAL: Upar wali news se batayo - USD strong hoga ya weak?
               Agar aaj High Impact USD news hai to Gold/BTC par kya asar parega?
               News se pehle trade lena chahiye ya wait karna chahiye?
            4. FINAL PLAN:
               - Bias: Buy / Sell / WAIT (News ki wajah se)
               - Entry Zone, SL, TP1, TP2
               - Agar news qareeb hai to bolna "News tak WAIT karo"

            Roman Urdu me simple me samjhao.
            """
            response = model.generate_content([prompt] + images)
            st.success("Analysis Ready!")
            st.markdown(response.text)

st.caption("Disclaimer: Educational only, financial advice nahi hai.")
