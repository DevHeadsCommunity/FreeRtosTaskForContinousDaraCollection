## Hardware Setup

### Connecting MPU to Esp32 Microcontroller:

- Connecting SDA and SCL pins from the MPU to the microcontroller's I2C pins.

- Pull-up resistors (4.7kΩ) may be required for I2C.

- Provide power (VCC and GND) to the MPU.

### Configuring MPU using I2C:

- Initialize the communication.

- Write to the MPU registers to set the desired sampling rate, sensitivity (±2g for accelerometer), and other parameters.

## libraries 
MPU6050 Arduino Library

## Create FreeRTOS Tasks:

### Task 1: Sensor Data Collection

- Repeated data reading on MPU at a specified frequency (100Hz).

- Read accelerometer and gyroscope data using I2C.


### Task 2: Preprocessing

- Filter data (low-pass filter to remove noise).

- Normalize or scale data for consistency.


### Task 3: ML Inference

- Feed preprocessed data to a Tflite model for inference.



```
void vTaskReadMPU(void *pvParameters) {
    while (1) {
        int16_t ax, ay, az, gx, gy, gz;

// Custom read function
        MPU_ReadAccelGyro(&ax, &ay, &az, &gx, &gy, &gz); 


// Send data to preprocessing task
        xQueueSend(queueHandle, &ax, 0); 


// 100Hz samp
 vTaskDelay(pdMS_TO_TICKS(10)); 

    }
}

// Preprocess and buffer data
void vTaskPreprocess(void *pvParameters) {
    int16_t ax, ay, az;
    while (1) {
        if (xQueueReceive(queueHandle, &ax, portMAX_DELAY)) {
            
            normalize(&ax, &ay, &az); // Custom normalization function
        }
    }
}
```


## Collect Shake Data for ML Training

### Collect Data:

- Label data for shake gestures ("shake") and non-shake gestures ("still").

- Log raw accelerometer and gyroscope data to a file (CSV format) for training.

- Use tools like Edge Impulse to upload and annotate data.


### Example of Data:

timestamp, ax, ay, az, gx, gy, gz, label
0, 0.12, 0.45, -9.81, 0.01, 0.02, 0.03, shake
1, 0.10, 0.40, -9.80, 0.00, 0.01, 0.02, still



## Train an ML Model

1. Use ML frameworks like TensorFlow.


2. Train with the collected dataset to classify gestures.


3. Convert the model to a format suitable for microcontroller TensorFlow Lite).



## Integrate ML Model

1. Deploy the trained model to the Esp32 microcontroller.


2. Run inference in a FreeRTOS task to detect shake gestures in real-time.



# Tools and References studied to compute the above 

FreeRTOS Tutorial: https://www.freertos.org/Documentation/FreeRTOS-documentation-and-books.html

MPU6050 Datasheet: https://invensense.tdk.com/products/motion-tracking/6-axis/mpu-6050/

Machine Learning: https://www.tensorflow.org/lite/microcontrollers

Edge Impulse: https://docs.edgeimpulse.com



