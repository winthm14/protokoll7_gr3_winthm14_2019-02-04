# Protokoll 7 <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/30/HTL_Kaindorf_Logo.svg/300px-HTL_Kaindorf_Logo.svg.png" alt="">  
  
Professor: SX  
Übungsort: Kaindorf   
Übungsdatum: 02.04.2018  
Anwesend: Vollmaier Alois, Wegl Patrick, Winter Matthias, Winter Thomas  
Abwesend: Sarah Vezonik, Mercedes Wesonig

## Aufgabenstellung  
Das Ziel war es, einen Modbusfähigen Temperatursensor einzurichten, welcher dann auf dem PC im Therminal ausgelesen wird.
  
## Grundlagen zu Modbus  
Das Modbus-Protokoll wurde 1979 zur Komunizierung mit SPS Geräten entwickelt. In der Automatisierungstechnik wird dieses Protokoll sehr gerne verwendet da es sich um einen offenen Standard handelt, welcher lösungen mit RS-232, RS-485 Busse und TCP/IP-Netzwerke Ermöglicht. Weiters ist Modbus ein zustandsloses Protokoll.  
  
Der Kommunikationsablauf beruht auf einem Server/Client Prinzip. Der Client (zum Beispiel ein PC) sendet einen Request zum Server (zum Beispiel ein Aktor oder Sensor). Dieser antworter mit einer Response.  
<img src="https://raw.githubusercontent.com/winthm14/Protokoll-5/master/Server%3AClient.tif" alt="">  
  
Bei der eigentlichen Datenübertragung werden drei Varianten unterschieden:  
 * **Modbus ASCII**  
    - Rein Textuelle byteweise Übertragung von Daten. Frames beginnen mit einem Doppelpunkt.  
 * **Modbus RTU**  
    - Binäre byteweise Übertragung von Daten.  
 * **Modbus TCP**  
    - Übertragung der Daten in TCP Paketen
### Modbus-Gateway  
Ein Modbus-Gateway ist in der Lage verschiedene Modbus-Varianten miteinander zu verbinden, also zum Beispiel die Verbindung eines über die UART-Schnittstelle erreichbaren Sensors mit einem über TCP/IP erreichbaren PC.  
Das Modbus Application Layer Protocol definiert dabei als Frame sogenannte **Protocol Data Units (PDU)**. Diese enthalten noch kein Adressierungsschema, da unterschiedliche Varianten (UART, TCP, ...) auch unterschiedliche Adressierungsarten verwenden.  
Die zusätzlichen Spezifikationen für die jeweiligen Varianten definieren dann auch zusätzliche Frame-Felder für die Adressierung und Fehlererkennung, wodurch dann die **Application Data Unit (ADU)** entsteht.  
  
<img src="https://www.researchgate.net/profile/Naixue_Xiong3/publication/281692567/figure/fig2/AS:331936288526339@1456151186841/MODBUS-Protocol-PDU-and-ADU.png" alt="">   
  
### Daten-Modell  
Das Modbus Daten-Modell unterscheidet vier Tabellen (Adressräume) für:  
* **Discrete Inputs**: Einzelnes Bit, kann nur gelesen werden. **Beispiele**: Taster, Endschalter,...  
* **Coils**: Einzelnes Bit, kann geleschen und beschrieben werden. **Beispiele**: LED, Relais,...  
* **Input Registers**: 16-Bit Wert, kann nur gelesen werden. **Beispiele**: Temperatursensor, ADC,...  
* **Hold-Registers**: 16-Bit Wert, kann beschrieben und gelesen werden. **Beispiele**: PWM-Einheit, DAC,...  
  
## Lösungsansätze  
### Modbus-TCP  
Hiermit wäre es möglich einen kabellosen Temperatursensor über W-LAN ein zurichten. Hierfür würde die Möglichkeit bestehen einen kleinen Einplatienencomputer wie zum Beispiel den Raspberry Pi Zero als Modbus Gateway zu verwenden. Zusätzlich könnte man zum Auswerten eine Android App schreiben und die Messwerte Grafisch ausgeben lassen. 
<img src="https://user-images.githubusercontent.com/43165765/55355726-6b7ca000-54c9-11e9-93ee-ee4105a7c79a.png" alt="">  
### Modbus-ASCII  
Die zweite Lösung wäre, einen Kabelgebundenen Temperatur Sensor einzurichten welcher mit dem Modbus-ASCII Protokoll arbeitet. Hierfür könnte man einen UART zu RS485 Umsetzer nach dem Arduino Nano setzen und diesen dann dierekt an eine SPS anschließen. In diesem Fall wäre der Arduino der Client und die SPS der Server.  
Diese Lösung wird auch in einer etwas abgeänderten Form in unserer Laboreinheit verwendet. Hierfür werden die UART Signale nicht auf RS485 übersetzt sondern mit dem UART auf USB converter umgesetzt um die Messdaten direkt am PC im Therminal ausgeben zu lassen.  
<img src="https://user-images.githubusercontent.com/43165765/55355951-f2317d00-54c9-11e9-8542-667f165ebcd0.png" alt="">  
  
## Datenanfrage (Request)  
Dieses nachfolgende Datenframe wurde im Unterricht verwendet um die Request zum Auslesen des Sensors zu senden. 
```
:010400000001 _ _ <CR><LF>
```  

**Beschreibung des Modbus ASCII Frame's**  
  
:|01|04|0000|0001|_ _|<CR><LF>
  
```:``` -> Start Frame  
```01``` -> Adresse des Geräts am Bus  
```04``` -> Read Input Register  
```0000``` -> Inputregister 1 für die Temperatur  
```0001``` -> Anzahl der Gewählten Input Register  
```_ _``` -> LRC/Prüfsumme  
```<CR><LF>``` -> End-Frame   
  
### Bilden der Prüfsumme(LRC)  
Anhand des folgenden Beispiels wird die Prüfsumme gebildet:  
``:0401000A000868``  
Übertragen werden folgende Bytes:  
0x3a **0x30 0x34 0x30 0x31 0x30 0x30 0x30 0x41 0x30 0x30 0x30 0x38** 0x36 0x38 0x0d 0x0a  
Die fett geschribenen Bytes werden in einer 8-Bit Addition ohne berücksichtigung des Überlaufs zusammen gerechnet.  
Als Ergebnis bekommt man dan **0x98**  
Dieses Ergebnis wird im Anschluss dem Zweierkomplimentsystem unterzogen und man bekommt das Ergebnis: **0x68**  

## Antwort (Response)  
Dieses nachfolgende Datenframe wurde im Unterricht verwendet um die Response vom Microcontroller zum PC zu senden
```
:010402xxxx _ _ <CR><LF>
```  

**Beschreibung des Modbus ASCII Frame's**  
  
:|01|04|02|xxxx|_ _|<CR><LF>
  
```:``` -> Start Frame  
```01``` -> Adresse des Geräts am Bus  
```04``` -> Read Input Register  
```02``` -> Anzahl der Bytes  
```xxxx``` -> Temperaturwert  
```_ _``` -> LRC/Prüfsumme  
```<CR><LF>``` -> End-Frame  
  
## Übertragung von den Temperaturwerten  
Die Temperaturwerte werden als 16Bit Werte übertragen. Weiters werden die werte in Festkommacodierung übertragen somit sind links und rechts vom Komma 8 Bit.
Um nun vom Temperaturwert z.B 23,5°C zum hex Wert zu kommen muss man den wert zuerst mit 256 Multiplizieren und danach in eine Hexadezimalzahl umwandeln -> 23,5 * 256 = 6016 => 1780hex  
Somit würde die Response wie folgt aussehen:

```
:0104021780 _ _ <CR><LF>
```   
## Auslesen des Temperatursensors  
Nachdem die Grundlagen jedem bekannt waren, wurde der Fokus auf das Realisieren einer funktionsfähigen Temperatureinheit gelegt. Verwendet wurde dafür der Integrierte   
Temperatursensor des Atmega328P des Arduino Nano.  
Die meisten neueren AVR-Chips verfügen über einen internen Temperatursensor. Bei der verwendung dieses Temperatursensors ist zu beachten, dass dieser nicht sehr genau ist. Jedoch gibt es Anwendungen in denen er Verwendung findet.  
**Hier eine kleine Liste der AVR-Chips mit eingebautem Temperatursensor:**  
- ATmega168A  
- Atmega168P
- Atmega328
- Atmega328P
- Atmega32U4  
### Verwendung  
Dadurch dass der Temperatursensor des Atmega328P im Chip integriert ist, ist es unvermeidbar dass dieser durch das Arbeiten des Microcontrollers den Messwert verändert.  
Dadurch ist dieser Sensor für das Messen der Umgebungstemperatur nahezu ungeeignet. Ausgenommen der Microcontroller befindet sich für mindestens 10 Minuten im Sleep-Modus und   
misst dann nur kurz die Temperatur und wird danach wider in den Sleep-Modus zurückversetzt.  
Die sinnvollste Verwendung des integrierten Temperatursensors währe es die Innentemperatur, welche bei neueren AVR-Chips 85 Grad Celsius nicht überschreiten darf, zu überwachen.  
Falls doch der Fall eintritt, dass der Microcontroller zu heiß wird, kann dießer Abgeschaltet werden.

### Genauigkeit  
Laut Datenblatt kann die gemessene Temperatur um 10°C von der realen Temperatur abweichen. Diese genauigkeit kann aber noch verbessert werden. Wenn man die Verstärkung und den Offset  
misst, kann man eine Genauigkeit von +/-2 Grad Celsius erreichen.  
Da aufgrund von Fertigungstolleranzen jeder Chip des gleichen Typs andere Messwerrte Liefert, sollten diese immer kalibriert werden.  
## Programm-Code  
####Beschreibung des Programmes  
Im Atmega328P werden analoge Eingangsspannungen im Analog-Digital-Wandler mittels Successiver Approximation in 10-bit Digitalwerte umgewandeld. Der kleinste Wert enspricht GND,  
der maximale Wert entspricht der ausgewählten Referenzsspannung minus ein LSB.  
  
Die Referenzspannung für den ADC kann durch die Bits **REFS1** und **REFS0** im **ADMUX**-Register ausgewählt werden. Mögliche Refernzspannungen sind die Versorgungsspannung, eine  
externe Referenzspannung oder wie in unserem Fall die Interne Referenzspannung, welche die Bandgapspannung ist und **1,1V** beträgt.  
Das 10-bit Ergebnis welches vom ADC kommt wird in den ADC-Data Registern abgelegt. Dies kann optional linksbündig in das **ADCH**Register und **ADCL**Register geschrieben werden.  
Die Einstellung erfolgt mit dem **ADLAR**-Bit im ADMUX-Reister.  
Zuerst wurden diese Register des ADC’s initialisiert und der app.modbus.frameIndex auf -1 gesetzt um in einen Idle Zustand zu gehen.  
Im nächsten Schritt wird der Wert des ADCH Registers in eine Variable gespeichert und der ADC gestartet indem das ADCH Register Gesetzt wird. In diesem Fall genügt es nur das ADCH Register  
Auszulesen, da die letzten beiden Bits des Messergebnisses durch etwaige störungen zu ungenau und somit vernachlässigbar sind.  
Am Ende erfolgt die Übertragung ders Temperaturwertes auf die Konsole des Computer's.  
### app.c  
```c
void app_init (void)
{
  memset((void *)&app, 0, sizeof(app));
  ADMUX = (1 << REFS1) | (1<< REFS0) | ( 1<< ADLAR) | 0x08;
  ADCSRA = (1<< ADEN) | 7; // 7 bedeutet durch 128, dass sind 125 kHz
  app.modbus.frameIndex = -1;
}

void app_task_16ms (void) 
{
  app.adch = ADCH;
  ADCSRA |= (1<< ADSC); //Starte ADC
}

void app_handleUartByte(char c)
{
  struct Modbus *p = &app.modbus;
  if (p ->frameIndex < 0)
    {
      return;
    }
    if( c == ':')
      {
        p ->frameIndex = 0;
        p->frameError = 0;
      }
      else if ( c == '\n')
        {
        if (p->frameError == 0)
          {
            app_parseModbusFrame();
          }
        }
        else if (p ->frameIndex < 16)
          {
            p ->frameIndex [p ->frameIndex] = c;
          }
          else if (p -> errCnt == 0)
            {
              p->frameError = 1;
              if(p->errCnt < 0xffff)
                {
                  p-> errCnt++;
                }
            }
}


```  
### app.h  
```c
struct Modbus // Struktur um alle Komponenten
{
  char frame[16];
  int8_t frameIndex;
  uint16_t frameError;
  uint16_t errCnt;
};

struct App
{
  uint8_t adch;
  struct Modbus modbus;
};
```  
### sys.c
```c
ISR (SYS_UART_RECEIVE_VECTOR)
{
  static uint8_t lastChar;
  uint8_t c = SYS_UDR;
  
  if (c=='R' && lastChar=='@')
    {
      wdt_enable(WDTO_15MS);
      wdt_reset();
      while(1) {};
    }
    lastChar = c;
    app_handleUartByte(c);
```










