# Nordic Workshop


## HW Requirements
- nRF52 Development Kit
- Computer
- Micro USB Cable

## SW Requirements
All participants must download the following software:
- **nRF5 SDK v14.2.0 [download page](http://developer.nordicsemi.com/nRF5_SDK/nRF5_SDK_v14.x.x/)**
   <pre>
    1.	Download the zip file
    2.	Create a new folder on your C:\ drive and name it Nordic_Semiconductor
    3.	Create another new folder inside Nordic_Semiconductor called nRF5_SDK_14.2.0_17b948a
    and extract the content of nRF5_SDK_14.2.0_17b948a.zip to this folder.
   </pre>

- **nRF Command Line Tools [download page](http://infocenter.nordicsemi.com/topic/com.nordic.infocenter.tools/dita/tools/nrf5x_command_line_tools/nrf5x_installation.html?cp=5_1_1)**

- **Latest version of Segger Embedded Studio [download page](https://www.segger.com/downloads/embedded-studio/)**
    <pre>
    1.	Download and install Segger Embedded Studio for your operating system.
          Embedded Studio is available for Windows, macOS, and Linux.
    2.	Run the installer and follow the installer instructions.
    </pre>
    For commercial use of Embedded Studio, follow the steps in the SEGGER Embedded studio – Getting started video to
         install the free SES license. Link to video [Here](https://www.youtube.com/watch?v=YZouRE_Ol8g&t=334s)

- **nRF Connect for Mobile, [download page](https://www.nordicsemi.com/eng/Products/Nordic-mobile-Apps/nRF-Connect-for-mobile-previously-called-nRF-Master-Control-Panel)**
Available on the App Store for iOS devices, and Google Play for Android devices


# Hands-on Tasks
The hands-on tasks in this workshop ... .. .. ...

## TASK 1: Control LEDs using the nRF Toolbox App
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

<img src="https://github.com/sigurdnev/nordic_workshop/blob/master/images/Task1_Step2.PNG" width="800">


### Step 3

Find the function `nus_data_handler`. This function is called when data is sent to the Nordic UART Service(NUS) from the nRF Toolbox app and this is where we have to look for the specific commands. The data that has been received is stored in a array pointed to by the `p_data` pointer and we want to store it in a local char array for later use. This can be done by using the [memcpy](https://www.tutorialspoint.com/c_standard_library/c_function_memcpy.htm) function. It will copy the content from cell 0 to `length` in the array pointed to by p_data into the uart_string.      

```C  
    char uart_string[BLE_NUS_MAX_DATA_LEN];
    memset(uart_string,0,BLE_NUS_MAX_DATA_LEN);
    memcpy(uart_string, p_evt->params.rx_data.p_data, p_evt->params.rx_data.length);
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
        UNUSED_RETURN_VALUE(NRF_LOG_PROCESS());
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

Compile the project by pressing the F7 key, and flash it to the nRF52 DK. You can flash the project by clicking on the Target tab, and click on "Download ble_app_uart_pca10040_s132". The project will also be flahsed when start a debug seesion. A debug session can be started by pressing the F5 key twice.
The Segger Embedded Studio project have already been configured to flash the SoftDevice for us. LED 1 on the nRF52 DK should start blinking, indicating that its advertising.  We've now completed the configuration on the nRF52 side  

### Step 8

Install the nRF Toolbox app on you Android/iOS phone. You can find the app [here](https://www.nordicsemi.com/eng/Products/Nordic-mobile-Apps/nRF-Toolbox-App) on Google Play Store and [here](https://itunes.apple.com/us/app/nrf-toolbox/id820906058?mt=8) on Apple App Store. Open the nRF Toolbox app and click the UART symbol, which should display the picture under "UART Menu". Press EDIT in the top right corner, the menu should now turn orange and then press the top-left square in the 3x3 matrix. The app should now display the same image as shown under "Edit Button Menu". Enter the command shown in the "Configure Command 1" and select 1 as the icon. After pressing OK you should see return to the orange edit screen shown under "Edit Mode". Press "DONE" in the top-right corner and you should return to the blue UART menu as shown under "Edit Complete".  



nRF Toolbox Menu  | UART Menu     | Edit Button Menu| Configure Command 1 | Edit Mode | Edit Completed |
------------ | ------------- | ------------  | ------------  | ------------  | ------------  |
<img src="https://github.com/sigurdnev/nordic_workshop/blob/master/images/nrf_toolbox.png" width="200"> | <img src="https://github.com/sigurdnev/nordic_workshop/blob/master/images/default.png" width="200"> | <img src="https://github.com/sigurdnev/nordic_workshop/blob/master/images/edit.png" width="200"> | <img src="https://github.com/sigurdnev/nordic_workshop/blob/master/images/command_1.png" width="200"> | <img src="https://github.com/sigurdnev/nordic_workshop/blob/master/images/edit_done.png" width="200"> | <img src="https://github.com/sigurdnev/nordic_workshop/blob/master/images/done.png" width="200">


### Step 9

Press the Connect button, this should bring up a list of nearby BLE devices. Select the device with the name you assigned in step 1 of this task. It should be the one of the devices with the strongest signal. LED 1 on your nRF52 DK should now stop blinking and stay lit, indicating that its in a connected state.

<img src="https://github.com/sigurdnev/nordic_workshop/blob/master/images/device_list.png" width="200">

You can now press the button we configured to send "COMMAND_1" to the nRF52 DK. This should turn on LED 4 on the nRF52 DK. Pressing it again should turn it off. Congratulations, you've just controlled one of the GPIO pins of the nRF52 using Bluetooth Low Energy.

## Task 2: Control the Servo using the nRF Toolbox App

**Scope:** The goal of this task is to include the PWM library in the ble_app_uart example so that we can control the servo from our smartphone. The app_pwm library is documented on [this](https://infocenter.nordicsemi.com/topic/com.nordic.infocenter.sdk5.v12.2.0/lib_pwm.html?resultof=%22%61%70%70%5f%70%77%6d%5f%69%6e%69%74%22%20) Infocenter page

Connecting the Servo to your nRF52 DK:

The three wires coming from the SG92R Servo are:

Brown: 	Ground 				- Should be connected to one of the pins marked GND on your nRF52 DK. 

Red: 	5V 					- Should be connected to the pin marked 5V on your nRF52 DK.

Orange: PWM Control Signal 	- Should be connected to one of the unused GPIO pins of the nRF52 DK (for example P0.4, pin number 4).

1 - In order to use the app_pwm library in the ble_app_uart project you have to add the necessary .c and .h files to the project. The app_pwm library uses the following files:

* `app_pwm.h`
* `app_pwm.c`
* `nrf_drv_ppi.c`
* `nrf_drv_timer.c`

### Adding .h files 

<img src="https://github.com/bjornspockeli/elektra/blob/master/images/include_path.PNG" width="1000"> 
Click the "Options for target" button in Keil, then select the C/C++ tab and clik on the "..." on the side of the "Inlude Paths" window. Navigate to the components folder and then find the missing .h file in either nrf_drivers or libraries. 

### Adding .c files
 
<img src="https://github.com/bjornspockeli/elektra/blob/master/images/add_c_files.png" width="1000"> 
Right-clik the folder that you want to add the .c file to and select "Add existing files to Group '____'". Navigate to the components folder and then find the missing .c file in either nrf_drivers or libraries. 


You also have to make sure that the correct nRF_Drivers and nRF_Libraries are enabled in the sdk_config.h file. If you're having compilation issues and/or linker errors then select the Configuration Wizard Tab in the bottom of the text window after opening `sdk_config.h` in the ble_app_uart example and compare it to the one in the pwm_library example.

### Modifying sdk_config.c  

<img src="https://github.com/bjornspockeli/elektra/blob/master/images/sdk_config.PNG" width="1000">

Under nRF_Libraries the following boxes must be checked

* APP_PWM_ENABLED

Under nRF_Drivers the following boxes must be checked
* PPI_ENABLED
* TIMER_ENABLED
    * TIMER1_ENABLED


2 - The first thing we have to do in main.c is to include the header to the PWM library, `app_pwm.h` and create a PWM instance using the TIMER1 peripheral. This is done as shown below 

```C
    #include "app_pwm.h"
    
    APP_PWM_INSTANCE(PWM1,1);                       // Create the instance "PWM1" using TIMER1.
```
2 -	The second thing we have to do is creating the function `pwm_init()` where we configure, initialize and enable the PWM peripheral. You configure the pwm library by creating a app_pwm_config_t struct like shown below

```C
    app_pwm_config_t pwm_config = {
        .pins               = {4, APP_PWM_NOPIN},
        .pin_polarity       = {APP_PWM_POLARITY_ACTIVE_HIGH, APP_PWM_POLARITY_ACTIVE_LOW}, 
        .num_of_channels    = 1,                                                          
        .period_us          = 20000L                                                
    };
```
This struct contains the information about which pins that are used as PWM pins, which polarity the pisn should have, how many channels(the number of PWM outputs, this is limited to 2 per PWM instance) and the period of the PWM signal. The struct is given as an input to the [app_pwm_init](https://infocenter.nordicsemi.com/topic/com.nordic.infocenter.sdk5.v12.2.0/group__app__pwm.html#gae3b3e1d5404fd776bbf7bf22224b4b0d) function which initializes the PWM library. 

```C
    uint32_t err_code;
    err_code = app_pwm_init(&PWM1,&pwm_config,NULL);
    APP_ERROR_CHECK(err_code);
```

You can initialize the PWM library with a callback function that is called when duty cycle change process is finsihed, but this is not necessary for this example so we will just pass NULL as an argument. After initializng the PWM library you have to enable the PWM instance by calling [app_pwm_enable](https://infocenter.nordicsemi.com/topic/com.nordic.infocenter.sdk5.v12.2.0/group__app__pwm.html#ga94f5d824afec86aff163f7cccedaa436).

```C
    app_pwm_enable(&PWM1);
```
The pwm_init() function is now finished and can add it to the `main()` function before the infinite for-loop.

3 - Now that we have initialized the PWM library its time to set the duty cycle of the PWM signal to the servo using the  [app_pwm_channel_duty_set](https://infocenter.nordicsemi.com/topic/com.nordic.infocenter.sdk5.v12.2.0/group__app__pwm.html#ga071ee86851d8c0845f297df5d23a240d) function. This will set the duty cycle of the PWM signal, i.e. the percentage of the total time the signal is high or low depending on the polarity that has been chosen. If we want to set the PWM signal to be high 50% of the time, then we call `app_pwm_channel_duty_set` with the following parameters.

```C
    while (app_pwm_channel_duty_set(&PWM1, 0, 50) == NRF_ERROR_BUSY);
```
The `app_pwm_channel_duty_set` function should be called until it does not return `NRF_ERROR_BUSY` in order to make sure that the duty cycle is correctly set. 

4 - The goal of this task was to make the servo sweep from its maximum angle to its minimum angle. This can be done by calling app_pwm_channel_duty_set twice with a delay between the two calls in the main while-loop.

```C
    while (true)
    {
        while (app_pwm_channel_duty_set(&PWM1, 0, duty_cycle) == NRF_ERROR_BUSY);
        nrf_delay_ms(1000);
        while (app_pwm_channel_duty_set(&PWM1, 0, duty_cycle) == NRF_ERROR_BUSY);
        nrf_delay_ms(1000);
    }
    
```
The code snippet above sets the duty cycle to 0, you have to figure out the correct duty cycle values for the min and max angle. 

Tips:
* Period should be 20ms (20000us) and duty cycle  for the min and max angle corresponds to 1ms and 2ms respectivly.

3 - Add `SERVO_POS_1` and  `SERVO_POS_2` to the `uart_command_t` enumeration created in Task 7, step 2. Add these commands to the `nus_data_handler` and the `uart_command_handler`. Call `app_pwm_channel_duty_set` when the commands are processed by the `uart_command_handler`, i.e.
```C 
    case SERVO_POS_1:
        while (app_pwm_channel_duty_set(&PWM1, 0, duty_cycle) == NRF_ERROR_BUSY);
        break;
```

4 - Configure two buttons in the nRF Toolbox app to send the `SERVO_POS_1` and  `SERVO_POS_2` commands to your nRF52 DK.

5 - Compile the project, flash it to the nRF52 DK and control the servo using the nRF Toolbox App.

## TASK 3: Measure the die temperature of the nRF52 and send it to the nRF Toolbox app.
**Scope:**  Measure the temperature of the nRF52 die and use an application timer to send the measurement to the nRF Toolbox App.

### Step 1

In order to measure the temperature of the nRF52 die you have to read the registers of the TEMP peripheral of the nRF52, see [this](https://infocenter.nordicsemi.com/topic/com.nordic.infocenter.nrf52832.ps.v1.1/temp.html?cp=2_2_0_26#concept_fcz_vw4_sr) page on the Nordic Infocenter. However, the SoftDevice uses this peripheral to calibrate the clock of the nRF52 so that its accurate enough to be used for BLE. We can therefore not access the TEMP registers directly, we have to go through the SoftDevice and ask it to check what the temperature is. This is done by calling the [sd_temp_get](https://infocenter.nordicsemi.com/topic/com.nordic.infocenter.s132.api.v3.0.0/group___n_r_f___s_o_c___f_u_n_c_t_i_o_n_s.html#gade0ea69f513ff1feab2c4f6e1c393313
) function.


We can now create a function called `read_temperature()` that in turn calls `sd_temp_get` and returns the temperature in degrees celsius.

```C
    static double read_temperature()
    {
        int32_t temp;
        sd_temp_get(&temp);

        return ((double)temp*0.25);   
    }
```

### Step 2
Next, we're going to create an application timer that calls `read_temperature()` periodically. First, create the timer ID as shown below

```C
    APP_TIMER_DEF(m_temp_timer_id);                 // Create the timer ID "m_temp_timer_id" .
```

### Step 3
Next, we're going to create the application timer timeout handler that is going to call `read_temperature()`. We need to use the `ble_nus_string_send()` function to send the measured temperature to the nRF Toolbox app. This function takes the pointer to an `uint8_t` array so we need to format a string and place it in the array. The  [sprintf](https://www.tutorialspoint.com/c_standard_library/c_function_sprintf.htm) function does exactly this, i.e. copies the content of a string into an array.

```C
    void temp_timer_timeout_handler(void * p_context)
    {
        double temp = read_temperature();

        // Place the temperature measurement into the data array
        uint32_t err_code;
        uint8_t data[20];

        sprintf((char *)data, "Temperature: %0.2f", temp);
        uint16_t length = sizeof(data);

        //Send temperature measurement to nRF Toolbox app
        err_code = ble_nus_string_send(&m_nus, data, &length);
        APP_ERROR_CHECK(err_code);
    }
```

### Step 4
Next, we need to create the timer and specify that it should use `temp_timer_timeout_handler` as its timeout handler.

```C
    void create_timers()
    {
        uint32_t err_code;
        // Create  temperature timer
        err_code = app_timer_create(&m_temp_timer_id,
                                    APP_TIMER_MODE_REPEATED,
                                    temp_timer_timeout_handler);
        APP_ERROR_CHECK(err_code);
    }
```
### Step 5
Now, the only thing that remains is to start the application timer. However, it is important that the `ble_nus_string_send` function is not called when we're not connected to the nRF Toolbox app. If we try to send the string containing the temperature without being in a connection then the ble_nus_string_send function will return NRF_ERROR_INVALID_STATE, and this will be passed into the APP_ERROR_CHECK() error-handler. It should therefore only be possible to start the timer when we issue a command from nRF Toolbox.  

Add `TEMP_TIMER_START` and `TEMP_TIMER_STOP` to the uart_command_t enumeration and modify the `nus_data_handler` so that these commands are recognized.


Lastly, add the following cases to the `uart_command_handler`

```C
        case TEMP_TIMER_START:
            err_code = app_timer_start(m_temp_timer_id, APP_TIMER_TICKS(1000),NULL);
            APP_ERROR_CHECK(err_code);
            break;

        case TEMP_TIMER_STOP:
            err_code = app_timer_stop(m_temp_timer_id);
            APP_ERROR_CHECK(err_code);
            break;
```

### Step 6
Configure two buttons in the nRF Toolbox app to send the `TEMP_TIMER_START`and `TEMP_TIMER_STOP` commands

### Step 7
Before compiling the project, we need to configure Embedded Studio to allow the data type `double` to be used by the sprintf function.
In the Project Explorer on the left side, right click on **Project 'ble_app_uart_pca10040_s132**, and click `Edit Options...`

Select the Common configuration as shown in the image below.

<img src="https://github.com/sigurdnev/nordic_workshop/blob/master/images/Task2_Step7.png" width="200">

Click on Printf/Scanf, and set the option `Printf Floating Point Supported` to `Double`. Press OK to apply the change.

<img src="https://github.com/sigurdnev/nordic_workshop/blob/master/images/Task2_Step7_2.png" width="400">

### Step 8
Compile the project and flash it to you nRF52 DK.  After pressing the button you configured to send the `TEMP_TIMER_START` command you should be able to see the temperature in the nRF Toolbox app log ( you open this by holding your finger above the UART text to the left of the screen and swiping from left to right)  

<img src="https://github.com/sigurdnev/nordic_workshop/blob/master/images/temperature.png" width="500">

## TASK x: Measure the supply voltage using the SAADC and send it to the nRF Toolbox app.
Scope: Use the SAADC peripheral to measure the supply voltage, and use an application timer to send the measurement to the nRF Toolbox App.

### Step 1

### Step 2

### Step 3


## BONUS TASK: Creating a Custom Service
Scope: The aim of this task is simply to create one service with one characteristic without too much theory in between the steps. In this task we will be starting from scratch in the ble_app_template project.

This is a bonus task if you have managed to finish the previous task within the time period of the workshop. Note that this is a larger task than the previous tasks, and it takes more time to finish this task. 
