# Stacking Sats on Kraken

## ✋ Caveat

You need to install the dependency [kraken-api](https://github.com/nothingisdead/npm-kraken-api), which is a third-party package.
It has a minimal set of dependencies and I've done my best to audit its code.
Also the version is fixed, so that unwanted changes do not slip in.

However: Use this at your own risk and decide for yourself whether or not you want to run this script and its dependencies!

## 🔑 API Key

Obtain your Kraken API Key via [the API settings page](https://www.kraken.com/u/settings/api).
Generate a new API key dedicated for stacking using the "Query Funds" and "Modify Orders" permissions:

![Kraken API Key Permissions](./api-permissions.png)

Only check the "Withdraw Funds" option if you plan to automatically withdraw Bitcoin from Kraken.
See details below.

## ⚙️ Configuration

These are the environment variables used by the script:

```sh
# used to authenticate with Kraken
KRAKEN_API_KEY="apiKeyFromTheKrakenSettings"
KRAKEN_API_SECRET="privateKeyFromTheKrakenSettings"

# used for buying
KRAKEN_API_FIAT="USD" # the governmental shitcoin you are selling
BUY_CRYPTO="XBT" # what you are buying
KRAKEN_BUY_AMOUNT=21 # fiat amount you trade for the future of money

# used for withdrawal
KRAKEN_MAX_REL_FEE=0.5 # maximum fee in % that you are willing to pay
KRAKEN_WITHDRAW_KEY="descriptionOfWithdrawalAddress"

# remove this line after verifying everything works
KRAKEN_DRY_RUN_PLACE_NO_ORDER=1
```
## 📦 Setup

Prerequisite: At least the current LTS version of [Node.js](https://nodejs.org/).

Install the dependencies:

```sh
npm install
```

Setup the environment variables for the script.
See the [config section above](#-configuration).

Use a dry run to test the script and see the output without placing an order:

```sh
npm run test:stack
```

You should see something like this sample output:

```text
💰 Balance: 210000.00 USD / 21.0 XBT

📈 Ask: 21000.2 USD
📉 Bid: 21000.1 USD

💸 Order: buy 0.21212121 XBTUSD @ limit 21000.1
📎 Transaction ID: 2121212121
```

## 🤑 Stack sats

When you are good to go, execute this command in a regular interval:

```sh
npm run stack
```

The best and easiest way is to wrap it all up in a shell script.
This script can be triggered via cron job, e.g. weekly, daily or hourly.

Here's a sample `stacksats.sh` script:

```sh
#!/bin/bash
set -e

export KRAKEN_API_KEY="apiKeyFromTheKrakenSettings"
export KRAKEN_API_SECRET="privateKeyFromTheKrakenSettings"
export BUY_CRYPTO="XBT"
export KRAKEN_API_FIAT="EUR"
export KRAKEN_BUY_AMOUNT=10
export KRAKEN_MAX_REL_FEE=0.5
export KRAKEN_WITHDRAW_KEY="descriptionOfWithdrawalAddress"
export KRAKEN_DRY_RUN_PLACE_NO_ORDER=1

# run script
cd $(cd `dirname $0` && pwd)
cmd=${1:-"stack"}

if [[ "${KRAKEN_DRY_RUN_PLACE_NO_ORDER}" ]]; then
  result=$(npm run test:$cmd --silent 2>&1)
else
  result=$(npm run $cmd --silent 2>&1)
fi
echo "$result"

# optional: send yourself an email
recipient="satstacker@example.org"
echo "Subject: Sats got stacked
From: satstacker@example.org
To: $recipient $result" | /usr/sbin/sendmail $recipient
```

Make it executable with `chmod +x stacksats.sh` and go wild.

Here is an example for a daily cronjob at 6:15am ...

```sh
15 6 * * * /home/stackingsats/stacking-sats-kraken/stacksats.sh > /dev/null 2>&1
```

To run the script manually, use this command:

./stacking-sats-kraken/stacksats.sh

## 🔑 Withdrawal

Holding significant amounts on an exchange is never a good idea.
You should regularly take ownership of your coins by withdrawing to your own wallet.
This can either be done manually or it can be automated.
The script provided here will only withdraw to a previously defined Bitcoin address if the relative fees do not exceed a certain limit.

*It is optional to run the withdrawal script.*

### Example 1

- Max. relative fee: 0.5%
- Fixed Kraken fee: ₿ 0.00050
- Balance: ₿ 0.06000
➡️ No withdrawal since fee actual (0.83%) is too high

### Example 2

- Max. relative fee: 0.5%
- Fixed Kraken fee: ₿ 0.00050
- Balance: ₿ 0.12000
➡️ Withdrawal executed since actual fee (0.42%) is low enough

In case you plan to automatically withdraw from Kraken, a withdrawal method must first be defined.
If you already set up a methode you can reuse it.
Otherwise generate a new one by going to **Funding > Bitcoin (XBT) withdraw > Add address**.
The description field will later be used as an environment variable in the script.

To test the withdrawal of funds to your defined address run:

```sh
npm run test:withdraw
```

You should see something like this:

```text
💡  Relative fee of withdrawal amount: 5.57%
❌  Fee is too high – max rel. fee: 0.50%
```

It is recommended to run the withdrawal script every time you stacked sats:

```sh
npm run withdraw
```

Since it can take a couple seconds or minutes for your order to fill, you should run the following script a couple hours later after the stacking script.
Just set up a second cron job which executes the withdrawal script.

If you are using the aforementioned `stacksats.sh` script you can withdraw via this command:
`stacksats.sh withdraw`
