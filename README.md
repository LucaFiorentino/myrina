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

const uint32_t ramSIZE = 0x1FFFF; // dimensione della memoria
uint32_t address = 0;
uint8_t pattern1 = 0xAA; // pattern1=1010
uint8_t pattern2 = 0x55; // pattern2=0101


//tale funzione scrive un byte su un undirizzo di memoria N.B la selezione della memoria viene fatta nel loop()

void write_Byte(uint32_t addr, char data){
SPI.transfer(WR);
SPI.transfer((char)(addr >> 8));
SPI.transfer((char) addr);
SPI.transfer(data);
  
}

//tale funzione legge un byte da un undirizzo di memoria N.B la selezione della memoria viene fatta nel loop()

char read_Byte(uint32_t addr){

  char rdByte;
  
  SPI.transfer(RD);
  SPI.transfer((char)(addr >> 8));
  SPI.transfer((char)(addr));
  rdByte = SPI.transfer(0xFF);
  return rdByte;
  
}

//funzione che inizializza la comunicazione SPI a seconda se bisogna leggere o scrivere una word

void SetAddressMode(uint32_t addr, byte mode){
SPI.transfer(mode);
SPI.transfer((byte)(addr >> 16));
SPI.transfer((byte)(addr >> 8));
SPI.transfer((byte)(addr));  
}

//questa funzione riempie la memoria selezionata(nel loop) con i pattern dichiarati

void writeChessBoard(uint32_t address, byte val1, byte val2, uint32_t lenght){
  SetAddressMode(address,WR);
      for(uint32_t i =0; i<lenght; i++){
        if(i % 2 == 0){
          SPI.transfer(val1);
        }

        else{
          SPI.transfer(val2);
        }
      }  
      
  }

//questa funzione controlla se ci è stato un bit-flip, se si riscrive il pattern corretto e manda in seriale l'indirizzo della locazione e la word errata

void FlipDetect(uint32_t addr, byte val1, byte val2, uint32_t lenght){
SetAddressMode(address,RD);
byte f = read_Byte(address);
boolean err1 = false;
boolean err2 = false;
for(uint32_t i=0; i< lenght; i++)
{
  if(i % 2 == 0)
  {
    if(f != val1)
    {
      Serial.write((byte)(addr));
      delay(5);
      Serial.write(f);
      delay(5);
      err1 = true;
      write_Byte(addr,val1);
    }
  }
  else
  {
    if (f != val2)
    {
       Serial.write((byte)(addr));
      delay(5);
      Serial.write(f);
      delay(5);
      err2 = true;
      write_Byte(addr,val2);
      
    }
  }
  f= read_Byte(++address);
}

    
  }
    

   
//inizializzazione della SPI e della seriale. scrittura delle memorie con i pattern dichiarati
void setup() {
pinMode(CS2,OUTPUT);
pinMode(CS1,OUTPUT);
pinMode(CS0,OUTPUT);
Serial.begin(9600);
Serial.flush();
SPI.begin();
SPI.beginTransaction(SPISettings(1000, MSBFIRST, SPI_MODE0));

digitalWrite(CS2,LOW);
digitalWrite(CS1,LOW);
digitalWrite(CS0,LOW);
writeChessBoard(address,pattern1,pattern2,ramSIZE);

digitalWrite(CS2,LOW);
digitalWrite(CS1,LOW);
digitalWrite(CS0,HIGH);
writeChessBoard(address,pattern1,pattern2,ramSIZE);

digitalWrite(CS2,LOW);
digitalWrite(CS1,HIGH);
digitalWrite(CS0,LOW);
writeChessBoard(address,pattern1,pattern2,ramSIZE);

digitalWrite(CS2,LOW);
digitalWrite(CS1,HIGH);
digitalWrite(CS0,HIGH);
writeChessBoard(address,pattern1,pattern2,ramSIZE);

digitalWrite(CS2,HIGH);
digitalWrite(CS1,LOW);
digitalWrite(CS0,LOW);
writeChessBoard(address,pattern1,pattern2,ramSIZE);

digitalWrite(CS2,HIGH);
digitalWrite(CS1,LOW);
digitalWrite(CS0,HIGH);
writeChessBoard(address,pattern1,pattern2,ramSIZE);

digitalWrite(CS2,HIGH);
digitalWrite(CS1,HIGH);
digitalWrite(CS0,LOW);
writeChessBoard(address,pattern1,pattern2,ramSIZE);

digitalWrite(CS2,HIGH);
digitalWrite(CS1,HIGH);
digitalWrite(CS0,HIGH);
writeChessBoard(address,pattern1,pattern2,ramSIZE);


}

//detect del bit-flip 

void loop() {

digitalWrite(CS2,LOW);
digitalWrite(CS1,LOW);
digitalWrite(CS0,LOW);
FlipDetect(address,pattern1,pattern2,ramSIZE);

digitalWrite(CS2,LOW);
digitalWrite(CS1,LOW);
digitalWrite(CS0,HIGH);
FlipDetect(address,pattern1,pattern2,ramSIZE);

digitalWrite(CS2,LOW);
digitalWrite(CS1,HIGH);
digitalWrite(CS0,LOW);
FlipDetect(address,pattern1,pattern2,ramSIZE);

digitalWrite(CS2,LOW);
digitalWrite(CS1,HIGH);
digitalWrite(CS0,HIGH);
FlipDetect(address,pattern1,pattern2,ramSIZE);

digitalWrite(CS2,HIGH);
digitalWrite(CS1,LOW);
digitalWrite(CS0,LOW);
FlipDetect(address,pattern1,pattern2,ramSIZE);

digitalWrite(CS2,HIGH);
digitalWrite(CS1,LOW);
digitalWrite(CS0,HIGH);
FlipDetect(address,pattern1,pattern2,ramSIZE);

digitalWrite(CS2,HIGH);
digitalWrite(CS1,HIGH);
digitalWrite(CS0,LOW);
FlipDetect(address,pattern1,pattern2,ramSIZE);

digitalWrite(CS2,HIGH);
digitalWrite(CS1,HIGH);
digitalWrite(CS0,HIGH);
FlipDetect(address,pattern1,pattern2,ramSIZE);


}
