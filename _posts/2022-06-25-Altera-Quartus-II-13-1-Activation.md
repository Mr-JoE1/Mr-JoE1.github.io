---
layout: post
title: Quartus II 13.1 Activation 
categories: [FPGA]
tags: [embedded, fpga, ]
comments: true
---

I Have a project based on Altera Cyclone III chip which is supported only for Quartus version below v14.
So i need a Full version to make some modifaction on my project 

After 2 days digging the web for the crack or giveaway license, 
I got nothing , so i batched the program myself 

## Activation steps:
  - Download official Quartus 13.1:
			https://www.intel.com/content/www/us/en/software-kit/666220/intel-quartus-ii-web-edition-design-software-version-13-1-for-linux.html
  
  - Download Patched files  : 
			https://drive.google.com/file/d/1tlPxKtjD8hgnkkDcoESN2YVtF20UE0u2/view?usp=sharing
  - Repalce Dll files crossponded to your Machine  : 13.1 x86 bin/sys_cpt.dll or 13.1 x64 bin64/sys_cpt.dll 
  - change XXXXXXXXXXXX to the mac address of your network adapter in the license.dat file 
  
  
## Tips to batch other version 
## Preparing Rspi-zero-w OS:
  - 13.1 x86 bin/sys_cpt.dll 000E63D0: 55 8b ec -> 33 C0 C3
  - 13.1 x64 bin64/sys_cpt.dll 000A4D10: 4C 89 4C -> 33 C0 C3
  -	13.0 SP1 x86 bin/sys_cpt.dll 000E4F90: 55 8b ec -> 33 C0 C3
  - 13.0 SP1 x64 bin64/sys_cpt.dll 000A32C0: 4C 89 4C -> 33 C0 C3
  - 13.0 x86 bin/sys_cpt.dll 000E4F90: 55 8b ec -> 33 C0 C3
  - 13.0 x64 bin64/sys_cpt.dll 000A32C0: 4C 89 4C -> 33 C0 C3cs
 