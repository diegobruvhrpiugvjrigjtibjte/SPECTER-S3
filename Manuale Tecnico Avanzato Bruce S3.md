# **📘 Enciclopedia Tecnica e Manuale Operativo: Mini-Bruce S3 Edition (v3.4)**

**Piattaforma Hardware:** ESP32-S3 N16R8 (Dual-Core, 240MHz, 16MB Flash, 8MB PSRAM)

**Ambito di Applicazione:** Cybersecurity Research, Protocol Analysis, RF Auditing.

## **⚠️ DICHIARAZIONE DI RESPONSABILITÀ ED ETICA (MANDATORIA)**

**\[IT\] ATTENZIONE:** Questo dispositivo è uno strumento di analisi di rete estremamente potente. L'intercettazione di traffico dati, la disconnessione forzata di dispositivi (Deauth) e la creazione di falsi punti di accesso (Phishing) su reti non autorizzate costituiscono reati penali gravi in quasi tutte le giurisdizioni mondiali. Questo materiale è fornito **esclusivamente a scopo di studio e test di penetrazione autorizzati**. L'autore declina ogni responsabilità per danni, sanzioni legali o usi impropri. **L'ignoranza della legge non è una scusa.**

**\[EN\] WARNING:** This tool is a high-grade network analysis device. Unauthorized use for data interception or network disruption is a criminal offense. This documentation is for **educational use only**.

## **1\. ANALISI DETTAGLIATA DELL'HARDWARE (BOM)**

Per garantire la stabilità del sistema sotto carico radio (WiFi \+ BLE), è fondamentale utilizzare componenti di alta qualità.

### **A. Display OLED 1.3" (Interfaccia I2C)**

Il firmware è ottimizzato per il refresh rate elevato del driver SSD1306/SSD1315.

* **Link Prodotto Consigliato:** [AliExpress \- OLED 1.3 pollici 128x64 I2C](https://it.aliexpress.com/item/1005009484470072.html?spm=a2g0o.order_list.order_list_main.5.77831802NhHhkz&gatewayAdapt=glo2ita)  
* **Specifiche Tecniche:** Risoluzione 128x64 pixel. Tecnologia auto-emissiva (non richiede retroilluminazione).  
* **Consumo:** \~20mA con tutti i pixel accesi.

### **B. Il "Cervello": ESP32-S3 N16R8**

Perché questa versione specifica?

* **16MB Flash:** Necessaria per ospitare il file system (LittleFS) con le pagine HTML di phishing e i log delle credenziali.  
* **8MB PSRAM:** Fondamentale per il buffer dello scanner WiFi e la gestione dei pacchetti BLE, che richiedono molta memoria volatile durante l'analisi dei frame.

### **C. Modulo RFID RC522 (13.56 MHz)**

Utilizzato per l'interazione con tag NFC/RFID passivi. Richiede una tensione di 3.3V estremamente pulita, poiché il rumore elettrico può impedire la lettura degli UID.

### **D. Sensore Audio (Microfono Analogico)**

Un semplice modulo con amplificatore (LM393 o simile). L'uscita AO (Analog Out) è collegata all'ADC dell'ESP32 per visualizzare le onde sonore.

## **2\. SCHEMA DI CABLAGGIO MICROSCOPICO E PINOUT**

Il layout dei cavi è critico. Cavi troppo lunghi sul bus SPI possono causare interferenze durante le trasmissioni WiFi ad alta potenza.

### **I2C \- Sottosistema Display**

| Pin Display | Pin ESP32-S3 | Funzione | Note |
| :---- | :---- | :---- | :---- |
| **VCC** | 3.3V | Alimentazione | Non collegare ai 5V\! |
| **GND** | GND | Massa |  |
| **SCL** | GPIO 8 | Clock | Frequenza: 400kHz |
| **SDA** | GPIO 9 | Data |  |

### **SPI \- Sottosistema RFID RC522**

| Pin RFID | Pin ESP32-S3 | Segnale | Descrizione |
| :---- | :---- | :---- | :---- |
| **SDA (SS)** | GPIO 2 | Chip Select | Abilita la comunicazione SPI |
| **SCK** | GPIO 3 | Clock |  |
| **MOSI** | GPIO 10 | Data Out |  |
| **MISO** | GPIO 11 | Data In |  |
| **RST** | GPIO 12 | Reset |  |

### **Controlli, Audio e Alimentazione**

* **Pulsante UP:** GPIO 4 \-\> Massa (GND)  
* **Pulsante DOWN:** GPIO 5 \-\> Massa (GND)  
* **Pulsante OK:** GPIO 6 \-\> Massa (GND)  
* **Pulsante ESC:** GPIO 7 \-\> Massa (GND)  
* **Input Audio:** GPIO 1 (Canale ADC1\_0)  
* **Alimentazione USB:** Consigliato l'uso di un cavo schermato per evitare drop di tensione durante l'invio dei pacchetti Deauth.

## **3\. ARCHITETTURA DEL FIRMWARE E LOGICA DI SISTEMA**

Il firmware non è un semplice script Arduino, ma un'applicazione **multitasking** complessa basata su **FreeRTOS**.

### **🇮🇹 ITALIANO: Gestione Dual-Core**

1. **Core 0 (Radio Protocol Task):**  
   * Gestisce i driver di basso livello esp\_wifi.  
   * Iniezione di frame Raw 802.11 (livello 2 dello stack ISO/OSI).  
   * Gestione dello stack Bluetooth (NimBLE).  
   * *Perché il Core 0?* È il core solitamente riservato dal sistema per le funzioni di rete, garantendo che i processi radio non vengano interrotti dalla UI.  
2. **Core 1 (Application Logic & UI):**  
   * Rendering grafico su OLED tramite bus I2C.  
   * Campionamento segnali analogici (Audio/Oscilloscopio).  
   * Lettura RFID tramite SPI.  
   * Gestione degli eventi dei pulsanti.

### **🇺🇸 ENGLISH: Radio Protocols & Vulnerabilities**

* **Deauthentication Attack (Theory):** The attack targets the 802.11 management frames. Since these frames are unencrypted in legacy WPA2 networks, the device spoofs the MAC address of the Access Point and sends a "Deauth" command to the client. The client, believing the message comes from the router, drops the connection.  
* **Evil Portal (Social Engineering):** When a client connects, the **DNS Server** intercepts all traffic. If the user tries to go to example.com, the ESP32 returns its own IP address. The local **Web Server** then serves a fake login page.  
* **BLE Spam:** Modern smartphones are always scanning for "Fast Pair" or "Handoff" packets. By sending thousands of these packets with varying IDs, the ESP32-S3 can freeze the UI of nearby phones.

### **🇪🇸 ESPAÑOL: Procesamiento de Datos**

* **Filtrado de Señal:** El firmware aplica un filtro de media móvil a las lecturas del sensor de sonido para suavizar la visualización en pantalla.  
* **Almacenamiento:** Las credenciales capturadas se guardan en el file system interno (LittleFS). Esto permite que, incluso si el dispositivo se apaga, los datos capturados permanezcan seguros para su posterior análisis.

## **4\. GUIDA OPERATIVA AI MODULI DI ATTACCO**

### **A. Evil Portal (Phishing Setup)**

1. Impostare il nome della rete (SSID) tramite il menu "Imposta Nome".  
2. Avviare "Free Portal".  
3. L'ESP32 inizierà a trasmettere pacchetti Beacon. Una volta che una vittima si connette, il display mostrerà l'indirizzo IP assegnato.  
4. Quando l'utente inserisce i dati nella pagina fake, il display vibrerà (visivamente) e mostrerà USER e PASS in tempo reale.

### **B. Clone Attack (Beacon Spam)**

Questa funzione crea decine di reti WiFi fantasma con nomi simili o identici alle reti reali vicine. Confonde gli utenti e può essere usata per mappare il comportamento di riconnessione automatica dei dispositivi.

### **C. RFID Scan & Live Oscilloscope**

* **RFID:** Avvicinando un tag, il firmware decodifica l'header del protocollo ISO14443. Utile per identificare rapidamente il tipo di badge (es. Mifare Classic 1K).  
* **Oscilloscopio:** Mostra il "rumore" ambientale. Può essere usato per rilevare picchi di attività sonora o per testare sensori analogici esterni.

## **5\. MANUTENZIONE E RISOLUZIONE ERRORI AVANZATA**

### **Problemi Comuni (FAQ)**

1. **"I tasti non rispondono durante l'attacco":**  
   * Succede se il Task radio sta saturando il bus di sistema. Il firmware include un sistema di priorità (Priority 1 per Radio, Priority 2 per UI) per mitigare questo problema. Controlla che il cavo dei pulsanti non sia vicino all'antenna.  
2. **"L'OLED mostra pixel casuali":**  
   * Interferenza I2C. Ridurre la lunghezza dei cavi o aggiungere due resistenze di pull-up da 4.7k Ohm tra SDA/SCL e 3.3V.  
3. **"L'RFID non legge nulla":**  
   * Molti moduli RC522 su AliExpress sono difettosi o richiedono esattamente 3.3V. Se la tensione scende a 3.1V, il chip non si inizializza.  
4. **"Errore durante la compilazione (Sketch too large)":**  
   * Vai in Tools \-\> Partition Scheme e seleziona **"16M Flash (3MB APP/9.9MB FATFS)"**. Senza questa impostazione, il firmware non entrerà nella memoria.

## **6\. CONCLUSIONI E SVILUPPI FUTURI**

Il progetto Mini-Bruce S3 è in continua evoluzione. Grazie alla **PSRAM da 8MB**, è possibile in futuro aggiungere il supporto per il salvataggio dei file PCAP (sniffing del traffico) direttamente sulla memoria interna o su una scheda SD esterna.

**Ricorda:** La potenza di questo dispositivo richiede una grande responsabilità. Usalo per migliorare la tua comprensione delle reti, non per danneggiare gli altri.

*Manuale redatto per la community di ricerca S3 \- 2024*