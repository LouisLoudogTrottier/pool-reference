# Chia Pool Installation

## Ubuntu 20.04 server

### References
https://www.chia.net/

#### chia-blockchain
https://github.com/Chia-Network/chia-blockchain
https://github.com/Chia-Network/chia-blockchain/wiki/INSTALL#ubuntudebian
https://github.com/Chia-Network/pool-reference#install-and-run-testnet

## Notes:
- Some files created by pool-server.py ran as root have root ownership.
- Use `chia stop -d all` to disconnect when stoping to prevent 'port already in use' at next start.


## POOL INSTALLATION

### 0. Do system update and install dependencies
```
sudo apt update
sudo apt -y upgrade
sudo apt -y install git 

## For Ubuntu 18.04
#sudo apt -y install python3.7-venv python3.7-distutils python3.7-dev git lsb-release -y
```

### 1. Clone the pool-reference git and Install the chia-blockchain and checkout the pools.2021-june-4 branch
```
git clone https://github.com/Chia-Network/chia-blockchain.git -b pools.2021-june-4 --recurse-submodules
cd chia-blockchain
git status # verify you are unsing the right branch
```

### 2. Proceed with the installation and activate
```
sh install.sh
. ./activate
```
### 3. point the CHIA_ROOT to your hidden chia folder (usualy in your home)
```
export CHIA_ROOT="/home/user/.chia/testnet7"
```
### 4. run `chia init` to get the subtitute wallet and Modify the config.yaml file for testnet7 (skip of update)
```
chia init 
## Consider overriding xch[...] with txch[...]

nano /home/user/.chia/testnet7/config/config.yaml
## CTRL-W to find the xch[...] address and replace it with the txch[...]

```
### 5. Initialize and configure chia, also add the testnet-node connection.
```
chia init
chia configure --testnet true

```
### 6. Start sync to the the testnet.
```
chia start farmer 		## THIS CAN TAKE A MINUTE
chia show -a testnet-node.chia.net:58444
chia show -s -c #show --status --connection
```
### 7. Generate 2 (or 3 with farmer) set of keys (skip for update)
```
chia keys generate
chia keys generate
## NOTE: HERE YOU COULD GENERATE THE FARMER'S (3RD) KEYS AS WELL SO IT HAVE TIME TO START SYNC (step 13)
#chia keys generate  ## For STEP 13.
```
### 8. Use `chia wallet show` and note the wallet_fingerprint and wallet_id of the 1st key your just created.
```
chia wallet show
```
```
Choose wallet key:
1) 400991747	
2) 214632651
Enter a number to pick or q to quit: 1 		## Select the 1st key you created
No online backup file found, 
 Press S to skip restore from backup 
 Press F to use your own backup file: s 	
Wallet height: 0
Sync status: Not synced
Balances, fingerprint: 400991747		      ## Note the fingerprint number (400991747)
Wallet ID 1 type STANDARD_WALLET		      ## Note the Wallet ID (1)
   -Total Balance: 0.0 txch (0 mojo)
   -Pending Total Balance: 0.0 txch (0 mojo)
   -Spendable: 0.0 txch (0 mojo)
```

### 9. Use `chia keys show` and note the `First Wallet address` for the 2 keys you created.
```
chia keys show
```
>>>
    Showing all public keys derived from your private keys:

    Fingerprint: 400991747
    Master public key (m): b9356q7ue6[...]
    Farmer public key (m/12381/8444/0/0): 90bdq5vtqvq[...]
    Pool public key (m/12381/8444/1/0): a452b6981[...]
    First wallet address: txch1uk5vnyg2rkr8sltruaumftp4ttjal2p38nwr5fhu2t57gusr3syqhtqqeg  	## NOTE THIS 1ST ADDRESS 

    Fingerprint: 214632651
    Master public key (m): 8b29404a46[...]
    Farmer public key (m/12381/8444/0/0): ae1682[...]
    Pool public key (m/12381/8444/1/0): 96c8901[...]
    First wallet address: txch1eklggc5wzrsethmghurkyvkl6xmye03gk4r5ngvr028hyj02fcfqmehe4l 	## NOTE THIS 2ND ADDRESS
>>>

## pool-references

### 10. Install the pool-reference repo (while the node and wallet syncs)
```
git clone https://github.com/Chia-Network/pool-reference.git
cd ~/pool-reference
sudo apt install -y python3-pip

## NOT SURE ABOUT THE ORDER AND OR PIP VS PIP3
pip3 install aiohttp
pip3 install blspy
pip3 install --upgrade pip
```

### 11. In the pool-reference repo edit `pool/pool.py`
```
nano pool/pool.py
```
CTRL-W to find `wallet_fingerprint` and set the value to your 1st key `wallet_fingerprint`  
just above, set the `default_target_puzzle_hash` to the 1ST address you noted  
also set the `pool_fee_puzzle_hash` to the 2ND address you noted  
>>>
    self.default_target_puzzle_hash: bytes32 = bytes32(
        decode_puzzle_hash("txch1uk5vnyg2rkr8sltruaumftp4ttjal2p38nwr5fhu2t57gusr3syqhtqqeg") 	## SET THIS TO THE 1ST ADDRESS
    )

    # The pool fees will be sent to this address. This MUST be on a different key than the target_pu>
    # otherwise, the fees will be sent to the users. Using 690783650
    self.pool_fee_puzzle_hash: bytes32 = bytes32(
        decode_puzzle_hash("txch1eklggc5wzrsethmghurkyvkl6xmye03gk4r5ngvr028hyj02fcfqmehe4l") 	## SET THIS TO THE 2ND ADDRESS
    )

    # This is the wallet fingerprint and ID for the wallet spending the funds from `self.default_tar>
    self.wallet_fingerprint = 400991747		## SET THIS TO YOUR 1ST KEY FINGERPRINT (400991747)
    self.wallet_id = "1"			## SET THIS TO YOUR 1ST KEY WALLET ID (1)
>>>
CTRL-W to find `pool_url` and set the value corresonping to your domain (or ip)
>>>
        self.pool_url = "http://10.0.0.123"	## SET THIS VALUE TO YOUR POOL URL (or ip)
>>>


### 12. Use the chia-blockchain CHIA_ROOT and run the server.
```
python3 -m venv ./venv
source ./venv/bin/activate
pip install ../chia-blockchain/ 
sudo CHIA_ROOT="/home/user/.chia/testnet7" ./venv/bin/python pool/pool_server.py
```

### 13. Start a farmer on the chia-blockchain and create a key for your farmer (skip at update)
```
chia keys generate
chia keys show
```
>>>
    ## USE THE 3RD KEY AS THE FARMER KEY
    Fingerprint: 1357477852
    Master public key (m): b35976dc9cc[...]
    Farmer public key (m/12381/8444/0/0):    adcc9871d21f[...]
    Pool public key (m/12381/8444/1/0): a3817aa42b811da67[...]
    First wallet address: txch19k7z58kjc360rm93vyru992hh7wgxuf2nl9mh3qeyz8yjfpwxvksae0vqy	## NOTE THIS 3RD ADDRESS
>>>
Visit http://chia-faucet.com/ and enter the farmer's (3rd) wallet address to get some TXCH needed for the singleton
- Make sure you sent 1 full coin because you can only use it once.

##### HINT
- Wait your wallet to Sync, use `watch "chia wallet show -f 1357477852"` to watch the process (change the fingerprint number by yours)
>>>
Choose wallet key:
1) 400991747
2) 214632651
3) 1357477852
Enter a number to pick or q to quit: 3
Wallet height: 85				## WAIT FOR MAX HEIGHT
Sync status: Not synced				## WAIT FOR SYNC
Balances, fingerprint: 1357477852
Wallet ID 1 type STANDARD_WALLET
   -Total Balance: 0.0 txch (0 mojo)
   -Pending Total Balance: 0.0 txch (0 mojo)
   -Spendable: 0.0 txch (0 mojo)
>>>


### 14. Create a singleton and verify it's created, don't forget to associate the farmer's wallet with -f
```
chia plotnft create -u 192.168.0.250:80 -f 1357477852
chia plotnft show
```
>>>
Choose wallet key:
1) 400991747
2) 214632651
3) 1357477852
Enter a number to pick or q to quit: 3
Will create a plot NFT and join pool: 10.0.0.123:80.
{'description': '(example) The Reference Pool allows you to pool with low '
                'fees, paying out daily using Chia.',
 'fee': '0.01',
 'logo_url': 'https://www.chia.net/img/chia_logo.svg',
 'minimum_difficulty': 10,
 'name': 'The Reference Pool',
 'protocol_version': '1.0.0',
 'relative_lock_height': 100,
 'target_puzzle_hash': '0xe5a8c9910a1d86787d63e779b4ac355ae5dfa8313cdc3a26fc52e9e472038c08'}
Confirm [n]/y: y
Transaction submitted to nodes: [('33834fbb01f9f84f1070e8630c0e3469369d6ae67c7cadffc473eb23b7de549d', 1, None)]
Do chia wallet get_transaction -f 1357477852 -tx 0xd6f0e5b30a4995bf8291ea81898f237b19e0005a7acb0c11a285b56994ce72c8 to get status
>>>
```
chia wallet get_transaction -f 1357477852 -tx 0xd6f0e5b30a4995bf8291ea81898f237b19e0005a7acb0c11a285b56994ce72c8  ## FROM ABOVE
```
>>> 
Transaction d6f0e5b30a4995bf8291ea81898f237b19e0005a7acb0c11a285b56994ce72c8
Status: Confirmed
Amount: 1E-12 txch
To address: txch1c809tu2j7cl6n4g97hxh55cj0fvc2vp9q9gs2rczvxtsrm04kg2q0hqzsd    ## NOTE THIS ADDR
Created at: 2021-06-05 15:52:04

>>>

### 15. Start creating Plots using the singleton wallet address
```
chia plots create -c txch1c809tu2j7cl6n4g97hxh55cj0fvc2vp9q9gs2rczvxtsrm04kg2q0hqzsd -k25 --override-k
```
