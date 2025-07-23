# KTP_Protocol

## Overview

KTP_Protocol is a custom, kernel-level transport protocol implemented over UDP, designed to provide reliable, flow-controlled data transfer between two endpoints. The protocol mimics key features of TCP, such as sliding window flow control, retransmissions, and ordered delivery, but is implemented in user space using shared memory and semaphores for inter-process communication. This project demonstrates how reliable data transfer and flow control can be established over an unreliable transport like UDP.

## What This Project Establishes
- **Reliable Data Transfer over UDP:** By implementing acknowledgments, retransmissions, and sequence numbering, the protocol ensures that data sent over UDP is delivered reliably and in order.
- **Sliding Window Flow Control:** Both sender and receiver maintain sliding windows to manage the flow of packets, prevent buffer overflows, and maximize throughput.
- **Custom Socket API:** The project provides a custom socket API (`k_socket`, `k_bind`, `k_sendto`, `k_recvfrom`, `k_close`) that mimics the standard BSD socket interface but operates on the custom protocol.
- **Shared Memory and Semaphores:** All socket state and buffers are managed in shared memory, with semaphores ensuring safe concurrent access.
- **Packet Loss Simulation:** The protocol can simulate packet loss to test retransmission and robustness.

## How Flow Control is Achieved
- **Sender Window (`swnd`):** The sender maintains a window of unacknowledged packets. It only sends new packets if there is space in the window, preventing overwhelming the receiver.
- **Receiver Window (`rwnd`):** The receiver advertises its available buffer space to the sender via ACKs. If the receiver's buffer is full, it signals the sender to pause transmission.
- **Circular Buffers:** Both send and receive buffers are implemented as circular arrays, allowing efficient use of memory and seamless wrap-around.
- **Sequence Numbers with Wrap-Around:** Packets are assigned sequence numbers (1-255), and wrap-around is handled to ensure correct ordering and duplicate detection.
- **Timeouts and Retransmissions:** If an ACK is not received within a timeout, the sender retransmits the packet. The protocol adapts to packet loss and network delays.

## Project Structure
- `ksocket.h` / `ksocket.c`: Custom socket API and protocol logic.
- `initsocket.c`: Initializes shared memory, semaphores, and runs the protocol daemon (handles socket creation, binding, and cleanup).
- `user1.c`: Sender application. Reads from an input file and sends data using the custom protocol.
- `user2.c`: Receiver application. Receives data and writes to an output file.
- `makefile`: Build and run automation.
- `input.txt`: Example input file for transmission.
- `documentation.txt`: Detailed documentation of structures, functions, and protocol design.

## Building the Project

1. **Compile all components:**
   ```sh
   make all
   ```
   This will build the protocol library, the initialization daemon (`init`), and the user applications (`user1`, `user2`).

2. **Clean build artifacts:**
   ```sh
   make clean
   ```

## Running the Protocol

1. **Start the Protocol Daemon:**
   The daemon must be running to manage shared memory and socket state.
   ```sh
   make run_init
   # or
   ./init
   ```

2. **Start the Receiver (User 2):**
   In a new terminal, run:
   ```sh
   make run_user2
   # or
   ./user2 <Source IP> <Source Port> <Remote IP> <Remote Port> <outputfile.txt>
   # Example:
   ./user2 127.0.0.1 9093 127.0.0.1 8082 output.txt
   ```
   This will wait for incoming data and write it to `output.txt`.

3. **Start the Sender (User 1):**
   In another terminal, run:
   ```sh
   make run_user1
   # or
   ./user1 <Source IP> <Source Port> <Remote IP> <Remote Port> <inputfile.txt>
   # Example:
   ./user1 127.0.0.1 8082 127.0.0.1 9093 input.txt
   ```
   This will read from `input.txt` and send the data to the receiver.

## Example Usage

- To send the contents of `input.txt` from User 1 to User 2 and write the result to `output.txt`:
  1. Start `init` (daemon)
  2. Start `user2` (receiver)
  3. Start `user1` (sender)

## Protocol Details

- **Sliding Window:** Both sender and receiver use a window of size 10 (configurable via `MAX_WINDOW_SIZE`).
- **Packet Structure:** Each packet contains a sequence number, data, and an ACK flag. ACKs also carry the receiver's available window size.
- **Flow Control:** The sender pauses when the receiver's window is full and resumes when space is available.
- **Timeouts:** Retransmissions are triggered if ACKs are not received in time.
- **Packet Loss Simulation:** The protocol can simulate packet drops to test reliability.

## Performance

The protocol has been tested with various packet loss probabilities. See `documentation.txt` for a detailed performance table and further implementation details.

## References
- See `documentation.txt` for in-depth explanations of structures, functions, and protocol mechanisms.
- For assignment details, see `CS39006_Networks_Lab_Assignment4.pdf`.

---

**Author:** Devanshu Agrawal (22CS30066)
