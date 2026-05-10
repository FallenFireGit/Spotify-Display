#include <Arduino.h>
#include <ArduinoJson.h>
#include <Adafruit_GFX.h>
#include <Adafruit_ST7735.h>
#include <WiFi.h>
#include <SpotifyEsp32.h>
#include <SPI.h>

#define TFT_CS   1
#define TFT_RST  2
#define TFT_DC   3
#define TFT_SCLK 4
#define TFT_MOSI 5

#define BTN_PREV      6
#define BTN_PLAYPAUSE 7
#define BTN_NEXT      8
//Fill these in with your information
const char* SSID          = "";
const char* PASSWORD      = "";
const char* CLIENT_ID     = "";
const char* CLIENT_SECRET = "";

String lastArtist;
String lastTrackname;
bool isPlaying = true;

unsigned long lastPressPrev      = 0;
unsigned long lastPressPlayPause = 0;
unsigned long lastPressNext      = 0;
const unsigned long DEBOUNCE_MS  = 250;

Spotify         sp(CLIENT_ID, CLIENT_SECRET);
Adafruit_ST7735 tft = Adafruit_ST7735(TFT_CS, TFT_DC, TFT_MOSI, TFT_SCLK, TFT_RST);


bool debounced(int pin, unsigned long &lastTime) {
    if (digitalRead(pin) == LOW && (millis() - lastTime) > DEBOUNCE_MS) {
        lastTime = millis();
        return true;
    }
    return false;
}

void showStatus(const char* msg) {
    tft.fillRect(0, 100, tft.width(), 28, ST77XX_BLACK);
    tft.setCursor(10, 108);
    tft.setTextColor(ST77XX_YELLOW);
    tft.print(msg);
    tft.setTextColor(ST77XX_WHITE);
}


void setup() {
    Serial.begin(115200);

    pinMode(BTN_PREV,      INPUT_PULLUP);
    pinMode(BTN_PLAYPAUSE, INPUT_PULLUP);
    pinMode(BTN_NEXT,      INPUT_PULLUP);

    tft.initR(INITR_BLACKTAB);
    tft.setRotation(1);
    tft.fillScreen(ST77XX_BLACK);
    tft.setTextColor(ST77XX_WHITE);
    tft.setTextSize(1);
    Serial.println("TFT Initialized!");

    WiFi.begin(SSID, PASSWORD);
    Serial.print("Connecting to WiFi...");
    tft.setCursor(0, 0);
    tft.print("Connecting WiFi...");
    while (WiFi.status() != WL_CONNECTED) {
        delay(1000);
        Serial.print(".");
    }
    Serial.println("\nConnected!");

    tft.fillScreen(ST77XX_BLACK);
    tft.setCursor(0, 0);
    tft.print(WiFi.localIP().toString());

    sp.begin();
    while (!sp.is_auth()) {
        sp.handle_client();
    }
    Serial.println("Authenticated");
    tft.fillScreen(ST77XX_BLACK);
}


void loop() {
    // Previous track
    if (debounced(BTN_PREV, lastPressPrev)) {
        sp.previous();
        showStatus("<< Previous");
        lastArtist    = "";
        lastTrackname = "";
        isPlaying     = true;
        delay(300);
    }

    // Play / Pause
    if (debounced(BTN_PLAYPAUSE, lastPressPlayPause)) {
        sp.start_resume_playback();
        isPlaying = !isPlaying;
        showStatus(isPlaying ? "> Playing" : "|| Paused");
        delay(300);
    }

    // Next track
    if (debounced(BTN_NEXT, lastPressNext)) {
        sp.skip();
        showStatus(">> Next");
        lastArtist    = "";
        lastTrackname = "";
        isPlaying     = true;
        delay(300);
    }

    // Spotify polling
    String currentArtist    = sp.current_artist_names();
    String currentTrackname = sp.current_track_name();

    if (lastArtist != currentArtist &&
        currentArtist != "Something went wrong" &&
        !currentArtist.isEmpty()) {

        tft.fillScreen(ST77XX_BLACK);
        lastArtist = currentArtist;
        Serial.println("Artist: " + lastArtist);
        tft.setCursor(10, 10);
        tft.print(lastArtist.c_str());
    }

    if (lastTrackname != currentTrackname &&
        currentTrackname != "Something went wrong" &&
        currentTrackname != "null") {

        lastTrackname = currentTrackname;
        Serial.println("Track: " + lastTrackname);
        tft.setCursor(10, 40);
        tft.print(lastTrackname.c_str());
    }

    delay(2000);
}
