# DAO-treasury-with-proposal-to-send-ETH
DAO treasury with proposal to send ETH
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract TreasuryDAO {
    struct Proposal {
        address to;
        uint256 amount;
        uint256 yes;
        uint256 no;
        bool executed;
    }

    mapping(address => bool) public member;
    Proposal[] public proposals;
    mapping(uint256 => mapping(address => bool)) public voted;

    constructor() {
        member[msg.sender] = true;
    }

    receive() external payable {}

    function addMember(address m) public {
        require(member[msg.sender], "Not member");
        member[m] = true;
    }

    function create(address to, uint256 amount) public {
        require(member[msg.sender], "Not member");
        proposals.push(Proposal(to, amount, 0, 0, false));
    }

    function vote(uint256 id, bool yes) public {
        require(member[msg.sender], "Not member");
        require(!voted[id][msg.sender], "Voted");
        voted[id][msg.sender] = true;
        if (yes) proposals[id].yes++;
        else proposals[id].no++;
    }

    function execute(uint256 id) public {
        Proposal storage p = proposals[id];
        require(!p.executed, "Done");
        require(p.yes > p.no, "Not passed");
        p.executed = true;
        payable(p.to).transfer(p.amount);
    }
}
