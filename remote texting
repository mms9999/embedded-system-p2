//*****************************************************************************
//
// hello.c - Simple hello world example.
//
// Copyright (c) 2012-2014 Texas Instruments Incorporated.  All rights reserved.
// Software License Agreement
// 
// Texas Instruments (TI) is supplying this software for use solely and
// exclusively on TI's microcontroller products. The software is owned by
// TI and/or its suppliers, and is protected under applicable copyright
// laws. You may not combine this software with "viral" open-source
// software in order to form a larger program.
// 
// THIS SOFTWARE IS PROVIDED "AS IS" AND WITH ALL FAULTS.
// NO WARRANTIES, WHETHER EXPRESS, IMPLIED OR STATUTORY, INCLUDING, BUT
// NOT LIMITED TO, IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
// A PARTICULAR PURPOSE APPLY TO THIS SOFTWARE. TI SHALL NOT, UNDER ANY
// CIRCUMSTANCES, BE LIABLE FOR SPECIAL, INCIDENTAL, OR CONSEQUENTIAL
// DAMAGES, FOR ANY REASON WHATSOEVER.
// 
// This is part of revision 2.1.0.12573 of the EK-TM4C123GXL Firmware Package.
//
//*****************************************************************************

#include <stdint.h>
#include <stdbool.h>
#include "inc/hw_memmap.h"
#include "inc/hw_types.h"
#include "inc/hw_gpio.h"


#include "driverlib/debug.h"
#include "driverlib/fpu.h"
#include "driverlib/gpio.h"
#include "driverlib/pin_map.h"
#include "driverlib/rom.h"
#include "driverlib/sysctl.h"
#include "driverlib/uart.h"
#include "utils/uartstdio.h"
#include "driverlib/interrupt.h"
#include "inc/hw_ints.h"
#include "inc/hw_nvic.h"

#include "driverlib/timer.h"
#include "inc/tm4c123gh6pm.h"

static int interrupted = 0;
static int edge1 = 0;
static int edge2 = 0;
static int total_time = 0;
static int count = 0;
static int times[52];
static int done_flag = 0;
static int last_time = 0;
static int start_flag = 0;

#ifdef DEBUG
void
__error__(char *pcFilename, uint32_t ui32Line)
{
}
#endif


void
ConfigureUART(void)
{
    //
    // Enable the GPIO Peripheral used by the UART.
    //
    ROM_SysCtlPeripheralEnable(SYSCTL_PERIPH_GPIOA);

    //
    // Enable UART0
    //
    ROM_SysCtlPeripheralEnable(SYSCTL_PERIPH_UART0);

    //
    // Configure GPIO Pins for UART mode.
    //
    ROM_GPIOPinConfigure(GPIO_PA0_U0RX);
    ROM_GPIOPinConfigure(GPIO_PA1_U0TX);
    ROM_GPIOPinTypeUART(GPIO_PORTA_BASE, GPIO_PIN_0 | GPIO_PIN_1);

    //
    // Use the internal 16MHz oscillator as the UART clock source.
    //
    UARTClockSourceSet(UART0_BASE, UART_CLOCK_PIOSC);

    //
    // Initialize the UART for console I/O.
    //
    UARTStdioConfig(0, 115200, 16000000);
}


void IR_Handler (void) {
	// Disable interrupt for awhile to avoid switch bounce
	GPIOIntDisable(GPIO_PORTB_BASE, GPIO_INT_PIN_2);

	// Clear interrupt request
	GPIOIntClear(GPIO_PORTB_BASE, GPIO_INT_PIN_2);
	
// Get pulse width
	edge2 = edge1;
	edge1 = TimerValueGet(TIMER0_BASE,TIMER_A);
	total_time = edge2 - edge1;
	if (total_time < 0)
		total_time = total_time + SysCtlClockGet();
	
// assign time	
	if (count < 60)
		times[count] = total_time;
	else
		done_flag = 1;
	
// check if continuously held
	if (last_time != 425000)
	{
		if (total_time > 400000 && total_time < 410000)
		{
			start_flag = 1;
		}
		else
		{
			if ((last_time + total_time) > 420000 && (last_time + total_time) < 430000)
				start_flag = 1;
		}
	}
	if (total_time > 420000 && total_time < 430000)
	{
		total_time = 425000;
		done_flag = 1;
	}
	else
	{
		if ((last_time + total_time) > 420000 && (last_time + total_time) < 430000)
		{
			total_time = 425000;
			done_flag = 1;
		}
	}
// if not continuously held
	if (start_flag == 1)
	{
		count = count + 1;
	}
	
	last_time = total_time;
}	

void decode(int times[], int size);

// ===========================================================================================
// Main

int main(void)
{
		ConfigureUART();
    ROM_FPULazyStackingEnable();
		int ir_input;
	
    //
    // Set the clocking to run directly from the crystal.
    //
    ROM_SysCtlClockSet(SYSCTL_SYSDIV_4 | SYSCTL_USE_PLL | SYSCTL_XTAL_16MHZ | SYSCTL_OSC_MAIN);


    ROM_SysCtlPeripheralEnable(SYSCTL_PERIPH_GPIOF);
		ROM_SysCtlPeripheralEnable(SYSCTL_PERIPH_GPIOB);

    HWREG(GPIO_PORTF_BASE + GPIO_O_LOCK) = GPIO_LOCK_KEY;
    HWREG(GPIO_PORTF_BASE + GPIO_O_CR) |= 0x01;
    HWREG(GPIO_PORTF_BASE + GPIO_O_LOCK) = 0;

		ROM_SysCtlPeripheralEnable(SYSCTL_PERIPH_TIMER0);
		ROM_TimerConfigure(TIMER0_BASE, TIMER_CFG_PERIODIC);

		ROM_TimerLoadSet(TIMER0_BASE, TIMER_A, SysCtlClockGet());
		TimerEnable(TIMER0_BASE, TIMER_A);
		
    ROM_GPIOPinTypeGPIOInput(GPIO_PORTF_BASE, GPIO_PIN_0 | GPIO_PIN_4); // PF0, PF4 = switches
		GPIOPinTypeGPIOInput(GPIO_PORTB_BASE, GPIO_PIN_2);
		
    ROM_GPIOPadConfigSet(GPIO_PORTF_BASE, GPIO_PIN_0 | GPIO_PIN_4, GPIO_STRENGTH_8MA, GPIO_PIN_TYPE_STD_WPU);
		GPIOPadConfigSet(GPIO_PORTB_BASE, GPIO_PIN_2, GPIO_STRENGTH_4MA, GPIO_PIN_TYPE_STD);
		
    ROM_GPIOPinTypeGPIOOutput(GPIO_PORTF_BASE, GPIO_PIN_1 | GPIO_PIN_2 | GPIO_PIN_3);
		
		GPIOIntTypeSet(GPIO_PORTB_BASE, GPIO_PIN_2, GPIO_FALLING_EDGE);

		GPIOIntEnable(GPIO_PORTB_BASE, GPIO_INT_PIN_2);
		
		IntEnable(INT_GPIOB);
		
		IntMasterEnable();
		
		ROM_GPIOPinWrite(GPIO_PORTF_BASE, GPIO_PIN_1 | GPIO_PIN_2 | GPIO_PIN_3, GPIO_PIN_3);
		
		UARTprintf("Starting Program...\n\n");
		
		
// =================================================================================================
// While
		
		
    while(1)
    {               
      
			ROM_SysCtlSleep();
			
			
			if (done_flag == 1)
			{
				decode(times, count);
				done_flag = 0;
				count = 0;
				start_flag = 0;
			}
			GPIOIntClear(GPIO_PORTB_BASE, GPIO_INT_PIN_2);
			GPIOIntEnable(GPIO_PORTB_BASE, GPIO_INT_PIN_2);
    }
}


// ==================================================================================================
// Functions
// ==================================================================================================


void decode(int times[], int size)
{
	int value[60];
	int index = 0;
	int start = 0;
	int sequence = 0;
	int address = 0;
	
	
			for (int i=0; i<size;i++)
			{
				if (times[i] > 123000 && times[i] < 127000)
				{
					value[index] = 1;
					index = index + 1;
				}
		else
		{
			if (times[i] > 73000 && times[i] < 77000)
			{
				value[index] = 0;
				index = index + 1;
			}
			else
			{
				if ((times[i]+times[i+1]) > 123000 && (times[i]+times[i+1]) < 127000)
				{
					value[index] = 1;
					index = index+1;
					i++;
				}
				else
				{
					if ((times[i]+times[i+1]) > 73000 && (times[i]+times[i+1]) < 77000)
					{
						value[index] = 0;
						index = index+1;
						i++;
					}
					else
					{
						value[index] = -1;
						index = index+1;
					}
				}
			}
		}
			
	

	} // for(int i=0; i<size;i++)
		
	
	// check for 1111 ---- ---- 0000
	for (int i=0; i<index;i++)
	{
		if (value[i] == 1)
		{
			start = 1;
			for (int j=0; j<4;j++)
			{
				if (value[i+j] != 1)
					start = 0;
			}
			if (start == 1)
			{
				for (int j=0; j<4;j++)
				{
					if (value[i+j+12] != 0)
					start = 0;
				}
			}
		}
		if (start == 1)
		{
			sequence = 0;

			for (int j = 8; j<12;j++)
			{
				sequence = sequence * 10 + value[i+j];
			}
			for (int j = 4; j < 8; j++)
			{
				address = address * 10 + value[i+j];
			}
			if (address == 11)
			{
				switch(sequence)
				{
					case 0:
						UARTprintf("0\n");
						break;
					case 1:
						UARTprintf("1\n");
						break;
					case 10:
						UARTprintf("2\n");
						break;
					case 11:
						UARTprintf("3\n");
						break;
					case 100:
						UARTprintf("4\n");
						break;
					case 101:
						UARTprintf("5\n");
						break;
					case 110:
						UARTprintf("6\n");
						break;
					case 111:
						UARTprintf("7\n");
						break;
					case 1000:
						UARTprintf("8\n");
						break;
					case 1001:
						UARTprintf("9\n");
						break;
					case 1111:
						UARTprintf("Mute\n");
						break;
					default:
						UARTprintf("Error, sequence = %i\n",sequence);
						sequence = 0;
				}
			} // if address == 11
			if (address == 101)
			{
					
				switch(sequence)
				{
					case 110:
						UARTprintf("<\n");
						break;
					case 111:
						UARTprintf(">\n");
						break;
					case 1001:
						UARTprintf("^\n");
						break;
					case 1000:
						UARTprintf("v\n");
						break;
					default:
						UARTprintf("Error, sequence = %i, address = %i\n",sequence,address);
						sequence = 0;
						address = 0;
				}
			} // if address == 101
			if (address == 10)
			{
				switch(sequence)
				{
					case 1111:
						UARTprintf("vol +\n");
						break;
					case 1110:
						UARTprintf("vol -\n");
						break;
					case 1101:
						UARTprintf("ch +\n");
						break;
					case 1100:
						UARTprintf("ch -\n");
						break;
					default:
						UARTprintf("Error, sequence = %i, address = %i\n",sequence,address);
						sequence = 0;
						address = 0;
				}
			} // if address == 10
			if (address == 1111)
			{
				switch(sequence)
				{
					case 100:
						UARTprintf("Enter\n");
						break;
					default:
						UARTprintf("Error, sequence = %i, address = %i\n",sequence,address);
						sequence = 0;
						address = 0;
				}
			}
			ROM_SysCtlDelay(SysCtlClockGet()/15);
		}
	} // (int i=0; i<size;i++)
}
