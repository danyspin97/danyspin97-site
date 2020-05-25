---
title: 'Raspberry Pi e SSH🇮🇹'
date: "2020-05-17T20:30:00"
categories: ["workshop"]
tags: ["raspberry", "fablabrc", "security"]
---
class: center, middle
name: start

# Raspberry Pi e SSH ![:i](fab fa-raspberry-pi)

---
class: center, middle

# Sono

# Danilo Spinella

---
class: left, top

# Obbiettivi

Cos'è il Raspberry Pi

Protocollo SSH

---
class: center, middle

# Parte 1

![](/img/raspberry_ssh/raspberry_board.png)

---
class: center, middle

#Il Raspberry Pi è <br>computer a singola scheda

---
class: left, middle
# Caratteristiche tecniche

ARM Quad-core 64bit 1,5GHz

1/2/4GB di RAM

.center[.small[(Relative ad un Raspberry Pi 4)]]
---
class: left, middle
# Caratteristiche tecniche

USB 3.0

Micro HDMI, Mini DPI

.center[.small[(Relative ad un Raspberry Pi 4)]]
---
class: center, middle

# Progetti realizzati con il Raspberry

---
background-image: url(/img/raspberry_ssh/pi-top.jpg)

---

background-image: url(https://chickadeesolutions-woo-6drz229racfsh09p7h.netdna-ssl.com/wp-content/uploads/2019/05/61o9O1WwAdL.jpg)

---

background-image: url(https://www.windowscentral.com/sites/wpcentral.com/files/styles/xlarge/public/field/image/2017/02/kodi-17-windows-10.jpg?itok=kt-EXttz)

---
class: center, middle

# Rasbian

Distribuzione Linux basata su Debian

---
class: center, middle

# Problema

Come ci connettiamo in remoto?

---
class: center, middle

# Protocollo SSH

Secure Shell

(successore di telnet)

---

class: center, middle

SSH è un protocollo **crittografico** per la connessione _remota_ di due macchine

---
class: center, middle

```bash
ssh pi@192.168.1.<xxxx>
```

---
class: left, middle

# Crittografia

Simmetrica

Asimmetrica

---
class: center, middle

# Crittografia Simmetrica

Le due parti utilizzeranno la stessa chiave per cifrare e decifrare i messaggi

---
class: left, middle

# Vantaggi/svantaggi

\+ Operazioni di cifratura e decifratura molto veloci

\- Non è possibile verificare l'autenticità del mittente

---
class: center, middle

# Crittografia Asimmetrica (Cifratura)

Alice userà la chiave pubblica di Bob per cifrare i messaggi da mandare a Bob

---
class: center, middle

# Crittografia Asimmetrica (Decifratura)

Bob userà la propria chiave privata per decrifare i messaggi ricevuti da Alice

---
class: left, middle

# Vantaggi/Svantaggi

\+ Messaggio sicuro, solo il possessore della chiave privata può decifrare

\- Operazioni di cifratura/decifratura molto lente

---
class: center, middle

# Come funziona SSH

---
class: center, middle

Utilizzando la crittografia asimmetrica, Alice si accerta dell'identità di Bob (e viceversa)

---
class: center, middle

Alice e Bob si accordano su una chiave da utilizzare per cifrare la connessione

---
class: center, middle

D'ora in poi verrà utilizzata la crittografia asimmetrica

---
class: center, middle

# Generiamo la coppia di chiavi

(sul dispositivo da cui ci vogliamo connettere)

---
class: center, middle

```bash
$ ssh-keygen
```

---
class: left, middle

```
Generating public/private rsa key pair.
Enter file in which to save the key (/home/danyspin97/.ssh/id_rsa):
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/danyspin97/.ssh/id_rsa
Your public key has been saved in /home/danyspin97/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:4FgJiZ86Ezu/W/yV8TvzC3LBccMXyZBeAKiAcy2dyDU pi@raspbian
The key's randomart image is:
+---[RSA 3072]----+
|   .+.=E. ...o=..|
|  .o.B =..   o +.|
|   .o.* .   o = .|
|  . o+ o   . + o |
|   +. . S . o    |
|  *  .     + .   |
|   =  o   + +    |
|    .. . . ooo   |
|    oo  .   .+o. |
+----[SHA256]-----+

```

---
class: left, middle

# Vediamo la chiave pubblica

```bash
$ cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDF4gkxX4efIDkvmEwhPixQvmESvcYbakpZsDxISXD6xibPVFjsLOZSA09p97DOJFU4K6YPKZHWS6OlPV3OtZpu6dsxXkWt7WAEQGygHGTVg0lvGwM6KxeGHOzvOxwcDUDS5VJ6UmsqKkV+pr9v5ByZCbJoldXQi7QS8X89Q61gaTbJIAXgli7vmyaf0B6v8p5ayrgD1cdH1CWQpiZLL74A9Zw3BpKCSw/WaXCVsgYzLZ8oo/nk2kvMFc8osHjtEWZAWooJqkVgpSaXnC/M4EBWlqs7o0KdED7VUJFkNwgB5yCCNR5hSjv5xTFyookC6RqWPcVdZ+Z4HIbIQi4DZGBZXjN6IyhmnGm4qKKCdLMCRAXIRvN49PseyfBbM70MKA2CNjQdaedyGU00Vqcm7GTpMkH1XfTNobgS/lAGuIJ84v2meNXFnMmWmfAlqKRFmb82PyOSPqNynrHs6mWlMSX1Xz6/JNYQFIYK7VdlT0XguofynMoYNXXscZr5iWgMNT8= danyspin97@danyspin97-laptop
```

---
class: center, middle

Aggiungiamo questa chiave alla lista delle chiavi accettate sul Raspberry

```bash
~/.ssh/authorized_keys
```

```bash
ssh-copy-id
```

---
class: left, middle

# Grazie dell'attenzione

Ci sono domande?

---
template: start

