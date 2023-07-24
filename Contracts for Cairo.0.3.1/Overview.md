Contracts for Cairo
A library for secure smart contract development written in Cairo for StarkNet, a decentralized ZK Rollup.

Usage
This repo contains highly experimental code. Expect rapid iteration. Use at your own risk.
First time?
Before installing Cairo on your machine, you need to install gmp:

sudo apt install -y libgmp3-dev # linux
brew install gmp # mac
If you have any trouble installing gmp on your Apple M1 computer, here’s a list of potential solutions.
Set up your project
Create a directory for your project, then cd into it and create a Python virtual environment.

mkdir my-project
cd my-project
python3 -m venv env
source env/bin/activate
Install the Nile development environment and then run init to kickstart a new project. Nile will create the project directory structure and install the Cairo language, a local network, and a testing framework.

pip install cairo-nile
nile init
Install the library
pip install openzeppelin-cairo-contracts
Use a basic preset
Presets are ready-to-use contracts that you can deploy right away. They also serve as examples of how to use library modules. Read more about presets.

# contracts/MyToken.cairo

%lang starknet

from openzeppelin.token.erc20.presets.ERC20 import constructor
Compile and deploy it right away:

nile compile

nile deploy MyToken <name> <symbol> <decimals> <initial_supply> <recipient> --alias my_token
<initial_supply> is expected to be two integers i.e. 1 0. See uint256 for more information.
Write a custom contract using library modules
Read more about libraries.

%lang starknet

from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.cairo.common.uint256 import Uint256
from openzeppelin.security.pausable.library import Pausable
from openzeppelin.token.erc20.library import ERC20

(...)

@external
func transfer{
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }(recipient: felt, amount: Uint256) -> (success: felt):
    Pausable.assert_not_paused()
    ERC20.transfer(recipient, amount)
    return (TRUE)
end