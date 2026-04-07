# -Multi-Sig-Escrow-with-Timelock
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/access/Ownable.sol";

contract MultiSigEscrow is Ownable {
    address public buyer;
    address public seller;
    uint256 public amount;
    uint256 public releaseTime;
    bool public released;

    constructor(address _buyer, address _seller, uint256 _delayDays) payable Ownable(msg.sender) {
        buyer = _buyer;
        seller = _seller;
        amount = msg.value;
        releaseTime = block.timestamp + (_delayDays * 1 days);
    }

    function release() external {
        require(msg.sender == buyer || msg.sender == owner(), "Not authorized");
        require(block.timestamp >= releaseTime, "Timelock active");
        require(!released, "Already released");
        released = true;
        payable(seller).transfer(amount);
    }

    function refund() external {
        require(msg.sender == buyer || msg.sender == owner(), "Not authorized");
        require(block.timestamp < releaseTime, "Cannot refund");
        payable(buyer).transfer(amount);
    }
}
