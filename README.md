# NNTP-Monitor
Script to monitor a news group for keywords.
  
#!/bin/bash

 This script uses the small (200k) nntp client 'sinntp' to check for keywords contained in
 alt.privacy.anon-server messages.  If any of the keywords (case insensitive) are found,
 the script then will sends an email to the designated email address assigned to the
 'emailaddr=' variable below.  This can ensure that remailer sysops are quickly made
 aware that there are messages concerning them on APAS.
 Note: 'sinntp' requires python3.  Check presence by entering 'python3 -V'.

 1. Place this script in its entirety into a file named CheckApasMsgs.sh in the /home
    directory of any user (not root, not user mix, not any mixmaster user) and perform a
    'sudo chmod 755 CheckApasMsgs.sh'.
 2. Create a file named CheckApasMsgs.txt in the same /home directory and place keyword(s)
    to search for on separate lines within that file.  At the very least, your remailer
    name should be in CheckApasMsgs.txt so you will get notified of APAS messages to
    (or about) you.
 3. Download sinntp-1.6.tar.gz into the same /home directory and untar it:
    'wget --no-check-certificate https://github.com/jwilk/sinntp/releases/download/1.6/sinntp-1.6.tar.gz'
    'tar zxvpf sinntp-1.6.tar.gz'
 4. (sinntp-1.6.tar.gz can also be found here: https://inwtx.net/sinntp-1.6.tar.gz)
 5. Change the email address 'your@emailaddr.net' in the CheckApasMsgs.sh script.
 6. Before setting up the cronjob below, execute the script manually because it will take a
    few minutes to initially get the articles.  Only the last 20 articles will be downloaded
    to your serve at a given time.  Do not interrupt the download: ./CheckApasMsgs.sh
 7. A cronjob in the /home user is used to execute this script:
    */1 * * * * ./CheckApasMsgs.sh &> /dev/null

 Note: to test to see if the code is running correctly, temporarily put the single letter e
 on a single line within CheckApasMsgs.txt.  The script will check to see if the letter e
 is somewhere in the apas text (a certain discovery) and will send you an email.


filePath=${0%/*} # current file path  $filePath/  

use_server=fleegle.mixmin.net  
newsgp=alt.privacy.anon-server  
emailaddr=your@emailaddr.net  
varCAM=""  
  
if [ ! -e $filePath/CheckApasMsgs.txt ] || [[ ! $(cat $filePath/CheckApasMsgs.txt | wc -l) -gt 0 ]]; then  
   echo "Required file CheckApasMsgs.txt is empty or does not exist!"  
   echo "See instruction # 5."  
   echo "Exiting script!"  
fi  
  
  
sinntp-1.6/nntp-pull -S $use_server $newsgp --limit 20  
  
if [ -e $filePath/$newsgp ]; then  
   sed -i '/Date:/d' $newsgp   # delete unnecessary headers  
   sed -i '/From:/d' $newsgp  
   sed -i '/Injection-Info:/d' $newsgp  
   sed -i '/Message-ID:/d' $newsgp  
   sed -i '/Newsgroups:/d' $newsgp  
   sed -i '/Organization:/d' $newsgp  
   sed -i '/Path:/d' $newsgp  
   sed -i '/X-Abuse:/d' $newsgp  
   sed -i '/Xref:/d' $newsgp  
  
   while read line1; do  
      if [[ $line1 = "e" ]]; then  
         if [[ $(grep -i -c $line1 $filePath/$newsgp) -gt 0 ]]; then  
         mutt -s "Message in APAS." $emailaddr < $filePath/$newsgp  
         break  
         fi  
      fi  
  
      if [[ $(grep -i -c $line1 $filePath/$newsgp) -gt 0 ]]; then  
         varCAM=$(grep Subject $filePath/$newsgp)  
         mutt -s "APAS: $(sed 's/Subject\: '// <<<$varCAM)" $emailaddr < $filePath/$newsgp  
         break   # this added to stop multiple mails due to multiple words found  
      fi  
   done< $filePath/CheckApasMsgs.txt  
  
   rm $filePath/$newsgp  
fi  
  
exit 0    
  
