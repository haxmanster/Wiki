


###    VNC Quick Start Instructions

Overview
--------
VNC (Virtual Network Computing) is a client/server remote desktop sharing
system which uses the RFB (Remote FrameBuffer) protocol.
A VNC session is started on a remote machine with the vncserver command
and then viewed on the local machine with the vncviewer command.
VNC allows any platform version of vncviewer to view a desktop on any
platform version of vncserver.  Supported platforms are (including but
no limited to) Windows, Linux, Solaris, and Mac OS X.

The vncserver runs on two ports: 5900 + :"display number" and 
5800 + :"display number".  Port 5900 + :<num> is for the standalone
viewer.  Port 5800 + :<num> for a java based, web browser viewer.
Primarily we will be concerned with port 5900 + :<num>

The display number defines which virtual screen the vncserver runs on, and
differentiates among several concurrent VNC sessions.  The display number
has the format ":n" where n a small integer number.
On windows, by default the vnc server shows the console desktop and is :0 (so 
runs on port 5900/5800).  Both Linux and Solaris follow the X windows display 
number convention.  So they usually start on :1 or 5901/5801 for the first 
session (because :0 is taken by the console X server).  When you run vncserver,
it will automatically choose a port number that does not already have an
Xserver running.  Multiple vncservers can be started with different display
numbers or the display number can be set on the command line.

The VNC software can be downloaded from the VNC website: www.realvnc.com

Linux Commands
--------------
To start vncserver:

  vncserver [:<display num>]  [-depth <bit color depth>] [-geometry <xpixels>x<y pixels>]

or more simply
 
  vncserver

vncserver will display its display number in the command-line window.
Note: do not kill the "VNC config" application, or cut and paste
will not work correctly.

To run vncviewer unencrypted:
  vncviewer <hostname>:<display num>

To run vncviewer encrypted:
First, configure an ssh client to add a secure outgoing TCP tunnel
with both the listen and destination ports set to the port number
of your vncserver on the remote machine (5901 in this example), and
connect the ssh client to the remote machine.

  ssh user@remotehost -L 5901:remotehost:5901

Then fire up the viewer on the client desktop (using dislay number 1 in
this example):

  vncviewer :1 

Note that a hostname is not required because a tunnel has been set up
to the remote machine.

To kill a VNC server (on display number 1):

  vncserver -kill :1 

To change the VNC password:

  vncpasswd

VNC dot files:
The Linux VNC settings and setup files are located in
the user's directory, ~user/.vnc

  .vnc/xstartup - which X apps to start (such as a window manager)
  .vnc/passwd   - vnc password file
  .vmc/*.log    - vnc log files; some can grow large and need to be cleaned up.

VNC Client on Windows 7
-----------------------
We use Putty on Windows 7.  It is very simple to configure:
   1. Start Putty
   2. Enter the host name or IP of the linux box you want to use
   3. In the box labeled "Category"
        --Select "SSH" which is under "Connection"
        --Select "Tunnels"
   4. Enter a source port, e.g. 5909
      Enter a destination port, e.g. machine_name:5909
      NOTE: the 9 in this example is the display number
            This is up to you to decide on, 1-9
            If you the display number is already in use,
            select a different number
   5. Click on "ADD"
   6. Select "Session" at the top of the "Category" box
      You are now back at the beginning window
   7. Give your session a name, e.g. machine_name and 
      select "SAVE"
This session with the tunneling ports set up is now saved.
You now double click on the session name you created and a login 
     window appears.

Login using your CS Linux login and password.
You now need to start a VNC session.
Here is an example for starting a VNC session:
       vncserver :3 -geometry 1600x1200 &
In this example, the server is starting a sesion on display "3", so
I would have set up putty for port 5903.  If I set up for port 5909,
my startup command would be:
       vncserver :9 -geometry 1600x1200 &
The geometry settings should correspond to the resolution settings of
your monitor.

Now you simply start your VNC client and enter "machine_name:display_number"
For instance, hercules:9


Windows Commands
----------------
To run vncviewer unencrypted:
If you are connecting from off-campus, you'll need to use the VPN.
Run the vncviewer and enter

  <hostname>:<display num>

in the vncviewer gui "server" field

To run vncviewer encrypted:
(This procedure varies, depending on your particular ssh client.  This
example is for the ssh.com client.)
Open an ssh window and edit -> settings -> tunneling
Add a secure outgoing TCP tunnel with both the listen and destination
port set to the port number of your target vncserver.  Then connect
the ssh client to the target machine.

Run the vncviewer and enter

  :<display num>

in the vncviewer gui "server" field

Vncserver Implementations
-------------------------
On windows, vncserver implements the console desktop.  On Linux and Solaris
it implements a "lite" version of X windows.  It behaves like standard 
Xwindows, but applications that use certain extensions will probably break 
or have weird issues (I believe opengl apps break).

Other notes:  There are several "flavors" vnc, RealVnc is the newest and has 
the best performance and features.  Other flavors are TightVnc and the 
original AT&T vnc.  On Solaris for best results, install Software Companion 
version of vnc.  (Older, but works out of the box.)
