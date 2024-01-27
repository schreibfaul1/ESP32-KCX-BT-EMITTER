example:
````c++
#include "Arduino.h"
#include "src/KCX_BT_Emitter.h"

#define BT_EMITTER_RX    45  // TX pin - KCX Bluetooth Transmitter (-1 if not available)
#define BT_EMITTER_TX    38  // RX pin - KCX Bluetooth Transmitter (-1 if not available)
#define BT_EMITTER_LINK  19  // high if connected                  (-1 if not available)
#define BT_EMITTER_MODE  20  // high transmit - low receive        (-1 if not available)


KCX_BT_Emitter  bt_emitter(BT_EMITTER_RX, BT_EMITTER_TX, BT_EMITTER_LINK, BT_EMITTER_MODE);

char buff[100];

void setup() {
  Serial.begin(115200);
  bt_emitter.begin();
}

void loop() {
  bt_emitter.loop();
  if(Serial.available()){
    int p = Serial.readBytesUntil('\n', buff, 100);
    buff[p] = '\0';
    bt_emitter.userCommand(buff);
  }
}

void kcx_bt_info(const char* info, const char* val){
    Serial.printf("BT-Emitter: %s %s\n", info, val);
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
