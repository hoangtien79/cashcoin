# cashcoin
AI Token
pragma solidity ^0.8.20;
contract CashGoCoin {
    string public name = "CashGoCoin"; // Token name
    string public symbol = "CGC";      // Token symbol
    uint8 public decimals = 18;        // Standard decimal places
    uint256 public totalSupply;        // Total supply of tokens

    mapping(address => uint256) private balances; // Tracks token balances
    mapping(address => mapping(address => uint256)) private allowances; // Tracks allowances
    mapping(address => uint256) private stakingBalances; // Tracks staked balances

    address public owner; // Contract owner
    uint256 public rewardPool; // Pool for rewards distribution

    uint256 public burnRate = 2; // 2% of each transaction is burned
    uint256 public redistributionRate = 3; // 3% of each transaction is redistributed

    // Events
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
    event OwnershipRenounced(address indexed previousOwner);
    event TokensBurned(uint256 amount);
    event TokensRedistributed(uint256 amount);
    event TokensStaked(address indexed staker, uint256 amount);
    event RewardsClaimed(address indexed claimant, uint256 amount);

    // Modifier to restrict access to the owner
    modifier onlyOwner() {
        require(msg.sender == owner, "Access denied: Only owner");
        _;
    }

    // Constructor: Initializes the contract with total supply
    constructor() {
        owner = msg.sender; // Set the deployer as the owner
        totalSupply = 120000000000 * (10 ** uint256(decimals)); // Set total supply
        balances[msg.sender] = totalSupply; // Assign all tokens to the deployer
        emit Transfer(address(0), msg.sender, totalSupply); // Emit transfer event for mint
    }

    // Function to check the balance of an address
    function balanceOf(address account) public view returns (uint256) {
        return balances[account];
    }

    // Function to transfer tokens
    function transfer(address recipient, uint256 amount) public returns (bool) {
        require(balances[msg.sender] >= amount, "Insufficient balance");

        uint256 burnAmount = (amount * burnRate) / 100;
        uint256 redistributionAmount = (amount * redistributionRate) / 100;
        uint256 transferAmount = amount - burnAmount - redistributionAmount;

        balances[msg.sender] -= amount;
        balances[recipient] += transferAmount;

        // Burn tokens
        totalSupply -= burnAmount;
        emit TokensBurned(burnAmount);

        // Redistribute tokens
        rewardPool += redistributionAmount;
        emit TokensRedistributed(redistributionAmount);

        emit Transfer(msg.sender, recipient, transferAmount);
        return true;
    }

    // Function to approve another address to spend tokens
    function approve(address spender, uint256 amount) public returns (bool) {
        allowances[msg.sender][spender] = amount;
        emit Approval(msg.sender, spender, amount);
        return true;
    }

    // Function to check the allowance for a spender
    function allowance(address _owner, address spender) public view returns (uint256) {
        return allowances[_owner][spender];
    }

    // Function to transfer tokens on behalf of another address
    function transferFrom(address sender, address recipient, uint256 amount) public returns (bool) {
        require(balances[sender] >= amount, "Insufficient balance");
        require(allowances[sender][msg.sender] >= amount, "Allowance exceeded");

        uint256 burnAmount = (amount * burnRate) / 100;
        uint256 redistributionAmount = (amount * redistributionRate) / 100;
        uint256 transferAmount = amount - burnAmount - redistributionAmount;

        balances[sender] -= amount;
        balances[recipient] += transferAmount;
        allowances[sender][msg.sender] -= amount;

        // Burn tokens
        totalSupply -= burnAmount;
        emit TokensBurned(burnAmount);

        // Redistribute tokens
        rewardPool += redistributionAmount;
        emit TokensRedistributed(redistributionAmount);

        emit Transfer(sender, recipient, transferAmount);
        return true;
    }

    // Function to stake tokens
    function stakeTokens(uint256 amount) public {
        require(balances[msg.sender] >= amount, "Insufficient balance to stake");

        balances[msg.sender] -= amount;
        stakingBalances[msg.sender] += amount;
        rewardPool += amount;

        emit TokensStaked(msg.sender, amount);
    }

    // Function to claim rewards
    function claimRewards() public {
        uint256 reward = (stakingBalances[msg.sender] * rewardPool) / totalSupply;
        require(reward > 0, "No rewards available");

        balances[msg.sender] += reward;
        rewardPool -= reward;

        emit RewardsClaimed(msg.sender, reward);
    }

    // Function to renounce ownership
    function renounceOwnership() public onlyOwner {
        emit OwnershipRenounced(owner);
        owner = address(0); // Set owner to zero address
    }
}
