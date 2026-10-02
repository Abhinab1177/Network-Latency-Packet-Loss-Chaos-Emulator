# Stage 3 – System Design & Architecture

## 1. Introduction

Stage 3 defines the architecture and technical design of the Network Latency and Packet Loss Chaos Emulator.

The purpose of this stage is to describe how the seven core features will be organized and how the user-space C++ application, Linux device driver, and Linux networking components will communicate with each other.

The system is divided into user space, kernel space, and the Linux network layer.

The seven core features are:

1. Latency / Delay
2. Packet Loss
3. Jitter
4. Network Profiles
5. TCP Testing
6. UDP Testing
7. Linux Device Driver
---

## 2. Overall System Architecture

The system follows a layered architecture consisting of three main levels:

### 2.1 User Space

The user interacts with the C++ control application.

The application is responsible for:

- Accepting network configuration
- Selecting network profiles
- Communicating with the Linux device driver
- Starting TCP and UDP tests
- Displaying test status and results

### 2.2 Kernel Space

The Linux character device driver operates in kernel space.

The driver provides:

- Character device registration
- Device interface through `/dev/`
- Configuration storage
- Communication with the C++ application
- `ioctl` support for structured configuration

### 2.3 Linux Network Layer

The Linux networking subsystem applies the selected network conditions.

The network layer is responsible for:

- Latency / Delay
- Packet Loss
- Jitter

The C++ application controls the required Linux networking configuration from user space.

The device driver does not directly execute networking commands. Instead, it provides the kernel-level interface for configuration management, while the user-space application performs the required network control operations.
---

## 3. System Architecture Flow

The overall communication flow of the system is:

User
  ↓
C++ Control Application
  ↓
Linux Character Device
  ↓
Linux Device Driver
  ↓
Network Configuration
  ↓
Linux Network Interface
  ↓
TCP / UDP Communication

The C++ application acts as the main control component.

The application sends the required configuration to the Linux character device. The device driver receives and maintains this configuration.

The user-space application then applies the required network conditions through Linux networking facilities.

TCP or UDP testing is performed after the selected network conditions have been configured.
---

## 4. Component Architecture

The project will contain the following major components:

### 4.1 Linux Device Driver

The device driver will be implemented as a Linux kernel module using C.

Main responsibilities:

- Register the character device
- Create the device interface
- Receive configuration from user space
- Store the current configuration
- Provide `ioctl`, `read`, and `write` interfaces
- Validate basic configuration values

### 4.2 C++ Control Application

The C++ application will be the main user-space program.

Main responsibilities:

- Accept user input
- Select network profiles
- Build the required configuration
- Communicate with the device driver
- Configure Linux networking
- Start TCP or UDP tests
- Display operation status

### 4.3 Network Controller

The Network Controller will manage the selected network conditions.

It will configure:

- Latency / Delay
- Packet Loss
- Jitter

Linux `tc` and `netem` will be used for applying these network conditions.

### 4.4 Network Profiles

The Profile Manager will contain the four predefined profiles:

- 3G
- 4G
- Wi-Fi
- Satellite

Each profile will provide project-defined values for latency, packet loss, and jitter.

### 4.5 TCP Test Module

The TCP Test Module will implement basic TCP client/server communication.

It will be used to observe TCP communication under different network conditions.

### 4.6 UDP Test Module

The UDP Test Module will implement basic UDP client/server communication.

It will be used to observe UDP communication under different network conditions.
---

## 5. Data Structures

The system will use structured data to represent the selected network configuration.

### 5.1 Network Configuration Structure

The configuration will contain:

- Latency value
- Packet loss percentage
- Jitter value
- Selected network profile

A conceptual representation is:

NetworkConfig
├── latency
├── packet_loss
├── jitter
└── profile

This structure will be shared between the C++ application and the device-driver interface where required.

### 5.2 Network Profile Structure

Each predefined profile will contain:

- Profile name
- Latency
- Packet loss
- Jitter

Conceptual representation:

NetworkProfile
├── name
├── latency
├── packet_loss
└── jitter

### 5.3 Test Configuration

The transport test configuration will identify the selected protocol:

TestConfig
├── protocol
└── test parameters

The protocol value will identify either TCP or UDP.
---

## 6. System Architecture Diagram

The planned architecture can be represented as:

                    USER
                      |
                      v
             +------------------+
             | C++ Control App  |
             +------------------+
                |            |
                |            |
                v            v
        +---------------+  +----------------+
        | Linux Device  |  | TCP / UDP Test |
        |    Driver     |  |    Modules     |
        +---------------+  +----------------+
                |
                v
        +-------------------+
        | Network Controller|
        +-------------------+
                |
                v
        +-------------------+
        | Linux Network     |
        | Interface         |
        +-------------------+
                |
                v
             NETWORK

The C++ application acts as the central user-space controller.

The Linux device driver provides the kernel-space interface.

The Network Controller applies the selected latency, packet loss, and jitter conditions.

The TCP and UDP modules perform communication tests using the configured network conditions.
---

## 7. UML Class Design

The main classes planned for the C++ application are:

### NetworkConfig

Responsible for storing:

- Latency
- Packet loss
- Jitter
- Network profile

### ProfileManager

Responsible for:

- Storing predefined profiles
- Selecting a profile
- Returning the corresponding network configuration

### DriverInterface

Responsible for:

- Opening the Linux device
- Sending configuration to the driver
- Reading driver status
- Using the required `ioctl` operations

### NetworkController

Responsible for:

- Applying latency
- Applying packet loss
- Applying jitter
- Resetting the network configuration

### TCPTester

Responsible for:

- Creating TCP connections
- Sending and receiving test data
- Reporting TCP test status

### UDPTester

Responsible for:

- Creating UDP communication
- Sending and receiving test data
- Reporting UDP test status

The conceptual relationship is:

NetworkConfig
      |
      v
ProfileManager
      |
      v
DriverInterface ---> Linux Device Driver
      |
      v
NetworkController
      |
      +------> TCPTester
      |
      +------> UDPTester
      ---

## 8. UML Sequence Design

The main sequence of operations is planned as follows:

User
  |
  | Select profile / configure values
  v
C++ Control Application
  |
  | Send configuration
  v
Linux Character Device
  |
  | ioctl()
  v
Linux Device Driver
  |
  | Store configuration
  v
C++ Control Application
  |
  | Apply network configuration
  v
Network Controller
  |
  | Configure latency, packet loss and jitter
  v
Linux Network Interface
  |
  | Start test
  v
TCP / UDP Test Module
  |
  | Perform communication
  v
Test Result
  |
  v
C++ Control Application
  |
  v
User

---

## 9. UML State Machine Design

The main application will follow a sequence of operational states.

The planned state flow is:

START
  |
  v
INITIALIZE
  |
  v
WAIT FOR USER INPUT
  |
  +----------------------+
  |                      |
  v                      v
SELECT PROFILE       ENTER CONFIGURATION
  |                      |
  +----------+-----------+
             |
             v
SEND CONFIGURATION
             |
             v
CONFIGURATION APPLIED
             |
             v
SELECT TEST
     /              \
    v                v
TCP TEST          UDP TEST
    \                /
     +------->-------+
             |
             v
DISPLAY RESULT
             |
             v
RESET / NEW TEST
             |
             v
WAIT FOR USER INPUT
---

## 10. Implementation Plan

The implementation will be carried out in incremental steps.

### Step 1 – Linux Device Driver

- Create the kernel module source file.
- Implement the Linux character device.
- Register the device with the kernel.
- Create the device interface under `/dev/`.
- Implement configuration handling.
- Implement required `ioctl`, `read`, and `write` operations.
- Test communication from user space.

### Step 2 – C++ Application

- Create the main C++ application.
- Implement user input handling.
- Implement network configuration structures.
- Implement communication with the device driver.
- Implement input validation.

### Step 3 – Network Controller

- Identify the active Linux network interface.
- Implement latency configuration.
- Implement packet-loss configuration.
- Implement jitter configuration.
- Implement network reset functionality.

### Step 4 – Network Profiles

Implement the four predefined profiles:

- 3G
- 4G
- Wi-Fi
- Satellite

Each profile will provide project-defined latency, packet-loss, and jitter values.

### Step 5 – TCP and UDP Testing

- Implement TCP client/server communication.
- Implement UDP client/server communication.
- Run tests using the selected network configuration.
- Verify successful data transmission.

### Step 6 – Integration

Integrate all seven project features:

- Latency / Delay
- Packet Loss
- Jitter
- Network Profiles
- TCP Testing
- UDP Testing
- Linux Device Driver

### Step 7 – Testing and Debugging

- Compile all components.
- Test the driver independently.
- Test each network condition independently.
- Test each network profile.
- Test TCP communication.
- Test UDP communication.
- Perform complete system integration testing.
- Fix errors discovered during testing.
---

## 11. Development Environment and Build Structure

The project will be developed and tested on Ubuntu Linux.

The main development tools are:

- GCC
- G++
- CMake
- Make
- Git
- Linux kernel headers
- `iproute2`
- `tc`

The source code will be divided into user-space and kernel-space components.

The planned project structure is:

Network-Latency-Packet-Loss-Chaos-Emulator/
│
├── driver/
│   └── Linux device driver source
│
├── include/
│   └── Shared header files
│
├── src/
│   └── C++ application source
│
├── tests/
│   └── Test programs and test-related files
│
├── docs/
│   ├── Stage1/
│   ├── Stage2/
│   ├── Stage3/
│   ├── Stage4/
│   ├── Stage5/
│   └── Stage6/
│
├── CMakeLists.txt
├── README.md
└── .gitignore

The driver component will be compiled using the Linux kernel build system, while the user-space C++ application will be built using CMake.
---

## 12. Git Branching and Version Control Plan

Git will be used to maintain the complete development history of the project.

The `master` branch will contain the stable project version.

Development work will be organized into logical commits based on major implementation milestones.

The planned commit sequence is:

1. Initial project structure and Stage 1 documentation
2. Stage 2 requirements and development plan
3. Stage 3 system design and architecture
4. Linux device driver implementation
5. C++ control application
6. Network latency implementation
7. Packet loss implementation
8. Jitter implementation
9. Network profiles implementation
10. TCP testing implementation
11. UDP testing implementation
12. System integration
13. Testing and debugging
14. Final documentation

Each major change will be compiled and tested before committing it to Git.

The Git repository will contain the source code, documentation, build instructions, and README required to reproduce the project.
---

## 13. Documentation and Progress Tracking

The development progress will be documented throughout the project.

Each stage will contain:

- Work completed
- Implementation details
- Testing performed
- Problems encountered
- Solutions applied
- Screenshots or terminal evidence where applicable
- Git commit information

The documentation will be maintained in the following directories:

docs/Stage1/
docs/Stage2/
docs/Stage3/
docs/Stage4/
docs/Stage5/
docs/Stage6/

The Git history will also be used as evidence of continuous development.

The README file will contain:

- Project overview
- Features
- Requirements
- Build instructions
- Driver loading instructions
- Application execution instructions
- Testing instructions
- Project structure
---

## 14. Stage 3 Completion Criteria

Stage 3 will be considered complete when the following design elements have been documented:

- Overall system architecture
- Component architecture
- System architecture flow
- Data structures
- UML class design
- UML sequence design
- UML state machine design
- Implementation plan
- Development environment
- Project directory structure
- Git and version-control plan
- Documentation and progress-tracking plan

The design will serve as the technical foundation for the implementation stages.

The next stage will focus on developing the initial Linux device driver and C++ prototype based on this design.
