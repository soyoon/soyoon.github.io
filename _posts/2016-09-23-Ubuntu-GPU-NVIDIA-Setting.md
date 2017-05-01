---
layout: post
title:  "Ubuntu 16.04, GTX1080, CUDA 8.0, Tensorflow"
date:   2016-09-23 22:00:00
categories: ML
---

2016.09.23 Ubuntu 16.04, GTX1080, CUDA 8.0, Tensorflow

Problem : After install NVIDIA driver, Ubuntu 16.04 stuck in login loop
Solution
- This problem was caused due to secure boot and EFI_SECURE_BOOT_SIG_ENFORCE in linux kernel 4.4.0-20 and later
- ** IMPORTANT !! : First of all make sure secure boot is disabled in computer BIOS. Older computers won't even have this option.

sudo apt-get purge nvidia
sudo apt-get install nvidia-current
sudo shutdown -r now

This can also fix a login loop issue on 16.04 or a black screen with cursor problem

after solve this problem, install CUDA 8.0, cuDnn v5 ( to support GTX1080 )


http://textminingonline.com/dive-into-tensorflow-part-iii-gtx-1080-ubuntu16-04-cuda8-0-cudnn5-0-tensorflow
