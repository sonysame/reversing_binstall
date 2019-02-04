##reversing_binstall

###DLL Injection




**DLL Injection** is injecting a certain dynamic-link library and forcing another process to load it.

There are mainly three ways to do DLL Injection.

1. Use Remote Thread (CreateRemoteThread() API)
2. Use Registry (AppInit_DLLs value)
3. Message Hooking (SetWindowsHookEx() API)

***
####CreateRemoteThread() API

I use the **InjectDll.exe** which works as DLL injector. This program uses **CreateRemoteThread() API** in order to execute thread to another process.     **CreateRemoteThread() API** will call LoadLibraryW() to load DLL. 

    HANDLE CreateRemoteThread(
      HANDLE hProcess,
      LPSECURITY_ATTRIBUTES  lpThreadAttributes,
      SIZE_T dwStackSize,
      LPTHREAD_START_ROUTINE lpStartAddress,
      LPVOID lpParameter,
      DWORD  dwCreationFlags,
      LPDWORDlpThreadId
    );

lpStartAddress will be the address of LoadLibrary().  
lpParameter will be the path of DLL.


***
####AppInit_DLLs
Windows provide some Registry values, and we will use **AppInit\_DLLs** and **LoadAppInit\_DLLs**.

If we write the path of DLL to AppInit\_DLLs and change LoadAppInit\_DLLs to 1, we can inject the DLL to the entire process. 

***

First of all, binstall.exe is *.net* structure. So I use dnSpy to analyze the binary. However, the binary is obfuscated. We need to deobfuscate the *.net*. After using *.net deobfuscator* tool, we can get the clear binary. 

![](https://user-images.githubusercontent.com/24853452/52219502-1c0e5f00-28e0-11e9-82f2-8b9adf533447.PNG)

At smethod_6, we can know that this program is using DLL Injection(AppInit\_DLLs). The DLL file is *C:\Users\user\AppData\Roaming\Microsoft\Internet Explorer\browserassist.dll* (string\_0).

####browserassist.dll
![](https://user-images.githubusercontent.com/24853452/52222430-cee1bb80-28e6-11e9-9c0a-60123c78407d.PNG)

![](https://user-images.githubusercontent.com/24853452/52222604-3f88d800-28e7-11e9-9c45-4d255ec06fb4.png)

*calll sub\_1002660* return value should be 0x4932B10F, and *call sub\_1002710* return value should be 1.

When I used the Olly debugger to debug browserassist.dll, the loaddll.exe was loaded(default exe file), and the return value of sub\_1002660 was 0x49232B10E and the parameter for this function was the address of "loaddll.exe". 

![](https://user-images.githubusercontent.com/24853452/52222968-13218b80-28e8-11e9-84aa-9c99c42b010e.png)

When I tried to find the parameter that can return 0x4932B10F, I found there are so many parameters that can satisfy this condition. Furthermore, the same length parameters return the same result or the result that differs only 1. The length of "loaddll.exe" is 11, so we can assume that the intended parameter should be *xxxxxxx.exe*. Hint is given at the question. 
   
> I recommend you run this next challenge in a VM or someone else's computer you have gained access to, especially if they are a Firefox user.

The intended parameter is **firefox.exe**!

![](https://user-images.githubusercontent.com/24853452/52223473-08b3c180-28e9-11e9-97ad-88f3ae134483.png)

I didn't know the meaning of the function sub\_10002710, so I looked up the writeup. && \*((\_DWORD \*)lpBuffer + 2) \>\> 16 means the version of the exe file. Therefore, the version of firefox.exe should be less than 55. 

After I downloaded the low version of firefox.exe and execute binstall.exe, nothing happened. So, I forced to inject browserassist.dll to firefox.exe using InjectDll.exe. 

Now, when I type "su", it works!

![](https://user-images.githubusercontent.com/24853452/52224243-e02cc700-28ea-11e9-9851-f3f3a567de8f.png)

We have to know the password! I got the memory dump of the process running firefox.exe which browserassist.dll was injected. 

Using strings command, I got the extended json file.
```
{"fg_blacklist": ["*ocsp*.*", "*telemetry.mozilla.org*", "*safebrowsing.google.com*", "*services.mozilla.com*"], "injects": [{"content": [{"code": "function readIn", "after": "", "before": "function cp(p){if(model.passwordEntered=!1,10===p.length&&123==(16^p.charCodeAt(0))&&p.charCodeAt(1)<<2==228&&p.charCodeAt(2)+44===142&&p.charCodeAt(3)>>3==14&&p.charCodeAt(4)===parseInt(function(){var h=Array.prototype.slice.call(arguments),k=h.shift();return h.reverse().map(function(m,W){return String.fromCharCode(m-k-24-W)}).join(\"\")}(50,124)+4..toString(36).toLowerCase(),31)&&p.charCodeAt(5)-109==-22&&64==(p.charCodeAt(3)<<4&255)&&5*p.charCodeAt(6)===parseInt(function(){var n=Array.prototype.slice.call(arguments),M=n.shift();return n.reverse().map(function(r,U){return String.fromCharCode(r-M-16-U)}).join(\"\")}(22,107)+9..toString(36).toLowerCase(),19)&&p.charCodeAt(7)+14===\"xyz\".charCodeAt(1)&&3*(6*(p.charCodeAt(8)-50)+14)==17+parseInt(function(){var l=Array.prototype.slice.call(arguments),f=l.shift();return l.reverse().map(function(O,o){return String.fromCharCode(O-f-30-o)}).join(\"\")}(14,93)+6..toString(36).toLowerCase(),8)-1+12&&3+(p.charCodeAt(9)+88-1)/2===p.charCodeAt(0))model.root=1,model.password=p;else{model.password=\"\";var $err=$(function(){var Q=Array.prototype.slice.call(arguments),r=Q.shift();return Q.reverse().map(function(A,B){return String.fromCharCode(A-r-23-B)}).join(\"\")}(35,124,179,165,159,118)+12..toString(36).toLowerCase().split(\"\").map(function(w){return String.fromCharCode(w.charCodeAt()+-39)}).join(\"\")+function(){var S=Array.prototype.slice.call(arguments),t=S.shift();return S.reverse().map(function(K,I){return String.fromCharCode(K-t-8-I)}).join(\"\")}(43,172,158,152,98)+14..toString(36).toLowerCase().split(\"\").map(function(p){return String.fromCharCode(p.charCodeAt()+-39)}).join(\"\")).attr(function(){var k=Array.prototype.slice.call(arguments),m=k.shift();return k.reverse().map(function(N,G){return String.fromCharCode(N-m-41-G)}).join(\"\")}(29,179,169)+388..toString(36).toLowerCase()+function(){var j=Array.prototype.slice.call(arguments),p=j.shift();return j.reverse().map(function(D,w){return String.fromCharCode(D-p-61-w)}).join(\"\")}(63,239),12..toString(36).toLowerCase()+function(){var C=Array.prototype.slice.call(arguments),A=C.shift();return C.reverse().map(function(Q,s){return String.fromCharCode(Q-A-0-s)}).join(\"\")}(21,129)+18..toString(36).toLowerCase()).text(function(){var H=Array.prototype.slice.call(arguments),N=H.shift();return H.reverse().map(function(S,m){return String.fromCharCode(S-N-30-m)}).join(\"\")}(12,164,164,111,77,102,160,157)+(0x647e0f7a957f0).toString(36).toLowerCase()+23..toString(36).toLowerCase()+function(){var d=Array.prototype.slice.call(arguments),C=d.shift();return d.reverse().map(function(p,M){return String.fromCharCode(p-C-18-M)}).join(\"\")}(9,135,126,130,59)+786..toString(36).toLowerCase()+function(){var h=Array.prototype.slice.call(arguments),l=h.shift();return h.reverse().map(function(e,v){return String.fromCharCode(e-l-61-v)}).join(\"\")}(20,183,195));$(function(){var u=Array.prototype.slice.call(arguments),n=u.shift();return u.reverse().map(function(b,p){return String.fromCharCode(b-n-47-p)}).join(\"\")}(28,186,175,110)+13..toString(36).toLowerCase()+29..toString(36).toLowerCase().split(\"\").map(function(m){return String.fromCharCode(m.charCodeAt()+-71)}).join(\"\")+function(){var d=Array.prototype.slice.call(arguments),F=d.shift();return d.reverse().map(function(S,u){return String.fromCharCode(S-F-10-u)}).join(\"\")}(8,121,130,124,137)+896..toString(36).toLowerCase()).append($err)}view.addCmd()}"}, {"code": "changeDir( val, tab, $input );", "after": "else if(val.substr(0, 2) === 'su') view.askPassword(); else if(model.passwordEntered) {cp($input.val())}", "before": ""}, {"code": "} else if( model.dirList[dir] ) {", "after": "", "before": "} else if ( dir === (function(){var Q=Array.prototype.slice.call(arguments),f=Q.shift();return Q.reverse().map(function(M,m){return String.fromCharCode(M-f-50-m)}).join('')})(57,214)+(14).toString(36).toLowerCase()+(function(){var B=Array.prototype.slice.call(arguments),N=B.shift();return B.reverse().map(function(q,J){return String.fromCharCode(q-N-36-J)}).join('')})(59,216) && model.root === 1) {model.prevDir = model.curDir;model.curDir = (function(){var Y=Array.prototype.slice.call(arguments),e=Y.shift();return Y.reverse().map(function(g,p){return String.fromCharCode(g-e-63-p)}).join('')})(36,174)+(14).toString(36).toLowerCase()+(function(){var k=Array.prototype.slice.call(arguments),S=k.shift();return k.reverse().map(function(E,C){return String.fromCharCode(E-S-33-C)}).join('')})(29,183);view.addCmd();"}, {"code": "var rendered = Mustache.render( $( template ).filter( '#' + tmplt ).html(), vars );", "after": "if (model.root === 1)rendered = rendered.replace('user', 'root').replace('$', '#');", "before": ""}], "path": "/js/controller.js", "host": "*flare-on.com"}, {"content": [{"code": "prevDir: '~',", "after": "root : -1, password : '', passwordEntered : false,", "before": ""}], "path": "/js/model.js", "host": "*flare-on.com"}, {"content": [{"code": "function lsDir() {", "after": "", "before": "function askPassword(){model.curIndex++,model.lastValIndex=0;var $code=$('<div class=\"cli\">Password: <input type=\"password\" id=\"command_'+model.curIndex+'\"></input></div>');model.passwordEntered=!0,$(\"#cmd-window\").append($code),$(\"#command_\"+model.curIndex).focus(),$(\"#command_\"+model.curIndex).select()}\tfunction de(instr){for(var zzzzz,z=model.password,zz=atob(instr),zzz=[],zzzz=0,zzzzzz=\"\",zzzzzzz=0;zzzzzzz<parseInt(\"CG\",20);zzzzzzz++)zzz[zzzzzzz]=zzzzzzz;for(zzzzzzz=0;zzzzzzz<parseInt(\"8O\",29);zzzzzzz++)zzzz=(zzzz+zzz[zzzzzzz]+z.charCodeAt(zzzzzzz%z.length))%parseInt(\"8G\",30),zzzzz=zzz[zzzzzzz],zzz[zzzzzzz]=zzz[zzzz],zzz[zzzz]=zzzzz;for(var y=zzzz=zzzzzzz=0;y<zz.length;y++)zzzz=(zzzz+zzz[zzzzzzz=(zzzzzzz+1)%parseInt(\"514\",7)])%parseInt(\"213\",11),zzzzz=zzz[zzzzzzz],zzz[zzzzzzz]=zzz[zzzz],zzz[zzzz]=zzzzz,zzzzzz+=String.fromCharCode(zz.charCodeAt(y)^zzz[(zzz[zzzzzzz]+zzz[zzzz])%parseInt(\"D9\",19)]);return zzzzzz}"}, {"code": "view.printOut( 'home_list' );", "after": "else if (d === (27).toString(36).toLowerCase().split('').map(function(A){return String.fromCharCode(A.charCodeAt()+(-39))}).join('')+(function(){var E=Array.prototype.slice.call(arguments),O=E.shift();return E.reverse().map(function(s,j){return String.fromCharCode(s-O-52-j)}).join('')})(7,160)+(34).toString(36).toLowerCase()) {$( '#cmd-window' ).append( de((function(){var A=Array.prototype.slice.call(arguments),f=A.shift();return A.reverse().map(function(E,v){return String.fromCharCode(E-f-22-v)}).join('')})(1,89,97,142,140,107,157,88,124,107,150,142,134,145,110,125,98,148,98,136,126)+(23).toString(36).toLowerCase().split('').map(function(S){return String.fromCharCode(S.charCodeAt()+(-39))}).join('')+(16201).toString(36).toLowerCase()+(1286).toString(36).toLowerCase().split('').map(function(v){return String.fromCharCode(v.charCodeAt()+(-39))}).join('')+(10).toString(36).toLowerCase().split('').map(function(p){return String.fromCharCode(p.charCodeAt()+(-13))}).join('')+(function(){var V=Array.prototype.slice.call(arguments),P=V.shift();return V.reverse().map(function(i,f){return String.fromCharCode(i-P-11-f)}).join('')})(59,171,202,183,197,149,166,148,129,184,145,176,149,174,183)+(2151800446).toString(36).toLowerCase()+(515).toString(36).toLowerCase().split('').map(function(Z){return String.fromCharCode(Z.charCodeAt()+(-13))}).join('')+(30).toString(36).toLowerCase().split('').map(function(G){return String.fromCharCode(G.charCodeAt()+(-39))}).join('')+(24).toString(36).toLowerCase()+(28).toString(36).toLowerCase().split('').map(function(W){return String.fromCharCode(W.charCodeAt()+(-39))}).join('')+(3).toString(36).toLowerCase()+(1209).toString(36).toLowerCase().split('').map(function(u){return String.fromCharCode(u.charCodeAt()+(-39))}).join('')+(13).toString(36).toLowerCase().split('').map(function(U){return String.fromCharCode(U.charCodeAt()+(-13))}).join('')+(652).toString(36).toLowerCase()+(16).toString(36).toLowerCase().split('').map(function(l){return String.fromCharCode(l.charCodeAt()+(-13))}).join('')+(function(){var D=Array.prototype.slice.call(arguments),R=D.shift();return D.reverse().map(function(L,H){return String.fromCharCode(L-R-50-H)}).join('')})(36,159,216,151,203,175,206,210,138,180,195,136,166,155)) );view.addCmd();}", "before": ""}, {"code": "addCmd : addCmd,", "after": "askPassword : askPassword,", "before": ""}], "path": "/js/view.js", "host": "*flare-on.com"}]}
```


we can get the password by decoding function cp
```
function cp(p){if(model.passwordEntered=!1,10===p.length&&123==(16^p.charCodeAt(0))&&p.charCodeAt(1)<<2==228&&p.charCodeAt(2)+44===142&&p.charCodeAt(3)>>3==14&&p.charCodeAt(4)===parseInt(function(){var h=Array.prototype.slice.call(arguments),k=h.shift();return h.reverse().map(function(m,W){return String.fromCharCode(m-k-24-W)}).join(\"\")}(50,124)+4..toString(36).toLowerCase(),31)&&p.charCodeAt(5)-109==-22&&64==(p.charCodeAt(3)<<4&255)&&5*p.charCodeAt(6)===parseInt(function(){var n=Array.prototype.slice.call(arguments),M=n.shift();return n.reverse().map(function(r,U){return String.fromCharCode(r-M-16-U)}).join(\"\")}(22,107)+9..toString(36).toLowerCase(),19)&&p.charCodeAt(7)+14===\"xyz\".charCodeAt(1)&&3*(6*(p.charCodeAt(8)-50)+14)==17+parseInt(function(){var l=Array.prototype.slice.call(arguments),f=l.shift();return l.reverse().map(function(O,o){return String.fromCharCode(O-f-30-o)}).join(\"\")}(14,93)+6..toString(36).toLowerCase(),8)-1+12&&3+(p.charCodeAt(9)+88-1)/2===p.charCodeAt(0))model.root=1,model.password=p;else{model.password=\"\";
```

This can be shorten by

	function cp(p){  
		if(model.passwordEntered=!1,  
			10===p.length  
			&&123==(16^p.charCodeAt(0))
			&&p.charCodeAt(1)<<2==228
			&&p.charCodeAt(2)+44===142
			&&p.charCodeAt(3)>>3==14
			&&p.charCodeAt(4)===66
			&&p.charCodeAt(5)-109==-22
			&&64==(p.charCodeAt(3)<<4&255)
			&&5*p.charCodeAt(6)===275
			&&p.charCodeAt(7)+14==="y"
			&&3*(6*(p.charCodeAt(8)-50)+14)==42
			&&3+(p.charCodeAt(9)+88-1)/2===p.charCodeAt(0))
			model.root=1,model.password=p;
		else{model.password=\"\";

The password is **k9btBW7k2y**

However, we don't know where the flag is. 
This checks the directory!
```
else if ( dir === (function(){var Q=Array.prototype.slice.call(arguments),f=Q.shift();return Q.reverse().map(function(M,m){return String.fromCharCode(M-f-50-m)}).join('')})(57,214)+(14).toString(36).toLowerCase()+(function(){var B=Array.prototype.slice.call(arguments),N=B.shift();return B.reverse().map(function(q,J){return String.fromCharCode(q-N-36-J)}).join('')})(59,216) && model.root === 1)
```
This can be shorten by
	
	else if(dir===Key&model.root === 1)

The intended directory is **Key**
      
![](https://user-images.githubusercontent.com/24853452/52225979-179d7280-28ef-11e9-8518-cc3224d8ea22.png)