# securite-mobile-lab11x

## Preparation de lenverenement 

<img width="1089" height="266" alt="image" src="https://github.com/user-attachments/assets/83b499a9-cf64-4a13-87f4-814f1ff6be12" />    

## Lapk bien installer  

<img width="612" height="949" alt="image" src="https://github.com/user-attachments/assets/cd466c33-83a7-4ef9-9f4c-360712b86802" />  

## lencement de frida server on arire plant 

<img width="1144" height="160" alt="image" src="https://github.com/user-attachments/assets/762d6fc3-d844-4e62-9e52-f1893ec50cf0" />  

# Vérifier que l’appareil est visible

<img width="894" height="617" alt="image" src="https://github.com/user-attachments/assets/ca018a64-88c9-4d47-aebd-9dec941000fd" />  

## Panorama rapide des techniques de détection de root

Java (haut niveau):  

Lecture de android.os.Build.TAGS (présence de test-keys).   

Recherche de fichiers via java.io.File.exists() pour /system/xbin/su, /system/bin/su, busybox, etc.  

Exécution de commandes via Runtime.getRuntime().exec("su"), which su, busybox.  

Librairies tierces (RootBeer) avec méthodes isRooted() etc.  

Natif (JNI/C/C++):  

Appels à open/openat/access/stat/lstat vers chemins suspects.  

Lecture de /proc/mounts ou /proc/self/mounts.  

Anti-debug/anti-Frida (scan de ports, strings "frida").  

# Script Frida (bypass Java) prêt à l’emploi


<img width="1920" height="977" alt="image" src="https://github.com/user-attachments/assets/f6707987-4f15-42d4-a18d-f4e3a2120fb8" />


Objectif: forcer des retours « non root » et bloquer l’accès aux indicateurs.  

# Lancement
<img width="1920" height="548" alt="image" src="https://github.com/user-attachments/assets/91f7d2a5-4063-429d-9b54-f55b9dbc30de" />    
## lapp crashed
<img width="480" height="874" alt="image" src="https://github.com/user-attachments/assets/1f4f8574-c72e-48c2-9f38-ebe5d17bec2f" />  
<img width="1920" height="890" alt="image" src="https://github.com/user-attachments/assets/45436d73-851c-4cb2-a65b-4e536c65f998" />

# Ajouter des hooks natifs (la détection passe par du code C/C++)  
<img width="1918" height="546" alt="image" src="https://github.com/user-attachments/assets/16efe6e5-900d-4699-9690-f81cd4ed1e16" />  
  
<img width="764" height="908" alt="image" src="https://github.com/user-attachments/assets/7137f6b1-2678-4eac-ad26-e8792c91207a" />

  <img width="453" height="68" alt="image" src="https://github.com/user-attachments/assets/b920d2b1-fb64-4d37-aae6-a1fbee1b9051" />
# Vérifie que frida-server tourne sur l'émulateur
  <img width="1110" height="105" alt="image" src="https://github.com/user-attachments/assets/88f714b5-6a70-4a35-ac7b-eb8c175ab2de" />


# si le script namarche avec la version de frida la soultuion est :   
## collecter les ressources de prblesms :
<img width="1608" height="785" alt="image" src="https://github.com/user-attachments/assets/61d02a2d-ba72-4871-a794-267b8faf57f5" />

# le prob ets de la versiond efrida jai le downgarde elle fonctionne 

## le script 
// bypass_final.js

// Hook strstr pour cacher frida
Interceptor.attach(Module.findExportByName("libc.so", "strstr"), {
    onEnter: function(args) {
        try {
            var needle = args[1].readCString();
            if (needle && (needle.indexOf("frida") !== -1 || needle.indexOf("xposed") !== -1)) {
                this.fake = true;
                console.log("[+] strstr bloqué: " + needle);
            }
        } catch(e) {}
    },
    onLeave: function(retval) {
        if (this.fake) {
            retval.replace(ptr(0));
            this.fake = false;
        }
    }
});
console.log("[+] strstr hookée");

// Hook fgets pour cacher frida dans /proc/self/maps
var fgetsPtr = Module.findExportByName("libc.so", "fgets");
var fgets = new NativeFunction(fgetsPtr, 'pointer', ['pointer', 'int', 'pointer']);
Interceptor.replace(fgetsPtr, new NativeCallback(function(buffer, size, fp) {
    var retval = fgets(buffer, size, fp);
    try {
        var line = Memory.readUtf8String(buffer);
        if (line && line.indexOf("frida") !== -1) {
            Memory.writeUtf8String(buffer, "");
            console.log("[+] fgets: ligne frida effacée");
        }
    } catch(e) {}
    return retval;
}, 'pointer', ['pointer', 'int', 'pointer']));
console.log("[+] fgets hookée");

// Java bypass
Java.perform(function () {
    try {
        var MainActivity = Java.use("sg.vantagepoint.uncrackable3.MainActivity");
        MainActivity.showDialog.implementation = function(msg) {
            console.log("[+] showDialog bloqué: " + msg);
        };
        console.log("[+] showDialog OK");
    } catch(e) { console.log("[-] showDialog: " + e); }

    try {
        var RootDetection = Java.use("sg.vantagepoint.util.RootDetection");
        RootDetection.checkRoot1.implementation = function() { return false; };
        RootDetection.checkRoot2.implementation = function() { return false; };
        RootDetection.checkRoot3.implementation = function() { return false; };
        console.log("[+] RootDetection OK");
    } catch(e) { console.log("[-] RootDetection: " + e); }

    try {
        var MC = Java.use("sg.vantagepoint.uncrackable3.MainActivity");
        MC.verifyLibs.implementation = function() {
            console.log("[+] verifyLibs bloquée");
        };
        console.log("[+] verifyLibs OK");
    } catch(e) { console.log("[-] verifyLibs: " + e); }
});

<img width="508" height="1000" alt="image" src="https://github.com/user-attachments/assets/881b20f8-3445-429c-932d-3e1047e5e52a" />




