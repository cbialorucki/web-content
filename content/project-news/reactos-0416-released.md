---
title:       "Introducing ReactOS 0.4.16"
author:      "ReactOS Team"
date:        2026-07-02
tags:        [ "release" ]
---

We are pleased to announce the release of ReactOS 0.4.16!
After a year and a half of development, we’re excited to showcase the improvements we’ve made between a new graphical installer, a unified bootcd and livecd image, better driver support, networking and storage stack improvements, and third-party code syncs.

## Graphical Installer and the All-in-One Boot CD
Historically, ReactOS offered two images for download, a livecd which let you test ReactOS in a read-only environment, and a bootcd which let you install ReactOS to your hard disk using a text-based installer.
Thanks to the efforts of Hermès BÉLUSCA - MAÏTO ([hbelusca](https://github.com/hbelusca)), ReactOS 0.4.16 has a new graphical installer and a combined bootcd and livecd.
Now you can test and install ReactOS using the same image.
(insert a gallery of graphical installer here)

## Video
During 0.4.15 development, core developer Hervé Poussineau ([hpoussin](https://github.com/hpoussin)) put in the ground work for multi-monitor support and falling back to a VGA driver when display drivers fail to load.
This foundation enabled us to continue pursuing better video driver compatibility in 0.4.16.
For years ReactOS has been plagued by different issues with all major video driver vendors.

Nvidia GPUs in particular had been plagued by a slow down issue that many talented contributors and developers investigated.
Eventually Justin Miller ([The_DarkFire_](https://github.com/darkfire01)) recognized that the kernel was running out of system page table entries (PTEs) when loading third party drivers.
This limitation was most apparent with graphics drivers, which allocate more memory than most other drivers.
The_DarkFire_ changed the memory layout used by our memory manager to increase the amount of system PTEs.
This fixed the hard-to-debug slowdown bug with Nvidia graphics drivers.
On AMD video drivers, the OpenGL window would end up blank.
This was resolved by rewriting ExtEscape, inspired by a patch from the late core developer Jim Tabor ([jimtabor](https://github.com/jimtabor)).
These improvements improved stability, better handled resource management of the new devices, and fixed many edge case bugs in our win32k.
We thank our contributors and developers for their time as these fixes needed an incredible amount of research.

## Audio
For years ReactOS had unstable and incomplete HD audio support.
HD audio drivers depend on a bus driver (`hdaudbus.sys`), including drivers from AMD, IDT, NVIDIA, Realtek, and SigmaTel.
Our initial implementation was written long ago by Johannes Anderwald ([janderwald](https://github.com/janderwald)).
This implementation was never finished, and was a frequent source of bug checks when attempting to install audio codecs with an HD audio controller.
Core developer Oleg Dubinskiy ([oleg-dubinskiy](https://github.com/oleg-dubinskiy)) imported [sklhdaudbus](https://github.com/coolstar/sklhdaudbus), a new HD audio bus driver, to replace our old implementation.

HD audio controllers which are compatible with Windows XP and Windows Server 2003 should now work in ReactOS 0.4.16.
Here is a video showing a Realtek HD audio controller working on ReactOS 0.4.16:
(replace me with an image link and upload video to youtube)
https://private-user-images.githubusercontent.com/26385117/584169871-b6474c2c-160b-4eb8-992c-3608854c92f5.mp4?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3ODMwNjY1MjAsIm5iZiI6MTc4MzA2NjIyMCwicGF0aCI6Ii8yNjM4NTExNy81ODQxNjk4NzEtYjY0NzRjMmMtMTYwYi00ZWI4LTk5MmMtMzYwODg1NGM5MmY1Lm1wND9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNjA3MDMlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjYwNzAzVDA4MTAyMFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWE0YTk1NzZjMzk4Njk5ODVjOGQwYjI5M2VlOTJhNDY5NWViMzhiNzQ2Mzc4MmJjZmY2MTFiZTM4OWFlZjNkODAmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JnJlc3BvbnNlLWNvbnRlbnQtdHlwZT12aWRlbyUyRm1wNCJ9.FlnzOhDvwcJ-_Td4pbK12ZEQugwYisVGQqA0guMNi8Y

The new HD audio bus driver depends on the Kernel Mode Driver Framework (KMDF).
Microsoft open sourced KMDF as part of the [Windows-Driver-Frameworks](https://github.com/microsoft/Windows-Driver-Frameworks) repository.
The_DarkFire_ imported KMDF for the new HD audio bus driver, and now we can use KMDF to import or develop other drivers.

Oleg also fixed the volume and balance sliders in Sound Properties (`mmsys.cpl`) and Audio Volume Mixer (`sndvol32.exe`).
Now the volume and balance levels are saved and restored on reboot when using an HD audio codec.
In addition, Oleg updated the audio device enumeration code to support more sound cards.

## Networking
ReactOS 0.4.16 also adds asynchronous connection support.
This improves networking performance by allowing applications to execute networking operations without stalling.
This also improves application compatibility as many programs assume that these asynchronous connection APIs are always present.

## Storage
Since 2009, ReactOS has been using the UniATA storage driver to add SATA, AHCI, and support for partitions greater than 8GB.
This was a huge help to ReactOS then, but today UniATA is responsible for slow boot times and failing to load on many devices; leading to the dreaded `INACCESSIBLE_BOOT_DEVICE` (`0x7B`) bug check.
ReactOS 0.4.16 introduces a new ATA driver developed by contributor Dmitry Borisov ([disean](https://github.com/disean)).
This new ATA driver allows ReactOS to boot in far more environments, including inside Hyper-V.

In 2021 we imported and enabled the open-source Microsoft FastFAT driver.
Unfortunately, this broke our ability to repair FAT partitions using chkdsk.
Core developer Doug Lyons ([Doug-Lyons](https://github.com/Doug-Lyons)) fixed our FAT chkdsk routines to work with the Microsoft FastFAT driver.

Core developer Mark Jansen ([learn-more](https://github.com/learn-more)) added a disk cleanup utility in ReactOS 0.4.16.
The disk cleanup utility is compatible with extensions for the Windows disk cleanup utility, allowing third party programs to clean up disk usage as well as the operating system.

## Third-Party Code Syncs
ReactOS utilizes several other open-source projects as part of its code base.
One of the largest open-source projects we leverage is Wine, a re-implementation of several Windows APIs for Unix-like operating systems.
ReactOS uses a fork of Wine that interfaces directly with a Windows-like kernel instead of translating calls to a Unix-like one.

For many years, ReactOS was limited to Wine 2.x and 3.x due to compatibility concerns adopting APIs newer than those available to Windows Server 2003.
Towards the end of the 0.4.15 development cycle, we abandoned this strict adherence to Windows Server 2003 compatibility, which allowed us to slowly update our Wine fork to Wine 10.0.
This upgrade is still on going, but 0.4.16 has a significant amount of this work in it.
We anticipate that updating to Wine 11.0 or later versions will be significantly easier thanks to this effort to bring it up to Wine 10.0.

At this time, the ReactOS release image is still compiled with Windows Server 2003 exports only since there are several programs that expect all Windows Vista and newer exports available even if only some are exposed.
If you’d like to experiment with Windows Vista and newer application support, build ReactOS using the `-DDLL_EXPORT_VERSION` flag.
Instructions on how to build ReactOS are available [here](https://reactos.org/wiki/Building_ReactOS).

For the first time, ReactOS release images will include WineVDM, which increases compatibility with 16-bit Windows applications.
WineVDM is a project by [otya128](https://github.com/otya128), available [here](https://github.com/otya128/winevdm).

## Final Thoughts
The mission for ReactOS is to “[Run] your favorite Windows applications and drivers in an open-source environment you can trust.”
With each release we come closer to fulfilling this goal.
We look forward to sharing more developments and progress with you.

We extend our deepest gratitude to our community, contributors and donors.
Without our contributors, we wouldn’t be able to make any development progress.
Without our donors, we couldn’t fund our testing and hosting infrastructure or development contracts to accelerate progress.
And without our community, no one would know we exist.
Thank you for making ReactOS possible.

If you are interested in joining our community, contributing, or donating to the ReactOS project, check out our homepage to find our donation page, social media links, and our GitHub.

Sincerely,

The ReactOS Team

[Download ReactOS 0.4.16](https://reactos.org/download/)

## Statistics
Resolved Jira issues: 375 (REMOVE ME: as of 7/1/2026)

Commits: 2735 (REMOVE ME: as of 7/1/2026)

Oldest Jira issue resolved: [CORE-3804](https://jira.reactos.org/browse/CORE-3804) from February 3rd, 2009 

- [0.4.16 Resolved Issues](https://reactos.org/wiki/0.4.16_Resolved_Issues)
- [0.4.16 Commit History](https://reactos.org/wiki/0.4.16_Commit_History)

The [releases/0.4.16](https://github.com/reactos/reactos/tree/releases/0.4.16) branch was forked from [master](https://github.com/reactos/reactos) on April 28th, 2026 after commit [bac97c5](https://github.com/reactos/reactos/commit/bac97c58e25262ccf53dfd7d0de4bed638b65491)
