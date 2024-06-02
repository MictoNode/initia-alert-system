## Türkçe: Initia Validator İzleme ve Uyarı Sistemi

Bu rehber, Initia validator'unuzu izlemek ve blok sayısı 100'ü geçtiğinde sizi uyarmak için bir sistem kurmanıza yardımcı olacaktır. Adım adım nasıl yapacağınızı göstereceğim.

### Adım 1: Telegram Botu Oluşturma

1. **Telegram uygulamasında BotFather'ı bulun:**
   - Telegram uygulamasını açın ve arama kısmına `@BotFather` yazın.
   - BotFather'ı açın ve `/start` komutunu gönderin.
   - `/newbot` komutunu gönderin ve botunuza bir isim verin.
   - Botunuza bir kullanıcı adı belirleyin (kullanıcı adı `bot` ile bitmelidir, örneğin `InitiaBot`).

2. **API token'ını alın:**
   - BotFather size botunuzun API token'ını verecektir. Bu token'ı bir yere kaydedin.

### Adım 2: Chat ID'sini Bulma

1. **Botunuzu bir Telegram sohbetine ekleyin (grup veya bireysel sohbet).**
2. **Botunuza bir mesaj gönderin.**
3. **API token'ınızı kullanarak aşağıdaki URL'yi tarayıcınızda açın:**
   
   ```bash
   https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates
   ```
   
4. **Yanıt içinde `chat` objesi olacaktır ve `chat` objesi içinde `id` değeri yer alacaktır. Bu `id` değeri, mesajların gönderileceği `chat_id` olacaktır.**

### Adım 3: script.sh Dosyasını Oluşturma

1. **Home dizininizde `script.sh` adında bir dosya oluşturun:**

    ```bash
    nano ~/script.sh
    ```

2. **Aşağıdaki içeriği `script.sh` dosyasına yapıştırın:**

   Not:`local_height=$(curl -s localhost:15657/status | jq -r .result.sync_info.latest_block_height)` RPC PORT'u değiştirmeyi unutmayın.

    ```bash
    #!/bin/bash

    # Türkiye yerel saatini alıyoruz
    current_time=$(TZ="Europe/Istanbul" date +"%Y-%m-%d %H:%M:%S")

    # Local ve network height değerlerini alıyoruz
    local_height=$(curl -s localhost:15657/status | jq -r .result.sync_info.latest_block_height)
    network_height=$(curl -s https://b545809c-5562-4e60-b5a1-22e83df57748.initiation-1.mesa-rpc.ue1-prod.newmetric.xyz/status | jq -r .result.sync_info.latest_block_height)

    # Blocks left hesaplıyoruz
    blocks_left=$((network_height - local_height))

    # Değerleri ekrana ve log dosyasına yazdırıyoruz
    echo "Tarih ve Saat: $current_time" | tee -a ~/initia_log.txt
    echo "Node yüksekliğiniz: $local_height" | tee -a ~/initia_log.txt
    echo "Ağ yüksekliği: $network_height" | tee -a ~/initia_log.txt
    echo "Kalan blok sayısı: $blocks_left" | tee -a ~/initia_log.txt

    # Blocks left 100'den büyükse restart komutunu çalıştırıyoruz
    if [ $blocks_left -gt 100 ]; then
        echo "Kalan blok sayısı 100'den büyük. initiad yeniden başlatılıyor..." | tee -a ~/initia_log.txt
        sudo systemctl restart initiad

        # Telegram botu üzerinden uyarı gönderme
        BOT_TOKEN="YOUR_TELEGRAM_BOT_TOKEN"
        CHAT_ID="YOUR_CHAT_ID"
        MESSAGE="Uyarı! Kalan blok sayısı $blocks_left olarak tespit edildi. initiad yeniden başlatıldı."

        curl -s -X POST "https://api.telegram.org/bot$BOT_TOKEN/sendMessage" -d chat_id="$CHAT_ID" -d text="$MESSAGE" > /dev/null
    else
        echo "Kalan blok sayısı 100 veya daha az. Yeniden başlatmaya gerek yok." | tee -a ~/initia_log.txt
    fi
    ```

    `YOUR_TELEGRAM_BOT_TOKEN` ve `YOUR_CHAT_ID` ile kendi değerlerinizi değiştirin.

4. **Dosyayı çalıştırılabilir hale getirin:**

    ```bash
    chmod +x ~/script.sh
    ```

### Adım 4: Cron ile Script'in Periyodik Çalıştırılması

Script'in her 5 dakikada bir çalışması için `crontab`'ı kullanacağız.

1. **Crontab düzenleyiciyi açın:**

    ```bash
    crontab -e
    ```

2. **Aşağıdaki satırı ekleyin:**

    ```bash
    */5 * * * * ~/script.sh
    ```
    
3. **Cron'u tekrar başlatalım:**

    ```bash
    sudo service cron restart
    ```

### Adım 5: Kontrol

1. **Log komutu ile çalışıyor mu kontrol edelim:**

   ```bash
    cat ~/cron_log.txt
    ```

