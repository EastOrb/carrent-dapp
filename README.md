# Car Rent DApp - README

Welcome to the Car Rent DApp! This decentralized application (DApp) is designed for car rental ventures, providing a streamlined process for users to hire cars. Car owners can set their own hire rates, and the admin plays a crucial role in approving suitable cars for hire, ensuring the company's image is protected.


## Features

- User-Friendly Interface
- Car Listing: Car owners can list their vehicles for hire as well end user
- Deployed contract wallet address which is the admin approve and reject cars that are not suitable fo the company 
- Transaction History Users can view their previous transactions with address they have transact with 
- Real-time Availability-Instant updates on car availability, users can check if a particular car is currently on hire
- Payment Handling: Secure payment processing, users can only hire a car if the previous user has completed the payment.

## Technologies
CarRent is a project that utilizes the Toastify, RainbowKit, and Smart Contract technologies with Next.js.

## Installation

To run the project locally, follow these steps:

1. Clone the repository: `git clone https://github.com/your-username/CarRent.git`
2. Install dependencies: `npm install`
3. Start the development server: `npm run dev`

## Usage

Once the project is running, you can access it in your browser at `http://localhost:3000`. Here are some usage examples:

- Connect your wallet
- Navigate to Hire cars on your navbar
- Rent a car of your choice
- Make payment and View rental history

## Contributing

If you would like to contribute to CarRent, please follow these guidelines:

1. Fork the repository
2. Create a new branch: `git checkout -b my-feature`
3. Make your changes and commit them: `git commit -m 'Add new feature'`
4. Push to the branch: `git push origin my-feature`
5. Submit a pull request


**Explaining the smart contract**
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

// Importing AccessControl for role management.
import "@openzeppelin/contracts/access/AccessControl.sol";

// Interface for interacting with an ERC-20 token.
interface IERC20Token {
    function transfer(address, uint256) external returns (bool);
    function approve(address, uint256) external returns (bool);
    function transferFrom(address, address, uint256) external returns (bool);
    function totalSupply() external view returns (uint256);
    function balanceOf(address) external view returns (uint256);
    function allowance(address, address) external view returns (uint256);

    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
}

// CarBooking contract managing car rentals and approvals.
contract CarBooking is AccessControl {
    // Address of the Celo Dollar token.
    address internal cUsdTokenAddress = 0x874069Fa1Eb16D44d622F2e0Ca25eeA172369bC1;

    // Address of the administrator.
    address admin;
    
    // Number of cars and rents for tracking.
    uint256 public carLength;
    uint256 public rentLength;

    // Enum defining car status (not accepted, accepted, out of service).
    enum CarStatus {
        NOTACCEPT,
        ACCEPTED,
        OUT_OF_SERVICE
    }

    // Enum defining order status (open, in progress, cancelled, completed).
    enum OrderStatus {
        OPEN,
        INPROGRESS,
        CANCELLED,
        COMPLETED
    }
```

**Explanation of Imports and Contract Structure:**
- The contract imports `AccessControl` for role management.
- An interface `IERC20Token` is defined for interacting with an ERC-20 token.
- `CarBooking` manages car rentals and approvals.

```solidity
    // Struct representing a car rental.
    struct Rent {
        uint256 carID;
        address carAddress;
        address BookingAcount;
        string name;
        string destination;
        uint256 amount;
        bool paid;
    }

    // Struct representing a car.
    struct Car {
        address payable owner;
        address admin;
        string model;
        string image;
        string plateNumber;
        uint256 bookingPrice;
        uint256 rentCar;
        CarStatus carStatus;
        OrderStatus orderStatus;
        Rent[] carRent; // Store the indices of rents for this car
    }
```

**Explanation of Structs:**
- Structs `Rent` and `Car` define the structure of car rentals and cars.

```solidity
    // Constructor initializes the contract and grants admin role to the deployer.
    constructor() {
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        admin = msg.sender;
    }

    // Modifier ensuring only the admin can call a function.
    modifier onlyAdmin() {
        require(hasRole(DEFAULT_ADMIN_ROLE, msg.sender), "Only admin can call this function");
        _;
    }
    
    // Mapping associating car ID with car details.
    mapping (uint256 => Car) public cars;
```

**Explanation of Constructor, Modifier, and Mapping:**
- The constructor initializes the contract and grants the admin role.
- Modifier `onlyAdmin` restricts certain functions to the admin.
- The `cars` mapping associates a unique ID with each car, storing details about each car.

```solidity
    // Function to add a new car with specified details.
    function addCar(string memory _model, string memory _imageCar, string memory _plateNumber, uint256 _bookingPrice) public {
        Car storage newCar = cars[carLength];
        newCar.owner = payable(msg.sender);
        newCar.admin = admin;
        newCar.model = _model;
        newCar.image = _imageCar;
        newCar.plateNumber = _plateNumber;
        newCar.bookingPrice = _bookingPrice;
        carLength++;
    }
```

**Explanation of `addCar` Function:**
- The `addCar` function allows the addition of a new car with specified details.
- It creates a new car instance with the provided details and increments `carLength`.

```solidity
    // Function to approve a car for usage.
    function carApprove(uint256 _carId) public onlyAdmin {
        Car storage car = cars[_carId];
        require(car.carStatus == CarStatus.NOTACCEPT, "Car did not meet the service standards or already accepted");
        car.carStatus = CarStatus.ACCEPTED;
    }
```

**Explanation of `carApprove` Function:**
- The `carApprove` function approves a car for usage.
- It checks if the car is not already accepted and meets service standards.
- If conditions are met, it updates the car status to "ACCEPTED."

```solidity
    // Function to reject a previously accepted car.
    function rejectCar(uint256 _carId) public onlyAdmin {
        Car storage car = cars[_carId];
        require(car.carStatus == CarStatus.ACCEPTED, "Car not accepted");
        car.carStatus = CarStatus.NOTACCEPT;
    }
```

**Explanation of `rejectCar` Function:**
- The `rejectCar` function rejects a previously accepted car.
- It checks if the car is already accepted before updating the status to "NOTACCEPT."

```solidity
    // Function to mark an accepted car as out of service.
    function outOfServiceCar(uint256 _carId) public onlyAdmin {
        Car storage car = cars[_carId];
        require(car.carStatus == CarStatus.ACCEPTED, "Car is not approved");
        car.carStatus = CarStatus.OUT_OF_SERVICE;
    }
```

**Explanation of `outOfServiceCar` Function:**
- The `outOfServiceCar` function marks an accepted car as out of service.
- It checks if the car is approved before updating the status to "OUT_OF_SERVICE."

```solidity
    // Function to add a new car rental.
    function addRent(uint256 _carID, string memory _name, string memory _destination) public {
        require(_carID < carLength, "Invalid Car index");
        Car storage car = cars[_carID];
        require(car.carStatus == CarStatus.ACCEPTED, "Car is not approved for usage");
        require(car.orderStatus == OrderStatus.OPEN, "Car is already on hire");
        Rent memory newRent = Rent({
            carID: _carID,
            carAddress: car.owner,
            BookingAcount: msg.sender,
            name: _name,
            destination: _destination,
            amount: car.bookingPrice,
            paid: false
        });
        rentLength++;

        cars[_carID].orderStatus = OrderStatus.INPROGRESS; 
        cars[_carID].carRent.push(newRent);
    }
```

**Explanation of `addRent` Function:**
- The `addRent` function enables a user to rent a car.
- It checks if the car is approved for usage and not already on hire.
- If conditions are met, a new rent instance is created, and the car's order status is updated to "INPROGRESS."

```solidity
    // Function to get details of a specific car.
    function getCars(uint256 _index) public view returns (
        address, address, string memory, string memory,
        string memory, uint256,
        uint256,
        CarStatus, 
        OrderStatus
    ) {
        Car storage car = cars[_index];
        return (
            car.owner,
            car.admin,
            car.model,
            car.image,
            car.plateNumber,
            car.bookingPrice,
            car.rentCar,
            car.carStatus,
            car.orderStatus
        );
    }
```

**Explanation of `getCars` Function:**
- The `getCars` function retrieves details of a specific car using its index.
- It returns relevant information such as owner, model, image, etc.

```solidity
    // Function to get details of a specific car rental.
    function getRent(uint256 _index) public view returns (
        uint256,
        address,
        address,
        string memory,
        string memory,
        uint256,
        bool
    ) {
        Car storage car = cars[_index];
        return (
            car.carRent[_index].carID,
            car.carRent[_index].carAddress,
            car.carRent[_index].BookingAcount,
            car.carRent[_index].name,
            car.carRent[_index].destination,
            car.carRent[_index].amount,
            car.carRent[_index].paid
        );
    }
```

**Explanation of `getRent` Function:**
- The `getRent` function retrieves details of a specific car rental using its index.
- It returns information such as car ID, customer address, rental amount, etc.

```solidity
    // Function to get the total number of cars.
    function getCarLength() public view returns(uint256) {
        return carLength;
    }
```

**Explanation of `getCarLength` Function:**
- The `getCarLength` function returns the total number of cars.

```solidity
    // Function to get the total number of car rentals.
    function getRentLength() public view returns(uint256) {
        return rentLength;
    }
```

**Explanation of `getRentLength` Function:**
- The `getRentLength` function returns the total number of car rentals.

```solidity
    // Function for processing car rental payment.
    function carRentPayment(uint256 _index) external payable {
        Car storage car = cars[_index];
        require(msg.sender == car.carRent[_index].BookingAcount, "Must be the owner");
        require(IERC20Token(cUsdTokenAddress).transferFrom(
            msg.sender,
            cars[_index].owner,
            car.bookingPrice
         ), "Transfer failed");
        car.carRent[_index].paid = true;
        cars[_index].orderStatus = OrderStatus.OPEN; 
    }
}
```

**Explanation of `carRentPayment` Function:**
- The `carRentPayment` function processes car rental payment.
- It ensures that the caller is the owner of the rental.
- Transfers the rental amount using the Celo Dollar token and updates the payment status and order status.
