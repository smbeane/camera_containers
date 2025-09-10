## Camera Snapshot Container

This repository contains configuration files for a systemd service that runs a podman container. The container uses ffmpeg to connect to an RTSP stream, periodically capture snapshots, and save them to a local directory on your machine.

## Files

- `camera.env`: This file stores all the necessary environment variables for your camera's connection details and the output directory.

- `camera.container`: This file is a systemd unit that defines the container service, including the image to use, volumes to mount, and the ffmpeg command to execute.

## Configuration

`camera.env`

Before you start, you must edit this file to match your camera's details and your desired output path.


    USER=username
    PASS=password
    IP=192.168.1.11
    PORT=554

    DIRECTORY=/images/snapshots
    SUB_DIRECTORY=camera_%%m-%%d-%%Y


- `USER` and `PASS`: Your camera's RTSP stream credentials.

- `IP` and `PORT`: The IP address and port of your camera's RTSP stream. The default port is usually 554.

- `DIRECTORY`: The absolute path on your host system where you want to save the images. **Ensure this directory exists.**

- `SUB_DIRECTORY`: This is a subfolder format based on date. The strftime variables create a new directory for each month, day, and year.

`camera.container`

In this file, change the file directory and name of the environment file as needed. Can also edit depending on needs of the current box. 

- `Image`: Uses the docker.io/linuxserver/ffmpeg image to perform the snapshot task.

- `Exec`: The core command that runs inside the container.

    - `-i rtsp://${USER}:${PASS}@${IP}:${PORT}`: The input stream URL, using the variables from camera.env.
    - `vf fps=1/600`: This is the frame rate filter. It captures one frame every 600 seconds (10 minutes). You can adjust this value to change the snapshot frequency. 
    - `-strftime_mkdir 1`: This part of the program may or may not work depending on compatability. If not, create a crontab that creates the directories at 12:00 AM each morning.
    - `"/output/${SUB\_DIRECTORY}/%%m-%%d-%%Y\_%%H-%%M-%%S.png"`: The output path and filename format for the snapshots.
    
## Usage
1. Clone the repository or download the files.
2. Edit the camera.env file with your specific camera and directory details.
3. Create the target directory on your host system, as specified by the DIRECTORY variable in camera.env.
5. Copy the files into the systemd container directory, 

    `/etc/containers/systemd`
6. Reload the systemd daemon to make it aware of the new service file 
    
    `sudo systemctl daemon-reload`

7. Enable and start the service 

    `sudo systemctl enable --now camera.container`

    > Note: This command enables the service to start automatically at boot and starts it immediately. You can check the status of the service at any time by running: `sudo systemctl status camera.container`


