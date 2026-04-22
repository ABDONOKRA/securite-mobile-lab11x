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

Objectif: forcer des retours « non root » et bloquer l’accès aux indicateurs.  




