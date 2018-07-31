``#!/bin/bash

export DISPLAY=:0
export PATH=$PATH:/opt/scale/bin
DATE=`date +"%Y%m%d%H%M%S"`

sc vm show display detail | grep export -B 2 | grep GUID | awk '{print $3}' >> /root/automated-features/VMGUIDS
sc vm show display detail | grep export -B 2 | grep Name | awk '{print $3}' >> /root/automated-features/VMNAME

[[ $(cat /root/automated-features/VMGUIDS | wc -l) > 0 ]] || ( echo "No VMs tagged for export"; exit 1 )

paste /root/automated-features/VMGUIDS /root/automated-features/VMNAME | column -s $'\t' -t >> /root/automated-features/VMLIST

VMGUIDS=`cat /root/automated-features/VMLIST | awk '{print $1}'`
USER=`grep username /root/automated-features/authfile | awk '{print $3}'`
PASS=`grep password /root/automated-features/authfile | awk '{print $3}'`
DOMAIN=`grep domain /root/automated-features/authfile | awk '{print $3}'`
HOST=`grep host /root/automated-features/authfile | awk '{print $3}'`
SHARE=`grep share /root/automated-features/authfile | awk '{print $3}'`

smbclient -U $USER%$PASS -W $DOMAIN  //$HOST/$SHARE -c "mkdir $DATE"

for VM in $VMGUIDS
do
    VMNAME=`sc vm show guid $VM | egrep -v "GUID|===" | awk '{print $7}'`
    echo "$VMNAME -- smb://$HOST/$SHARE/$DATE/$VMNAME"
    sc vm export uri "smb://$DOMAIN;$USER:$PASS@$HOST/$SHARE/$DATE/$VMNAME" wait yes guid $VM
    if [[ $? -eq 0 ]] ; then
        echo "$VMNAME -- export completed" >> /tmp/ExportResults.txt
    else
        echo "$VMNAME -- export failed" >> /tmp/ExportResults.txt
    fi
sleep 5
done

sendEmail -f 'alerts3@scalecomputing.com' -t "itnotify@sos-retailservices.com" -u 'Export Results' -o message-file=/tmp/ExportResults.txt

rm -rf /tmp/ExportResults.txt
rm -rf /root/automated-features/VMGUIDS
rm -rf /root/automated-features/VMNAME
rm -rf /root/automated-features/VMLIST``
