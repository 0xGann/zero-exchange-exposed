## TLDR:
Zero Exchange source code is a fork of UniSwap with the ability to steal users' funds added. They have over 20 million USD of users funds in their contracts and the anonymous team can steal every dollar at any moment by minting ZERO token or wrapped Zero assets. All funds should be withdrawn immediately.

## ANALYSIS:
While doing some research on the Avalanche blockchain I became aware of an exchange called Zero Exchange, which was operating using a series of unverified smart contracts.

An unverified smart contract means that no-one using the smart contract can see what the code does. The code of an unverified smart contract could do anything. From the perspective of security, using an unverified smart contract is no different from sending your private keys to someone else and hoping they will not take your funds.

I raised the issue of the unverified smart contracts in the Zero Exchange Discord, expecting the team to address the issue immediately - given the seriousness of the situation. Instead I was met with a barrage of insults, and accused of spreading FUD. Apparently my posting had lead to a wave of scam accusations on Twitter, although I could find no such messages.

Eventually the team agreed to verify the contracts, and a few days later posted to inform me that their devs had verified the smart contracts.

However, checking the blockchain revealed that they had only verified their router and a factory contracts:
https://cchain.explorer.avax.network/address/0x85995d5f8ee9645cA855e92de16FA62D26398060/contracts
https://cchain.explorer.avax.network/address/0x2Ef422F30cdb7c5F1f7267AB5CF567A88974b308/transactions

All of the pools holding everyone’s funds remained unverified, but it’s viewable that they have all been launched using the router, and can therefore be expected to have the code from the router library.

However, my suspicions had been raised by the reaction of the team to asking for verified smart contracts, so I began to investigate further.

At the time of writing this report Zero Exchange is operating 6 pools on Avalanche blockchain, and each pool is paired against either ZERO token, or a wrapped zAsset. The addresses are:
```
0x332719570155dc61bEc2901A06d6B36faF02F184
0x4cEa032b4B3F59f31d6d52071258EE0d42b6cC7e
0x17766e5dd91FdC14C24CD9847e5E93fD62a05aFd
0xE1c458c5A0Cda501193c776a7A61de50DaDF3a9F
0x0751f9A49D921aA282257563C2041B9a0E00eB78
0x45c2755EEFA0eb96cE15C2f6FDc48346DA7f3A7e
```

Aside from this, they also have a pairing on Ethereum against their Zero token on Uniswap:
https://info.unitrade.app/token/0xf0939011a9bb95c3b791f0cb546377ed2693a574

The ZERO token contract code remains unverified on Avalanche. However, the ZERO token contract code on Ethereum IS verified and can be seen here:
https://etherscan.io/address/0xF0939011a9bb95c3B791f0cb546377Ed2693a574#code

On lines 51 to 53 we see a cap is specified for 1 billion tokens:
```
uint8 public constant decimals = 18;
uint256 public constant decimalFactor = 10 ** uint256(decimals);
uint256 private constant cap = 1000000000 * decimalFactor;
```

Then on lines 93, 140, and 69 we see a minting function, the control function for using it, and a modifier that locks minting to an owner address:
```
function _mint(address to, uint256 value) internal {
    require(totalSupply.add(value) <= cap, "ZERO:CAP_EXCEEDED");
    totalSupply = totalSupply.add(value);
    balanceOf[to] = balanceOf[to].add(value);
    emit Transfer(address(0), to, value);
}

function mint(address to, uint256 value) external onlyMinter returns (bool) {
    _mint(to, value);
    return true;
}

modifier onlyMinter {
    require(msg.sender == minter, "ZERO:NOT_MINTER");
    _;
}
```

Calling the contract minter variable reveals an address `0xf26eae2a9e571e60a166e4fa91f92e63984eeb2b`, which has the ability to create as many ZERO tokens as they want

Clearly, this is the same code as the ZERO token on Avalanche, and of course the reason they were refusing to verify it.

Checking the code for all of the wrapped assets shows they all have the same functionality, for example:

https://cchain.explorer.avax.network/address/0xf6F3EEa905ac1da6F6DD37d06810C6Fcb0EF5183/contracts
```
function mint(address to, uint256 amount) public {
    require(hasRole(MINTER_ROLE, _msgSender()), "ERC20PresetMinterPauser: musthave minter role to mint");
    _mint(to, amount);
}
```

The code for the ZERO token and these mintable assets is not published on their Github like their router and peripheral contracts. It's clear that the team wish to keep this functionality of their code hidden.

The code they have published is nothing other than the Uniswap code with the names of variables changed - zero innovation or new features added. You can view a comparison here:  
https://github.com/Uniswap/uniswap-v2-core/compare/master...zeroexchange:master

The sum of total assets in Zero Exchange related tools is currently over 20 million USD at time of writing. The code itself proves that at any second, the Zero Exchange team can activate the minting methods on all the assets they control to drain these pools.

Apart from the minting methods the Zero Exchange code is just a fork of Uniswap with the names of variables changed. They have taken Uniswap code, rebranded a simple website, and secretly added the ability for themselves to drain all funds, and are now managing over 20 million USD of people’s funds.

The team behind Zero Exchange is 100% anonymous, having taken significant steps to hide their identities. Despite their ability to take everyone’s funds at any money, their website brands their service as “Zero Custody”.

I am writing this on the 25th of February 2021 as an alert for the community and all users of Zero Exchange. I hope it gets out in time. The source code of this protocol is intentionally designed to give the anonymous owners the ability to take the funds of users.
