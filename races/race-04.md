All 8 questions in this quiz are based on the InSecureum contract. This is the same contract you will see for all the 8
questions in this quiz. InSecureum is adapted from a widely used ERC20 contract. The question is below the shown
contract.

```solidity
pragma solidity 0.8.10;

import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/IERC20.sol";
import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/extensions/IERC20Metadata.sol";
import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Context.sol";

contract InSecureum is Context, IERC20, IERC20Metadata {
    mapping(address => uint256) private _balances;
    mapping(address => mapping(address => uint256)) private _allowances;
    uint256 private _totalSupply;
    string private _name;
    string private _symbol;

    constructor(string memory name_, string memory symbol_) {
        _name = name_;
        _symbol = symbol_;
    }

    function name() public view virtual override returns (string memory) {
        return _name;
    }

    function symbol() public view virtual override returns (string memory) {
        return _symbol;
    }

    function decimals() public view virtual override returns (uint8) {
        return 8;
    }

    function totalSupply() public view virtual override returns (uint256) {
        return _totalSupply;
    }

    function balanceOf(address account) public view virtual override returns (uint256) {
        return _balances[account];
    }

    function transfer(address recipient, uint256 amount) public virtual override returns (bool) {
        _transfer(_msgSender(), recipient, amount);
        return true;
    }

    function allowance(address owner, address spender) public view virtual override returns (uint256) {
        return _allowances[owner][spender];
    }

    function approve(address spender, uint256 amount) public virtual override returns (bool) {
        _approve(_msgSender(), spender, amount);
        return true;
    }

    function transferFrom(
        address sender,
        address recipient,
        uint256 amount
    ) public virtual override returns (bool) {
        uint256 currentAllowance = _allowances[_msgSender()][sender];
        if (currentAllowance != type(uint256).max) {
            unchecked {
                _approve(sender, _msgSender(), currentAllowance - amount);
            }
        }
        _transfer(sender, recipient, amount);
        return true;
    }

    function increaseAllowance(address spender, uint256 addedValue) public virtual returns (bool) {
        _approve(_msgSender(), spender, _allowances[_msgSender()][spender] + addedValue);
        return true;
    }

    function decreaseAllowance(address spender, uint256 subtractedValue) public virtual returns (bool) {
        uint256 currentAllowance = _allowances[_msgSender()][spender];
        require(currentAllowance > subtractedValue, "ERC20: decreased allowance below zero");
        _approve(_msgSender(), spender, currentAllowance - subtractedValue);
        return true;
    }

    function _transfer(
        address sender,
        address recipient,
        uint256 amount
    ) internal virtual {
        require(sender != address(0), "ERC20: transfer from the zero address");
        require(recipient != address(0), "ERC20: transfer to the zero address");
        uint256 senderBalance = _balances[sender];
        require(senderBalance >= amount, "ERC20: transfer amount exceeds balance");
        unchecked {
            _balances[sender] = senderBalance - amount;
        }
        _balances[recipient] += amount;
        emit Transfer(sender, recipient, amount);
    }

    function _mint(address account, uint256 amount) external virtual {
        _totalSupply += amount;
        _balances[account] = amount;
        emit Transfer(address(0), account, amount);
    }

    function _burn(address account, uint256 amount) internal virtual {
        require(account != address(0), "ERC20: burn from the zero address");
        require(_balances[account] >= amount, "ERC20: burn amount exceeds balance");
        unchecked {
            _balances[account] = _balances[account] - amount;
        }
        _totalSupply -= amount;
        emit Transfer(address(0), account, amount);
    }

    function _approve(
        address owner,
        address spender,
        uint256 amount
    ) internal virtual {
        require(spender != address(0), "ERC20: approve from the zero address");
        require(owner != address(0), "ERC20: approve to the zero address");
        _allowances[owner][spender] += amount;
        emit Approval(owner, spender, amount);
    }
}
```

#### Q1. InSecureum implements

- [ ] (A): Atypical decimals value
- [ ] (B): Non-standard decreaseAllowance and increaseAllowance
- [ ] (C): Non-standard transfer
- [ ] (D): None of the above

<details>
  <summary>Answers</summary>

#### Correct answers: A, B

[Here is EIP ERC-20](https://eips.ethereum.org/EIPS/eip-20) that every auditor should be familiar with

##### A: [standard decimals](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol#L92) value is 18, current implementation returns 8

##### B: decreaseAllowance and increaseAllowance functions are not in EIP, they were introduced due An Attack Vector on the [Approve/TransferFrom Methods](https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/edit#heading=h.m9fhqynw2xvt) in  [openzeppelin ERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol)

##### C: It's incorrect, transfer specified in EIP

##### D: It's incorrect

</details>

#### Q2. InSecureum

- [ ] (A): decimals() can have pure state mutability instead of view
- [ ] (B): _burn() can have external visibility instead of internal
- [ ] (C): _mint() should have internal visibility instead of external
- [ ] (D): None of the above

<details>
  <summary>Answers</summary>

[Here is EIP ERC-20](https://eips.ethereum.org/EIPS/eip-20) that every auditor should be familiar with

#### Correct answers: A, C

##### A:Considering that the decimals() function exclusively provides a constant hardcoded value and does not access any storage or non-calldata information, it is deemed appropriate to declare it as a pure function.

##### B: Making burn function public will allow anyone to burn(decrease) balance of anyone.

##### C: Mint function should be internal to not allow anyone to increase their tokens

##### D: It's incorrect

</details>

#### Q3. InSecureum transferFrom()

- [ ] (A): Is susceptible to an integer underflow
- [ ] (B): Has an incorrect allowance check
- [ ] (C): Has an optimisation indicative of unlimited approvals
- [ ] (D): None of the above

<details>
  <summary>Answers</summary>

#### Correct answers: A, B, C

##### A: Substraction inside unchecked block will not revert like it will without it. E.x. 0 - 1 = type(uint256).max. Check it in remix

##### B: Due to unchecked block, allowance check doesn't make any sense.

##### C: Whenever there is an unlimited approval - function will skip calling `_approve` thus not executing variable updates in storage

##### D: It's incorrect

</details>

#### Q4. InSecureum transferFrom()

- [ ] (A): increaseAllowance is susceptible to an integer overflow
- [ ] (B): decreaseAllowance is susceptible to an integer overflow
- [ ] (C): decreaseAllowance does not allow reducing allowance to zero
- [ ] (D): decreaseAllowance can be optimised with unchecked{}

<details>
  <summary>Answers</summary>

#### Correct answers: C, D

##### A: Substraction inside unchecked block will not revert like it will without it. E.x. 0 - 1 = type(uint256).max. Check it in remix

##### B: Due to unchecked block, allowance check doesn't make any sense.

##### C: Whenever there is an unlimited approval - function will skip calling `_approve` thus not executing variable updates in storage

##### D: It's incorrect

</details>




[Q3] InSecureum transferFrom()

[Answers]: A, B, C
Rajeev | Secureum — 02/10/2022 4:56 PM
[Q4] In InSecureum
(A): increaseAllowance is susceptible to an integer overflow
(B): decreaseAllowance is susceptible to an integer overflow
(C): decreaseAllowance does not allow reducing allowance to zero
(D): decreaseAllowance can be optimised with unchecked{}
[Answers]: C, D

[Q5] InSecureum _transfer()
(A): Is missing a zero-address validation
(B): Is susceptible to an integer overflow
(C): Is susceptible to an integer underflow
(D): None of the above
[Answers]: D
[Q6] InSecureum _mint()
(A): Is missing a zero-address validation
(B): Has an incorrect event emission
(C): Has an incorrect update of account balance
(D): None of the above
[Answers]: A, C
[Q7] InSecureum _burn()
(A): Is missing a zero-address validation
(B): Has an incorrect event emission
(C): Has an incorrect update of account balance
(D): None of the above
[Answers]: B
[Q8] InSecureum _approve()
(A): Is missing a zero-address validation
(B): Has incorrect error messages
(C): Has an incorrect update of allowance
(D): None of the above
[Answers]: B, C
