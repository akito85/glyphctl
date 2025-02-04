**Error:**
Please unset QT_IM_MODULE and GTK_IM_MODULE environment variables and 'ibus-daemon --panel disable'

**Solution:**
Go to `/etc/environment.d`  
Create a file called `ibus-custom.conf`

Add these two lines:

`unset GTK_IM_MODULE unset QT_IM_MODULE`