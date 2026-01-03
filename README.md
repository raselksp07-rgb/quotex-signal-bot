from flask import Flask, render_template
import yfinance as yf
import ta

app = Flask(__name__)

PAIR = "EURUSD=X"

def get_signal():
    df = yf.download(PAIR, period="1d", interval="1m")

    df['rsi'] = ta.momentum.RSIIndicator(df['Close'], 14).rsi()
    df['ema9'] = ta.trend.EMAIndicator(df['Close'], 9).ema_indicator()
    df['ema21'] = ta.trend.EMAIndicator(df['Close'], 21).ema_indicator()

    last = df.iloc[-1]

    if last['rsi'] < 30 and last['ema9'] > last['ema21']:
        return "🟢 STRONG BUY (CALL)"
    elif last['rsi'] > 70 and last['ema9'] < last['ema21']:
        return "🔴 STRONG SELL (PUT)"
    else:
        return "⚪ NO TRADE"

@app.route("/")
def index():
    return render_template("index.html", signal=get_signal())

if __name__ == "__main__":
    app.run()
