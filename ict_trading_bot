import yfinance as yf
import pandas as pd
import gspread
from oauth2client.service_account import ServiceAccountCredentials
from datetime import datetime

def connect_sheet(sheet_name="ICT_Trading_Log", creds_file="your-creds.json"):
    scope = ["https://spreadsheets.google.com/feeds",'https://www.googleapis.com/auth/drive']
    creds = ServiceAccountCredentials.from_json_keyfile_name(creds_file, scope)
    client = gspread.authorize(creds)
    return client.open(sheet_name).sheet1

def log_to_sheet(sheet, data_row):
    sheet.append_row(data_row)

def detect_bos(df, lookback=20):
    signals = []
    for i in range(lookback, len(df)):
        recent_high = df['High'].iloc[i-lookback:i].max()
        recent_low = df['Low'].iloc[i-lookback:i].min()
        close = df['Close'].iloc[i]
        timestamp = df.index[i]
        if close > recent_high:
            signals.append((timestamp, 'Bullish BOS', close))
        elif close < recent_low:
            signals.append((timestamp, 'Bearish BOS', close))
    return signals

def detect_fvg(df):
    signals = []
    for i in range(2, len(df)):
        prev2 = df.iloc[i - 2]
        current = df.iloc[i]
        timestamp = df.index[i]
        if current['Low'] > prev2['High']:
            signals.append((timestamp, 'Bullish FVG', prev2['High'], current['Low']))
        elif current['High'] < prev2['Low']:
            signals.append((timestamp, 'Bearish FVG', current['High'], prev2['Low']))
    return signals

def is_in_session(timestamp, session="london"):
    hour = timestamp.hour
    if session == "london":
        return 8 <= hour < 11
    elif session == "new_york":
        return 13 <= hour < 16
    return False

def run_ict_bot(ticker="AAPL", session_filter="new_york", creds_file="your-creds.json"):
    df = yf.download(ticker, interval="5m", period="7d").dropna()
    sheet = connect_sheet(creds_file=creds_file)

    bos_signals = detect_bos(df)
    fvg_signals = detect_fvg(df)

    for timestamp, signal_type, price in bos_signals:
        if is_in_session(timestamp, session_filter):
            trade_id = f"{ticker}_{timestamp.strftime('%Y%m%d_%H%M')}"
            log_to_sheet(sheet, [str(timestamp), ticker, "Stock", "Signal", price, signal_type, "", trade_id])

    for timestamp, signal_type, from_price, to_price in fvg_signals:
        if is_in_session(timestamp, session_filter):
            trade_id = f"{ticker}_{timestamp.strftime('%Y%m%d_%H%M')}"
            note = f"Gap: {from_price} → {to_price}"
            log_to_sheet(sheet, [str(timestamp), ticker, "Stock", "Signal", from_price, signal_type, note, trade_id])

if __name__ == "__main__":
    run_ict_bot("AAPL", "new_york", "your-creds.json")
