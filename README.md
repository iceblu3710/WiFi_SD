# ESP32-S3 USB Drive with Web File Manager

## 1. Project Overview

### Summary
This project aims to create a custom USB gadget using the LilyGO T-Dongle S3 (ESP32-S3). The device will function as a standard USB flash drive (Mass Storage Device) when connected to a computer. Additionally, it will host a web-based file manager, accessible over the local Wi-Fi network, allowing users to upload, list, and manage files on an onboard SD card. A crucial aspect is implementing a mode-switching mechanism to prevent data corruption by ensuring that either the USB host or the web interface has exclusive access to the SD card's FAT32 filesystem at any given time.

### Goals
* Emulate a FAT32 USB Mass Storage Device using the ESP32-S3, making the SD card accessible as a standard flash drive to a host computer.
* Provide a web-based file manager interface (accessible via Wi-Fi) for:
    * Listing files and directories on the SD card.
    * Uploading new files to the SD card from the network.
    * Deleting files from the SD card.
* Implement a robust mode-switching mechanism that allows the user to explicitly switch between "USB Drive Mode" and "Web Management Mode" to prevent simultaneous filesystem access and potential data corruption.
* Utilize the LilyGO T-Dongle S3 development board and its onboard SD card slot.
* Provide clear user feedback regarding the current operational mode.

### Key Components
* **Hardware:**
    * LilyGO T-Dongle S3 (featuring ESP32-S3 microcontroller with Wi-Fi and USB OTG)
    * Micro SD Card (formatted as FAT32)
* **Software & Firmware Libraries/APIs:**
    * **ESP32-S3 Development Framework:** ESP-IDF or Arduino Core for ESP32.
    * **USB Mass Storage Class (MSC):** TinyUSB library for emulating a USB drive.
    * **SD Card Interaction:** Standard SD/SD_MMC library (for Arduino) or `esp_vfs_fat` and `sdmmc_host` (for ESP-IDF) to manage the FAT32 filesystem.
    * **Wi-Fi Connectivity:** `WiFi.h` (Arduino) or `esp_wifi.h` (ESP-IDF).
    * **Web Server:** `ESPAsyncWebServer.h` (Arduino recommended for file uploads) or `esp_http_server.h` (ESP-IDF).
    * **Mode Switching Logic:** Custom application logic to manage states and control access to USB MSC and SD card operations.
    * **HTML/CSS/JavaScript:** For the client-side web interface.

## 2. Development Roadmap

This roadmap outlines the major phases of the project, from initial setup to a refined, functional device.

1.  **Phase 1: Foundation & Core SD Card Functionality**
    * *Goal: Establish the development environment and ensure basic read/write access to the SD card.*
2.  **Phase 2: USB Mass Storage Emulation**
    * *Goal: Enable the ESP32-S3 to be recognized as a USB flash drive by a host PC, with access to the SD card.*
3.  **Phase 3: Basic Web Server & Wi-Fi Connectivity**
    * *Goal: Connect the ESP32-S3 to Wi-Fi and serve a simple web page.*
4.  **Phase 4: Web-Based File Listing**
    * *Goal: Display the contents of the SD card on a web page.*
5.  **Phase 5: Web-Based File Upload**
    * *Goal: Allow users to upload files to the SD card via the web interface.*
6.  **Phase 6: Mode Switching Implementation**
    * *Goal: Develop and integrate the logic to switch between USB Drive Mode and Web Management Mode.*
7.  **Phase 7: Web-Based File Deletion & Management**
    * *Goal: Add functionality to delete files (and optionally manage directories/download files) via the web interface.*
8.  **Phase 8: UI Refinement & Robust Error Handling**
    * *Goal: Improve the web interface usability and implement comprehensive error handling.*
9.  **Phase 9: Final Testing, Optimization & Documentation**
    * *Goal: Ensure stability, optimize performance, and document the project.*

## 3. Detailed Development Plan

### Phase 1: Foundation & Core SD Card Functionality

* **Task 1.1: Setup ESP32-S3 Development Environment**
    * Sub-task 1.1.1: Install and configure the ESP-IDF (Espressif IoT Development Framework) or the Arduino Core for ESP32.
    * Sub-task 1.1.2: Set up the toolchain for the ESP32-S3 chip.
    * Sub-task 1.1.3: Configure the development environment for the LilyGO T-Dongle S3 board, ensuring correct board definitions and pinouts are recognized.
    * Sub-task 1.1.4: Compile and upload a basic "Blink" example to the T-Dongle S3 to verify the toolchain, board communication, and basic operation.
* **Task 1.2: SD Card Initialization & Access**
    * Sub-task 1.2.1: Confirm the GPIO pins used by the T-Dongle S3's built-in MicroSD card slot (typically uses SDMMC peripheral).
    * Sub-task 1.2.2: Integrate the appropriate SD Card library (e.g., `SD_MMC.h` for Arduino, or `esp_vfs_fat` with `sdmmc_host` driver for ESP-IDF).
    * Sub-task 1.2.3: Write firmware to initialize the SD card in SDMMC mode (1-bit or 4-bit).
    * Sub-task 1.2.4: Implement checks for SD card presence, correct formatting (FAT32), and report errors to the Serial Monitor.
* **Task 1.3: Basic SD Card File Operations (via Serial Monitor)**
    * Sub-task 1.3.1: Write code to create a new test file (e.g., `test.txt`) on the SD card and write some sample data into it.
    * Sub-task 1.3.2: Write code to read the contents of the test file from the SD card and print it to the Serial Monitor.
    * Sub-task 1.3.3: Write code to list all files and directories in the root of the SD card, printing their names to the Serial Monitor.
    * Sub-task 1.3.4: Write code to delete the test file from the SD card.
    * Sub-task 1.3.5: Verify all operations through feedback on the Serial Monitor.

### Phase 2: USB Mass Storage Emulation

* **Task 2.1: Integrate TinyUSB Library**
    * Sub-task 2.1.1: Add the TinyUSB library to your project. (It's typically included with recent ESP-IDF versions and available for Arduino).
    * Sub-task 2.1.2: Familiarize yourself with TinyUSB Mass Storage Class (MSC) device examples, specifically for the ESP32-S3.
* **Task 2.2: Implement Basic MSC Device Functionality**
    * Sub-task 2.2.1: Configure TinyUSB to present the SD card as a single Logical Unit Number (LUN).
    * Sub-task 2.2.2: Implement the necessary TinyUSB MSC callbacks:
        * `tud_msc_inquiry_cb`: To provide device identification.
        * `tud_msc_capacity_cb`: To report the SD card's capacity.
        * `tud_msc_read10_cb`: To handle read requests from the host, reading data from the SD card.
        * `tud_msc_write10_cb`: To handle write requests from the host, writing data to the SD card.
        * `tud_msc_scsi_cb`: For other SCSI commands like `TEST UNIT READY`.
    * Sub-task 2.2.3: Ensure proper mapping of MSC read/write block operations to the SD card's sector read/write functions.
* **Task 2.3: Testing USB MSC Functionality**
    * Sub-task 2.3.1: Compile and upload the firmware with MSC enabled to the T-Dongle S3.
    * Sub-task 2.3.2: Connect the T-Dongle S3 to a host computer via its USB-C port (the one connected to the ESP32-S3's native USB pins).
    * Sub-task 2.3.3: Verify that the host computer recognizes the ESP32-S3 as a USB Mass Storage Device (e.g., a new drive letter appears).
    * Sub-task 2.3.4: Test reading files (created in Phase 1) from the emulated drive on the host PC.
    * Sub-task 2.3.5: Test writing new files and directories to the emulated drive from the host PC.
    * Sub-task 2.3.6: Verify (e.g., by re-listing via Serial Monitor or later via Web UI) that files written via USB are correctly stored on the SD card.
    * Sub-task 2.3.7: Test ejecting the drive from the host OS and re-connecting.

### Phase 3: Basic Web Server & Wi-Fi Connectivity

* **Task 3.1: Wi-Fi Connection Setup**
    * Sub-task 3.1.1: Integrate the Wi-Fi library (`WiFi.h` for Arduino, `esp_wifi.h` for ESP-IDF).
    * Sub-task 3.1.2: Implement code to connect the ESP32-S3 to your local Wi-Fi network using hardcoded SSID and password (for initial testing).
    * Sub-task 3.1.3: Implement code to obtain and print the ESP32-S3's IP address to the Serial Monitor upon successful connection.
    * Sub-task 3.1.4: Implement Wi-Fi reconnection logic in case of network dropout.
* **Task 3.2: Basic Web Server Implementation**
    * Sub-task 3.2.1: Integrate a web server library (Recommended: `ESPAsyncWebServer.h` for Arduino due to its efficient handling of asynchronous requests and file uploads; or `esp_http_server.h` for ESP-IDF).
    * Sub-task 3.2.2: Implement a simple request handler for the root URL ("/").
    * Sub-task 3.2.3: Have this handler serve a basic "Hello World" HTML page or a project status page.
    * Sub-task 3.2.4: Start the web server after a successful Wi-Fi connection.
    * Sub-task 3.2.5: Test accessing the served web page from a browser on a device connected to the same network, using the ESP32-S3's IP address.

### Phase 4: Web-Based File Listing

* **Task 4.1: Design Basic HTML Structure for File Listing**
    * Sub-task 4.1.1: Create an HTML structure (e.g., using a table or an unordered list) to display filenames, file sizes, and types (file/directory).
    * Sub-task 4.1.2: Include placeholders for future action buttons (e.g., delete, download).
    * Sub-task 4.1.3: Consider how to handle directory navigation (initially, list root directory).
* **Task 4.2: Implement Web Server Endpoint for Dynamic File Listing**
    * Sub-task 4.2.1: Create a new web server route (e.g., `/listfiles` or handle on root `/`).
    * Sub-task 4.2.2: In the request handler for this route:
        * Access the SD card using the file system functions developed in Phase 1.
        * Read the contents of the current directory (initially root).
        * Iterate through the files and directories.
        * Dynamically generate HTML output that populates the structure designed in Task 4.1 with the actual file information.
* **Task 4.3: Test Web-Based File Listing**
    * Sub-task 4.3.1: Ensure that files and directories previously created on the SD card (either via USB or test code) are correctly listed on the web page.
    * Sub-task 4.3.2: Test with an empty SD card or empty directories.
    * Sub-task 4.3.3: Test with a mix of files and subdirectories.

### Phase 5: Web-Based File Upload

* **Task 5.1: Design HTML Form for File Upload**
    * Sub-task 5.1.1: Add an HTML form to your web page with:
        * An `<input type="file" name="filetoupload">` element.
        * A submit button (`<input type="submit" value="Upload">`).
    * Sub-task 5.1.2: Set the form's `method` attribute to `POST` and `enctype` attribute to `multipart/form-data`.
    * Sub-task 5.1.3: Define the `action` attribute of the form to point to the upload handling endpoint (e.g., `/upload`).
* **Task 5.2: Implement Web Server Endpoint for File Upload Handling**
    * Sub-task 5.2.1: Create a new web server route (e.g., `/upload`) that accepts POST requests.
    * Sub-task 5.2.2: If using `ESPAsyncWebServer`, configure a file upload handler (e.g., using `onFileUpload`). This handler will be called when a file is received.
    * Sub-task 5.2.3: Inside the upload handler:
        * Get the uploaded filename and file data.
        * Open a new file on the SD card with the received filename.
        * Write the received data chunks to the file on the SD card.
        * Close the file once the upload is complete.
    * Sub-task 5.2.4: Implement basic error handling (e.g., SD card write errors, SD card full).
    * Sub-task 5.2.5: Redirect the user back to the file listing page or provide a success/failure message after the upload attempt.
* **Task 5.3: Test File Upload Functionality**
    * Sub-task 5.3.1: Attempt to upload various small file types (e.g., .txt, .jpg) through the web interface.
    * Sub-task 5.3.2: Verify that the uploaded files are correctly saved on the SD card (e.g., by checking the file list on the web page or via USB MSC mode if implemented carefully).
    * Sub-task 5.3.3: Test behavior when trying to upload a file with an existing name (decide on overwrite policy or error).
    * Sub-task 5.3.4: Test upload limits and behavior with larger files (be mindful of ESP32 RAM constraints for buffering; ESPAsyncWebServer handles this well by streaming).

### Phase 6: Mode Switching Implementation

* **Task 6.1: Define Operational Modes and Control Variables**
    * Sub-task 6.1.1: Define two distinct modes: `USB_MSC_MODE` and `WEB_UI_MODE`.
    * Sub-task 6.1.2: Implement a global state variable (e.g., `current_mode`) to track the active mode.
    * Sub-task 6.1.3: Set a default mode on boot-up (e.g., `USB_MSC_MODE`).
* **Task 6.2: Implement Functions to Enable/Disable USB MSC**
    * Sub-task 6.2.1: Create a function `enable_usb_msc()`:
        * This function will initialize and start the TinyUSB MSC stack, making the SD card available to the host PC.
    * Sub-task 6.2.2: Create a function `disable_usb_msc()`:
        * This function must safely stop or de-initialize the TinyUSB MSC stack. This might involve calling a TinyUSB disconnect function or stopping its task and ensuring the USB peripheral is reset or idle. This is critical to prevent host OS errors.
        * Ensure any SD card resources are released if exclusively used by MSC.
* **Task 6.3: Implement Web UI Controls for Mode Switching**
    * Sub-task 6.3.1: On the web interface, add a button: "Switch to USB Drive Mode".
        * When clicked, this button should send a request to an ESP32 endpoint (e.g., `/enable_usb`).
        * The handler for `/enable_usb` should:
            1.  Set `current_mode = USB_MSC_MODE`.
            2.  Call `enable_usb_msc()`.
            3.  Potentially disable web-based file modification operations.
            4.  Update the web page to indicate it's now in USB mode (or the page might become unresponsive if the server is heavily impacted).
    * Sub-task 6.3.2: On the web interface, add a button: "Switch to Web Management Mode". (This button should ideally only be active/visible if the device is in USB_MSC_MODE, or it's the action to take control from USB).
        * When clicked, this button sends a request to an ESP32 endpoint (e.g., `/enable_web`).
        * The handler for `/enable_web` should:
            1.  Call `disable_usb_msc()`. This "ejects" the drive from the host.
            2.  Set `current_mode = WEB_UI_MODE`.
            3.  Enable full web-based file operations.
            4.  Update the web page to indicate it's in Web Management mode.
* **Task 6.4: Enforce Mutually Exclusive SD Card Access**
    * Sub-task 6.4.1: Before any SD card operation (read, write, list) via the web server, check if `current_mode == WEB_UI_MODE`. If not, disallow or return an error.
    * Sub-task 6.4.2: Ensure that when `current_mode == USB_MSC_MODE`, the web server does not attempt to modify SD card contents. Listing might be permissible if implemented carefully, but write operations must be blocked.
    * Sub-task 6.4.3: Ensure that before switching from `WEB_UI_MODE` to `USB_MSC_MODE`, all file handles opened by the web server are closed and data is flushed to the SD card.
* **Task 6.5: Test Mode Switching Robustness**
    * Sub-task 6.5.1: Connect to PC. Verify USB drive appears when switched to `USB_MSC_MODE`.
    * Sub-task 6.5.2: Switch to `WEB_UI_MODE`. Verify USB drive "ejects" or becomes inaccessible to the PC. Verify web UI file operations are now enabled.
    * Sub-task 6.5.3: Upload a file via web, then switch to `USB_MSC_MODE`. Verify the file is accessible via PC.
    * Sub-task 6.5.4: Write a file via PC, then switch to `WEB_UI_MODE`. Verify the new file is listed on the web UI.
    * Sub-task 6.5.5: Test for data integrity after multiple rapid switches. Test behavior if switching while a file transfer (USB or Web) is hypothetically in progress (should be prevented by UI design).

### Phase 7: Web-Based File Deletion & Management

* **Task 7.1: Implement File Deletion via Web Interface**
    * Sub-task 7.1.1: In the file listing on the web page (from Phase 4), add a "Delete" button or link next to each file.
    * Sub-task 7.1.2: Make these buttons trigger a request (e.g., a POST or GET request with the filename as a parameter) to a new endpoint (e.g., `/deletefile`).
    * Sub-task 7.1.3: Implement the handler for `/deletefile`:
        * Ensure `current_mode == WEB_UI_MODE`.
        * Extract the filename to be deleted from the request.
        * Use the SD card library's function to delete the specified file (e.g., `SD.remove(filename)`).
        * Provide feedback to the user (e.g., redirect to updated file list, show success/error message).
* **Task 7.2: (Optional) Implement Directory Creation**
    * Sub-task 7.2.1: Add a form/button to the web UI to input a new directory name.
    * Sub-task 7.2.2: Create an endpoint (e.g., `/createdir`) to handle this request.
    * Sub-task 7.2.3: Use `SD.mkdir(dirname)` in the handler.
* **Task 7.3: (Optional) Implement File Download**
    * Sub-task 7.3.1: Add "Download" links next to files in the web UI.
    * Sub-task 7.3.2: Create an endpoint (e.g., `/downloadfile`) that takes a filename parameter.
    * Sub-task 7.3.3: In the handler, set appropriate HTTP headers (`Content-Disposition: attachment; filename="<filename>"`, `Content-Type: application/octet-stream`).
    * Sub-task 7.3.4: Stream the file content from the SD card to the HTTP response.
* **Task 7.4: Test Enhanced File Management Features**
    * Sub-task 7.4.1: Test deleting various files and verify they are removed from the SD card and the web listing.
    * Sub-task 7.4.2: If directory creation is implemented, test creating and navigating into/out of directories.
    * Sub-task 7.4.3: If downloads are implemented, test downloading files and verify their integrity.

### Phase 8: UI Refinement & Robust Error Handling

* **Task 8.1: Improve Web Interface Aesthetics and Usability**
    * Sub-task 8.1.1: Apply basic CSS styling to make the web interface more visually appealing and user-friendly.
    * Sub-task 8.1.2: Ensure responsive design for different screen sizes (if a priority).
    * Sub-task 8.1.3: Improve navigation (e.g., breadcrumbs for directory Browse).
    * Sub-task 8.1.4: Consider embedding CSS/JS into firmware or serving from PROGMEM to save filesystem space if not using SPIFFS/LittleFS for web assets.
* **Task 8.2: Implement Comprehensive Error Handling**
    * Sub-task 8.2.1: For all SD card operations (init, read, write, delete, list, mkdir): check return codes and handle potential errors (e.g., card not present, card full, file not found, permission denied).
    * Sub-task 8.2.2: For Wi-Fi operations: handle connection failures, disconnections.
    * Sub-task 8.2.3: For web server operations: handle invalid requests, upload errors.
    * Sub-task 8.2.4: Display clear, user-friendly error messages on the web interface instead of crashing or showing cryptic codes.
    * Sub-task 8.2.5: Log detailed errors to the Serial Monitor for debugging.
* **Task 8.3: Enhance User Feedback and Status Indicators**
    * Sub-task 8.3.1: Clearly display the current operational mode (`USB_MSC_MODE` or `WEB_UI_MODE`) on the web page.
    * Sub-task 8.3.2: Provide visual feedback during file uploads (e.g., a simple "Uploading..." message, or a progress bar if implementing more advanced client-side JavaScript).
    * Sub-task 8.3.3: Show confirmation messages for successful actions (e.g., "File uploaded successfully," "File deleted," "Mode switched").
    * Sub-task 8.3.4: Indicate SD card free/total space.

### Phase 9: Final Testing, Optimization & Documentation

* **Task 9.1: Comprehensive End-to-End System Testing**
    * Sub-task 9.1.1: Test all functionalities (Wi-Fi connect, web UI access, list, upload, delete, mode switch, USB MSC access) across different web browsers (Chrome, Firefox, Edge) and operating systems (Windows, macOS, Linux for USB access).
    * Sub-task 9.1.2: Test with various SD card sizes (e.g., 4GB, 16GB, 32GB) and states (nearly empty, nearly full, some fragmentation).
    * Sub-task 9.1.3: Perform stress testing: multiple quick operations, repeated mode switches, uploading multiple files sequentially.
    * Sub-task 9.1.4: Test for long-term stability: leave the device running for an extended period in both modes and check for crashes or performance degradation.
* **Task 9.2: Code Cleanup, Refactoring, and Optimization**
    * Sub-task 9.2.1: Review and refactor code for clarity, maintainability, and efficiency.
    * Sub-task 9.2.2: Remove any unused variables, functions, or debug print statements (that are not for intentional logging).
    * Sub-task 9.2.3: Optimize for memory usage (RAM and flash) if close to limits. Check for memory leaks.
    * Sub-task 9.2.4: Optimize web asset sizes if served from ESP32.
* **Task 9.3: Project Documentation**
    * Sub-task 9.3.1: Ensure code is well-commented, explaining complex sections.
    * Sub-task 9.3.2: Update this Markdown document to reflect the final state of the project if there were deviations.
    * Sub-task 9.3.3: Create a brief user guide:
        * How to connect the device to Wi-Fi (if not hardcoded, e.g., if a captive portal or serial config is added).
        * How to access the web interface.
        * How to switch between USB Drive Mode and Web Management Mode.
        * Troubleshooting common issues.
* **Task 9.4: (Optional) Persistent Configuration**
    * Sub-task 9.4.1: If desired, implement saving Wi-Fi credentials to NVS (Non-Volatile Storage) on the ESP32, so they don't need to be hardcoded.
    * Sub-task 9.4.2: Consider implementing a captive portal or a configuration mode (e.g., via Serial) for initial Wi-Fi setup if credentials are not hardcoded.
