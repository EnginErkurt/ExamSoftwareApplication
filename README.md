# Secure Exam Client

This project is an advanced desktop client application developed to ensure the security, fairness, and integrity of computer-based exams conducted in educational institutions and corporate environments. It monitors student activities on their devices, detects potential rule violations, and securely transmits all data to a central server.

## Key Features

Advanced and Secure Authentication
- Local Active Directory (AD) Integration: To prevent passwords from being transmitted over the network on Windows systems, local authentication is performed using the LogonUserW API via check_user.exe (a native tool written in C++).
- HMAC-SHA256 Tokenization: Following successful authentication, a unique token generated with HMAC-SHA256 is sent to the server instead of the actual password.
- Hybrid Support: When necessary, CATS (school student information system) and AD integration can work simultaneously.

Comprehensive System Monitoring Modules
The application includes independent modules that run continuously in the background to prevent cheating attempts and suspicious activities:
- Process Monitor: Monitors background applications using psutil. Instantly detects when pre-determined blacklisted applications are launched (e.g., browsers, messaging apps).
- Focused Window Monitor: Tracks which application and screen the student is actively using (Window Title, Process Path). Reports suspicious patterns such as "rapid app switching".
- Hardware Monitor: Checks whether a new USB device, monitor, or different disk is connected to the computer during the exam. Logs changes in Network Interfaces.
- Idle Monitor: Listens to the user's keyboard and mouse movements (via Windows GetLastInputInfo and Linux xprintidle) and sends an alert to the server when there is no activity for an extended period.

Automatic Screen Recording (Replay Recorder)
- FFmpeg Integration: Silently records the desktop screen in the background in MPEG-TS format (rolling segments) with low performance consumption.
- Incident-Based Evidence: In the event of any rule violation (incident), the current screen recording segments are merged and uploaded directly to the server as concrete evidence.

Offline Resilience and Buffering (Incident Buffer)
- The exam process does not stop in case of unexpected network or Wi-Fi interruptions. All incidents, logs, and system reports are securely and locally saved to the disk (data/client/{uuid}/buffer/).
- When the network connection is restored, all buffered data is sequentially transmitted to the server via WebSocketSession (using sequence numbers), preventing any data loss.

Dynamic Graphical User Interface (GUI)
- Hybrid GUI: Selects the most optimized interface based on the user's hardware. While providing a modern and advanced interface with PySide6 (Qt), it automatically falls back to the Tkinter interface if system dependency issues occur.
- Exam Countdown and Notifications: Displays the remaining time, exam status, and notifications from the server to the student using an "Always-on-top" widget.

Auto-Discovery
- Listens for UDP Multicast/Broadcast packets to find the exam server on the local network (LAN/Wi-Fi). Automatically connects to the server without requiring students to manually enter an IP address or Port number.

## Technologies Used

- Python 3.x: Main application, logic engine, WebSocket communication.
- C++ (Win32 API): Secure Active Directory authentication (check_user.cpp).
- PySide6 (Qt) / Tkinter: Graphical User Interfaces.
- FFmpeg: Continuous and segmented video recording engine.
- Network & Async: aiohttp, asyncio, websockets.
- System Analysis: psutil, ctypes (Windows API integrations).

## Installation and Usage

Requirements
For the project to run correctly, Python 3.x and the following libraries must be installed on the system:

    pip install aiohttp psutil PySide6

(Note: For the screen recording feature to work, ffmpeg must be present in the application's runtime environment, or its path must be specified via the EXAM_FFMPEG_PATH environment variable.)

Launching
The main launcher script client_launcher.py is used to start the client:

    # Launch by auto-selecting the UI engine (Qt first, fallback to Tkinter)
    python client_launcher.py --ui auto

    # Force launch with only the Qt interface
    python client_launcher.py --ui qt

    # Connect directly with test and debug parameters from the command line
    python -m client.main --login-id <USERNAME> --password <PASSWORD> --id <SERVER_ID>

## File and Folder Structure (Summary)

- client/auth_util/: AD integration and C++ native binary files.
- client/custommodules/: Independently running system monitoring modules (Hardware, Screen recording, Process monitor, etc.).
- client/ui/: Qt and Tkinter GUI source codes.
- client/exam.py & submission.py: Extraction of exam materials, file transfers, and uploading final packages (.zip) to the server.
- client/incident_buffer.py: Storing incidents in the local database during offline situations.
- client/main.py: The main file that initializes the application's network, discovery, and event loop center.

---
This software actively monitors the system only during the exam within the scope of student privacy policies and completely terminates recording mechanisms when the exam ends or the application is closed.
