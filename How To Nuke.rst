Performing a Cluster Reset on HyperCore and Resolving Leftover Remnants


A cluster reset is ONLY to be performed per customer request. If a customer requests a cluster reset it is required to have written permission to proceed. 



Resolution

Permission to Proceed

To obtain written permission send the following e-mail template to the customer and ensure they have verbally stated it is acceptable to proceed with the reset.  Make sure they have provided the Backplane IPs and the support tunnel number they have opened to the cluster. 

Dear <Name>,
 
During our conversation, we discussed resetting the cluster back to factory default settings. Please verify that you are in fact interested in resetting the cluster, wiping all current data, clearing current networking information, and starting from default settings to re-install the cluster. If so, please provide the Backplane IPs of all nodes in the cluster, the support tunnel number you have opened to the cluster, and respond affirmatively to this email in order to authorize the Support Engineer to run the script at the scheduled time.
 
Thank you,
<Name>

Resetting the Cluster

After the customer has responded, verify you have logged into the correct support tunnel and the Backplane IPs they have provided match the cluster you are on. Once confirmed run the following command:
scclusterreset --nuke

You will be prompted with the Backplane IPs of all nodes in the cluster and have to enter yes to proceed.

The cluster reset is typically a very quick process and once complete the nodes can be formed into a new cluster following the Scale Computing HyperCore Installation Guide. 

Resolving Issues After Cluster Reset

It is not uncommon for scclusterreset --nuke to not properly clear all necessary files and configurations, leaving the node in a state that will prevent a successful node add or cluster initialization.

NOTE: An engineering ticket has been created for this issue. Once resolved this section will be removed.

Not every step may be needed depending on what the reset has already resolved. Follow each step listed below and move past any sections that are not necessary. These commands will have to be performed for each node. 

The previous gateway may still be listed in /etc/resolv.conf. To clear out the file run the following:
> /etc/resolv.conf


The previous node IPs and GUIDs may still be stored in /opt/scale/var/nodes. To empty the file run the following:
> /opt/scale/var/nodes

The next item to check is a directory that may contain the GUIDs of each node added to the cluster. If this directory has any GUIDs listed within it run the following command to clear its contents.
rm -rf /opt/scale/var/incoming-nodes.added/*

If a vmBlacklist file exists we need to remove it:
rm -rf /opt/scale/var/vmBlacklist

The last step is to remove the backplane IP from /etc/xinetd.conf
vi /etc/xinetd.conf

You will need to remove the line that displays this:
bind = <backplane IP>

This can be removed by arrowing down to this line, and pressing 
dd

This will remove the line.  Once the line is gone, press 
:wq <enter>

This will save the file, minus the backplane IP stored in the xinetd.conf file.

There are three files that need cleaned up to resolve issues with rsh, rexec, and rlogin. Run the following command to remove the necessary contents and restart the xinetd deamon.
sed -i '/bind/d' /etc/xinetd.d/{rsh,rexec,rlogin} && service xinetd restart

Now we want to verify all scservices have been stopped.
scservices status

If any services are running we want to stop them.
service <daemon> stop

If any services are dead or locked remove the appropriate services file from /var/lock/subsys.
rm -rf /var/lock/subsys/<daemon>

Verify all services are stopped again.
scservices status

If you have any further issues run scclusterreset --nuke on each node. However, since the /opt/scale/var/nodes file is now empty we will need to provide it two columns. First column can be any text while the second column should be the backplane IP for this node. Use this command to easily append content to the file. 
echo "foo <backplaneIP>" > /opt/scale/var/nodes

Now execute the cluster reset command again.
scclusterreset --nuke

Using rsh move to the next node and perform the same steps. Once completed the nodes should be able to successfully perform node initializations, cluster initializations, and node adds. 
