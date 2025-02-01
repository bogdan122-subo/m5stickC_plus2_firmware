NULL,               // Task parameters 
        2,                  // Task priority (0 to 3), loopTask has priority 2. 
        &xHandle            // Task handle (not used) 
    ); 
#endif 
 
  boot_screen_anim(); 
 
  startup_sound(); 
 
  if (bruceConfig.wifiAtStartup) { 
    displayInfo("Connecting WiFi..."); 
    wifiConnectTask(); 
    tft.fillScreen(bruceConfig.bgColor); 
  } 
 
  #if ! defined(HAS_SCREEN) 
    // start a task to handle serial commands while the webui is running 
    startSerialCommandsHandlerTask(); 
  #endif 
 
  delay(200); 
  wakeUpScreen(); 
 
  if (bruceConfig.startupApp != "" && !startupApp.startApp(bruceConfig.startupApp)) { 
    bruceConfig.setStartupApp(""); 
  } 
} 
 
/************** 
**  Function: loop 
**  Main loop 
**************/ 
#if defined(HAS_SCREEN) 
void loop() { 
  #if defined(HAS_RTC) 
    RTC_TimeTypeDef _time; 
  #endif 
  bool redraw = true; 
  long clock_update=0; 
  mainMenu.begin(); 
 
  // Interpreter must be ran in the loop() function, otherwise it breaks 
  // called by 'stack canary watchpoint triggered (loopTask)' 
#if !defined(LITE_VERSION) 
  #if !defined(ARDUINO_M5STACK_CORE) && !defined(ARDUINO_M5STACK_CORE2) 
    if(interpreter_start) { 
      interpreter_start=false; 
      interpreter(); 
      previousMillis = millis(); // ensure that will not dim screen when get back to menu 
      //goto END; 
    } 
  #endif 
#endif 
  tft.fillScreen(bruceConfig.bgColor); 
  bruceConfig.fromFile(); 
 
 
  while(1){ 
    if(interpreter_start) goto END; 
    if (returnToMenu) { 
      returnToMenu = false; 
      tft.fillScreen(bruceConfig.bgColor); //fix any problem with the mainMenu screen when coming back from submenus or functions 
      redraw=true; 
    } 
 
    if (redraw) { 
      if(bruceConfig.rotation & 0b01) mainMenu.draw(float((float)tftHeight/(float)135)); 
      else mainMenu.draw(float((float)tftWidth/(float)240)); 
      clock_update=0; // forces clock drawing 
      redraw = false; 
    } 
 
    handleSerialCommands(); 
#ifdef HAS_KEYBOARD 
    checkShortcutPress();  // shortctus to quickly start apps without navigating the menus 
#endif 
 
    if (check(PrevPress)) { 
      checkReboot(); 
      mainMenu.previous(); 
      redraw = true; 
    } 
    /* DW Btn to next item */ 
    if (check(NextPress)) { 
      mainMenu.next(); 
      redraw = true; 
    } 
 
    /* Select and run function */ 
    if (check(SelPress)) { 
      mainMenu.openMenuOptions(); 
      drawMainBorder(true); 
      redraw=true; 
    } 
    // update battery and clock once every 30 seconds 
    // it was added to avoid delays in btns readings from Core and improves overall performance 
    if(millis()-clock_update>30000) { 
      drawBatteryStatus(); 
      if (clock_set) { 
        #if defined(HAS_RTC) 
          _rtc.GetTime(&_time); 
          setTftDisplay(12, 12, bruceConfig.priColor, 1, bruceConfig.bgColor); 
          snprintf(timeStr, sizeof(timeStr), "%02d:%02d", _time.Hours, _time.Minutes); 
          tft.print(timeStr); 
        #else 
          updateTimeStr(rtc.getTimeStruct()); 
          setTftDisplay(12, 12, bruceConfig.priColor, 1, bruceConfig.bgColor); 
          tft.print(timeStr); 
        #endif 
      } 
      else { 
        setTftDisplay(12, 12, bruceConfig.priColor, 1, bruceConfig.bgColor); 
        tft.print("BRUCE " + String(BRUCE_VERSION)); 
      } 
      clock_update=millis(); 
    } 
  } 
  END: 
  delay(1); 
} 
#else 
 
// alternative loop function for headless boards 
#include "modules/others/webInterface.h" 
 
void loop() { 
  setupSdCard(); 
  bruceConfig.fromFile(); 
 
  if(!wifiConnected) { 
    Serial.println("wifiConnect"); 
    wifiConnectMenu(WIFI_AP);  // TODO: read mode from config file 
  } 
  Serial.println("startWebUi"); 
  startWebUi(true);  // MEMO: will quit when check(EscPress) 
} 
#endif
// Определяем логотип дельфина (пример в формате XBM)
const unsigned char dolphin_logo[] PROGMEM = {
  0x00, 0x3C, 0x42, 0xA5, 0x81, 0xA5, 0x99, 0x42, 0x3C, 0x00
}; // Этот массив можно заменить на более подходящий для дельфинчика в XBM

void setup() {
    M5.begin(); // Инициализация M5StickC Plus
    M5.Lcd.fillScreen(TFT_ORANGE);  // Устанавливаем оранжевый фон экрана
    M5.Lcd.drawXBitmap(40, 20, dolphin_logo, 64, 64, TFT_BLACK); // Рисуем дельфинчика
    delay(2000);  // Задержка 2 секунды для отображения дельфина
}

void loop() {
    // Пусто, так как это только экран загрузки
}
