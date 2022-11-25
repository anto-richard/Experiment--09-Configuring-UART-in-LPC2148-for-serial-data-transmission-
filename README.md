## EXPERIMENT -09- CONFIGURATING-UART-IN-LPC2148-FOR-SERIAL-DATA-TRANSMISSION...

Name : Anto Richard. S

Reg no : 212221240005 

Date of experiment : 25-11-22

### CONFIGURATING UART IN LPC2148 FOR SERIAL DATA TRANSMISSION:

### AIM: 

To configure internal UART for transferring serial data and display it on the Virtual terminal.

### COMPONENTS REQUIRED: 

Proteus ISIS professional suite,

Kiel μ vision 5 Development environment.

### THEORY: 

The UART Protocol uses only two wires (or pins in a device like microcontroller) to transmit the data. In that, one is for transmitting the data and the pin is called TX pin in the device. The other pin is used to receive the data and is called RX pin.

As UART is a serial communication, the data is transmitted in a series of packets. Usually, a packet consists of 4 parts: a start bit, the actual data, a parity bit and stop bits. The following image shows a typical structure of the data packet in UART.

### FIGURE -01- UART PACKET: 

  ![image](https://user-images.githubusercontent.com/36288975/203727146-383ce4b4-677b-44c3-bb13-a9e203950760.png)

### UART IN LPC2148:

Coming to UART in LPC2148, the LPC214x series of MCUs have two UART blocks called UART0 and UART1. Each UART block is associated with two pins, one for transmission and the other for receiving.

In UART0 block, the TXD0 (Transmit) and RXD0 (Receive) pins in the device are P0.0 and P0.1 respectively. In case of UART1, the TXD1 and RXD1 pins are P0.8 and P0.9 respectively.

UART0   UART1

TXD0	P0.0	TXD1	P0.8

RXD0	P0.1	RXD1	P0.9

Both the UART modules are identical, except the UART1 block has an additional full modem interface. This includes all the pins for RS232 compatibility like flow control pins (CTS, RTS) etc.

Both the UART blocks have 16 byte Receive and Transmit FIFO structures to hold the transmit and receive data. In order to control the data access and assembly, the UART blocks have two registers each.

For the transmitter operation, the TX has two special registers called Transmit Holding Register (THR) and Transmit Shift Register (TSR). In order to transmit the data, it is first sent to THR and then moved to TSR.

For the receiver operation, the RX has two special registers called Receiver Buffer Register (RBR) and Receive Shift Register (RSR). When the data is received, it is first stored in the RSR and then moved to RBR.

### REGISTERS ASSOCIATED WITH UART IN LPC2148:

There are many registers involved with the UART blocks. UART0 and UART1 have a similar register structure and here we are mentioning some of the important registers of the UART0 block. In order to access the registers in UART1 block, simply replace the ‘0’ with ‘1’.

#### UART0 RECEIVER BUFFER REGISTER (U0RBR): 

The Receiver Buffer Register consists of the top byte of the RX FIFO. This is the data that is first arrived and Bit 0 contains the oldest data. In case the received data is less than 8 bits, the remaining bits are padded with 0’s. In order to access the U0RBR register, the Divisor Latch Access bit (DLAB) in the UART0 Line Control Register (U0LCR) must be set to 0. With reference to the user manual, it is advised that the U0LSR register is read first and then the U0RBR register is read.

#### UART0 TRANSMIT HOLDING REGISTER (U0THR):

The Transmit Holding Register consists of the top byte of the TX FIFO i.e. the oldest data. Bit 0 (LSB) is the first data to be transmitted. In order to access the U0THR register, the Divisor Latch Access Bit (DLAB) bit in the UART0 Line Control Register (U0LCR) must be made zero.

#### UART0 DIVISOR LATCH REGISTERS (U0DLL and U0DLM):

The Divisor Latch Registers determine the baud rate of the UART0. The Divisor Latch is a part of the Baud Rate Generator. The U0DLL and U0DLM registers contains the lower and higher 8 – bits of the divisor and together they form the 16 – bit divisor value. Since it contains the divisor value, the U0DLL cannot contain 0x00 as division by zero is invalid. Hence, the minimum value for the U0DLM:U0DLL combination is 0x00:0x01. In order to access the Divisor Latch Registers, the Divisor Latch Access Bit (DLAB) bit in the UART0 Line Control Register (U0LCR) must be set to 1.

#### UART0 FRACTIONAL DIVIDER REGISTER (U0FDR):

The Fractional Divider Register contains the bits that control the prescale for the baud rate generator. U0FDR contains the divisor and multiplier values for prescaling. Both the divisor (DIVADDVAL) and multiplier (MULVAL) values are stored as 4 – bit values. Bit 0 to Bit 3 in U0FDR contains the pre – scaler divisor value for baud rate generation. For the baud rate generator to have an impact on the baud rate of the UART, the divisor value must not be 0. Bit 4 to bit 7 in U0FDR contains the pre – scaler multiplier value. For proper working of the UART0, the value in multiplier must be greater than or equal to 1. This is also applicable even if the baud rate generator is not used.

#### UART0 INTERRUPT ENABLE REGISTER (U0IER):

The Interrupt Enable Register is used to enable or disable the interrupts corresponding to UART0. Bit 0 is for RBR Interrupt, Bit 1 is for THRE interrupt, Bit 2 is for RX Line Status Interrupt, Bit 8 is for End of Auto – baud interrupt and Bit 9 is for Auto – baud Time out interrupt. When a bit is 0, the interrupt is disabled and when the bit is 1, the interrupt is enabled.

#### UART0 INTERRUPT IDENTIFICATION REGISTER (U0IIR):

The Interrupt Identification Register is used to provide the code for priority and status of the pending interrupt.

#### UART0 FIFO CONTROL REGISTER (U0FCR): 

The FIFO Control Register controls the operation of the RX and TX FIFOs in UART0. Bit 0 is used to enable or disable the FIFO. Bit 1 is used to reset the RX FIFO. Bit 2 is used to reset the TX FIFO. Bits 6 and 7 are used to control when the interrupt must occur i.e. after how many receiver characters.

#### UART0 LINE CONTROL REGISTER (U0LCR): 

The Line Control Register is used to set the format of the data which is transmitted or received.

### FIGURE -02- UART INTERFACE VIRTUAL TERMINAL:

  ![image](https://user-images.githubusercontent.com/36288975/203729175-35823e84-cdad-4cd2-8334-2a7477de528f.png)

### KIEL-PROGRAM: 

```c

#include <LPC213x.H>             
char a;
void uart0_init()
{
  PINSEL0 = 0x00000005;           
  U0LCR = 0x83;                   
  U0DLL = 97;                  
  U0LCR = 0x03;              
}
void uart0_putc(char c)
{
 while(!(U0LSR & 0x20));
 U0THR = c; 
}
int uart0_getc (void)
{                     
  while (!(U0LSR & 0x01));
  return (U0RBR);
}
int main (void) 
{                
  uart0_init();      
  while (1) 
  {                          
   a=uart0_getc();
   uart0_putc(a);
  }                               
}

```

### OUTPUT SCREENSHOTS:

#### BEFORE SIMULATION:

![img1](https://user-images.githubusercontent.com/93427534/204021664-a84ce1a9-203a-43e5-95f5-cf7ddd8a2e92.png)

#### AFTER SIMULATION:

![img2](https://user-images.githubusercontent.com/93427534/204021686-fc70ce4a-342a-4875-aea6-11dac6ada69b.png)

#### CIRCUIT DIAGRAM:

![img3](https://user-images.githubusercontent.com/93427534/204021693-469da419-1eb3-489f-b0cd-e4977021defc.png)

### RESULT:

UART is programmed for transmitting serial data on virtual terminal.  
