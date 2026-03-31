# Lucky10
BaseLucky10.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.34;

contract BaseLucky10 {
    address public immutable owner;
    uint256 public treasuryBalance;

    event NumberGuessed(
        address indexed player,
        uint8 guessedNumber,
        uint8 luckyNumber,
        uint256 bet,
        uint256 payout,
        uint256 timestamp
    );

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Only owner can call this");
        _;
    }

    function guess(uint8 _number) external payable {
        require(msg.value == 0.001 ether, "Must pay exactly 0.001 ETH");
        require(_number >= 1 && _number <= 10, "Number must be between 1 and 10");

        // 生成幸运数字 1~10
        uint256 random = uint256(keccak256(abi.encodePacked(
            block.prevrandao,
            msg.sender,
            block.timestamp,
            block.number
        ))) % 10 + 1;

        uint8 luckyNumber = uint8(random);
        uint256 payout = 0;

        if (_number == luckyNumber) {
            payout = 0.018 ether;           // 18x 大奖
            payable(msg.sender).transfer(payout);
        } else {
            treasuryBalance += msg.value;   // 未中奖进入金库
        }

        emit NumberGuessed(msg.sender, _number, luckyNumber, msg.value, payout, block.timestamp);
    }

    function getTreasury() external view returns (uint256) {
        return treasuryBalance;
    }

    function withdrawTreasury() external onlyOwner {
        uint256 amount = treasuryBalance;
        treasuryBalance = 0;
        payable(owner).transfer(amount);
    }

    function getContractInfo() external pure returns (string memory) {
        return "BaseLucky10 - Pay 0.001 ETH to guess 1-10 and win 18x!";
    }
}
# Lucky UPDATA
