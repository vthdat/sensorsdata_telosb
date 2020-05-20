// $Id: BlinkC.nc,v 1.6 2010-06-29 22:07:16 scipio Exp $

/*									tab:4
 * Copyright (c) 2000-2005 The Regents of the University  of California.  
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions
 * are met:
 *
 * - Redistributions of source code must retain the above copyright
 *   notice, this list of conditions and the following disclaimer.
 * - Redistributions in binary form must reproduce the above copyright
 *   notice, this list of conditions and the following disclaimer in the
 *   documentation and/or other materials provided with the
 *   distribution.
 * - Neither the name of the University of California nor the names of
 *   its contributors may be used to endorse or promote products derived
 *   from this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
 * "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
 * LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS
 * FOR A PARTICULAR PURPOSE ARE DISCLAIMED.  IN NO EVENT SHALL
 * THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT,
 * INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
 * (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION)
 * HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT,
 * STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED
 * OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 * Copyright (c) 2002-2003 Intel Corporation
 * All rights reserved.
 *
 * This file is distributed under the terms in the attached INTEL-LICENSE     
 * file. If you do not find these files, copies can be found by writing to
 * Intel Research Berkeley, 2150 Shattuck Avenue, Suite 1300, Berkeley, CA, 
 * 94704.  Attention:  Intel License Inquiry.
 */

/**
 * Implementation for Blink application.  Toggle the red LED when a
 * Timer fires.
 **/

#include "Timer.h"
#include "printf.h"
#include "RadioCountToLeds.h"

module BlinkC @safe()
{
  uses interface Timer<TMilli> as Timer0;
  uses interface Leds;
  uses interface Boot;

   uses interface Receive;
   uses interface AMSend;
   uses interface SplitControl as AMControl;
   uses interface Packet;

  uses interface Read<uint16_t> as Light;
  uses interface Read<uint16_t> as Temperature;
  uses interface Read<uint16_t> as Humidity; 
}
implementation
{

  uint16_t light = 0;
  uint16_t temperature = 0;
  uint16_t humidity = 0;
  uint16_t count = 0;
  message_t packet;

  bool locked;
  event void Boot.booted() {
    call AMControl.start();
  }

  event void AMControl.startDone(error_t err) {
    if (err == SUCCESS) {
      call Timer0.startPeriodic(1000);
    }
    else {
      call AMControl.start();
    }
  }

  event void AMControl.stopDone(error_t err) {
    // do nothing
  }


  event void Timer0.fired()
  {
    if (TOS_NODE_ID != 0) return;
    dbg("BlinkC", "Timer 0 fired @ %s.\n", sim_time_string());
    call Light.read();
    call Leds.led0On();

    call Temperature.read();
    call Leds.led1On();

    call Humidity.read();
    call Leds.led2On();
	if (TOS_NODE_ID == 0 && count == 3)
	{
	dbg("RadioCountToLedsC", "RadioCountToLedsC: timer fired, counter is %hu.\n", counter);
     	if (locked) {
      	return;
   	}
    	else {
      	radio_count_msg_t* rcm = (radio_count_msg_t*)call Packet.getPayload(&packet, sizeof(radio_count_msg_t));
      	if (rcm == NULL) {
	return;
      	}
      	rcm->light = light;
	rcm->temperature = temperature;
	rcm->humidity = humidity;
      	if (call AMSend.send(AM_BROADCAST_ADDR, &packet, sizeof(radio_count_msg_t)) == SUCCESS) {
	dbg("RadioCountToLedsC", "RadioCountToLedsC: packet sent.\n", counter);	
	locked = TRUE;
      	}
		
		call Leds.led0Off();
	
		
		call Leds.led1Off();

		
		call Leds.led2Off();

		count = 0;
	}
  }

}

  event void AMSend.sendDone(message_t* bufPtr, error_t error) {
    if (&packet == bufPtr) {
      locked = FALSE;
    }
  }

  event message_t* Receive.receive(message_t* bufPtr, void* payload, uint8_t len) {
	if (TOS_NODE_ID == 0) return;
	else{
		dbg("RadioCountToLedsC", "Received packet of length %hhu.\n", len);
   	if (len != sizeof(radio_count_msg_t)) {return bufPtr;}
    	else {
      	radio_count_msg_t* rcm = (radio_count_msg_t*)payload;
	
	light = rcm->light;
	temperature = rcm->temperature;
	humidity = rcm->humidity;
	
	if (light > 1000)
	{
		call Leds.led0On();
		call Leds.led1Off();
		call Leds.led2Off();
	}
	else if (light > 100)
	{
		call Leds.led0Off();
		call Leds.led1On();
		call Leds.led2Off();
	}
	else
	{
		call Leds.led0Off();
		call Leds.led1Off();
		call Leds.led2On();
	}
	printf("Light: %d\n", light);
	printf("Temperature: %d\n", temperature);
	printf("Humidity: %d\n", humidity);

      	return bufPtr;
    }
}  
  }

  event void Light.readDone(error_t result, uint16_t data)
  {
  	if (result == SUCCESS){
	  light = data;
          count++;
	}
  }

  event void Temperature.readDone(error_t result, uint16_t data)
  {
  	if (result == SUCCESS){
	  temperature = data;
          count++;
	}
  }

  event void Humidity.readDone(error_t result, uint16_t data)
  {
  	if (result == SUCCESS){
	  humidity = data;
          count++;
	}
  }
}

