# पहले जरूरी टूल्स इंस्टॉल करें
!pip install streamlit yfinance pandas ta pyngrok -q

%%writefile app.py
import streamlit as st
import yfinance as yf
import pandas as pd
from ta.momentum import RSIIndicator
from ta.trend import SMAIndicator

st.title("📈 शेयर ट्रेडिंग सिग्नल ऐप")
st.write("किसी भी शेयर का नाम लिखें और जानें कि अभी क्या करना चाहिए!")

# यूजर से शेयर का नाम इनपुट लेना
ticker = st.text_input("शेयर का टिकर दर्ज करें (जैसे RELIANCE.NS, TCS.NS, TATAMOTORS.NS):", "RELIANCE.NS")

if st.button("एनालिसिस करें"):
    with st.spinner('डेटा लोड हो रहा है...'):
        try:
            stock = yf.Ticker(ticker)
            df = stock.history(period="6mo")
            
            if df.empty:
                st.error("गलत टिकर या डेटा नहीं मिला। कृपया सही नाम डालें (जैसे .NS जोड़कर)।")
            else:
                # इंडिकेटर कैलकुलेट करना
                df['SMA_50'] = SMAIndicator(close=df["Close"], window=50).sma_indicator()
                df['RSI'] = RSIIndicator(close=df["Close"], window=14).rsi()
                
                latest_close = df['Close'].iloc[-1]
                latest_sma = df['SMA_50'].iloc[-1]
                latest_rsi = df['RSI'].iloc[-1]
                
                # रिजल्ट दिखाना
                st.subheader(f"नतीजे: {ticker}")
                st.write(f"**लेटेस्ट कीमत:** ₹{latest_close:.2f}")
                st.write(f"**50-दिन का एवरेज (SMA):** ₹{latest_sma:.2f}")
                st.write(f"**RSI (14):** {latest_rsi:.2f}")
                
                # सिग्नल लॉजिक
                if latest_close > latest_sma and latest_rsi < 70:
                    st.success("👉 सिग्नल: **BUY (खरीदने का मौका - बुलिश ट्रेंड)**")
                elif latest_rsi > 70:
                    st.warning("⚠️ सिग्नल: **SELL / PROFIT BOOK (बाजार ओवरबॉट है)**")
                elif latest_close < latest_sma and latest_rsi < 30:
                    st.info("💡 सिग्नल: **POTENTIAL BUY / OVERSOLD (कीमत बहुत गिर चुकी है, नजर रखें)**")
                else:
                    st.info("⚖️ सिग्नल: **HOLD (कोइ स्पष्ट ट्रेंड नहीं, इंतजार करें)**")
        except Exception as e:
            st.error(f"त्रुटि आई: {e}")
 Stock-app
This tool is created for educational and analytical purposes to assist in technical analysis. Users should perform their own research and risk management before executing actual trades in the stock market.
