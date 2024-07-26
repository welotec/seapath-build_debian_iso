# SEAPATH installation media

This repository contains customizations geared towards the realtime capabilities of
Welotecs RSAPC MK2 platform with the 8C/16T configuration. We also provide an installation image.
> [!WARNING]  
> The image contains default passwords AND ssh keys. Make sure none of them end up on a production device
> see section below for instructions how to change either of these.


## How to use the Image

Download and verify the latest image from the release section of this repository.
If you're on Linux simply use dd to prepare a USB thumb drive for booting.
As root do the following.

```bash
dd if=~/Downloads/seapath.iso of=/dev/sdX bs=4M && sync # where sdX is your thumbdrive
```

If you are on Windows we recommend using rufus to burn the seapath.iso onto your USB thumb drive.
Use the DD mode when doing so.

Then plug the USB thumb drive int the RSAPC MK2 and boot off of it, for 
instruction on how to do so refer to the official manual. The installation will 
be fully automatic and will overwrite the first disk it finds. If there is an 
NVMe installed it's going to be that. Otherwise it will be whichever SSD is 
installed in the first slot. It is recommended to not have any drive 
installed which has sensitive data whose loss you can't afford.

After this your device will reboot into the operating system. You can then go ahead
and use it like you'd use any other RT-os. However, you still have to enable
the Welotec customizations to get the most benefit from your device.
To do so simply activate the welo-sp-rt tuned profile by typing in the following

```bash
tuned-adm profile welo-sp-rt
```

In order for some of the settings to take effect you will also have to reboot
the device. The next section will discuss custumizations so you can plan out
how to deploy your realtime application.


## Customizations

Customizations ontop of the regular SEAPATH image can be found in the usercustomizations folder.
They include a custom tuned profile which is deactivated by default. As well as a service
and corresponding shell script called hw_setup. This will prepare the machines last 4
physical CPU cores (cores 4-7) for realtime tasks. Which means it does the following.

The pair of cores 4, 5 and 6, 7 have dedicated 6MB L3 Cache allocated to each pair.
The logical sibling threads (HyperThreads) for these cores are disabled. Their base clock is
increased to 2.4 GHz to allow for higher single thread performance.

If you want to implement your own customizations for your usecase please refer to the
official seapath debian\_build\_iso repository. If you want to change the preinstalled
keys and passwords head to the USERCUSTOMIZATIONS.var file and change them accordingly.
Then rebuild the image with `./build_iso.sh`. You're user needs to be in the docker group
and be able to execute docker-compose. Also your system needs to be booted in UEFI mode.


# Security

This image is only meant for testing and evaluation purposes. Therefore we
preinstalled some SSH keys and set easy default passwords. Should you for some
reason decide to roll this image out to production, or can for some reason not
trust your local network, follow these steps to ensure your system is secure enough.

## Change passwords and keys

Currently this still has to be done by hand and is thus a tad tedious, in the future
there will be ansible playbooks or scripts allowing it to easily change 
keys and passwords.

First you need to generate new keys, unless you've already got some.
Watch out that you are not overwriting any existing keys in your ~/.ssh directory.

```bash
ssh-keygen -t ed25519 -C "<your_email@here.com>" -f ~/.ssh/sp_root_key
ssh-keygen -t ed25519 -C "<your_email@here.com>" -f ~/.ssh/sp_ansible_key
ssh-keygen -t ed25519 -C "<your_email@here.com>" -f ~/.ssh/sp_user_key
```

> [!Note]
> It is common to put your E-Mail as a comment at the end of your keys, however 
> that is not necessary, you can enter any information that will help you 
> identify the purpose of that key.

Next you need to push those new keys to the target system. Push keys for each user:

```bash
ssh-copy-id root@192.168.0.10 # Swap the IP with the IP or hostname of your server
ssh-copy-id ansible@192.168.0.10 # Swap the IP with the IP or hostname of your server
ssh-copy-id virtu@192.168.0.10 # Swap the IP with the IP or hostname of your server
```

Once you've done this, you should check that you can actually login using the new keys.
If thats the case you can go ahead and delete the old generic keys from the key
authorization file. You can identify the generic keys by their comment 
`WELOTEC - DO NOT TRUST`

When you've logged in to the device using the editor of your choice delete the
key mentionend above in the following files.

```bash
/root/.ssh/authorized_keys
/home/ansible/.ssh/authorized_keys
/home/virtu/.ssh/authorized_keys
```
After you've done that you should verify that they are gone, by trying to log in
with the old keys, which should now fail.

To change the passwords simply login to the users root and virtu and and interactively
change the password as desired using `passwd`

If anything is still unclear, please don't hesitate to open an issue explaining
where you are having trouble. I will try to address your issue and update 
the Instructions accordingly.


