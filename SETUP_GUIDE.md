# TQQQ ladder bot: step-by-step setup guide

Written for add-on version 0.1.68.

This guide is for someone setting up the TQQQ ladder bot for the first time, with a helper sitting next to them (in person or on a call). Do the steps in order. Each step says **who** does it: **You** (the account owner) or **Helper**. Tick the box when a step is done.

## What this thing is, in plain words

- You put money in a brokerage account at **Interactive Brokers (IBKR)**.
- A **Google Sheet** works out the plan: at which prices to buy TQQQ, at which prices to sell, and how many shares each time. It buys a little more each time the price drops, and sells each piece for a small profit when the price bounces back up. That is the "ladder".
- A **small computer** in your home runs the **bot**. The bot reads the Sheet, places the orders at IBKR, and writes back what happened. It never makes up its own prices. It just follows the Sheet.
- **Your phone** approves the bot's login once a week and can get alerts when something needs a human.

**Important to know before you start:** TQQQ is a 3x leveraged fund. It can drop a lot, and quickly. In bad years the account can be worth much less than you put in, even while the bot is booking small profits. Only use money you can leave alone through a big drop.

## What you need

- About $150–$300 for a used mini PC (Step 6), plus a network cable.
- A Google account.
- A smartphone.
- A password manager, or a safe place to write passwords. You will create several.
- The money you want to trade, ready to transfer.
- About half a day for Steps 6–11, plus a few days of waiting for IBKR to approve the account and for the money to arrive.

You do **not** need to pay IBKR for a live market data subscription. The bot works with IBKR's free delayed prices.

## Part 1: The brokerage account (IBKR)

### Before you start: how many accounts? (You, with Helper)

Each IBKR account gets its **own bot, its own Google Sheet and its own bot login**. Most people need one or two:

- **One account is simplest.** It can be a regular account or a tax-sheltered one.
- **Tax-sheltered options:** an **IRA** (Roth or traditional) at IBKR, or, if you are self-employed, a **Solo 401k**. A Solo 401k is usually set up by a Solo 401k provider first, and then IBKR opens an account owned by the plan. Old 401k or IRA money can move in with a **direct rollover** (it goes straight from the old company to IBKR, never through your hands).
- **Paper account:** one is enough for practice. IBKR gives one to each live account.
- **Two accounts on one box** work out of the box (the Account 1 and Account 2 add-ons). Each extra account means one more IBKR Mobile approval every Sunday.
- Which account types fit you is a tax question. Ask a tax preparer or adviser before you move retirement money.

The steps below are for one account. For a second account, repeat Steps 1–5 and 10–13 with the Account 2 add-on.

### Step 1. Open and fund the IBKR account (You)

- [ ] Go to interactivebrokers.com and open an **Individual** account. Approval can take a few days.
- [ ] **Account type:** ask your Helper whether to pick margin or cash. With a cash account, money from a sale takes a day to settle, which can block the bot's fast buy-then-sell cycles. A margin account avoids that, and the bot never borrows on its own because the Sheet sizes every order from your cash. An IRA or 401k cannot borrow at all; ask your Helper which IRA option IBKR offers that avoids the settlement wait.
- [ ] **Trading permissions:** turn on **US stocks/ETFs**. If IBKR asks you to accept a risk notice for leveraged or complex ETFs, accept it, because TQQQ is one.
- [ ] Install the **IBKR Mobile** app on your phone and turn on two-factor login with it (IBKR calls it IBKR Mobile Authentication or IB Key).
- [ ] **Move the money in** (bank transfer). Wait until it shows as available cash.
- [ ] Write down your **live account number** (starts with **U**).
- [ ] Turn on the **paper trading** (practice) account in Settings. Write down the **paper account number** (starts with **DU**). We test with this first.

### Step 2. Set the tax lot method to LIFO (You)

When you own shares you bought at different prices and sell some, IBKR has to pick which ones you sold. The ladder always sells the shares it bought most recently, so we tell IBKR "last in, first out" (LIFO).

- [ ] In IBKR's website: **Settings → Account Settings → Tax Optimizer** (or "Lot Matching Method"), set the default to **LIFO**.

This changes how your taxes are calculated. It is what your Helper uses, but check with your own tax preparer if you are unsure.

### Step 3. Make a separate login just for the bot (You)

The bot needs its own username and password. Why: IBKR only lets one session per username trade at a time. If the bot used your login, every time you opened the IBKR app on your phone you would knock the bot offline (or it would knock you off).

- [ ] In IBKR's website: **Settings → Users & Access Rights → add a user** (+).
- [ ] Give the new user **trading access** to your account. It does not need permission to move money out, so leave withdrawals and transfers off.
- [ ] Finish the new user's setup: confirm its email and set its password.
- [ ] Set up **two-factor login for the bot's user** in IBKR Mobile on **your** phone. Once a week your phone will get a push asking you to approve the bot's login (see Part 5).
- [ ] Save the bot's **username** and **password** in your password manager. If the paper account has its own username for this user, save that too.

## Part 2: The Google Sheet (the brain)

### Step 4. Get your own copy of the Sheet (Helper, then You)

- [ ] Helper shares the ladder Sheet template with you.
- [ ] Open it and click **File → Make a copy**. Save the copy in your own Google Drive. Use **your copy** from now on.
- [ ] Check it has these tabs: **TQQQ_Tracker**, **Fills**, **Health**, **Errors**. Do not rename them.
- [ ] With your Helper, fill in the yellow input cells on **TQQQ_Tracker**: **I2** = the cash you are giving the ladder, **F3** = reduction factor (Helper picks), **H3** = `1.64`.
- [ ] Type the **starting anchor price** into **G7** by hand. Your Helper picks it. This is the price the whole ladder is built from. After that the bot updates G7 by itself (for example, after a run-up it writes the price the anchor was bought back at).
- [ ] Copy the **Sheet ID**. It is the long code in the Sheet's web address between `/d/` and `/edit`. Save it.

### Step 5. Give the bot a "robot" Google account (You, with Helper)

The bot cannot log in as you, so Google gives it a robot account, called a **service account**, with its own email address and a key file.

- [ ] Go to **console.cloud.google.com** and sign in with your Google account.
- [ ] Create a **new project** (any name, for example `tqqq-bot`).
- [ ] In **APIs & Services → Library**, turn on **Google Sheets API** and **Google Drive API**.
- [ ] In **IAM & Admin → Service Accounts**, click **Create service account**. Any name. Skip the optional role steps.
- [ ] Open the new service account, go to **Keys → Add key → Create new key → JSON**. A small `.json` file downloads. **This file is a password.** Do not email it or post it anywhere.
- [ ] Copy the service account's **email address**. It looks like `tqqq-bot@your-project.iam.gserviceaccount.com`.
- [ ] Back in your Sheet, click **Share**, paste that email, and give it **Editor** access.

## Part 3: The little computer

### Step 6. Buy the computer (You)

- [ ] Get a small **Intel or AMD** mini PC ("x86-64"). **Not a Raspberry Pi**: the bot is only built for Intel/AMD computers. A used business mini PC (Dell OptiPlex Micro, Lenovo ThinkCentre Tiny, HP EliteDesk Mini) or a new Intel N100 box all work.
- [ ] At least **8 GB of memory** and a **64 GB or bigger** SSD.
- [ ] Plan to plug it into your router with a **network cable**. Wi-Fi is not supported by Home Assistant OS on most of these.
- [ ] Optional but nice: a small battery backup (UPS) so short power blips do not restart it.

### Step 7. Install Home Assistant OS on it (Helper, with You)

Home Assistant OS is the free operating system that runs the bot. It replaces Windows on that computer; everything on it gets erased.

- [ ] Follow the official guide at **home-assistant.io → Installation → Generic x86-64**. In short: write the Home Assistant OS image onto the mini PC's drive, set the computer's BIOS to boot UEFI, and start it.
- [ ] In the BIOS, also turn on **"Restore on AC power loss" / "Power on after power failure"** so it turns itself back on after a power cut.
- [ ] On another computer on the same network, open **http://homeassistant.local:8123**. Wait for it to finish setting up (can take 20 minutes).
- [ ] Create the **owner account** with a strong password. Save it.
- [ ] Set your **location and time zone** when asked.

### Step 8. Set up remote access with Tailscale (Helper, with You)

Tailscale lets your Helper reach the computer from anywhere to fix problems, without opening anything to the internet.

- [ ] Make a free account at **tailscale.com** (You).
- [ ] In Home Assistant: **Settings → Add-ons → Add-on Store**, find **Tailscale**, **Install**, turn on **Start on boot**, then **Start**.
- [ ] Open its **Web UI** and log in with your Tailscale account to add this computer.
- [ ] In the Tailscale admin page, **share this machine** with your Helper (or invite them).
- [ ] In Home Assistant, **Settings → People → Add person** for your Helper, with a login and admin rights.
- [ ] Helper checks they can open Home Assistant from their own place.

## Part 4: The bot

### Step 9. Install the bot add-on (Helper)

- [ ] **Settings → Add-ons → Add-on Store**, then the **⋮** menu (top right) → **Repositories**.
- [ ] Paste `https://github.com/Wakeboardsam/shorts-and-ladders` and click **Add**.
- [ ] Refresh the page. Install **V7_tqqq_bot**. (The "Account 2" add-on is only for a second IBKR account; skip it.)
- [ ] **Do not start it yet.**

### Step 10. Put the passwords and settings in (You type, Helper guides)

Open the add-on and go to the **Configuration** tab. Start in **practice mode**:

**Account and trading**

- [ ] Trading mode: **paper**
- [ ] IBKR account ID: your **paper** account number (DU…)
- [ ] IBKR username and password: the **bot's** login from Step 3 (the paper one, if it is different)
- [ ] Watch only: **on**
- [ ] Gateway read-only: **on**
- [ ] License key: the key your Helper gives you (it starts with `SL1.`)

**Google Sheet**

- [ ] Sheet ID: from Step 4
- [ ] Service account key: open the `.json` file from Step 5 and paste **all of it on one single line**. Your Helper can help squash it to one line.

**Time zone and schedule**

- [ ] Time zone: yours. Leave the other times as they are unless your Helper changes them.

**Strategy and safeguards, Advanced**

- [ ] Leave as they are.

Then:

- [ ] Click **Save**.
- [ ] On the add-on's **Info** tab, turn on **Start on boot** and **Watchdog** (Watchdog restarts the bot by itself if it gets stuck).

### Step 11. First start in practice mode (Helper, with You)

- [ ] Click **Start**, then open the **Log** tab.
- [ ] Wait for the IBKR Gateway to log in and for the bot to say it started (a few minutes).
- [ ] In your Sheet, check that **C1** (heartbeat) keeps updating and the **Health** tab gets new rows.
- [ ] If the log says the Gateway is stuck on a pop-up, your Helper turns on the Gateway screen (VNC) once to click it, then turns VNC off again.

### Step 12. Phone alerts (You, with Helper; optional but recommended)

- [ ] Install the **Home Assistant** app on your phone and log in.
- [ ] Helper creates a Home Assistant automation: trigger **Webhook**, action **send a notification to your phone**. Copy its webhook URL.
- [ ] In the add-on's **Phone notifications** section: turn on **enabled**, paste the webhook URL, and turn on at least: halts, bridge halt, share mismatch, Gateway login needed, watchdog restart, errors. Fills are optional.
- [ ] Save and restart the add-on. Check that the "bot started" alert arrives (if you turned it on).

### Step 13. Go live (You, with Helper)

First, reset the Sheet so the practice trades are gone. If you skip this, the bot compares the practice rows with your real account, finds the shares do not match, and stops itself.

- [ ] **Stop** the add-on.
- [ ] On **TQQQ_Tracker**, rows 7–100: clear column **C** (an empty cell means `IDLE`) and remove every `Y` in column **D**.
- [ ] Set **I2** to the real cash for the ladder, and type a fresh starting anchor price into **G7** (your Helper picks it).
- [ ] Optional: delete the practice rows (not the header row) on the **Fills**, **Health** and **Errors** tabs, so profit totals only count real trades.

Then, live but watching only:

- [ ] Change **all four together**: Trading mode **live**, account ID = your **live** number (U…), the bot's **live** username and password.
- [ ] Keep **Watch only** and **Gateway read-only** on. Save, then **Start**.
- [ ] Your phone gets an **IBKR Mobile push**. Approve it within about 3 minutes.
- [ ] Helper checks the log: the bot sees the right account, the cash, and the Sheet.

Then, when your Helper says it all looks right, turn trading on:

- [ ] **Stop** the add-on. Turn **Watch only off** and **Gateway read-only off**. Save, **Start**, approve the push.
- [ ] The bot places its first order (the anchor buy at row 7). From now on it runs by itself.

## Part 5: Living with the bot

### Your weekly job

- **Every Sunday at 9:00 your time** (the default), the bot logs back in for the week. **Approve the IBKR Mobile push** on your phone. If you miss it, it tries again about an hour later.
- The bot turns itself off from **Friday 6:15 pm to Sunday 9:00 am** (default times). That is normal.
- A short pause every night just before midnight is also normal.

### Rules so the bot stays safe

- **Do not buy or sell TQQQ by hand** in this account, and do not place your own TQQQ orders. The bot will see orders it does not recognize and **stop on purpose** to protect you.
- **Do not edit** the Sheet's status columns (C and D) or the price columns. Only change the yellow input cells, and only with your Helper.
- **Adding more money:** do it on a weekend, with your Helper, so the Sheet can be updated to match.
- Leave the little computer **plugged in and on the network**.
- Never share your passwords, the `.json` key, or screenshots of the Configuration page.

### If something goes wrong

- If you get an alert that says **halt**, **mismatch** or **login needed**, do not try to fix it in IBKR yourself. Message your Helper.
- The bot stops itself whenever the Sheet and IBKR disagree. That is a safety feature, not a crash. Your Helper compares the two, fixes the Sheet, and restarts the bot.

## Things to save (fill in, keep private)

| What | Where it came from |
|---|---|
| IBKR live account number (U…) | Step 1 |
| IBKR paper account number (DU…) | Step 1 |
| Your own IBKR login | Step 1 |
| Bot's IBKR username and password (live and paper) | Step 3 |
| Google Sheet ID | Step 4 |
| Service account email and `.json` key file | Step 5 |
| Home Assistant owner login | Step 7 |
| Tailscale account | Step 8 |
| Home Assistant webhook URL | Step 12 |
