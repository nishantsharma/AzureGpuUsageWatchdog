This project can be used to watch usage of any costly Azure VM and shut it down when necessary.
If the CPU is not in use, the Azure Ubuntu instance where it is running is automatically shut down
and deallocated.

The intention is to save money  on unused cloud VMs.

Pre Conditions:
    These settings have only been tested on Ubuntu Azure VMs although other Linuxen
    should work similarily.

VM Configuration Steps:
    Install Azure CLI on the VM by following the steps below.
        https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-apt?view=azure-cli-latest

    Install pip
        sudo apt install python3-pip
    Install python modules for Azure commands on the VM by following the steps below.
        sudo pip install -r requirements.txt

    Enable managed service identity for the VM in question.
        Go to https://portal.azure.com and select the VM.
        Clink on Identity in the settings section.
        Make sure that the status is set to ON.
        For details, check 
            https://docs.microsoft.com/en-us/azure/active-directory/managed-service-identity/tutorial-linux-vm-access-arm

    Grant your VM access/ability to shutdown or reboot itself.
        Go to portal.azure.com and select the VM.
        Click access control(IAM).
        Click Add
            Select the role to "DevTest Labs User".
            Select Assign access to "Virtual Machine".
            Select the Virtual machine applicable.

    To ensure that the machine does not shut itself down when a user is active on command line,
    add following lines to /etc/bash.bashrc
        PROMPT_COMMAND="touch /var/run/MachineUsageWatchdog;$PROMPT_COMMAND"
    This ensures that whenever bash console is used, the shutdown by watchdog is postponed.

    Copy the project folder(MachineUsageWatchdog) to /usr/local/src.
        git clone nisshar@vs-ssh.visualstudio.com:v3/nisshar/Hier2Hier/Hier2Hier
        sudo cp src/MachineUsageWatchdog to /usr/local/src/

    Update machineUsageWatchdogConfig in MachineUsageWatchdog.py
        sudo vi /usr/local/src/MachineUsageWatchdog.py

        # Add following lines before the definition of first function.
        machineUsageWatchdogConfig = AttrDict({})
        machineUsageWatchdogConfig.vm_name = "gpu-vm"
        machineUsageWatchdogConfig.resource_group_name = "learn"

    Configure for the MachineUsageWatchdog to be run at startup.
    Add following lines to /etc/rc.local
        # Create the watchdog file which triggers shutdown.
        touch /var/run/MachineUsageWatchdog
        chmod uog+tw /var/run/MachineUsageWatchdog
        nohup python3 /usr/local/src/MachineUsageWatchdog/MachineUsageWatchdog.py &

    For processes we want to run in background, install stayAliveForCmd.sh and stayAliveForMinutes.sh in /usr/local/bin/.
    Also install stayAliveForMinutes.sh in /usr/local/bin/.
        sudo ln -s //usr/local/src/*.sh /usr/local/bin/

Other references
    https://docs.microsoft.com/en-us/azure/azure-stack/azure-stack-create-service-principals
    https://stackoverflow.com/questions/44648067/deallocate-a-ubuntu-vm-from-itself

