# Smart-Traffic-Light-Management-System-with-Countdown-Display
#include <LPC21xx.h>

// 7-segment pins (A-DP) connected to P1.16 to P1.23
#define SEGMENT_PORT (0xFF<<16)  // P1.16 to P1.23
#define DSEL1        (1 << 24)    // P1.24 for tens digit
#define DSEL2        (1 << 25)    // P1.25 for units digit

// LED pins for traffic lights
#define RED_LED      (1 << 0)  // P0.0
#define YELLOW_LED   (1 << 1)  // P0.1
#define GREEN_LED    (1 << 2)  // P0.2

// Lookup table for 7-segment display
const unsigned char segLUT[] = {
    0xC0, // 0 - 11000000 - A B C D E F
    0xF9, // 1 - 11111001 - B C
    0xA4, // 2 - 10100100 - A B D E G
    0xB0, // 3 - 10110000 - A B C D G
    0x99, // 4 - 10011001 - B C F G
    0x92, // 5 - 10010010 - A C D F G
    0x82, // 6 - 10000010 - A C D E F G
    0xF8, // 7 - 11111000 - A B C
    0x80, // 8 - 10000000 - A B C D E F G
    0x90  // 9 - 10010000 - A B C D F G
};

void delay_ms(unsigned int ms) {
    unsigned int i, j;
    for (i = 0; i < ms; i++) {
        for (j = 0; j < 12000; j++) {
            // Empty loop for delay
        }
    }
}

void init_ports() {
    // Configure P1.16 to P1.23 for 7-segment
    IODIR1 |= SEGMENT_PORT;

    // Configure P1.24 and P1.25 for digit selection
    IODIR1 |= (DSEL1 | DSEL2);

    // Configure LEDs for traffic lights
    IODIR0 |= (RED_LED | YELLOW_LED | GREEN_LED);
    IOCLR0 = (RED_LED | YELLOW_LED | GREEN_LED);  // Turn off all LEDs
}

void display_number(unsigned int number) {
    // Display tens digit
    IOCLR1 = SEGMENT_PORT;               // Clear segment bits
    IOSET1 = (segLUT[number / 10] << 16); // Set tens digit
    IOSET1 = DSEL1;                      // Select tens digit
    IOCLR1 = DSEL2;                      // Deselect units digit
    delay_ms(1);
    IOCLR1 = DSEL1;                      // Deselect tens digit

    // Display units digit
    IOCLR1 = SEGMENT_PORT;               // Clear segment bits
    IOSET1 = (segLUT[number % 10] << 16); // Set units digit
    IOSET1 = DSEL2;                      // Select units digit
    IOCLR1 = DSEL1;                      // Deselect tens digit
    delay_ms(1);
    IOCLR1 = DSEL2;                      // Deselect units digit
}

void traffic_light_sequence() {
    unsigned int count;
    unsigned int i;
    // Red light with countdown from 60 to 0
    IOSET0 = RED_LED;
    IOCLR0 = YELLOW_LED | GREEN_LED;
    for (count = 60; count > 0; count--) {
        for (i = 0; i < 200; i++) {  // Approx 1 second with multiplexing
					   delay_ms(1);
            display_number(count);
        }
    }
		
    // Green light with countdown from 20 to 0
    IOSET0 = GREEN_LED;
    IOCLR0 = RED_LED | YELLOW_LED;
    for (count = 20; count > 0; count--) {
        for (i = 0; i < 200; i++) {  // Approx 1 second with multiplexing
            display_number(count);
        }
    }

    // Yellow light glows for 10 seconds
    IOSET0 = YELLOW_LED;
    IOCLR0 = RED_LED | GREEN_LED;
    for (i = 0; i < 1000; i++) {  // Approx 10 seconds
        display_number(0);  // Display "00" on the 7-segment display
        delay_ms(1);
    }

     //Red light glows continuously after yellow
    IOSET0 = RED_LED;
    IOCLR0 = YELLOW_LED | GREEN_LED;
    while (1) {
        display_number(0);  // Keep displaying "00"
        delay_ms(1);
    }
}

int main() {
    init_ports();
    while (1) {
        traffic_light_sequence();
    }
}
