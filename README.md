# Rareskills Ppgradable Proxy Contracts Questions

1. The OZ upgrade tool for hardhat defends against 6 kinds of mistakes. What are they and why do they matter?
    - constructor() in the implementation contract. When we have a constructor inside the implementation contract it sets the state variables inside the implementation contract but that is not reflected inside the proxy contract.
    - assigning initial values to state variables inside the implementation contract. This will have the same effect as using the contructor.
    - being able to call initialize() more than once. If the initialization function from the implementation contract can be called more than once, it can affect the state variables inside the proxy contract/s.
    - selfdestruct() in the implementation contract. If the implementation contract is deleted the proxy will delegatecall to non-existing contract and every call to the proxy contract will fail.
    - delegatecall() in the implementation contract. If the implementation contract delegates call to some other contract, this other contract can have selfdestruct in it, and because of delegatecall the implementation contract will be destroyed.
    - storage layout reordering between different implementation version contracts. If the order of the state variables is changed in the implementation contract, this may cause assigning of unexpected values to the state variables of the proxy.

2. What is a beacon proxy used for?
    - Beacon proxy allows multiple proxies to be upgraded to a different implementation in a single transaction. In this pattern, the proxy contract doesn’t hold the implementation address in storage like an ERC1967 proxy. Instead, the address is stored in a separate beacon contract. The upgrade operations are sent to the beacon instead of to the proxy contract, and all proxies that follow that beacon are automatically upgraded. If a project needs multiple proxies referring to the same implementation then beacon proxy is good option to go with.

3. Why does the openzeppelin upgradeable tool insert something like ```uint256[50] private __gap;``` inside the contracts? To see it, create an upgradeable smart contract that has a parent contract and look in the parent.
    - Given the following scenario:
    
    ```
    contract Base {
      uint256 base1;
    }

    contract Child is Base {
			uint256 child;
		}
    ```
    If Base is modified to add an extra variable:

    ```
    contract Base {
      uint256 base1;
      uint256 base2;
    }
    ```
    Then the variable ```base2``` would be assigned the slot that child had in the previous version, meaning we cannot add new variables to base contracts, if the child has any variables of its own. A workaround for this is to declare unused variables or storage gaps in base contracts that we want to extend in the future, 
		as a means of "reserving" those slots.

    Example:

    ```
    contract Base {
      uint256 base1;
      uint256[49] __gap;
    }

    contract Child is Base {
      uint256 child;
    }
    ```

    If Base is later modified to add extra variable(s), we can reduce the appropriate number of slots from the storage gap

    ```
    contract Base {
      uint256 base1;
      uint256 base2; // 32 bytes
      uint256[48] __gap;
    }
    ```

4. What is the difference between initializing the proxy and initializing the implementation? Do you need to do both? When do they need to be done?

5. What is the use for the ```reinitializer```? Provide a minimal example of proper use in Solidity