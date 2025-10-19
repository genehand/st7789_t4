# st7789_t4

A DMA enabled SPI driver for ST7789 LCDs. This driver code for communicating with SPI connected ST7789 LCD devices on the Teensy 4.x

This library relies on https://github.com/codewitch-honey-crisis/lcd_spi_driver_t4

```cpp
#include <Arduino.h>
#include "st7789_t4.hpp"
#include "lvgl.h"

// Screen dimension
const short SCREEN_WIDTH = 240;
const short SCREEN_HEIGHT = 320;

// Pins
const byte CS_PIN = 10; // for CS1: 38
const byte DC_PIN = 8;
const byte RST_PIN = 9;
const byte DIN_PIN = 11; // for MOSI1: 26
const byte CLK_PIN = 13; // for SCK1: 27
st7789_t4 lcd(st7789_t4_res_t::ST7789_240x320, CS_PIN,DC_PIN,RST_PIN);
static constexpr const size_t lcd_transfer_buffer_size = (240*320*2)/10;
static void* lcd_transfer_buffer1 = nullptr;
static void* lcd_transfer_buffer2 = nullptr;

static uint32_t lvgl_tick(void)
{
    return millis();
}

void lvgl_disp_flush( lv_display_t *disp, const lv_area_t *area, uint8_t * px_map)
{
    lcd.flush_async(area->x1,area->y1,area->x2,area->y2, px_map, true);
}
void lcd_on_flush_complete(void* state) {
    lv_display_flush_ready((lv_display_t *)state);
}
void setup()
{
    
    Serial.begin(115200);
    lv_init();
    lcd_transfer_buffer1 = malloc(lcd_transfer_buffer_size);
    lcd_transfer_buffer2 = malloc(lcd_transfer_buffer_size);
    SPI.begin();
    lcd.begin();
    lcd.rotation(0);
    lv_display_t * disp = lv_display_create(SCREEN_WIDTH, SCREEN_HEIGHT);
    lv_display_set_flush_cb(disp, lvgl_disp_flush);
    lv_tick_set_cb(lvgl_tick);
    lv_display_set_buffers(disp, lcd_transfer_buffer1, lcd_transfer_buffer2, lcd_transfer_buffer_size, LV_DISPLAY_RENDER_MODE_PARTIAL);
    lcd.on_flush_complete_callback(lcd_on_flush_complete,disp);
}
void loop() {
    lv_timer_handler();
}
```
