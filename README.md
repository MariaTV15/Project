# BountyHunter

BountyHunter is a decentralized bounty board where users can create bounties with specific rewards. Bounty hunters can claim tasks, submit proof of completion, and receive rewards upon approval from the bounty creator.

## Features
- Create bounties with ETH rewards
- Hunters can claim and submit proof of task completion
- Bounty creators approve submissions and release rewards

## Getting Started

### Deploying the Contract
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract BountyBoard {
    struct Bounty {
        address creator;
        string description;
        uint256 reward;
        bool completed;
        address hunter;
    }

    Bounty[] public bounties;
    mapping(uint256 => string) public submissions;

    event BountyCreated(uint256 indexed bountyId, address indexed creator, uint256 reward);
    event BountyClaimed(uint256 indexed bountyId, address indexed hunter);
    event BountyCompleted(uint256 indexed bountyId, address indexed hunter, uint256 reward);

    function createBounty(string memory _description) external payable {
        require(msg.value > 0, "Reward must be greater than 0");

        bounties.push(Bounty({
            creator: msg.sender,
            description: _description,
            reward: msg.value,
            completed: false,
            hunter: address(0)
        }));

        emit BountyCreated(bounties.length - 1, msg.sender, msg.value);
    }

    function claimBounty(uint256 _bountyId, string memory _submission) external {
        require(_bountyId < bounties.length, "Invalid bounty ID");
        Bounty storage bounty = bounties[_bountyId];
        require(!bounty.completed, "Bounty already completed");
        require(bounty.hunter == address(0), "Bounty already claimed");

        bounty.hunter = msg.sender;
        submissions[_bountyId] = _submission;

        emit BountyClaimed(_bountyId, msg.sender);
    }

    function completeBounty(uint256 _bountyId) external {
        require(_bountyId < bounties.length, "Invalid bounty ID");
        Bounty storage bounty = bounties[_bountyId];
        require(msg.sender == bounty.creator, "Only creator can complete");
        require(bounty.hunter != address(0), "No hunter assigned");
        require(!bounty.completed, "Already completed");

        bounty.completed = true;
        payable(bounty.hunter).transfer(bounty.reward);

        emit BountyCompleted(_bountyId, bounty.hunter, bounty.reward);
    }

    function getBountyCount() external view returns (uint256) {
        return bounties.length;
    }

    function getBounty(uint256 _bountyId) external view returns (address, string memory, uint256, bool, address) {
        require(_bountyId < bounties.length, "Invalid bounty ID");
        Bounty storage bounty = bounties[_bountyId];
        return (bounty.creator, bounty.description, bounty.reward, bounty.completed, bounty.hunter);
    }
}
```

### Interacting with the Contract
- **Create a Bounty**: Send ETH with a description to create a bounty.
- **Claim a Bounty**: Hunters can submit proof to claim a bounty.
- **Complete a Bounty**: Creators approve submissions and release funds.

## License
This project is licensed under the MIT License.

