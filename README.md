My First Computer Vision Project: A CCTV/YOLO Tracking Log

This project documents my journey in setting up a real-time object detection and tracking system. The goal was to learn the fundamentals of video surveillance, from accessing a video stream to applying AI models (like YOLO) to track objects.

This log details the real-world setup process, the problems I encountered, and the solutions I implemented.

Core Technologies Used

OS: Kali Linux (running in a VirtualBox VM)

Python: 3.13

Core AI Models: YOLOv3 / YOLOv8

Core Libraries: OpenCV, Ultralytics, pip

Environment: Python Virtual Environments (venv)

Source Project: PhazerTech/yolo-rtsp-security-cam

The Project: A Step-by-Step Troubleshooting Log

My goal was to get the yolo-rtsp-security-cam project running, which is designed to detect and track objects from a CCTV stream. Here is the process I followed.

Step 1: Initial Setup & The First Wall

My first step was to clone the project and install its dependencies.

# Clone the project
git clone [https://github.com/PhazerTech/yolo-rtsp-security-cam.git](https://github.com/PhazerTech/yolo-rtsp-security-cam.git)
cd yolo-rtsp-security-cam

# Attempt to install requirements
pip install -r requirements.txt


Problem: This failed immediately with the error:

error: externally-managed-environment

Investigation & Solution:
I learned that modern Kali Linux (and other Debian-based systems) protect the "system" Python from being modified by pip. This is a safety feature (per PEP 668) to prevent breaking the operating system's own tools.

The correct, professional solution is to create an isolated virtual environment (venv) for the project.

The Fix:

# Create a virtual environment folder named "venv"
python3 -m venv venv

# Activate the environment (the "sandbox")
source venv/bin/activate

# Now, install the packages *inside* the sandbox
pip install -r requirements.txt


Step 2: The "Out of Space" Error

With the venv active, I tried the installation again. It started downloading large packages like torch and opencv, but then failed with a new error.

Problem:

pip._vendor.urllib3.exceptions.ProtocolError: ("Connection broken: OSError(28, 'No space left on device')")

Investigation & Solution:
This seemed impossible, as my VM has 56GB of free space.

I ran df -h to check my disk usage.

I discovered that my main hard drive (/dev/sda1) was fine (26% full), but my /tmp (temporary) folder was a tmpfs (a RAM disk) limited to only 987MB.

pip was downloading the multi-gigabyte packages to this tiny /tmp folder in RAM, filling it up, and crashing.

The Fix:
I needed to tell pip to use a temporary folder on my main hard drive, which had plenty of space.

# 1. Create a new temp folder inside my project
mkdir build_temp

# 2. Re-run the install, forcing it to use my new temp folder
TMPDIR=./build_temp pip install -r requirements.txt


This worked, and all packages installed successfully.

Step 3: The Connection & "Zero" Error

With everything installed, I tried to run the project on a public test stream.

python3 yolo-rtsp-security-cam.py --stream "rtsp://..." --yolo person --monitor


Problem: The script immediately crashed with two errors.

[tcp @ ...] Connection refused

ZeroDivisionError: float division by zero

Investigation & Solution:

The Connection refused error told me the problem wasn't my code; the public test stream was offline or blocking my connection (perhaps due to my VPN).

The ZeroDivisionError was just a symptom. The script failed to get a video, so its fps (frames per second) variable was 0. When the code tried to calculate 1 / fps, it was trying to divide by zero, which caused the crash.

The Fix:
Instead of relying on an unstable internet stream, I downloaded a local video file to test with.

# 1. Download a stable, local test video
wget [https://github.com/intel-iot-devkit/sample-videos/raw/master/person-bicycle-car-detection.mp4](https://github.com/intel-iot-devkit/sample-videos/raw/master/person-bicycle-car-detection.mp4)

# 2. Run the script on the local file
python3 yolo-rtsp-security-cam.py --stream person-bicycle-car-detection.mp4 --yolo person,car,bicycle --monitor


Final Result: Success!

The script launched, a window appeared, and I could see the AI successfully tracking objects in the video ("a person walking on a bike").

I also noted that the project's code has a minor bug. It throws an FFmpegError when the video finishes, because it tries to stop a recording process that isn't running. This is a script bug, not a setup failure.

What I Learned

This project was a success. I learned that setting up an AI project is rarely a simple "run and done" process. I successfully diagnosed and solved:

A Python environment conflict (the venv issue).

A Linux system configuration issue (the /tmp RAM disk).

A network/input error (the dead RTSP stream).

This troubleshooting process is the real skill of a developer.
