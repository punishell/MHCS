# Mobile Hacking Cheat Sheet
Mobile Hacking Cheat Sheet

### Install System CA
```
# Convert DER to PEM
openssl x509 -inform DER -in cacert.der -out cacert.pem

# Get subject_hash_old (or subject_hash if OpenSSL < 1.0)
openssl x509 -inform PEM -subject_hash_old -in cacert.pem |head -1

# Rename cacert.pem to <hash>.0
mv cacert.pem 9a5ba575.0

# Remount and copy cert to device
adb root
adb remount
adb push 9a5ba575.0 /sdcard/
adb shell
vbox86p:/ # mv /sdcard/9a5ba575.0 /system/etc/security/cacerts/
vbox86p:/ # chmod 644 /system/etc/security/cacerts/9a5ba575.0
vbox86p:/ # reboot
```
### Modify APK

```
apktool d TestApp.apk
vim TestApp\res\xml\network_security_config.xml
#Content:
        <network-security-config> 
        <base-config> 
        <trust-anchors> 
        <!-- Trust preinstalled CAs --> 
        <certificates src="system" /> 
        <!-- Additionally trust user added CAs --> 
        <certificates src="user" /> 
        </trust-anchors> 
        </base-config> 
        </network-security-config>

vim TestApp\AndroidManifest.xml
# Add to <application > tag:
        android:networkSecurityConfig="@xml/network_security_config"

# Rebuild and self-sign
keytool -genkey -v -keystore test.keystore -storepass password -alias android -keypass password -keyalg RSA -keysize 2048 -validity 10000

apktool b TestApp

jarsigner -keystore test.keystore -storepass password -keypass password TestApp\dist\TestApp.apk android

# Install new APK
adb install TestApp\dist\TestApp.apk

# Install Burp CA to User Certs
mv cacert.der cacert.cer
adb push burpca.cer /mnt/sdcard
Settings -> Security -> Install from SD Card
```
