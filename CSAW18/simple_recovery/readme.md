Forensics
simple_recovery

Solved : 306/1448
Difficulty : ✦✧✧✧✧


Simple Recovery Try to recover the data from these RAID 5 images!

after downloading the 2 images, and extracting them ,doing some strings and file commands on them 

<b>0x890$ file disk.img</b>
disk.img: Hitachi SH big-endian COFF object, not stripped

actualiy its a HITASHI COFF object and not stripped, let's try to see strings on one of the files. 

<b>0x890$ strings disk.img</b>
EFI PART
XpFE?
fXfX
#Ehr
GLt$
8jfi@


we can spot fome words like flag, okey lets try to grep it. 

<b>0x890$ strings disk.img | grep flag{</b>
                  <photoshop:LayerName>flag{dis_week_evry_week_dnt_be_securty_weak}</photoshop:LayerName>
                  <photoshop:LayerText>flag{dis_week_evry_week_dnt_be_securty_weak}</photoshopTOSHOP
                  
we can see from the grep result that there is a Photoshop file has a layer named flag{... and containts the flag on a text.

<b>the flag is : flag{dis_week_evry_week_dnt_be_securty_weak}</b>
