# Práctica 6. Gerard Cots y Joel J. Morera

## Ejercicio practico 1 : Lectura/Escritura de memoria SD

###### **Funcionamiento**

Paragraph

###### **Código del programa**

```cpp
#include <SPI.h>
#include <SD.h>

File myFile;

void setup()
{
  Serial.begin(115200);

  Serial.print("Iniciando SD ...");
  if (!SD.begin(4))
  {
    Serial.println("No se pudo inicializar");
    return;
  }
  Serial.println("inicializacion exitosa");
 
  myFile = SD.open("archivo.txt");//abrimos  el archivo 
  if (myFile)
  {
    Serial.println("archivo.txt:");
    while (myFile.available()) 
    {
    	Serial.write(myFile.read());
    }
    myFile.close(); //cerramos el archivo
  } 
  else 
  {
    Serial.println("Error al abrir el archivo");
  }
}

void loop() {}
```

###### **Salida del puerto serie**

Paragraph

```
```

###### **Montaje**

![Montaje escaner](/images/escan_mount.png)

## Ejercicio practico 2 : Lectura de etiqueta RFID

###### **Funcionamiento**

Paragraph

###### **Código del programa**

- platformio.ini:

```
[env:esp32doit-devkit-v1]
platform = espressif32
board = esp32doit-devkit-v1
framework = arduino
monitor_speed = 115200
monitor_port = /dev/ttyUSB0
lib_deps = miguelbalboa/MFRC522 @ ^1.4.10
```

- main.cpp:

```cpp
#include <SPI.h>
#include <MFRC522.h>

#define RST_PIN	9    //Pin 9 para el reset del RC522
#define SS_PIN	10   //Pin 10 para el SS (SDA) del RC522
MFRC522 mfrc522(SS_PIN, RST_PIN); //Creamos el objeto para el RC522

void setup() 
{
	Serial.begin(9600); //Iniciamos la comunicación  serial
	SPI.begin();        //Iniciamos el Bus SPI
	mfrc522.PCD_Init(); // Iniciamos  el MFRC522
	Serial.println("Lectura del UID");
}

void loop()
{
	// Revisamos si hay nuevas tarjetas  presentes
	if ( mfrc522.PICC_IsNewCardPresent()) 
    {  
  		//Seleccionamos una tarjeta
        if ( mfrc522.PICC_ReadCardSerial()) 
        {
                // Enviamos serialemente su UID
                Serial.print("Card UID:");
                for (byte i = 0; i < mfrc522.uid.size; i++) 
                {
                        Serial.print(mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " ");
                        Serial.print(mfrc522.uid.uidByte[i], HEX);   
                } 
                Serial.println();
                // Terminamos la lectura de la tarjeta  actual
                mfrc522.PICC_HaltA();         
        }      
	}	
}
```

###### **Salida del puerto serie**

Paragraph

```

```

## Ejercicios de subida de nota

### Ejercicio de subida de nota. Parte 1 : Lectura de codigos RFID y escritura en memoria SD

###### **Funcionamiento**

Paragraph

###### **Código del programa**

-platformio.ini :

```
[env:esp32doit-devkit-v1]
platform = espressif32
board = esp32doit-devkit-v1
framework = arduino
monitor_speed = 115200
monitor_port = /dev/ttyUSB0
lib_deps = miguelbalboa/MFRC522 @ ^1.4.10
```

- main.cpp :

```cpp
#include <SPI.h>
#include <SD.h>
#include <MFRC522.h>

#define RST_PIN	9    //Pin 9 para el reset del RC522
#define SS_PIN	10   //Pin 10 para el SS (SDA) del RC522
MFRC522 mfrc522(SS_PIN, RST_PIN); //Creamos el objeto para el RC522

File myFile;

void getNewCards();
void initSD();
void writeCodes();

void setup() 
{
	Serial.begin(115200); //Iniciamos la comunicación  serial
	SPI.begin();        //Iniciamos el Bus SPI
	mfrc522.PCD_Init(); // Iniciamos  el MFRC522
	Serial.println("Lectura del UID");
    initSD();
}

void loop()
{
    getNewCards();
    writeCodes();
}

void initSD()
{
    Serial.print("Iniciando SD ...");
    if (!SD.begin(4))
    {
        Serial.println("No se pudo inicializar");
        return;
    }
    Serial.println("Inicializacion exitosa");
    
    myFile = SD.open("archive.log");//abrimos  el archivo 
}

void writeCodes()
{
    if (myFile)
    {
        while (myFile.available()) 
        {
            String ln = act_time + " - " + message ;
            myFile.write(ln);
        }
    } 
    else 
    {
        Serial.println("Error al abrir el archivo");
    }
}

void getNewCards()
{
    // Revisamos si hay nuevas tarjetas  presentes
	if ( mfrc522.PICC_IsNewCardPresent()) 
    {  
  		//Seleccionamos una tarjeta
        if ( mfrc522.PICC_ReadCardSerial()) 
        {
                // Enviamos serialemente su UID
                Serial.print("Card UID:");
                for (byte i = 0; i < mfrc522.uid.size; i++) 
                {
                    message = message + String(mfrc522.uuid.uidByte[i],HEX);
                    Serial.print(mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " ");
                    Serial.print(mfrc522.uid.uidByte[i], HEX);   
                } 
                Serial.println();
                // Terminamos la lectura de la tarjeta  actual
                mfrc522.PICC_HaltA();         
        }      
	}
}
```

###### **Salida del puerto serie**

Paragraph

```

```

###### **Fichero de la memoria SD**

```

```

###### **Montaje**

![Montage RFID y SD](./images/rfid_sd.png)

### Ejercicio de subida de nota. Parte 2 : Página web donde se pueda consultar la lectura del lector RFID

###### **Funcionamiento**

Paragraph

###### **Código del programa**

- platformio.ini:

```
[env:esp32doit-devkit-v1]
platform = espressif32
board = esp32doit-devkit-v1
framework = arduino
monitor_speed = 115200
monitor_port = /dev/ttyUSB0
lib_deps =  miguelbalboa/MFRC522 @ ^1.4.10
            ottowinter/ESPAsyncWebServer-esphome @ ^3.0.0
```

- main.cpp:

```cpp
#include <SPI.h>
#include <MFRC522.h>
#include "WiFi.h"
#include "SPIFFS.h"
#include "ESPAsyncWebServer.h"

//Server vars.
const char* ssid = "*****";
const char* password =  "******";
 
AsyncWebServer server(80);
AsyncWebSocket ws("/ws");
 
AsyncWebSocketClient * globalClient = NULL;
 
String  message;  // Var. Message

#define RST_PIN	9    //Pin 9 para el reset del RC522
#define SS_PIN	10   //Pin 10 para el SS (SDA) del RC522
MFRC522 mfrc522(SS_PIN, RST_PIN); //Creamos el objeto para el RC522

//Function declaration
void getNewCards();
void onWsEvent(AsyncWebSocket * server, AsyncWebSocketClient * client, AwsEventType type, void * arg, uint8_t *data, size_t len);
void initSPIFFS();
void initWiFi();
void initServer();

void setup() 
{
	Serial.begin(115200); //Iniciamos la comunicación  serial
	SPI.begin();        //Iniciamos el Bus SPI
	mfrc522.PCD_Init(); // Iniciamos  el MFRC522
	Serial.println("Lectura del UID");
    initSPIFFS();
    initWiFi();
    initServer();
}

void loop()
{
    getNewCards();
    if(globalClient != NULL && globalClient->status() == WS_CONNECTED)
    {
        globalClient -> text(message);
    }
}

void getNewCards()
{
    // Revisamos si hay nuevas tarjetas  presentes
	if ( mfrc522.PICC_IsNewCardPresent()) 
    {  
  		//Seleccionamos una tarjeta
        if ( mfrc522.PICC_ReadCardSerial()) 
        {
                // Enviamos serialemente su UID
                Serial.print("Card UID:");
                for (byte i = 0; i < mfrc522.uid.size; i++) 
                {
                    message = message + String(mfrc522.uid.uidByte[i],HEX);
                    Serial.print(mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " ");
                    Serial.print(mfrc522.uid.uidByte[i], HEX);   
                } 
                Serial.println();
                // Terminamos la lectura de la tarjeta  actual
                mfrc522.PICC_HaltA();         
        }      
	}
}
//Server functions
void initSPIFFS()
{
  if(!SPIFFS.begin()){
     Serial.println("An Error has occurred while mounting SPIFFS");
     for(;;);
  }
}

void initWiFi()
{
  WiFi.begin(ssid, password);
 
  Serial.print("Connecting to WiFi..");
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("IP: ");
  Serial.print(WiFi.localIP());
  Serial.println("");
}

void initServer()
{
  ws.onEvent(onWsEvent);
  server.addHandler(&ws);
 
  server.on("/", HTTP_GET, [](AsyncWebServerRequest *request){
    request->send(SPIFFS, "/index.html", "text/html");
  });
 
  server.begin();
}

void onWsEvent(AsyncWebSocket * server, AsyncWebSocketClient * client, AwsEventType type, void * arg, uint8_t *data, size_t len)
{
 
  if(type == WS_EVT_CONNECT){
 
    Serial.println("Websocket client connection received");
    globalClient = client;
 
  } else if(type == WS_EVT_DISCONNECT){
 
    Serial.println("Websocket client connection finished");
    globalClient = NULL;
 
  }
}
```

###### **Salida del puerto serie**

Paragraph

```

```

###### **Visualización de la página web**

- Página web:

![Página web](./images/web_rfid_p2.png)

- Código HTML:

```html
<!DOCTYPE html>
<html class="no-js" lang ='es'>
    <head>
        <meta charset='utf-8'>
        <title>Heart Rate & SPO2</title>
        <link rel='preconnect' href='https://fonts.googleapis.com'>
        <link rel='preconnect' href='https://fonts.gstatic.com' crossorigin>
        <link href='https://fonts.googleapis.com/css2?family=Poppins:wght@200&display=swap' rel='stylesheet'>
        <script type="text/javascript">
            var ws = new WebSocket("ws://" + location.hostname + "/ws");

            ws.onopen = function() {
                console.log("WebSocket connected");
            };

            ws.onmessage = function(evt) {
                var raw = evt.data;
                const data_array = raw.split(";");
                document.getElementById("time").innerHTML = data_array[0];
                document.getElementById("RFID").innerHTML = data_array[1];
            };

            ws.onerror = function (error) {
                console.log("WebSocket Error ", error);
            };

            ws.onclose = function(){
                console.log("WebSocket connection closed");
            }
        </script>
    </head>
    <style>
        html{
            display: inline-block;
            margin: 1px auto;
            text-align: center;
        }
        body{
            background-color: #252525;
            color: #c1bbbb;
            font: 200 22px 'Poppins',sans-serif
        }
        h1{
            color: #ffffff;
            font-size: 36px;
            text-align: center;
        }
        p{
            text-align: center;
        }
        div{
            background-color: #6a676734;
            border-radius: 15px;
            margin: 2% 15% 2% 15%;
            padding: 0.5%;
        }
    </style>
            
    <body>
        <div>
            <h1>RFID's codes</h1>
            <p>Time : <div id="time">---</div></p>
            <p>RFID Code : <div id="RFID">---</div></p>
        </div>
    </body>
</html>
```

> **Nota:** Se puede ver un video de la actualización de los datos en tiempo real en `images/web_rfid.webm`

###### **Montaje**

![Montage pagina web](./images/web_rfid_mount.png)
