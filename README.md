-----

# LeRobot Installation and SO-101 Setup Tutorial

This tutorial covers the installation of the LeRobot environment and the assembly, configuration, and teleoperation setup for the SO-101 robot, designed for users on **macOS**, **Windows**, and **Linux**.

-----

## 1\. Environment Setup

### Install Miniforge (All Platforms)

You'll use **Miniforge** and **conda** to manage the virtual environment.

1.  **Download and Install Miniforge:**

      * Find the latest installer for your operating system (macOS, Windows, Linux) on the [Miniforge GitHub releases page](https://github.com/conda-forge/miniforge/releases).
      * **On Linux/macOS:**
        ```bash
        [cite_start]wget "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh" [cite: 1]
        [cite_start]bash Miniforge3-$(uname)-$(uname -m).sh [cite: 1]
        ```
      * Follow the prompts to complete the installation.

2.  **Create and Activate the `lerobot` Environment:**

      * Create a conda environment with Python 3.10:
        ```bash
        [cite_start]conda create -y -n lerobot python=3.10 [cite: 1]
        ```
      * Activate the environment. **This must be done every time you open a new shell** to use LeRobot:
        ```bash
        [cite_start]conda activate lerobot [cite: 1]
        ```

3.  **Install FFmpeg:**

      * Install FFmpeg via conda. This is required for video encoding/decoding:
        ```bash
        [cite_start]conda install ffmpeg -c conda-forge [cite: 1]
        ```
      * *Note: This usually installs FFmpeg 7.X with `libsvtav1` encoder support.*
      * **Troubleshooting FFmpeg:** If you encounter issues or need a specific version, you can install it explicitly:
        ```bash
        [cite_start]conda install ffmpeg=7.1.1 -c conda-forge [cite: 2]
        ```

-----

## 2\. Install LeRobot Library

### Option A: Install from PyPI (Recommended for Users)

Install the core library and the necessary **Feetech** motor support extras for the SO-101 robot.

1.  **Install Core and Feetech Extras:**
    ```bash
    [cite_start]pip install 'lerobot[feetech]' [cite: 6]
    ```

### Option B: Install from Source (For Developers)

[cite_start]Use this option only if you plan to modify and contribute to the LeRobot code[cite: 4].

1.  **Clone the Repository:**
    ```bash
    [cite_start]git clone https://github.com/huggingface/lerobot.git [cite: 3]
    [cite_start]cd lerobot [cite: 3]
    ```
2.  **Install in Editable Mode with Feetech Extras:**
    ```bash
    [cite_start]pip install -e ".[feetech]" [cite: 3, 11]
    ```

### Optional Dependencies / Troubleshooting (Linux Example)

If you encounter build errors, you may need to install development dependencies.

  * **On Linux (Debian/Ubuntu):**
    ```bash
    [cite_start]sudo apt-get install cmake build-essential python3-dev pkg-config libavformat-dev libavcodec-dev libavdevice-dev libavutil-dev libswscale-dev libswresample-dev libavfilter-dev [cite: 8]
    ```
  * [cite_start]For other systems, refer to the documentation for "Compiling PyAV"[cite: 8].

-----

## 3\. SO-101 Robot Assembly and Configuration

### Assembly (Summary)

The SO-101 uses **Feetech STS3215 motors**. [cite_start]The follower arm uses 1/345 gearing, while the leader arm uses various gear ratios for different joints[cite: 17, 18].

  * [cite_start]**Parts Sourcing:** The bill of materials, including links and 3D printing instructions, is available in the associated README[cite: 14].
  * **Assembly Steps:**
    1.  [cite_start]Clean all support material from 3D-printed parts[cite: 47].
    2.  [cite_start]Assemble the joints sequentially, from Joint 1 (Base/Shoulder Pan) to Joint 6 (Gripper)[cite: 50, 66].
    3.  [cite_start]Use **M2x6mm** screws for fastening motors and **M3x6mm** screws for securing motor horns and attaching arm parts[cite: 50, 52].
    <!-- end list -->
      * [cite_start]*Note: It is advisable to install one 3-pin cable in the motor after placing them but before continuing assembly.* [cite: 49]

### Configure the Motors (Crucial Step)

[cite_start]The motors need unique IDs and a matching baudrate for proper communication[cite: 25, 27]. [cite_start]This is a one-time process as the parameters are written to the motors' internal memory (EEPROM)[cite: 29].

#### 3.1. Find the MotorBus USB Ports

1.  [cite_start]Connect the MotorBus (controller) to your computer via USB and power[cite: 20].
2.  Run the script:
    ```bash
    [cite_start]lerobot-find-port [cite: 21]
    ```
3.  The script will list available ports. [cite_start]Disconnect the MotorBus when prompted and press **Enter**[cite: 22].
4.  [cite_start]The script will identify the port associated with that MotorBus (e.g., `/dev/tty.usbmodem575E0032081`)[cite: 23]. Note this port for the next steps.
5.  [cite_start]Reconnect the USB cable[cite: 24].

#### 3.2. Set Motor IDs and Baudrates

[cite_start]You will set the ID and baudrate for each motor individually, following the script's instructions[cite: 37]. **Only one motor must be connected at a time during this process.**

**A. Follower Arm:**

1.  [cite_start]Connect the USB cable and power supply to the **follower arm's** controller board[cite: 33].
2.  Run the setup command, replacing the port with the one you found previously:
    ```bash
    lerobot-setup-motors \
        --robot.type=so101_follower \
        [cite_start]--robot.port=/dev/tty.usbmodem585A0076841  # <- REPLACE WITH YOUR PORT [cite: 34, 35]
    ```
3.  [cite_start]The script will prompt you to connect the first motor, starting with the **'gripper'** motor[cite: 36].
4.  Plug **only** the gripper's motor into the controller board. [cite_start]**Ensure no other motors are connected or daisy-chained**[cite: 37, 42].
5.  Press **Enter**. [cite_start]The script will set the ID and baudrate and prompt you for the next motor (e.g., 'wrist\_roll')[cite: 38, 39].
6.  [cite_start]Repeat for each motor as instructed[cite: 42].
7.  [cite_start]Once finished, the script will end, and the motors are ready[cite: 45].
8.  [cite_start]You can now daisy-chain the 3-pin cables between motors and connect the first motor (`shoulder pan` with id=1) to the controller board[cite: 46].

**B. Leader Arm:**

1.  Repeat the same steps for the **leader arm**, using its unique port and the following command:
    ```bash
    lerobot-setup-motors \
        --teleop.type=so101_leader \
        [cite_start]--teleop.port=/dev/tty.usbmodem575E0031751  # <- REPLACE WITH YOUR PORT [cite: 47]
    ```

-----

## 4\. Calibration

[cite_start]Calibration is essential to ensure both arms have matching position values when physically aligned, allowing trained neural networks to transfer across robots[cite: 67, 68].

### Follower Arm Calibration

1.  Run the calibration command (replace port and ID):
    ```bash
    lerobot-calibrate \
        --robot.type=so101_follower \
        [cite_start]--robot.port=/dev/tty.usbmodem58760431551 \ # <- The port of your follower arm [cite: 69]
        [cite_start]--robot.id=my_awesome_follower_arm # <- Give the robot a unique name [cite: 69]
    ```
2.  [cite_start]The script will first ask you to move the robot to the position where **all joints are in the middle of their ranges**[cite: 70].
3.  [cite_start]After pressing enter, you will have to move **each joint through its full range of motion**[cite: 71].

### Leader Arm Calibration

1.  Repeat the same steps for the leader arm:
    ```bash
    lerobot-calibrate \
        --teleop.type=so101_leader \
        [cite_start]--teleop.port=/dev/tty.usbmodem58760431551 \ # <- The port of your leader arm [cite: 72]
        [cite_start]--teleop.id=my_awesome_leader_arm # <- Give the robot a unique name [cite: 72]
    ```

-----

## 5\. Teleoperation

Once both arms are calibrated, you can begin the teleoperation step. [cite_start]This is used to control the robot and is the starting point for recording training data[cite: 77, 78].

  * [cite_start]If any calibration is missing, the command will automatically initiate the calibration procedure[cite: 85].
  * [cite_start]**Crucially, use the same IDs for the robot and teleop device as in the calibration step, as this is how the calibration files are linked**[cite: 83].

### Start Teleoperation

Run the following command, replacing the port and ID arguments with your specific values for the follower robot and the leader teleoperation device:

```bash
lerobot-teleoperate \
    --robot.type=so101_follower \
    [cite_start]--robot.port=/dev/tty.usbmodem58760431541 \ # Follower Port [cite: 84]
    [cite_start]--robot.id=my_awesome_follower_arm \ # Follower ID [cite: 84]
    --teleop.type=so101_leader \
    [cite_start]--teleop.port=/dev/tty.usbmodem58760431551 \ # Leader Port [cite: 84]
    [cite_start]--teleop.id=my_awesome_leader_arm # Leader ID [cite: 84]
```

[cite_start]This command connects the robot and teleop device and starts the teleoperation session[cite: 85].

-----
