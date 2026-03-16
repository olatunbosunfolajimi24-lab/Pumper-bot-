# Pumper-bot-
import requests
import time
from datetime import datetime

def send_telegram_message(bot_token, chat_id, message):
    """
    Sends a text message to a specific Telegram chat.
    """
    url = f"https://api.telegram.org/bot{bot_token}/sendMessage"
    payload = {
        "chat_id": chat_id,
        "text": message,
        "parse_mode": "HTML" # Allows us to use bolding and links in the message
    }
    
    try:
        response = requests.post(url, json=payload)
        response.raise_for_status()
    except Exception as e:
        print(f"Failed to send Telegram alert: {e}")

def monitor_pumpfun_token(token_address, bot_token, chat_id, target_mcap=15000):
    """
    Monitors a token and triggers a Telegram alert when target MCAP is hit.
    """
    url = f"https://api.dexscreener.com/latest/dex/tokens/{token_address}"
    total_supply = 1_000_000_000 
    
    print(f"[{datetime.now().strftime('%H:%M:%S')}] Monitoring {token_address}...")
    print(f"Alert will be sent to Telegram at ${target_mcap:,} Market Cap.\n")
    
    while True:
        try:
            response = requests.get(url).json()
            
            if response.get('pairs'):
                pair = response['pairs'][0]
                price_usd = float(pair.get('priceUsd', 0))
                current_mcap = price_usd * total_supply
                
                print(f"[{datetime.now().strftime('%H:%M:%S')}] Current Market Cap: ${current_mcap:,.2f}")
                
                if current_mcap >= target_mcap:
                    # 1. Format the alert message
                    dexscreener_link = f"https://dexscreener.com/solana/{token_address}"
                    alert_msg = (
                        f"🚨 <b>PUMP.FUN ALERT</b> 🚨\n\n"
                        f"<b>Token:</b> <code>{token_address}</code>\n"
                        f"<b>Market Cap:</b> ${current_mcap:,.2f}\n"
                        f"<b>Price:</b> ${price_usd:.6f}\n\n"
                        f"<a href='{dexscreener_link}'>View on DexScreener</a>"
                    )
                    
                    # 2. Send it to Telegram
                    send_telegram_message(bot_token, chat_id, alert_msg)
                    
                    # 3. Print to console and stop monitoring
                    print("\n🚨 TARGET REACHED! TELEGRAM ALERT SENT. 🚨")
                    break 
            else:
                print(f"[{datetime.now().strftime('%H:%M:%S')}] No trading data found yet.")
                
        except Exception as e:
            print(f"Error fetching data: {e}")
            
        time.sleep(5)

# --- EXECUTION ---
if __name__ == "__main__":
    # --- ENTER YOUR CREDENTIALS HERE ---
    TELEGRAM_BOT_TOKEN = "8426488451:AAHnLxsLdU2u059UY_hQ66MSpaifN70tRnc"
    TELEGRAM_CHAT_ID = "6620107353"
    TARGET_TOKEN = "FksjYMq38iQigRRozNZGkpjWj3AGxoUw45fnS8Nnpump" 
    
    monitor_pumpfun_token(TARGET_TOKEN, TELEGRAM_BOT_TOKEN, TELEGRAM_CHAT_ID, target_mcap=15000)