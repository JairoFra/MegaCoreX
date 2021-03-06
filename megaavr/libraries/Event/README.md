# Event
A library for interfacing with the built-in Event system on the megaAVR-0 series MCUs.  
Developed by [MCUdude](https://github.com/MCUdude/).  

**From the datasheet:**
> The Event System (EVSYS) enables direct peripheral-to-peripheral signaling. It allows a change in one peripheral (the event generator) to trigger actions in other peripherals (the event users) through event channels, without using the CPU. It is designed to provide short and predictable response times between peripherals, allowing for autonomous peripheral control and interaction, and also for synchronized timing of actions in several peripheral modules. It is thus a powerful tool for reducing the complexity, size, and execution time of the software.

More information about the Event system and how it works can be found in the [Microchip Application Note AN2451](http://ww1.microchip.com/downloads/en/AppNotes/DS00002451B.pdf) and in the [megaAVR-0 family data sheet](http://ww1.microchip.com/downloads/en/DeviceDoc/megaAVR0-series-Family-Data-Sheet-DS40002015B.pdf).


### Level vs. Pulse events
There are two types of events - a "pulse" interrupt, which lasts for the duration of a single clock cycle (either `CLK_PER` or a relevant (slower) clock - for example, the USART XCK generator provides a pulse event which lasts one XCK period, whuich is far slower than CLK_PER), or a "level" interrupt which lasts for the duration of some condition. 
Often for a given even generator or user only one or the other makes sense. Less often, for some reason or another, you may need a level event, but all you have is a pulse event - or the other way around. A [CCL module (Logic.h)](../Logic/README.md) event between the two at the cost of the logic module and one event channel. In the case of timer WO (PWM) channels, the CCL already has level inputs.


### Synchronization
The event system, under the hood, is asynchronous - it can react faster than the system clock (often a lot faster).
The fact that it is asynchronous usually doesn't matter, but it is one of the things one should keep in mind when using these features, because every so often it does.


### Some of these events are *weird*
At first glance, nore than half of the users and generators seem, at best, odd - and a good few of them might appear entirely useless. Most of the event system can only truly be understood when considering the full range of generators and users - particularly the CCL. One of the tragedies of a datasheet is that it - generally - lacks a "why". Behind every mysterious event, there is a use case that seems obscure to most people - but within some sub-field, it's common and essential. There are also times when something may seem surprising until you're more familiar with the event and logic systems in general.



## Event
Class for interfacing with the built-in Event system. Each event generator channel has its own object. 
Use the predefined objects `Event0`, `Event1`, `Event2`, `Event3`, `Event4`, `Event5`, `Event6` or `Event7`. Note that channels have different functionality, so make sure you use the right channel for the task.

In short terms:  
* `genN::rtc_div8192`, `genN::rtc_div4096`, `genN::rtc_div2048` and `genN::rtc_div1024` are only available on odd numbered channels
* `genN::rtc_div512`, `genN::rtc_div256`, `genN::rtc_div128` and `genN::rtc_div64` are only available on even numbered channels
* PIN PA0..7 and PB0..5 can only be used as event generators on channel 0 and 1
* PIN PC0..7 and PD0..7 can only be used as event generators on channel 2 and 3
* PIN PE0..3 and PF0..6 can only be used as event generators on channel 4 and 5


## get_channel_number()
Function to get the current channel number. Useful if the channel object has been passed to a function as reference.

### Usage
``` c++
uint8_t this_channel = Event0.get_channel_number();  // In this case, get_channel_number will return 0
```


## get_user_channel()
Function to get what event channel a user is connected to. Returns -1 if not connected to any channel. Note that we use `user::` as prefix when we refer to event users. Also, note that we don't have to specify an object to determine what channel the user is connected to. if you're not sure, use `Event::get_user_channel`.
An event generator can have multiple event users, but an event user can only have one event generator.

### Usage
```c++
uint8_t connected_to = Event::get_user_channel(user::ccl0_event_a); // Returns the channel number ccl0_event_a is connected to
```


## set_generator()
Function to assign an event generator to a channel. Note that we use the prefix genN:: (where N is the channel number) when referring to generators unique to this particular channel. we use gen:: when referring to generators available on all generators.

### Usage
```c++
Event0.set_generator(gen::ccl0_out); // Use the output of logic block 0 (CCL0) as an event generator for Event0
Event2.set_generator(gen2::pin_pc0); // Use pin PC0 as an event generator for Event2
```

### Generator table
Below is a table with all possible generators for each channel.

| All event channels   | Event0                                              | Event1                                              | Event2                                              | Event3                                              | Event4                                               | Event5                                               | Event6              | Event7             |
|----------------------|-----------------------------------------------------|-----------------------------------------------------|-----------------------------------------------------|-----------------------------------------------------|------------------------------------------------------|------------------------------------------------------|---------------------|--------------------|
| `gen::disable`       | `gen0::disable`                                     | `gen1::disable`                                     | `gen2::disable`                                     | `gen3::disable`                                     | `gen4::disable`                                      | `gen5::disable`                                      | `gen6::disable`     | `gen7::disable`    |
| `gen::updi_synch`    | `gen0::rtc_div8192`                                 | `gen1::rtc_div512`                                  | `gen2::rtc_div8192`                                 | `gen3::rtc_div512`                                  | `gen4::rtc_div8192`                                  | `gen5::rtc_div512`                                   | `gen6::rtc_div8192` | `gen7::rtc_div512` |
| `gen::rtc_ovf`       | `gen0::rtc_div4096`                                 | `gen1::rtc_div256`                                  | `gen2::rtc_div4096`                                 | `gen3::rtc_div256`                                  | `gen4::rtc_div4096`                                  | `gen5::rtc_div256`                                   | `gen6::rtc_div4096` | `gen7::rtc_div256` |
| `gen::rtc_cmp`       | `gen0::rtc_div2048`                                 | `gen1::rtc_div128`                                  | `gen2::rtc_div2048`                                 | `gen3::rtc_div128`                                  | `gen4::rtc_div2048`                                  | `gen5::rtc_div128`                                   | `gen6::rtc_div2048` | `gen7::rtc_div128` |
| `gen::ccl0_out`      | `gen0::rtc_div1024`                                 | `gen1::rtc_div64`                                   | `gen2::rtc_div1024`                                 | `gen3::rtc_div64`                                   | `gen4::rtc_div1024`                                  | `gen5::rtc_div64`                                    | `gen6::rtc_div1024` | `gen7::rtc_div64`  |
| `gen::ccl1_out`      | `gen0::pin_pa0`                                     | `gen1::pin_pa0`                                     | `gen2::pin_pc0`                                     | `gen3::pin_pc0`                                     | `gen4::pin_pe0`<br/> (Only available on ATmegaX809)  | `gen5::pin_pe0`<br/> (Only available on ATmegaX809)  |                     |                    |
| `gen::ccl2_out`      | `gen0::pin_pa1`                                     | `gen1::pin_pa1`                                     | `gen2::pin_pc1`                                     | `gen3::pin_pc1`                                     | `gen4::pin_pe1`<br/> (Only available on ATmegaX809)  | `gen5::pin_pe1`<br/> (Only available on ATmegaX809)  |                     |                    |
| `gen::ccl3_out`      | `gen0::pin_pa2`                                     | `gen1::pin_pa2`                                     | `gen2::pin_pc2`                                     | `gen3::pin_pc2`                                     | `gen4::pin_pe2`<br/> (Only available on ATmegaX809)  | `gen5::pin_pe2`<br/> (Only available on ATmegaX809)  |                     |                    |
| `gen::ac0_out`       | `gen0::pin_pa3`                                     | `gen1::pin_pa3`                                     | `gen2::pin_pc3`                                     | `gen3::pin_pc3`                                     | `gen4::pin_pe3`<br/> (Only available on ATmegaX809)  | `gen5::pin_pe3`<br/> (Only available on ATmegaX809)  |                     |                    |
| `gen::adc0_ready`    | `gen0::pin_pa4`                                     | `gen1::pin_pa4`                                     | `gen2::pin_pc4`<br/> (Only available on ATmegaX809) | `gen3::pin_pc4`<br/> (Only available on ATmegaX809) |                                                      |                                                      |                     |                    |
| `gen::usart0_xck`    | `gen0::pin_pa5`                                     | `gen1::pin_pa5`                                     | `gen2::pin_pc5`<br/> (Only available on ATmegaX809) | `gen3::pin_pc5`<br/> (Only available on ATmegaX809) |                                                      |                                                      |                     |                    |
| `gen::usart1_xck`    | `gen0::pin_pa6`                                     | `gen1::pin_pa6`                                     | `gen2::pin_pc6`<br/> (Only available on ATmegaX809) | `gen3::pin_pc6`<br/> (Only available on ATmegaX809) |                                                      |                                                      |                     |                    |
| `gen::usart2_xck`    | `gen0::pin_pa7`                                     | `gen1::pin_pa7`                                     | `gen2::pin_pc7`<br/> (Only available on ATmegaX809) | `gen3::pin_pc7`<br/> (Only available on ATmegaX809) |                                                      |                                                      |                     |                    |
| `gen::usart3_xck`    | `gen0::pin_pb0`<br/> (Only available on ATmegaX809) | `gen1::pin_pb0`<br/> (Only available on ATmegaX809) | `gen2::pin_pd0`                                     | `gen3::pin_pd0`                                     | `gen4::pin_pf0`                                      | `gen5::pin_pf0`                                      |                     |                    |
| `gen::spi0_sck`      | `gen0::pin_pb1`<br/> (Only available on ATmegaX809) | `gen1::pin_pb1`<br/> (Only available on ATmegaX809) | `gen2::pin_pd1`                                     | `gen3::pin_pd1`                                     | `gen4::pin_pf1`                                      | `gen5::pin_pf1`                                      |                     |                    |
| `gen::tca0_ovf_lunf` | `gen0::pin_pb2`<br/> (Only available on ATmegaX809) | `gen1::pin_pb2`<br/> (Only available on ATmegaX809) | `gen2::pin_pd2`                                     | `gen3::pin_pd2`                                     | `gen4::pin_pf2`<br/> (Not available on 28-pin parts) | `gen5::pin_pf2`<br/> (Not available on 28-pin parts) |                     |                    |
| `gen::tca0_hunf`     | `gen0::pin_pb3`<br/> (Only available on ATmegaX809) | `gen1::pin_pb3`<br/> (Only available on ATmegaX809) | `gen2::pin_pd3`                                     | `gen3::pin_pd3`                                     | `gen4::pin_pf3`<br/> (Not available on 28-pin parts) | `gen5::pin_pf3`<br/> (Not available on 28-pin parts) |                     |                    |
| `gen::tca0_cmp0`     | `gen0::pin_pb4`<br/> (Only available on ATmegaX809) | `gen1::pin_pb4`<br/> (Only available on ATmegaX809) | `gen2::pin_pd4`                                     | `gen3::pin_pd4`                                     | `gen4::pin_pf4`<br/> (Not available on 28-pin parts) | `gen5::pin_pf4`<br/> (Not available on 28-pin parts) |                     |                    |
| `gen::tca0_cmp1`     | `gen0::pin_pb5`<br/> (Only available on ATmegaX809) | `gen1::pin_pb5`<br/> (Only available on ATmegaX809) | `gen2::pin_pd5`                                     | `gen3::pin_pd5`                                     | `gen4::pin_pf5`<br/> (Not available on 28-pin parts) | `gen5::pin_pf5`<br/> (Not available on 28-pin parts) |                     |                    |
| `gen::tca0_cmp2`     |                                                     |                                                     | `gen2::pin_pd6`                                     | `gen3::pin_pd6`                                     | `gen4::pin_pf6`                                      | `gen5::pin_pf6`                                      |                     |                    |
| `gen::tcb0_capt`     |                                                     |                                                     | `gen2::pin_pd7`                                     | `gen3::pin_pd7`                                     |                                                      |                                                      |                     |                    |
| `gen::tcb1_capt`     |                                                     |                                                     |                                                     |                                                     |                                                      |                                                      |                     |                    |
| `gen::tcb2_capt`     |                                                     |                                                     |                                                     |                                                     |                                                      |                                                      |                     |                    |
| `gen::tcb3_capt`     |                                                     |                                                     |                                                     |                                                     |                                                      |                                                      |                     |                    |
| `gen::tcb3_capt`     |                                                     |                                                     |                                                     |                                                     |                                                      |                                                      |                     |                    |


## get_generator()
Function to get the generator used for a particular channel.

### Usage
```c++
uint8_t generator_used = Event0.get_generator();
if(generator_used == gen::ccl0_out) {
  Serial.println("We're using gen::ccl0_out as generator");
}
```


## set_user()
Function to connect an event user to an event generator. Note that a generator can have multiple users.

### Usage
```c++
Event0.set_generator(gen0::pin_pa0); // Set pin PA0` as event generator for Event0
Event0.set_user(user::evoutc);       // Set EVOUTC (pin PC2) as event user
Event0.set_user(user::evoutd);       // Set evoutD (pin PD2) as event user
```

### User table
Below is a table with all possible event users.   
Note that `evoutN_pin_pN7` is the same as `evoutN_pin_pN2` but where the pin is swapped from 2 to 7. This means that for instance, `evouta_pin_pa2` can't be used in combination with `evouta_pin_pa7.`

| Event users            | Notes                                                                 |
|------------------------|-----------------------------------------------------------------------|
| `user::ccl0_event_a`   |                                                                       |
| `user::ccl0_event_b`   |                                                                       |
| `user::ccl1_event_a`   |                                                                       |
| `user::ccl1_event_b`   |                                                                       |
| `user::ccl2_event_a`   |                                                                       |
| `user::ccl2_event_b`   |                                                                       |
| `user::ccl3_event_a`   |                                                                       |
| `user::ccl3_event_b`   |                                                                       |
| `user::adc0_start`     |                                                                       |
| `user::evouta_pin_pa2` |                                                                       |
| `user::evouta_pin_pa7` | Pin swapped variant of `evouta_pin_pa2`                               |
| `user::evoutb_pin_pb2` | Only available on ATmegaX809                                          |
| `user::evoutc_pin_pc2` |                                                                       |
| `user::evoutc_pin_pc7` | Pin swapped variant of `evoutc_pin_pc2`. Only available on ATmegaX809 |
| `user::evoutd_pin_pd2` |                                                                       |
| `user::evoutd_pin_pd7` | Pin swapped variant of `evoutd_pin_pd2`                               |
| `user::evoute_pin_pe2` | Only available on ATmegaX809                                          |
| `user::evoutf_pin_pf2` | Not available on 28-pin parts                                         |
| `user::usart0_irda`    |                                                                       |
| `user::usart1_irda`    |                                                                       |
| `user::usart2_irda`    |                                                                       |
| `user::usart3_irda`    |                                                                       |
| `user::tca0`           |                                                                       |
| `user::tcb0`           |                                                                       |
| `user::tcb1`           |                                                                       |
| `user::tcb2`           |                                                                       |
| `user::tcb3`           |                                                                       |


## clear_user()
Function to detach a user from a channel. Note that you don't need to know what channel to detach from, simply use `Event::clear_user()`.

### Usage
```c++
Event::clear_user(user::evouta); // Remove the user::evouta from whatever event channel it is connected to
```


## soft_event()
Creates a single software event - users connected to that channel will react to it in the same way as they would to one caused by the generator the channel is connected to. 
Great if you have to force trigger something. Note that a software event only lasts a single system clock cycle, so it's rather fast! 
The software events will invert the channel, and so will trigger something regardless of whether it needs a the event channel to go high or low.

### Usage
```c++
Event0.soft_event(); // Create a single software event on Event0
```


## start()
Starts an event generator channel by writing the generator selected by `set_generator()` function.

### Usage
```c++
Event0.start(); // Starts the Event0 generator channel
```


## stop()
Stops an event generator channel. The `Eventn` object retains memory of what generator it was previously set to.

### Usage
```c++
Event0.stop(); // Stops the Event0 generator channel
```
