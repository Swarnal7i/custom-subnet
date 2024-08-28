# Creating EVM subnet with Ava Labs

- Created my own (custom) Subnet using the Avalanche-Cli.

## Steps to Follow

- avalanche subnet create swarnali (chain ID : 10338) (Token Symbol : SWR)
- avalanche subnet deploy ayush
- Importing the account & Copying the private key in the metamask.
- Adding the network through RPC URL and chain ID
- Interacting with the game using the REMIX IDE

![App Screenshot](https://res.cloudinary.com/dsprifizw/image/upload/v1724856568/Screenshot_2024-08-28_151232_enmdpf.png)

![App Screenshot](https://res.cloudinary.com/dsprifizw/image/upload/v1724856568/Screenshot_2024-08-28_151821_afpcoa.png)

## Objective

The DiceDuel contract is an ERC20-based game where players can bet tokens on a virtual dice roll. Players roll a dice against the contract, and depending on the outcome, they either win double their bet or lose their tokens. The contract also includes functionality for the owner to mint and burn tokens, ensuring the game's integrity and managing the in-game economy.

## Code by Code Explanation.

```Solidity

uint randomNumber = uint(keccak256(abi.encodePacked(block.timestamp, block.difficulty))) % 2;

```

This code is implemented in my game a random number range from 0-1 for deciding the winner.

## Complete Code

### ERC20 contact

```Solidity
// SPDX-License-Identifier: MIT
// OpenZeppelin Contracts (last updated v5.0.0) (token/ERC20/ERC20.sol)

pragma solidity ^0.8.20;

import {IERC20} from "./IERC20.sol";

error ERC20InvalidSender(address _nullAddress);
error ERC20InvalidReceiver(address _nullAddress);
error ERC20InsufficientBalance(address,address,uint);
error ERC20InsufficientBalances(address,uint,uint);
error ERC20InsufficientAllowance(address,uint,uint);


contract ERC20 is  IERC20 {
    mapping(address account => uint256) private _balances;

    mapping(address account => mapping(address spender => uint256)) private _allowances;

    uint256 private _totalSupply;

    string private _name;
    string private _symbol;


    constructor(string memory name_, string memory symbol_) {
        _name = name_;
        _symbol = symbol_;
    }

    function name() public view virtual returns (string memory) {
        return _name;
    }

    function symbol() public view virtual returns (string memory) {
        return _symbol;
    }

    function decimals() public view virtual returns (uint8) {
        return 18;
    }

    function mintTokens(address owner,uint _amount) external{
        _mint(owner, _amount);
    }

    function burnTokens(address account, uint256 value) external {
        _burn(account , value);
    }

    function totalSupply() public view virtual returns (uint256) {
        return _totalSupply;
    }

    function balanceOf(address account) public view virtual returns (uint256) {
        return _balances[account];
    }


    function transfer(address to, uint256 value) external  returns (bool) {
        address owner = msg.sender;
        _transfer(owner, to, value);
        return true;
    }

    function allowance(address owner, address spender) public view virtual returns (uint256) {
        return _allowances[owner][spender];
    }

    function approve(address spender, uint256 value) public virtual returns (bool) {
        address owner = msg.sender;
        _approve(owner, spender, value);
        return true;
    }

    function transferFrom(address from, address to, uint256 value) public virtual returns (bool) {
        address spender = msg.sender;
        _spendAllowance(from, spender, value);
        _transfer(from, to, value);
        return true;
    }

    function _transfer(address from, address to, uint256 value) internal {
        if (from == address(0)) {
            revert ERC20InvalidSender(address(0));
        }
        if (to == address(0)) {
            revert ERC20InvalidReceiver(address(0));
        }
        _update(from, to, value);
    }
    function transferFunc(address sender, address recepient, uint _amount) external{
        _transfer(sender, recepient, _amount);
    }

    function _update(address from, address to, uint256 value) internal virtual {
        if (from == address(0)) {
            // Overflow check required: The rest of the code assumes that totalSupply never overflows
            _totalSupply += value;
        } else {
            uint256 fromBalance = _balances[from];
            if (fromBalance < value) {
                revert ERC20InsufficientBalances(from, fromBalance, value);
            }
            unchecked {
                // Overflow not possible: value <= fromBalance <= totalSupply.
                _balances[from] = fromBalance - value;
            }
        }

        if (to == address(0)) {
            unchecked {
                // Overflow not possible: value <= totalSupply or value <= fromBalance <= totalSupply.
                _totalSupply -= value;
            }
        } else {
            unchecked {
                // Overflow not possible: balance + value is at most totalSupply, which we know fits into a uint256.
                _balances[to] += value;
            }
        }

        emit Transfer(from, to, value);
    }

    function _mint(address account, uint256 value) internal {
        if (account == address(0)) {
            revert ERC20InvalidReceiver(address(0));
        }
        _update(address(0), account, value);
    }

    function _burn(address account, uint256 value) internal   {
        if (account == address(0)) {
            revert ERC20InvalidSender(address(0));
        }
        _update(account, address(0), value);
    }

    function _approve(address owner, address spender, uint256 value) internal {
        _approve(owner, spender, value, true);
    }

    function _approve(address owner, address spender, uint256 value, bool emitEvent) internal virtual {
        if (owner == address(0)) {
            revert ERC20InvalidSender(address(0));
        }
        if (spender == address(0)) {
            revert ERC20InvalidSender(address(0));
        }
        _allowances[owner][spender] = value;
        if (emitEvent) {
            emit Approval(owner, spender, value);
        }
    }

    function _spendAllowance(address owner, address spender, uint256 value) internal virtual {
        uint256 currentAllowance = allowance(owner, spender);
        if (currentAllowance != type(uint256).max) {
            if (currentAllowance < value) {
                revert ERC20InsufficientAllowance(spender, currentAllowance, value);
            }
            unchecked {
                _approve(owner, spender, currentAllowance - value, false);
            }
        }
    }
}
```

### Vault contact

```Solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.17;
import "./ERC20.sol";

contract Vault {
    ERC20 token;
    uint public shares;
    address public _owner;

    uint public totalSupply;
    uint public totalShares;
    mapping(address => uint) public balanceOf;

    constructor() {
        token = new ERC20("swarnali","SWR");
        // _owner = msg.sender;
        // _tokenaddress = _token;
        totalSupply = token.totalSupply();
    }

   function transferFunc(address owner,address _recepient,uint _amount) internal{
    token.transferFunc(owner,_recepient,_amount);
   }

    function _mint(address _to, uint _shares) internal {
        totalShares += _shares;
        balanceOf[_to] += _shares;
    }

    function _burn(address _from, uint _shares) internal {
        totalShares -= _shares;
        balanceOf[_from] -= _shares;
    }

    // function deposit(uint _amount) external {
    //     /*
    //     a = amount
    //     B = balance of token before deposit
    //     T = total supply
    //     s = shares to mint

    //     (T + s) / T = (a + B) / B 

    //     s = aT / B
    //     */
    //     uint shares;
    //     if (totalSupply == 0) {
    //         shares = _amount;
    //     } else {
    //         shares = (_amount  * totalSupply) / token.balanceOf(msg.sender);
    //     }

    //     require(shares >0,"Shares is 0");

    //     _mint(msg.sender, shares);
    //    bool res =  token.transferFrom(msg.sender, address(this), _amount);
    //    require(res,"faild");
    // }



  
   function deposit(uint _amount) external {
        require(_amount > 0, "Amount must be greater than 0");

        uint256 tokenBalance = token.balanceOf(msg.sender);

        if (totalSupply == 0) {
            shares = _amount;
        } else {
            require(tokenBalance > 0, "Token balance must be greater than 0");
            shares = (_amount* totalSupply)/ tokenBalance;
        }

       _mint(msg.sender,shares);
       transferFunc(msg.sender,address(this),_amount);
    }

    function balance() external view returns(uint){
        return token.balanceOf(msg.sender);
    }

    function withdraw(uint _shares) external {
        /*
        a = amount
        B = balance of token before withdraw
        T = total supply
        s = shares to burn

        (T - s) / T = (B - a) / B 

        a = sB / T
        */
        // uint amount = (_shares * token.balanceOf(msg.sender)) / totalSupply;
        _burn(msg.sender, _shares);
        transferFunc(address(this), msg.sender, shares);
    }

    function getTotalSupply() external{
          totalSupply = token.totalSupply();
    }

  
}
```

### Battle contact

```Solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.22;
import "./ERC20.sol";

contract Battle  {
    uint public battleCount;
    event BattleStarted(uint indexed battleId, address indexed player1, address indexed player2, uint stake);
    event BattleWon(uint indexed battleId, address indexed winner, uint reward);

    ERC20 erc20;


    struct BattleInfo {
        address player1;
        address player2;
        uint stake;
        bool battleConcluded;
        address winner;
    }

    mapping(uint => BattleInfo) public battles;

    constructor() {
        erc20 = new ERC20("swarnali","SWR"); //
    }

    function mintToken(uint _amount) external{
        erc20.mintTokens(msg.sender,_amount);
    }

    function balance() external view returns(uint) {
       return erc20.balanceOf(msg.sender);
    }



    // Start a new battle between two players
    function startBattle(address _opponent, uint _stake) external {
        require(erc20.balanceOf(msg.sender) >= _stake, "Insufficient balance to stake");
        require(erc20.balanceOf(_opponent) >= _stake, "Opponent has insufficient balance");

        battleCount++;
        BattleInfo storage newBattle = battles[battleCount];

        newBattle.player1 = msg.sender;
        newBattle.player2 = _opponent;
        newBattle.stake = _stake;

        erc20.burnTokens(msg.sender, _stake);
        erc20.burnTokens(_opponent, _stake);

        emit BattleStarted(battleCount, msg.sender, _opponent, _stake);


    }

    // Conclude the battle (for example, using random dice rolls)
    function concludeBattle(uint _battleId) external {
        BattleInfo storage battle = battles[_battleId];
        require(!battle.battleConcluded, "Battle already concluded");

        // A simple random determination of the winner (this can be replaced with more sophisticated logic)
        uint randomNumber = uint(keccak256(abi.encodePacked(block.timestamp, block.difficulty))) % 2;

        if (randomNumber == 0) {
            battle.winner = battle.player1;
        } else {
            battle.winner = battle.player2;
        }

        uint reward = battle.stake * 2; // Winner takes all
        erc20.mintTokens(battle.winner, reward);
        battle.battleConcluded = true;

        emit BattleWon(_battleId, battle.winner, reward);
    }
}


```
## Author 

- swarnali das


## License

This project is licensed under the MIT License - see the LICENSE.md file for details
