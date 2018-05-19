# EOS - Step by Step Setup Tutorial and TicTacToe Game

#### Notes
- Based on eos-dawn-v3.0.0 but should also work on the latest release eos-dawn-v4.0.0
- Instructions are compiled on Mac but are generic. If the paths are different, searching them on Google should help you guide

#### Contents
- [Installation](#installation)
- [Setting up the Node](#setting-up-the-node)
- [Basic Commands](#basic-commands)
- [Deploying the Game](#deploying-the-game)
- [Steps to play TicTacToe](#steps-to-play-tictactoe)
- [References](#references)

#### Installation
(As EOSIO are getting ready for V1.0, you might see a loot of changes on the steps with every release. Refer to EOSIO wiki for changes to this flow in case if they don't work)  
  - `mkdir /home/eos-dawn-v3.0.0`
  - `cd /home/eos-dawn-v3.0.0`
  - `git clone https://github.com/eosio/eos --recursive`
  - `cd eos`

  - `git checkout DAWN-2018-04-27-ALPHA`
  - `git submodule update --init --recursive`
  - `./eosio_build.sh`
    *  to install dependencies type 1 and click enter.
  - `cd build`
  - `sudo make install`

#### Setting up the Node
  - It is a good idea to save some of the information alongside the steps to refer later on in the process. It is referred as 'EOSNotes' here.
  - The command to stat the node once the installation is complete is "nodeos"
    -  `nodeos`
  - We will need to make changes to the configuration file (Stored in Config.ini). As soon as  nodoes cmd is run, config.ini and genesis.json are created (for the first time).
    -  `Press "ctrl+c" to stop nodeos now that it has created a config.ini file.`
  - Let's find the genesis file. On Mac the file will be available at `~/Library/Application Support/eosio/nodeos/config`. If not find it using the cmd `sudo find / -name genesis.json`
  - Amend the config file:
    - Towards the bottom of the config file there should be a private key starting with "5", save it in your EOSNotes file for reference later on.
    - Edit the config.ini file, adding/updating the following settings to the defaults already in place:

      ```
      	# Find and change the ip to localhost
      	http-server-address = 127.0.0.1:8888

      	# Verify the Genesis path.
      	genesis-json = "~/Library/Application Support/eosio/nodeos/config/genesis.json"

      	# Find and switch to true
      	enable-stale-production = true

      	# Find, remove "#", and change to "eosio"
      	producer-name = eosio

      	# Copy the following plugins to the bottom of the file
      	plugin = eosio::producer_plugin
      	plugin = eosio::wallet_api_plugin
      	plugin = eosio::chain_api_plugin
      	plugin = eosio::http_plugin
      	plugin = eosio::account_history_api_plugin
      ```
  - Once the changes are saved, Run "nodeos" and let it keep running in it's own window.
  - EOS is now running on the local node, and producing Blocks.

#### Basic Commands
Creating a wallet
  - `cleos wallet create`
    - This will create a wallet and return a password for you. Save it in your 'EOSNotes' file. Save password to use in the future to unlock this wallet.Without password imported keys will not be retrievable.
  - Create two sets of keypairs using `cleos create key`, Save each keypair in your EOSNotes file and label the first "Owner Key" and the second "Active Key"
  - Import the private keys you just created.
    - `cleos wallet import <PRIVATE_KEY>`
  - The private key from the config.ini should already be inside your wallet. To check all the imported keys:
    - `cleos wallet keys`
    - Make sure there are 3 private keys in your wallet, if it has 2 import the key from the config.ini
  - Notes:
    - At any time is `nodeos` is restarted, wallet needs to be unlocked before using the keys.
    - `cleos wallet unlock` to unlock the wallet
    - `cleos wallet lock` to lock the wallet

Loading the BIOS contract
  - BIOS contract is the default System contract that enables you to have direct control over the resource allocation of other accounts and to access other privileged API calls. `eosio.bios` can be used for development.
  - eosio.bios contract can be found in the contracts/eosio.bios folder of your EOSIO build.
  - `cd ~/eos/contracts/eosio.bios/`
  - `eosiocpp -o eosio.bios.wast eosio.bios.cpp`
    - To complie the contract to WebAssembly
  - `cleos set contract eosio ../eosio.bios -p eosio`
    - To publish the contract to Blockchain

Creating Accounts
  - Now that we have setup the basic system contract, we can start to create our own accounts.For the purpose of this guide, we will set-up three accounts.
    - `tictactoe`: Account to hold the smart contract for the Game
    - `host`: The player account. One who hosts the Game
    - `challenger`: Second Player account. One who challenges the host for a game  
  - We will need to associate a Key to each account so that we can control them. For the example, we can use the same key. Remember,
    - to create key : `cleos create key`
    - to import key to wallet: `cleos wallet import privatekey`
  - Create `tictactoe` account:
    - `cleos create account eosio tictactoe publickkey publickkey`
    - The create account subcommand requires two keys, one for the OwnerKey (which in a production environment should be kept highly secure) and one for the ActiveKey. In this tutorial example, the same key is used for both.
  - create `host` account:
    -  `cleos create account eosio host publickkey publickkey`
  - create `challenger` account:
      -  `cleos create account eosio challenger publickkey publickkey`

Other useful commands:
  - `cleos` to get a list of all Commands
  - `cleos wallet keys` to see all the keys
  - `cleos get accounts publickey` to see accounts controlled by a key

#### Deploying the Game
- Clone the tictactoe contract
  - `git clone https://github.com/Lazymindz/eos-sample-contracts`
- change the directory to `eos-sample-contracts/tictactoe`
- `eosiocpp -o tictactoe.wast tictactoe.cpp`
  - to compile the contract. the repository should already have the .wast file in it
-  `cleos set contract tictactoe ../tictactoe`
  - to publish the contract to Blockchain

#### Steps to play TicTacToe
Game Notes:
- we are using a standard 3x3 tic tac toe board.
- Players are divided into two roles: host and challenger.
- Host always makes the first move.
- Each pair of players can ONLY have up to two games at the same time, one where the first player becomes the host and the other one where the second player becomes the host.

Board:
- We use 1 to denote movement by host,
- 2 to denote movement by challenger,
- and 0 to denote empty cell.
- One dimensional array is used to store the board in a Table on the Blockhain and retrieve.

Game Actions:
- `create`: create a new game
- `restart`: restart an existing game, host or challenger is allowed to do this
- `close`: close an existing game, which frees up the storage used to store the game, only host is allowed to do this
- `move`: make a movement

Game Action commands:
- `create`: create a new game
- `restart`: restart an existing game, host or challenger is allowed to do this
- `close`: close an existing game, which frees up the storage used to store the game, only host is allowed to do this
- `move`: make a movement
- To check the board:
  - `cleos get table tictactoe host games`

Examples:
- create:
  - `cleos push action tictactoe create '{"challenger":"challenger", "host":"host"}' -p host@active`
- move by host:
  - `cleos push action tictactoe move '{"challenger":"challenger", "host":"host", "by":"host", "mvt":{"row":0, "column":0} }' -p host@active`
- move by challenger:
  - `cleos push action tictactoe move '{"challenger":"challenger", "host":"host", "by":"challenger", "mvt":{"row":1, "column":0} }' -p challenger@active`
- restarting the game:
  - `cleos push action tictactoe restart '{"challenger":"challenger", "host":"host", "by":"host"}' -p host@active`
- closing the game:
  -  `cleos push action tic.tac.toe close '{"challenger":"challenger", "host":"host"}' -p host@active`

#### References
- [EOS Wiki](https://github.com/EOSIO/eos/wiki)
- [Understanding Smart Contracts](https://www.youtube.com/watch?v=EbWDHrm2ETY)
- [Tic-Tac-Toe Steps breakdown](https://github.com/EOSIO/eos/wiki/Tutorial-Tic-Tac-Toe)
- [TicTacToe contract code](https://github.com/Lazymindz/eos-sample-contracts/tree/master/tictactoe)
