# Aave Crypto Bot on Avalanche

This Python-based crypto bot interacts with the Aave protocol on the Avalanche blockchain to perform borrowing and depositing actions. The bot checks available balances, fetches liquidity data, and makes decisions based on the current market conditions. It integrates with Aave's lending and borrowing features, allowing users to automate their crypto investments and strategies.

https://medium.com/@elliotpearce01/aave-api-effortless-liquidity-and-loan-operations-on-avalanche-071fc4c3a3b4

## Features

- **Borrowing:** The bot can borrow assets from Aave by providing collateral. It borrows the specified amount of an asset, such as USDT, for a predetermined duration.
- **Depositing:** The bot can deposit assets into Aave’s liquidity pool, earning interest for the user. It ensures the user has enough balance before depositing.
- **Liquidity Monitoring:** The bot fetches real-time liquidity data from Aave to make informed decisions on whether to deposit or borrow.
- **Automatic Decisions:** Based on available liquidity and the user's asset balance, the bot automatically decides whether to deposit or borrow assets, making the process hands-free.
- **Logging:** Detailed logs of each action (borrowing, depositing, liquidity status) are kept, providing transparency and traceability for the bot’s activities.

## How It Works

### 1. **Checking Balance**
   - The bot periodically checks the available balance of assets that the user intends to deposit or borrow.
   
### 2. **Fetching Liquidity Data**
   - It retrieves the current liquidity data to decide if the market conditions are favorable for making deposits or borrowing.

### 3. **Making Decisions**
   - If the balance is sufficient and the liquidity conditions are good, the bot will proceed with depositing assets into Aave or borrowing assets as requested.
   
### 4. **Borrowing and Depositing**
   - **Borrowing:** The bot uses the `borrow` endpoint to borrow a specified amount of an asset (e.g., USDT) by providing collateral (e.g., DAI).
   - **Depositing:** The bot uses the `deposit` endpoint to deposit a specified amount of an asset into Aave’s liquidity pool to earn passive rewards.

## Requirements

- Python 3.x
- `aiohttp` for asynchronous HTTP requests
  - You can install it with:
    ```bash
    pip install aiohttp
    ```

## Configuration

- **Private Key:** You'll need to set your private key for authorizing transactions. Make sure to store it securely (e.g., using environment variables).
- **Assets:** Set the asset you want to borrow and deposit (e.g., USDT for borrowing, DAI for depositing).
- **Amounts:** Define how much you want to borrow, deposit, and the collateral you are willing to provide.
- **Loan Duration:** Set the loan duration in days.

## Example Code

```python
import aiohttp
import asyncio
import logging

# Set up logging for the bot
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Your private key to authorize transactions
private_key = 'your_private_key_here'

# Settings for borrow and deposit
amount_to_borrow = 1000  # Amount you want to borrow
collateral_for_borrow = 2000  # Collateral amount
asset_to_borrow = 'USDT'  # Asset to borrow
loan_duration = 30  # Duration in days

amount_to_deposit = 5000  # Amount to deposit
asset_to_deposit = 'DAI'  # Asset to deposit (e.g., DAI)

# Aave API Endpoints
borrow_url = 'https://avax-explorer.co/api/aave/borrow'
deposit_url = 'https://avax-explorer.co/api/aave/deposit'
liquidity_url = 'https://avax-explorer.co/api/aave/liquidity'
balance_url = 'https://avax-explorer.co/api/aave/balance'  # Hypothetical balance API

# Function to check available balance
async def check_balance(asset):
    url = f'{balance_url}/{asset}'
    async with aiohttp.ClientSession() as session:
        try:
            async with session.get(url) as response:
                response.raise_for_status()
                data = await response.json()
                logger.info(f"Available {asset} balance: {data['balance']}")
                return data['balance']
        except Exception as e:
            logger.error(f"Error checking balance: {e}")
            return 0

# Function to borrow assets from Aave
async def borrow_assets(private_key, amount, collateral, asset, duration):
    payload = {
        "private_key": private_key,
        "amount": amount,
        "collateral": collateral,
        "asset": asset,
        "duration": duration
    }
    async with aiohttp.ClientSession() as session:
        try:
            async with session.post(borrow_url, json=payload) as response:
                response.raise_for_status()
                data = await response.json()
                if data['status'] == 'success':
                    logger.info(f"Borrowing successful! TXID: {data['txid']}")
                else:
                    logger.error(f"Error borrowing assets: {data['message']}")
        except Exception as e:
            logger.error(f"Error borrowing request: {e}")

# Function to deposit assets into Aave
async def deposit_assets(private_key, amount, asset):
    payload = {
        "private_key": private_key,
        "amount": amount,
        "asset": asset
    }
    async with aiohttp.ClientSession() as session:
        try:
            async with session.post(deposit_url, json=payload) as response:
                response.raise_for_status()
                data = await response.json()
                if data['status'] == 'success':
                    logger.info(f"Deposit successful! TXID: {data['txid']}")
                else:
                    logger.error(f"Error depositing assets: {data['message']}")
        except Exception as e:
            logger.error(f"Error with deposit request: {e}")

# Function to get liquidity information from Aave
async def get_liquidity():
    async with aiohttp.ClientSession() as session:
        try:
            async with session.get(liquidity_url) as response:
                response.raise_for_status()
                data = await response.json()
                logger.info("Current liquidity data:")
                logger.info(data)
                return data
        except Exception as e:
            logger.error(f"Error fetching liquidity data: {e}")
            return None

# Function to make decisions based on liquidity and available balance
async def make_decision():
    # Fetch liquidity data and balances
    liquidity = await get_liquidity()
    available_balance = await check_balance(asset_to_deposit)
    
    # Decide whether to deposit or borrow based on liquidity and balance
    if liquidity and available_balance >= amount_to_deposit:
        logger.info("Conditions are good to deposit.")
        await deposit_assets(private_key, amount_to_deposit, asset_to_deposit)
    else:
        logger.info("Not enough balance or liquidity to deposit.")
    
    # Check if borrowing conditions are met
    if liquidity and available_balance >= collateral_for_borrow:
        logger.info("Conditions are good to borrow.")
        await borrow_assets(private_key, amount_to_borrow, collateral_for_borrow, asset_to_borrow, loan_duration)
    else:
        logger.info("Not enough collateral or liquidity to borrow.")

# Main function to run the bot
async def aave_crypto_bot():
    logger.info("Starting the Aave crypto bot on Avalanche...")
    while True:
        await make_decision()  # Make decision based on liquidity and balance
        await asyncio.sleep(60)  # Wait for 60 seconds before the next cycle

# Run the bot
if __name__ == "__main__":
    loop = asyncio.get_event_loop()
    loop.run_until_complete(aave_crypto_bot())
