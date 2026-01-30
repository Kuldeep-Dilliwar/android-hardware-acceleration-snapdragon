# How to enable snapdragon hardware acceleration in native Termux?
## Here is how to:
```
# 1. Update everything silently (Force "Yes" to all prompts)
pkg update -y -o Dpkg::Options::="--force-confnew"
pkg upgrade -y -o Dpkg::Options::="--force-confnew"

# 2. Install Repos and Packages silently
pkg install x11-repo tur-repo -y -o Dpkg::Options::="--force-confnew"
pkg install mesa mesa-vulkan-icd-freedreno termux-x11-nightly xfce4 xfce4-session mesa-demos glmark2 -y -o Dpkg::Options::="--force-confnew"

# 3. Clean up old locks
pkill -f termux-x11
rm -rf $PREFIX/tmp/.X0-lock

# 4. Create the setup in .bashrc (so it remembers next time)
echo '
export DISPLAY=:0
export MESA_LOADER_DRIVER_OVERRIDE=zink
export VK_ICD_FILENAMES=$PREFIX/share/vulkan/icd.d/freedreno_icd.aarch64.json
export TU_DEBUG=noconform
export MESA_VK_WSI_PRESENT_MODE=immediate
export vblank_mode=0

alias gpu="MESA_LOADER_DRIVER_OVERRIDE=zink VK_ICD_FILENAMES=$PREFIX/share/vulkan/icd.d/freedreno_icd.aarch64.json TU_DEBUG=noconform MESA_VK_WSI_PRESENT_MODE=immediate vblank_mode=0"
alias start-desktop="dbus-launch --exit-with-session xfce4-session"
' > ~/.bashrc

# 5. Load the new settings immediately
source ~/.bashrc

# 6. Start the X11 Server
termux-x11 :0 &
sleep 3

# 7. Start XFCE Desktop in the background
dbus-launch --exit-with-session xfce4-session &
sleep 5

# 8. RUN THE TESTS
echo "---------------------------------------------------"
echo "CHECKING DRIVER (Look for 'Zink' or 'Turnip' below)"
echo "---------------------------------------------------"
glxinfo -B | grep -E "OpenGL renderer|Device"

echo ""
echo "---------------------------------------------------"
echo "STARTING BENCHMARK (Switch to X11 App to watch!)"
echo "---------------------------------------------------"
glmark2
```
## Quick check 
```
# Verify the driver is "Zink" or "Turnip"
gpu glxinfo -B
```
## How to start it next time
```
termux-x11 :0 &
start-desktop
```
