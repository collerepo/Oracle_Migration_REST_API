# Oracle_Migration_REST_API
A python REST API that can interpret a migration script that deploys a solidity oracle contract onto the ethereum blockchain using the Flask framework.

To create a Python REST API that can interpret a migration script that deploys an oracle contract onto the Ethereum blockchain, you'll need to follow these steps:

1) Install the required libraries. You'll need the 'web3' and 'Flask' libraries to interact with the Ethereum blockchain and create the REST API, respectively. You can install them using 'pip':
------------------------------------
pip install web3
pip install Flask
------------------------------------
2) Next, you'll need to import the required libraries and your oracle contract into your Python script. You can do this using the following code:
------------------------------------
from web3 import Web3
from flask import Flask, request
from solc import compile_source

# Read the Solidity source code
with open('contracts/Oracle.sol', 'r') as f:
    contract_source_code = f.read()

# Compile the Solidity code
compiled_sol = compile_source(contract_source_code)

# Extract the ABI and bytecode for the Oracle contract
oracle_abi = compiled_sol['<stdin>:Oracle']['abi']
oracle_bytecode = compiled_sol['<stdin>:Oracle']['bin']
------------------------------------
3) Next, you'll need to create a function to deploy the oracle contract to the Ethereum blockchain. This function should take two arguments: the 'web3' object and the 'price_feed_address', and return the address of the deployed contract. Here's an example of how you might do this:
------------------------------------
def deploy_oracle(web3, price_feed_address):
    # Create a contract object for the Oracle contract
    Oracle = web3.eth.contract(abi=oracle_abi, bytecode=oracle_bytecode)

    # Get the account to use for the deployment
    account = web3.eth.accounts[0]

    # Set the update interval (in seconds) for the price data
    update_interval = 3600

    # Deploy the contract
    tx_hash = Oracle.constructor(price_feed_address, update_interval).transact({'from': account})

    # Wait for the transaction to be mined
    web3.eth.waitForTransactionReceipt(tx_hash)

    # Get the contract address
    contract_address = web3.eth.getTransactionReceipt(tx_hash).contractAddress

    return contract_address
------------------------------------
4) Next, you'll need to create a Flask app to expose the deployment function as a REST API. Here's an example of how you might do this:
------------------------------------
# Create the Flask app
app = Flask(__name__)

# Set the Ethereum RPC URL
rpc_url = "http://localhost:8545"

# Connect to the Ethereum node
web3 = Web3(Web3.HTTPProvider(rpc_url))

# Create a route for the deployment API
@app.route('/deploy', methods=['POST'])
def deploy():
    # Get the price feed address from the request body
    price_feed_address = request.json['price_feed_address']

    # Deploy the oracle contract
    contract_address = deploy_oracle(web3, price_feed_address)

    # Return the contract address
    return {'contract_address': contract_address}

# Start
------------------------------------
