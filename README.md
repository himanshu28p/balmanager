# BalanceManager Smart Contract

This is a simple Solidity smart contract for managing user balances with functionalities for depositing, withdrawing, and transferring ownership. It also includes an emergency withdrawal function for the contract owner.

## Features

- **Deposit Funds**: Users can deposit funds into their balance.
- **Withdraw Funds**: Users can withdraw funds from their balance.
- **Transfer Ownership**: The owner can transfer the contract ownership to a new address.
- **Check Balance**: Users can check their balance.
- **Emergency Withdraw**: The owner can withdraw all the funds from the contract in case of an emergency.

## Prerequisites

- [Remix IDE](https://remix.ethereum.org/)
- Ethereum wallet (e.g., MetaMask) for interacting with the contract on a test network

## Getting Started

### 1. Open Remix IDE

Go to [Remix IDE](https://remix.ethereum.org/).

### 2. Create a New File

- Click on the "File Explorer" icon on the left.
- Click on the "New File" icon.
- Name your file `BalanceManager.sol`.

### 3. Paste the Contract Code

Copy and paste the following Solidity code into the newly created file:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract BalanceManager {
    mapping(address => uint256) private balances;
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    // Modifier to restrict access to the owner
    modifier onlyOwner() {
        require(msg.sender == owner, "Caller is not the owner");
        _;
    }

    // Function to deposit funds
    function deposit() public payable {
        require(msg.value > 0, "Deposit amount must be greater than zero");
        balances[msg.sender] += msg.value;
    }

    // Function to withdraw funds
    function withdraw(uint256 amount) public {
        require(amount <= balances[msg.sender], "Insufficient balance");
        balances[msg.sender] -= amount;
        payable(msg.sender).transfer(amount);
    }

    // Function to transfer ownership
    function transferOwnership(address newOwner) public onlyOwner {
        require(newOwner != address(0), "New owner is the zero address");
        owner = newOwner;
    }

    // Function to get the balance of an account
    function getBalance(address account) public view returns (uint256) {
        return balances[account];
    }

    // Example of using assert()
    function checkBalance(address account) public view {
        uint256 balance = balances[account];
        assert(balance >= 0); // This should always be true
    }

    // Example of using revert()
    function emergencyWithdraw() public onlyOwner {
        uint256 contractBalance = address(this).balance;
        require(contractBalance > 0, "Contract balance is zero");
        
        // Transfer the contract balance to the owner
        (bool success, ) = owner.call{value: contractBalance}("");
        if (!success) {
            revert("Emergency withdrawal failed");
        }
        
        // Ensure the contract balance is zero after withdrawal
        assert(address(this).balance == 0);
    }
}
