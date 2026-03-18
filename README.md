
# Github CLI

- Push changes
- `gh run watch` to view process
- `gh run list`
- `gh run view`
when done, if succesful
- `gh run download` to download the latest firmware.
    - or `gh run download --dir ./firmware`

# Update List

2024/12/21
Added support for zmk-studio (just refresh the left hand to use).
2024/10/24
Modified power supply mode to reduce power consumption.
Fixed the automatic shut-off feature for RGB power supply.
2025/8/22
update the soft off.When you press the keys Q, S and Z simultaneously and hold them for 2 seconds, the keyboard will enter a deep sleep state and cannot be awakened by pressing the keys. This function can be used when carrying it outside. The activation method is to press the reset switch once.
This month, I also updated the ultra-thin versions of the corne and sofle cases. The frame and base plate have been thickened, and the opening of the reset switch has been adjusted, so that the reset switch can be easily pressed. At present, we are still conceptualizing how to design the shell with an inclined bracket.If you have carefully examined a PCB, you will notice that there are reserved interfaces for expansion IO. I wonder if anyone has been able to utilize them,I will try it！
The GIF animations on the right-hand keyboard screen have been removed, which will significantly reduce the power consumption of the right-hand keyboard.

> If your keyboard was updated before October 24, please update to the latest firmware.
> 
---

# Sofle Keymap
<img width="1004" height="1997" alt="image" src="https://github.com/user-attachments/assets/e96a967c-50e7-453f-8ba6-d0a265de2a26" />

## Known Issues

**Board Repository Pinned (March 2026):**
- The external board repository (`a741725193/zmk-sofle`) is currently pinned to commit `59a9f80` (Dec 27, 2025)
- Reason: March 2026 upstream updates introduced build failures (`KeyError: 'qualifiers'`) due to Zephyr 4.1 incompatibilities
- Impact: None on firmware functionality - all features work as expected
- Maintenance: Monthly checks scheduled to test for upstream compatibility fixes (see `AGENTS.md` for details)
- Ownership: To be assigned

This is a temporary workaround until the upstream board repository is updated for full Zephyr 4.1 support.


