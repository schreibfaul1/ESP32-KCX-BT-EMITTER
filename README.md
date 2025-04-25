# ESP32 KCX_BT_EMITTER

![KCX_BT_EMITTER](https://github.com/schreibfaul1/ESP32-KCX-BT-EMITTER/assets/26044260/4857e72a-ae46-493c-8352-fbcc0a9edc78)
![KCX_BT_EMITTER_back](https://github.com/schreibfaul1/ESP32-KCX-BT-EMITTER/assets/26044260/c884e1fb-5922-4dd6-a23b-bdfb6ad25d2d)
![image](https://github.com/schreibfaul1/ESP32-KCX-BT-EMITTER/assets/26044260/9d4e56c2-2904-44a2-a356-20bae7a136a3)

example:
````c++
#include "Arduino.h"
#include "KCX_BT_Emitter.h"

#define BT_EMITTER_RX    45  // TX pin - KCX Bluetooth Transmitter
#define BT_EMITTER_TX    38  // RX pin - KCX Bluetooth Transmitter
#define BT_EMITTER_LINK  19  // high if connected
#define BT_EMITTER_MODE  20  // high transmit - low receive


KCX_BT_Emitter  bt_emitter(BT_EMITTER_RX, BT_EMITTER_TX, BT_EMITTER_LINK, BT_EMITTER_MODE);

char buff[100];
bool f_bt_emitter = false;

void setup() {
    Serial.begin(115200);
    bt_emitter.begin();
    bt_emitter.userCommand("AT+GMR?");      // get version
    bt_emitter.userCommand("AT+VMLINK?");   // get all mem vmlinks
    bt_emitter.userCommand("AT+VOL?");      // get volume (in receiver mode 0 ... 31)
    bt_emitter.userCommand("AT+BT_MODE?");  // transmitter or receiver
}

void loop(){
    bt_emitter.loop();
    vTaskDelay(1);
    if (Serial.available()){
        String r = Serial.readStringUntil('\n');
        Serial.printf("Serial: %s\n", r.c_str());
        if (r.startsWith("btp")){
            uint16_t i = 0;
            while (bt_emitter.list_protokol(i)){
                log_e("%s", bt_emitter.list_protokol(i));
                i++;
            }
        }
    }
}

void kcx_bt_info(const char* info, const char* val){
    Serial.printf("BT-Emitter: %s %s\n", info, val);
    if(!strcmp(info, "KCX_BT_Emitter found"))f_bt_emitter = true;
}

void kcx_bt_status(bool status) { // is always called when the status changes fron disconnected to connected and vice versa
    if(status) { Serial.printf("BT-Emitter:  Status -> Connected\n");   }
    else       { Serial.printf("BT-Emitter:  Status -> Disconnected\n");}
}

void kcx_bt_memItems(const char* jsonItems){ // Every time an item (name or address) was added, a JSON string is passed here
    Serial.printf("bt_memItems %s\n", jsonItems);
}

void kcx_bt_scanItems(const char* jsonItems){ // Every time an item (name and address) was scanned, a JSON string is passed here
    Serial.printf("bt_scanItems %s\n", jsonItems);
}

void kcx_bt_modeChanged(const char* m){ // Every time the mode has changed
    if(strcmp("RX", m) == 0) {
      Serial.printf("receiver mode");
    }
    if(strcmp("TX", m) == 0) {
      Serial.printf("emitter mode");
    }
}




````

Serial port protocol:
The last 100 read/write commands are stored
````c++
    uint16_t i = 0;
    while(bt_emitter.list_protokol(i)){
        log_i("%s", bt_emitter.list_protokol(i));
        i++;
    }
````
