# Boot in recovery mode.
#
# Disable the security by running: 
#
#   ~~~shell
#   $ csrutil disable
#   ~~~
#
#
# Apple has a layer of security in macOS; the layer removes some privileges from root. 
# The spindump file has a restricted flag. 
#
# Only restricted processes which are signed by Apple will be able to
# modify these files. However, you can disable this security system by 
# booting in recovery mode and disabling it in a Terminal by doing: csrutil disable.

# Examples:
#
#   ~~~shell
#   $ sudo mv spindump spindump-disabled
#   mv: rename spindump to spindump-disabled: Operation not permitted
#
#   $ ls -ldeO@ .
#   drwxr-xr-x  246 root  wheel  restricted 8364 Apr 24 04:01 .
#
#   $ ls -leO@ spindump
#   -rwxr-xr-x  1 root  wheel  restricted,compressed 381024 Apr 13 03:45 spindump
#
#   $ touch x
#   touch: x: Operation not permitted
#   ~~~

# Become root
sudo -i

trap 'csrutil enable' INT TERM EXIT

# Modify the System Integrity Protection configuration. 
# All configuration changes apply to the entire machine.
# We temporarily disable it here, so we can modify files. 
# Later in this scrip we will enable it again.
csrutil disable

# Unload spindump now and going forward
sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.spindump.plist

# Create a fake spindump command that runs forever doing nothing
cat << EOF > /usr/sbin/spindump-fake
#!/bin/sh
while true
do
sleep 60000
done
EOF

# Link the spindump command to the fake one
cd /usr/sbin/
mv spindump spindump-real
ln -sf spindump-fake spindump

# Restore System Integrity Protection
csrutil enable
