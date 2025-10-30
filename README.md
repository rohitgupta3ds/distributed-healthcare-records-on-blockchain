// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

/**
 * @title Distributed Healthcare Record System
 * @notice Example smart contract for managing patient health records on blockchain
 * @dev Works directly in Remix IDE
 */
contract HealthRecords {

    // --- STRUCTS ---

    struct Record {
        string recordHash;     // IPFS CID or hash reference
        address addedBy;       // healthcare provider
        uint256 timestamp;     // when the record was added
    }

    struct Patient {
        bool registered;
        address patientAddress;
        mapping(address => bool) authorizedProviders;
        Record[] records;
    }

    // --- STATE VARIABLES ---

    address public admin;
    mapping(address => bool) public healthcareProviders;
    mapping(address => Patient) private patients;

    // --- EVENTS ---

    event PatientRegistered(address indexed patient);
    event ProviderRegistered(address indexed provider);
    event ProviderRevoked(address indexed provider);
    event RecordAdded(address indexed patient, address indexed provider, string recordHash);
    event ProviderAuthorized(address indexed patient, address indexed provider);
    event ProviderAccessRevoked(address indexed patient, address indexed provider);

    // --- MODIFIERS ---

    modifier onlyAdmin() {
        require(msg.sender == admin, "Not admin");
        _;
    }

    modifier onlyProvider() {
        require(healthcareProviders[msg.sender], "Not a healthcare provider");
        _;
    }

    modifier onlyAuthorized(address _patient) {
        require(
            msg.sender == _patient ||
            patients[_patient].authorizedProviders[msg.sender],
            "Not authorized"
        );
        _;
    }

    // --- CONSTRUCTOR ---

    constructor() {
        admin = msg.sender; // Deployer becomes admin
    }

    // --- ADMIN FUNCTIONS ---

    function registerProvider(address _provider) external onlyAdmin {
        require(!healthcareProviders[_provider], "Provider already registered");
        healthcareProviders[_provider] = true;
        emit ProviderRegistered(_provider);
    }

    function revokeProvider(address _provider) external onlyAdmin {
        require(healthcareProviders[_provider], "Provider not registered");
        healthcareProviders[_provider] = false;
        emit ProviderRevoked(_provider);
    }

    // --- PATIENT FUNCTIONS ---

    function registerPatient() external {
        require(!patients[msg.sender].registered, "Already registered");
        patients[msg.sender].registered = true;
        patients[msg.sender].patientAddress = msg.sender;
        emit PatientRegistered(msg.sender);
    }

    function authorizeProvider(address _provider) external {
        require(patients[msg.sender].registered, "Not a registered patient");
        require(healthcareProviders[_provider], "Provider not valid");
        patients[msg.sender].authorizedProviders[_provider] = true;
        emit ProviderAuthorized(msg.sender, _provider);
    }

    function revokeProviderAccess(address _provider) external {
        require(patients[msg.sender].registered, "Not a registered patient");
        patients[msg.sender].authorizedProviders[_provider] = false;
        emit ProviderAccessRevoked(msg.sender, _provider);
    }

    // --- RECORD FUNCTIONS ---

    function addRecord(address _patient, string memory _recordHash)
        external
        onlyProvider
        onlyAuthorized(_patient)
    {
        Record memory newRecord = Record({
            recordHash: _recordHash,
            addedBy: msg.sender,
            timestamp: block.timestamp
        });

        patients[_patient].records.push(newRecord);
        emit RecordAdded(_patient, msg.sender, _recordHash);
    }

    function getRecordCount(address _patient)
        external
        view
        onlyAuthorized(_patient)
        returns (uint256)
    {
        return patients[_patient].records.length;
    }

    function getRecord(address _patient, uint256 index)
        external
        view
        onlyAuthorized(_patient)
        returns (string memory recordHash, address addedBy, uint256 timestamp)
    {
        Record storage rec = patients[_patient].records[index];
        return (rec.recordHash, rec.addedBy, rec.timestamp);
    }
}

contract address: 0xd9145CCE52D386f254917e481eB44e9943F39138
<img width="1410" height="914" alt="rohit add" src="https://github.com/user-attachments/assets/fdb61388-925e-4ac9-9ea7-7d0f632cff69" />


