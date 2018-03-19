# myrina

#include <SPI.h>

// OPCODE SRAM

#define RD 3

#define WR 2

// PIN

#define MOSI 51

#define MISO 50

#define SCK  52

#define CS2 12 //MSB della parola binaria di ingresso del decoder

#define CS1 11

#define CS0 10 //LSB della parola binaria di ingresso del decoder
