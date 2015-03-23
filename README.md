# coreos-launch: Create a CoreOS cluster on KVM

This script is designed to simplify the process of launching a CoreOS cluster with bridged networking on Linux/KVM. It reads configuration from a YAML file describing the cluster, and launches each machine within a ```screen``` session giving access to the console if required.

## Requirements

 * You'll need a bridge interface (e.g. ```br0```) configured against your host's primary network interface, for the script to connect ```tap``` devices to (one per virtual machine). See your host distribution's documentation for details on how to set up a bridge.
 * You'll need the following commands available on the host:
   - ```/usr/bin/brctl```
   - ```/sbin/ifconfig```
   - ```/sbin/ip```
   - ```/usr/bin/qemu-system-x86_64```
   - ```/usr/bin/screen```
   - ```/usr/bin/sudo```
   - ```/usr/sbin/tunctl```
 * Unless you want to be running everything as root (which you don't), you'll want the script to be able to launch several privileged commands without requiring interactive authentication. This can be achieved using ```sudo```:
   - Add each user who will be using the ```coreos-launch``` script to the ```kvm``` group
   - Using ```visudo```, edit the ```sudo``` configuration to allow these users access to certain commands. For example, add the following lines:

```Cmnd_Alias KVM = /usr/sbin/tunctl, /sbin/ifconfig, /usr/sbin/brctl, /sbin/ip```
```%kvm ALL=(ALL) NOPASSWD: KVM```

Note that the position in the file of the ```%kvm``` line above matters! If you already have existing group rules, e.g. for ```wheel```, make sure you add this line *after* or it will not work.
 * You'll obviously need a CoreOS image of your choice, which you can download from the [CoreOS website](https://coreos.com/docs/running-coreos/platforms/qemu/) (ignore the wrapper script and just download the image)
 * Python 2.x, and the PyYAML module installed

## Usage

One all requirements are met, usage is rather straightforward:

```$ ./bin/coreos-launch etc/coreos-launch.yaml```

For information on what the YAML configuration file should look like, see the example. The script will then attempt to launch each machine as defined in the configuration, forking and attaching the machine's console to a ```screen``` session with the same name as the machine. When this screen session exits, the script will then clean up (bring down the machine's tap interface, etc). Note that halting CoreOS is not sufficient to end the screen session, but rather QEMU itself must be killed. How to use screen/QEMU is outside the scope of this document, but to save you some time:

 * Connect to the screen session with ```screen -r <machine_name>```
 * Press ```Ctrl+a``` then ```a``` to send a "Ctrl+a" escape sequence past screen and through to QEMU
 * Press ```x``` to shut down the virtual machine ("Ctrl+a" then "x" is QEMUs sequence for killing a machine)

Using the QEMU console from within a screen session is a bit tricky due to all the escape sequences, but you'll get used to it. You shouldn't be spending much time at the virtual console anyway. Enjoy!
