# WiFi Lessons: Cracking WiFi Passwords with Brucegotchi and `handshakeCracker`

Welcome to WiFi Lessons! This guide will walk you through the process of capturing WiFi handshakes using your Brucegotchi (running [Bruce Firmware](https://github.com/pr3y/Bruce)) and then cracking those handshakes to reveal WiFi passwords using the [handshakeCracker](https://github.com/Mysteriza/handshakeCracker) tool.

**Important Disclaimer:** This tutorial is for educational and personal network security testing purposes only. **Using these methods on networks you do not own, or without explicit written permission from the network owner, is illegal and unethical. Always act responsibly and within legal boundaries.**

## Table of Contents

* [1. Preliminary Requirements](#1-preliminary-requirements)
* [2. Capturing Handshakes with Brucegotchi](#2-capturing-handshakes-with-brucegotchi)
* [3. Transferring Handshake Files to Your PC](#3-transferring-handshake-files-to-your-pc)
* [4. Cracking Handshakes with `handshakeCracker`](#4-cracking-handshakes-with-handshakecracker)
    * [Setting Up WSL Kali Linux](#setting-up-wsl-kali-linux)
    * [Installing `handshakeCracker` and Dependencies](#installing-handshakecracker-and-dependencies)
    * [Moving Handshake Files to Kali Linux](#moving-handshake-files-to-kali-linux)
    * [Running the Cracking Process](#running-the-cracking-process)
* [5. Understanding Cracking Results & Troubleshooting](#5-understanding-cracking-results--troubleshooting)

---

## 1. Preliminary Requirements

Before we dive in, please ensure you have the following prerequisites covered. This guide will **not** cover their installation, and you are expected to set them up yourself:

1.  **A PC with WSL Kali Linux installed on Windows, or Kali Linux installed in a Virtual Machine** (e.g., VMWare or VirtualBox). This tutorial will specifically use WSL Kali Linux. [How to install WSL Kali Linux on Windows](https://www.youtube.com/results?search_query=how+to+install+wsl+kali+linux+on+windows+11).
2.  **A device with Bruce Firmware installed** (e.g., Lilygo, M5Stack, CYD, ESP32). [How to flash Bruce Firmware](https://bruce.computer/flasher).

If you meet these requirements, you're ready to proceed! Take your time to set up WSL Kali Linux if you haven't already; it's a crucial step. This tutorial reflects my personal workflow, but once you grasp the core concepts, feel free to adapt it to your preferred, faster, or easier methods.

## 2. Capturing Handshakes with Brucegotchi

In this stage, we'll use your Brucegotchi device to capture WiFi handshakes.

**What is a WiFi Handshake?**
In the context of WiFi security, a "handshake" is a sequence of messages exchanged between a WiFi client (like your phone) and a router (Access Point or AP) when the client attempts to connect to a WPA/WPA2/WPA3-secured network. This handshake contains cryptographic information that, while not the password itself, can be used to verify if a guessed password is correct. Capturing this handshake is the first step in trying to uncover the WiFi password.

**Steps:**

1.  **Power On Your Brucegotchi:** Turn on your device with Bruce Firmware installed.
2.  **Navigate to Brucegotchi Mode:** On the Bruce Firmware menu, go to the "WiFi" menu, scroll until you find the "Brucegotchi" option, and select it.
3.  **Automatic Operation:** Once you've entered Brucegotchi mode, your device will automatically begin scanning for networks, performing deauthentication attacks (to force clients to re-authenticate and generate handshakes), and capturing WiFi handshakes. You don't need to do anything further; it works autonomously.
4.  **Monitor Handshake Count:** Look at the top of your device's screen for the "HS" indicator. This shows the current number of handshakes captured. A higher number means more handshakes obtained. Ensure your device is within the target WiFi's range.
![20250706_113840](https://github.com/user-attachments/assets/c16b4c08-7300-4f29-ba0b-b342063bb86e)

6.  **Collect Sufficient Handshakes:** When you've gathered enough handshakes (more than one is ideal), you're ready to transfer these `.pcap` files to your PC for cracking.

## 3. Transferring Handshake Files to Your PC

Now, let's move the captured handshake files from your device to your PC running Kali Linux. The quickest and easiest method (for me) is using Mass Storage. If your device doesn't use an SD card, you might need to use a WebUI or other file transfer methods provided by your Bruce Firmware.

**Steps:**

1.  **Connect Device to PC:** Connect your Brucegotchi device to your PC using a USB cable.
2.  **Enable Mass Storage:** On your Bruce Firmware device, navigate to the "Files" menu, then find and select the "Mass Storage" option.
3.  **Access SD Card:** On your PC, your device's SD card will appear as a removable drive. Open it and navigate to the `BrucePCAP` folder. Inside, you'll find the `handshakes` folder, which contains all your captured `.pcap` files.
4.  **Copy Files to Local Disk:** Select all the `.pcap` files and copy them to a readily accessible location on your PC's local disk (e.g., `C:\CapturedHandshakes`).
5.  **Disconnect Device:** Once the files are copied, you can safely disconnect Mass Storage from your device. We will now focus entirely on your PC.

## 4. Cracking Handshakes with `handshakeCracker`

This section details the cracking process using WSL Kali Linux and `handshakeCracker`.

### Setting Up WSL Kali Linux

1.  **Open WSL Kali Linux:** Launch your WSL Kali Linux terminal. You'll be greeted with a command-line interface.
2.  **Update and Install Dependencies:** In the terminal, execute these commands one by one to update your system and install necessary tools:
    ```
    sudo apt update
    ```
    ```
    sudo apt install aircrack-ng -y
    ```
    ```
    sudo apt install python3 python3-venv -y
    ```

### Installing `handshakeCracker` and Dependencies

1.  **Clone the Repository:** From your Kali Linux terminal, clone the `handshakeCracker` repository:
    ```bash
    git clone https://github.com/Mysteriza/handshakeCracker
    ```
2.  **Navigate to Tool Directory:**
    ```bash
    cd handshakeCracker
    ```
3.  **Create and Activate Virtual Environment:** To isolate the project's Python dependencies, create and activate a virtual environment:
    ```
    python3 -m venv venv
    ```
    ```
    source venv/bin/activate
    ```
    You will see `(venv)` appear at the beginning of your terminal prompt, indicating the virtual environment is active.

### Moving Handshake Files to Kali Linux

1.  **Access WSL Directory from Windows:** On your Windows PC, open File Explorer. In the address bar, type `\\wsl$` and press Enter. This will show your WSL distributions. Navigate into your Kali Linux distribution (e.g., `Kali-Linux`), then `home`, then your Linux username, and finally the `handshakeCracker` directory you just cloned.
2.  **Create `handshakes` Folder:** Inside the `handshakeCracker` directory in your WSL Kali Linux file system, create a new folder named `handshakes`.
3.  **Transfer `.pcap` Files:** Copy all the `.pcap` files you previously saved on your Windows local disk (e.g., `C:\CapturedHandshakes`) into this newly created `handshakes` folder within your WSL Kali Linux `handshakeCracker` directory.

### Running the Cracking Process

1.  **Return to Terminal:** Go back to your WSL Kali Linux terminal.
2.  **Run `crack_handshake.py`:** Execute the main cracking script:
    ```bash
    python crack_handshake.py
    ```
    The tool will automatically install any remaining required Python libraries. If all installations are successful, it will then launch the program.
3.  **Follow the Prompts:** The program will guide you through the process:
    * **Choose Input Mode:**
        * Enter `0` for `Auto` to automatically scan the `handshakes/` directory for all `.cap`/`.pcap` files. **(Choose this option, as the program will automatically crack all handshakes located in your `handshakes/` folder).**
        * Enter `1` for `Manual` to input handshake file paths one by one.
        * Enter `3` to `Exit` the program.
    * **Enter Handshake Paths (Manual Mode):** If you selected Manual mode, you will be prompted to enter handshake file paths one by one. Use `TAB` for auto-completion. Type `done` or `q` when you have finished adding files.

The program will then process the handshakes in the queue, displaying its progress and any results.
![Screenshot 2025-07-06 113001](https://github.com/user-attachments/assets/775d518d-2ba3-481f-80c2-05232dd73523)


## 5. Understanding Cracking Results & Troubleshooting

The cracking process is now complete. If a match is found between a handshake and your wordlist, the correct password will be displayed! And voilà, you've successfully cracked a handshake.

However, if no WiFi password is found, there are typically two main possibilities:

1.  **Invalid or Incomplete Handshake File:** It's normal for Brucegotchi, being a compact device, to sometimes capture incomplete or invalid handshakes. Not every captured handshake will be crackable.
2.  **Wordlist Does Not Contain the Password:** This is also common, especially if the WiFi password is complex or unique. The success of cracking highly relies on the quality and comprehensiveness of your wordlist.

Remember, this process does not guarantee a 100% success rate in obtaining your desired WiFi password, as it's highly dependent on your wordlist.
