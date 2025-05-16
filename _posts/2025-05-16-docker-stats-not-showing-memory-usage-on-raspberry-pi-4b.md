---
layout: post
---

I noticed that `docker stats` wasn't showing memory usage on my Raspberry Pi 4B (64-bit Debian).

The memory-related columns like **MEM USAGE / LIMIT** and **MEM %** were showing zeros for all containers.
![0 memory usage](/assets/img/0-memory-usage.png)

## Solution

Enable memory cgroup support by modifying the boot parameters:

1. Open the boot configuration file:
    ```bash
    sudo nano /boot/firmware/cmdline.txt
    ```

2. Append the following parameters at the end of the existing line (do not create a new line):
    ```bash
    cgroup_enable=cpuset cgroup_enable=memory group_memory=1
    ```

3. sudo reboot it.

---

After rebooting, `docker stats` should now correctly show memory usage for all containers.
![showing memory usage](/assets/img/showing-memory-usage.png)


## 🔗 References

* [https://github.com/docker/for-linux/issues/1112#issuecomment-699512425](https://github.com/docker/for-linux/issues/1112#issuecomment-699512425)
