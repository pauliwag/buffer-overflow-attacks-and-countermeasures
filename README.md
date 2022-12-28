# Buffer-Overflow Attacks and Countermeasures

## Summary
These are my solutions to Dr. Du's SEED lab *Buffer-Overflow Attack (Server Version)*, for the purpose of preparing penetration-testing training materials for new QA engineer hires at a software development firm, so that they can better understand buffer-overflow vulnerabilities, attacks, and countermeasures.

## Setup
1. To set up the necessary VM environment: 
    1. Download the Ubuntu VM image from [DigitalOcean](https://seed.nyc3.cdn.digitaloceanspaces.com/SEED-Ubuntu20.04.zip). This will be a ZIP file—when unzipped, it will give you a VDI file which contains the Ubuntu 20.04 VM image. 
    2. Spin up a VM (type: Linux, version: 64-bit Ubuntu) using the aforementioned pre-built VM file (previous step) and a virtualization software of your choice (e.g., VirtualBox, VMWare, Parallels).
2. Disable address space layout randomization (ASLR) by executing `sudo /sbin/sysctl -w kernel.randomize_va_space=0`.
3. In the `server-code` directory, execute `make` followed by `make-install`.
4. Spin up the Docker containers from the root directory by:
    1. building the Docker image via `sudo docker-compose build`, and then
    2. instantiating the containers via `sudo docker-compose up`.
5. (Optional but useful) Open a new terminal tab and execute `sudo docker ps --format "{{.ID}} {{.Names}}"` to obtain the running container IDs.

## Attacks
The goal of the buffer-overflow attacks is to acquire a reverse shell on the target container. The `<TaskNum>-exploit.py` files in the `attack-code` directory each generate a `badfile` comprising the payload. You can spin up a Netcat server by executing `nc -nv -l 9090` in a new terminal tab, and then feed a `badfile` to the target server via (in the old tab) `cat badfile | nc <targetIP> 9090` to acquire a reverse shell in the new tab.

- [`Task2-exploit.py`](attack-code/Task2-exploit.py): Generates the payload for attacking a 32-bit `server-1` program *knowing the buffer size and address*.
- [`Task3-exploit.py`](attack-code/Task3-exploit.py): Generates the payload for attacking a 32-bit `server-2` program *knowing the range of the buffer size*.
- [`Task4-exploit.py`](attack-code/Task4-exploit.py): Generates the payload for attacking a 64-bit `server-3` program *knowing the buffer size and address*.
- [`Task5-exploit.py`](attack-code/Task5-exploit.py): Generates the payload for attacking a 64-bit `server-4` program *with a small buffer*.
- [`brute-force.sh`](attack-code/brute-force.sh): Executes a brute-force attack on `server-1` to demonstrate the insufficiency of the ASLR countermeasure.

## Countermeasures
- **ASLR** – Can be reenabled (after initially disabling it in [Setup step 2](#setup)) by executing `sudo /sbin/sysctl -w kernel.randomize_va_space=2`.
- **StackGuard** – Can be enabled by removing the `-fno-stack-protector` flag from the `gcc` flag in [`server-code/Makefile`](server-code/Makefile), enabling the detection and prevention of stack smashing.
- **Non-executable stack** – Can be enabled by removing the `-z execstack` flag from the `gcc` flag in [`shellcode/Makefile`](shellcode/Makefile), actuating segmentation faults to prevent the acquisition of reverse shells.

## Known Issues
If you are very unlucky with your execution of `brute-force.sh`, it can run long enough to completely fill your root logical volume (LV) `ubuntu-lv`, resulting in memory issues which pervade and hinder most of the critical commands/operations required for this training module. You can resolve this memory issue by reallocating free space on volume group `ubuntu-vg` to the aforementioned root LV (`ubuntu-lv`) by executing, as root,
1. `lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv` in the `lvm` CLI, which extends the LV to the maximum size usable, followed by
2. (after exiting the `lvm` CLI) `resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv`, which extends the root filesystem on top of increasing the size of the block volume where the filesystem resides.