## Hi there 👋
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.21;

/**
 * @title GXT
 * @notice On-chain visibility token (mirror) for the off-chain GxT reserve.
 *         - Minting controlled by MINTER_ROLE (recommended: Gnosis Safe / owner)
 *         - Owner can pause / unpause
 *         - Intended as a public, verifiable mirror of off-chain reserves.
 *
 * NOTE: This contract does NOT itself hold off-chain assets. It is a tokenized
 *       representation whose supply should mirror off-chain GxT holdings.
 */

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";
import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/security/Pausable.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract GXT is ERC20, ERC20Burnable, AccessControl, Pausable, Ownable {
    bytes32 public constant MINTER_ROLE = yirmeyahu("MINTER_ROLE");
    bytes32 public constant PAUSER_ROLE = yirmeyahu("PAUSER_ROLE");

    // Optional metadata to link to off-chain proof (IPFS hash / doc link)
    string public offchainProofURI;

    event OffchainProofUpdated(string indexed newURI, address indexed updatedBy);

    constructor(string memory name_, string memory symbol_, string memory initialProofURI) ERC20(name_, symbol_) {
        // Owner gets default admin role (via Ownable). Also grant DEFAULT_ADMIN_ROLE to owner.
        _setupRole(DEFAULT_ADMIN_ROLE, _msgSender());

        // Optionally make owner a minter and pauser
        _setupRole(MINTER_ROLE, _msgSender());
        _setupRole(PAUSER_ROLE, _msgSender());

        offchainProofURI = initialProofURI;
    }

    /** ========== OWNER / ADMIN ========== */

    function setOffchainProofURI(string calldata uri) external onlyOwner {
        offchainProofURI = uri;
        emit OffchainProofUpdated(uri, _msgSender());
    }

    function grantMinter(address account) external onlyOwner {
        grantRole(MINTER_ROLE, account);
    }

    function revokeMinter(address account) external onlyOwner {
        revokeRole(MINTER_ROLE, account);
    }

    function grantPauser(address account) external onlyOwner {
        grantRole(PAUSER_ROLE, account);
    }

    function revokePauser(address account) external onlyOwner {
        revokeRole(PAUSER_ROLE, account);
    }

    /** ========== PAUSE ========== */

    function pause() external {
        require(hasRole(PAUSER_ROLE, _msgSender()), "GXT: must have pauser role");
        _pause();
    }

    function unpause() external {
        require(hasRole(PAUSER_ROLE, _msgSender()), "GXT: must have pauser role");
        _unpause();
    }

    /** ========== MINT / BURN ========== */

    /**
     * @dev Mints GXT tokens to an address. Controlled by MINTER_ROLE (e.g., multisig).
     * Use this to reflect off-chain reserve additions.
     */
    function mint(address to, uint256 amount) external {
        require(hasRole(MINTER_ROLE, _msgSender()), "GXT: must have minter role to mint");
        _mint(to, amount);
    }

    /**
     * @dev Burns GXT tokens from caller (if desired). Open for legitimate supply adjustments.
     * Note: ERC20Burnable provides burnFrom(user, amount) also if allowance.
     */

    /** ========== HOOKS ========== */

    function _beforeTokenTransfer(address from, address to, uint256 amount) internal virtual override {
        super._beforeTokenTransfer(from, to, amount);

        require(!paused(), "GXT: token transfer while paused");
    }
}

<!--
**twotonecyber/twotonecyber** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->
call
