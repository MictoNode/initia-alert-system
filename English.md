## English: Initia Validator Monitoring and Alert System

This guide will help you set up a system to monitor your Initia validator and alert you via Telegram when the block count exceeds 100. I will show you step by step how to do it.

### Step 1: Creating a Telegram Bot

1. **Find BotFather on the Telegram app:**
   - Open the Telegram app and search for `@BotFather`.
   - Open BotFather and send the `/start` command.
   - Send the `/newbot` command and give your bot a name.
   - Choose a username for your bot (the username must end with `bot`, e.g., `InitiaBot`).

2. **Get the API token:**
   - BotFather will provide you with the API token for your bot. Save this token as you will need it shortly.

### Step 2: Finding the Chat ID

1. **Add your bot to a Telegram chat (group or individual chat).**
2. **Send a message to your bot.**
3. **Open the following URL in your browser using your API token:**
   ```bash
   https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates
   ```
4. **You will receive a JSON response containing the message you sent to your bot. Inside this response, there will be a `chat` object, and within this object, there will be an `id` field. This `id` value is the `chat_id` where messages will be sent.**

### Step 3: Creating the script.sh File

1. **Create a file named `script.sh` in your home directory:**

    ```bash
    nano ~/script.sh
    ```

2. **Paste the following content into the `script.sh` file:**

    ```bash
    #!/bin/bash

    # Get the current time in UTC
    current_time=$(date -u +"%Y-%m-%d %H:%M:%S")

    # Get local and network heights
    local_height=$(curl -s localhost:15657/status | jq -r .result.sync_info.latest_block_height)
    network_height=$(curl -s https://b545809c-5562-4e60-b5a1-22e83df57748.initiation-1.mesa-rpc.ue1-prod.newmetric.xyz/status | jq -r .result.sync_info.latest_block_height)

    # Calculate blocks left
    blocks_left=$((network_height - local_height))

    # Output values to the screen and log file
    echo "Date and Time: $current_time" | tee -a ~/initia_log.txt
    echo "Your node height: $local_height" | tee -a ~/initia_log.txt
    echo "Network height: $network_height" | tee -a ~/initia_log.txt
    echo "Blocks left: $blocks_left" | tee -a ~/initia_log.txt

    # If blocks left is greater than 100, run the restart command
    if [ $blocks_left -gt 100 ]; then
        echo "Blocks left is greater than 100. Restarting initiad..." | tee -a ~/initia_log.txt
        sudo systemctl restart initiad

        # Send alert via Telegram bot
        BOT_TOKEN="YOUR_TELEGRAM_BOT_TOKEN"
        CHAT_ID="YOUR_CHAT_ID"
        MESSAGE="Alert! Blocks left is $blocks_left. initiad has been restarted."

        curl -s -X POST "https://api.telegram.org/bot$BOT_TOKEN/sendMessage" -d chat_id="$CHAT_ID" -d text="$MESSAGE" > /dev/null
    else
        echo "Blocks left is 100 or less. No restart needed." | tee -a ~/initia_log.txt
    fi
    ```

    Replace `YOUR_TELEGRAM_BOT_TOKEN` and `YOUR_CHAT_ID` with your actual values.

3. **Make the file executable:**

    ```bash
    chmod +x ~/script.sh
    ```

### Step 4: Running the Script Periodically with Cron

We will use `crontab` to run the script every 5 minutes.

1. **Open the crontab editor:**

    ```bash
    crontab -e
    ```

2. **Add the following line:**

    ```bash
    */5 * * * * ~/script.sh
    ```

3. **Restart Cron**

    ```bash
    sudo service cron restart
    ```

### Step 5: Check

1. **Let's check if it works with log command:**

    ```bash
    cat ~/cron_log.txt
    ```
