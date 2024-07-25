# SEAPATH installation media

This repository contains customizations geared towards the realtime capabilities of
Welotecs RSAPC MK2 platform. We also provide an installation image (not quite yet).
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

## Customizations

Customizations ontop of the regular SEAPATH image can be found in the usercustomizations folder.
They include a custom tuned profile which is deactivated by default. As well as a service
and corresponding shell script called hw_setup. This will take prepare the machines last 4
physical CPU cores (cores 4-7) for realtime tasks. Which means it does the following.

The pair of cores 4, 5 and 6, 7 have dedicated 6MB L3 Cache allocated to each pair.
The logical sibling threads (HyperThreads) for these cores are disabled. Their base clock is
increased to 2.4 GHz to allow for higher single thread performance.

## Exchange passwords and keys

TODO
