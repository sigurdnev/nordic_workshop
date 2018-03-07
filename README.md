# Nordic Workshop


## HW Requirements
- nRF52 Development Kit 
- Computer
- Micro USB Cable

## SW Requirements
All participants must download the following software:
- nRF5 SDK v14.2.0 [download page](http://developer.nordicsemi.com/nRF5_SDK/nRF5_SDK_v14.x.x/)
   <pre>
    1.	Download the zip file
    2.	Create a new folder on your C:\ drive and name it Nordic_Semiconductor
    3.	Create another new folder inside Nordic_Semiconductor called nRF5_SDK_14.2.0_17b948a
    and extract the content of nRF5_SDK_14.2.0_17b948a.zip to this folder.
   </pre>


- Latest version of Segger Embedded Studio [download page](https://www.segger.com/downloads/embedded-studio/)
    <pre>
    1.	Download and install Segger Embedded Studio for your operating system .
    2.	Run the installer and follow the installer instructions.
    </pre>
    For commercial use of Embedded Studio, follow the steps in the SEGGER Embedded studio – Getting started video to
         install the free SES license. Link to video [Here](https://www.youtube.com/watch?v=YZouRE_Ol8g&t=334s)

- nRF Connect for Mobile, [download page](https://www.nordicsemi.com/eng/Products/Nordic-mobile-Apps/nRF-Connect-for-mobile-previously-called-nRF-Master-Control-Panel)
- nRF Command Line Tools [download page](http://infocenter.nordicsemi.com/topic/com.nordic.infocenter.tools/dita/tools/nrf5x_command_line_tools/nrf5x_installation.html?cp=5_1_1)

# Hands-on Tasks 
The hands-on tasks in this workshop ... .. .. ...

## TASK 1: Control LEDs using the nRF Toolbox App and 
Scope: Modify the ble_app_uart example to recognise specific commands sent from the nRF Toolbox app and turn on a LED when one of these commands are received.

### Step 1: Change the device name

Open the ble_app_uart example found in the nRF5_SDK_14.2.0\examples\ble_peripheral\ble_app_uart\pca10040\s132\ses folder. Find the `DEVICE_NAME` define and change the device to a unique name that is easily recognisable, for example.

#define DEVICE_NAME                     "Sigurd_UART"       


### Step 2

We need a variable to keep track of the current command that the nRF52 should handle. We can do this by creating an enumeration, which is basically a list of commands that are assigned a number from 0 and upwards. We create an enumeration  like this

```C    
    typedef enum {
        COMMAND_1,
        COMMAND_2,
        COMMAND_3,
        NO_COMMAND
    } uart_command_t;
```
Every variable of the uart_command_t type can be set to one of the commands in the list.  We've  added a `NO_COMMAND` command which is going to be the default state when no command has been received or the last command has been completed. After declaring the enumeration type `uart_command_t` we need to create a variable `m_command` of the `uart_command_t` type and initialize it to `NO_COMMAND`, i.e.

```C  
    uart_command_t m_command = NO_COMMAND;
```

In Segger Embedded Studio this should look something like this:

<img src="https://github.com/sigurdnev/nordic_workshop/blob/master/images/Task1_Step2.PNG" width="600"> 


### Step 3

Find the function `nus_data_handler`. This function is called when data is sent to the Nordic UART Service(NUS) from the nRF Toolbox app and this is where we have to look for the specific commands. The data that has been received is stored in a array pointed to by the `p_data` pointer and we need to store it a local array for later use. This can be done by using the [memcpy](https://www.tutorialspoint.com/c_standard_library/c_function_memcpy.htm) function. It will copy the content from cell 0 to `length` in the array pointed to by p_data into the uart_string.      

```C  
    char uart_string[BLE_NUS_MAX_DATA_LEN];
    memset(uart_string,0,BLE_NUS_MAX_DATA_LEN);
    memcpy( uart_string, p_data, length);
```
### Step 4

Now that we have copied the received data into the `uart_string` array we want to compare the content of `uart_string` with a known command. This can be done by using the [strcmp](https://www.tutorialspoint.com/c_standard_library/c_function_strcmp.htm) function, which will return 0 if `uart_string` is equal to the 

```C  
    if(strcmp(uart_string,"COMMAND_1") == 0 )
    {
        m_command = COMMAND_1;    
    }
    else if(strcmp(uart_string,"COMMAND_2") == 0 )
    {
        m_command = COMMAND_2;
    }
    else if(strcmp(uart_string,"COMMAND_3") == 0)
    {
        m_command = COMMAND_3;
    }
    else
    {
        m_command = NO_COMMAND;
    }
```
If the uart_string that we received is equal to the known "COMMAND_1" string, then we set the `m_command` variable to the corresponding command in enumeration we created in step 1.   

### Step 5

Now that the `m_command` variable is set to the correct command if the correct string is received, the last thing we need is a function that checks these commands at a regular interval and runs the code we have assosiated with that command. We'll call this function `uart_command_handler` and it takes the pointer to the m_command variable as an input. Inside the function we have a [switch](https://www.tutorialspoint.com/cprogramming/switch_statement_in_c.htm) statement, which is very useful when comparing a variable against a list of values, like our `uart_command_t` enumeration. The `uart_command_handler` should look something like this

```C  
    void uart_command_handler(uart_command_t * m_command)
    {
        uint32_t err_code = NRF_SUCCESS;

        switch(*m_command)
        {
            case COMMAND_1:
                // Put Action to COMMAND_1 here.
                break;

            case COMMAND_2:
                // Put Action to COMMAND_2 here.
                break;

            case COMMAND_3:
                // Put Action to COMMAND_3 here.
                break;

            case NO_COMMAND:
                // No command has been received -> Do nothing
                break;
                
            default:
                // Invalid command -> Do nothing.
                break;
        }
        /* Reset the command variable to NO_COMMAND after a command has been handled */
        *m_command = NO_COMMAND;

        // Check for errors 
        APP_ERROR_CHECK(err_code);
    }
```
After declaring the `uart_command_handler` we add the uart_command_handler to the infinite for-loop in main as shown below. 
```C 
    for (;;)
        {
            uart_command_handler(&m_command);
            power_manage();
        }
```

### Step 6

Now we want to toggle a led when we receive "COMMAND_1". Since the ble_app_uart example uses LED_1 on the nRF52 DK to indicate if the device is advertising or connected to central, so we have to toggle LED_4 instead. 
```C 
    case COMMAND_1:
        nrf_gpio_pin_toggle(LED_4); 
        break;
```
We also have to configure the pin connected to LED_4 as an output so make sure that you add to main()
```C 
    nrf_gpio_cfg_output(LED_4); 
```

### Step 7

Compile the project and flash it to the nRF52 DK. Make sure that you've also flashed the S132 v3.0 SoftDevice to your board. LED 1 on the nRF52 DK should start blinking, indicating that its advertising.  We've now completed the configuration on the nRF52 side  

### Step 8

Install the nRF Toolbox app on you Android/iOS phone. You can find the app [here](https://www.nordicsemi.com/eng/Products/Nordic-mobile-Apps/nRF-Toolbox-App) on Google Play Store and [here](https://itunes.apple.com/us/app/nrf-toolbox/id820906058?mt=8) on Apple App Store. Open the nRF Toolbox app and click the UART symbol, which should display the picture under "UART Menu". Press EDIT in the top right corner, the menu should now turn orange and then press the top-left square in the 3x3 matrix. The app should now display the same image as shown under "Edit Button Menu". Enter the command shown in the "Configure Command 1" and select 1 as the icon. After pressing OK you should see return to the orange edit screen shown under "Edit Mode". Press "DONE" in the top-right corner and you should return to the blue UART menu as shown under "Edit Complete".  



nRF Toolbox Menu  | UART Menu     | Edit Button Menu| Configure Command 1 | Edit Mode | Edit Completed |
------------ | ------------- | ------------  | ------------  | ------------  | ------------  |
<img src="https://github.com/bjornspockeli/elektra/blob/master/images/nrf_toolbox.png" width="200"> | <img src="https://github.com/bjornspockeli/elektra/blob/master/images/default.png" width="200"> | <img src="https://github.com/bjornspockeli/elektra/blob/master/images/edit.png" width="200"> | <img src="https://github.com/bjornspockeli/elektra/blob/master/images/command_1.png" width="200"> | <img src="https://github.com/bjornspockeli/elektra/blob/master/images/edit_done.png" width="200"> | <img src="https://github.com/bjornspockeli/elektra/blob/master/images/done.png" width="200">


### Step 9

Press the Connect button, this should bring up a list of nearby BLE devices. Select the device with the name you assigned in step 1 of this task. It should be the one of the devices with the strongest signal. LED 1 on your nRF52 DK should now stop blinking and stay lit, indicating that its in a connected state. 

<img src="https://github.com/bjornspockeli/elektra/blob/master/images/device_list.png" width="200">

### Step 10

Pressing the button we configured to send "COMMAND_1" to the nRF52 DK should turn on LED 4 on the nRF52 DK. Pressing it again should turn it off. Congratulations, you've just controlled one of the GPIO pins of the nRF52 using Bluetooth Low Energy.

