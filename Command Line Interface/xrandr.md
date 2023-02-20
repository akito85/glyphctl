`xrandr --output eDP-1 --mode 1920x1080 --right-of HDMI-1 --output HDMI-1 --primary --mode 2560x1080`

<br/>

Lets break down
--
- xrandr is a xorg command line tool to configure x, if you use wayland it might be use different command
- output is your display output in this case i use eDP-1 and HDMI-1
- primary is to set which display is your primary
- right-of is the position of secondary display from your primary display
- mode is the resolution for each display