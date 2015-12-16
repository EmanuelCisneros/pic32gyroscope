# pic32gyroscope

/* Giroscopio L3GD20
* Este programa está diseñado para establecer comunicación
* con el giroscopio por medio de SPI y obtener los valores
* de aceleración angular. Con los valores obtenidos se realiza
* una corrección del offset, cálculo del nivel de ruido y la
* integración de la velocidad para obtener los grados.
*/

// Se incluyen las librerías necesarias
#include <Wire.h>
#include <L3G.h>

L3G gyro;  // Declaración del giroscopio

int sampleNum=500;  // Número de muestras para el promediar el offset

// Declaración de variables de offset y ruido
int dc_offsetX=0; // Offset en X
double noiseX=0;  // Nivel de ruido en X

int dc_offsetY=0;  // Offset en Y
double noiseY=0;  // Nivel de ruido en Y

int dc_offsetZ=0;  // Offset en Z
double noiseZ=0;  // Nivel de ruido en Z

// Declaración de variables para muestras de la velocidad
unsigned long time;
int sampleTime=10;
int rateX;  // Medición actual de velocidad X
int rateY;  // Medición actual de velocidad Y
int rateZ;  // Medición actual de velocidad Z

// Declaración de variables para medición del ángulo
int prev_rateX=0;  // Medición previa de velocidad X
double angleX=0;  // Valor del ángulo en Z

int prev_rateY=0;  // Medición previa de velocidad Y
double angleY=0;  // Valor del ángulo en Y

int prev_rateZ=0;  // Medición previa de velocidad Z
double angleZ=0;  // Valor del ángulo en Z

// Bloque de configuración
void setup() {
  Serial.begin(9600);  // Se abre la comunicación con el PC
  Wire.begin();  // Se abre la comunicación con el sensor

  // Si no hay respuesta del sensor se muestra un mensaje y
  // el programa se detiene
  if (!gyro.init())
  {
    Serial.println("Ha fallado la comunicacion con el sensor");
    while (1);
  }

  gyro.enableDefault();  // Si hay comunicación se configura
                         // por defecto el sensor
 
  calculoOffset();  // Función para calcular el offset de cada eje
}

// Bloque de Programa
void loop() {
  if(millis() - time > sampleTime)
  {
    time = millis(); // Tiempo de la próxima actualización
    gyro.read();  // Lectura de medición de velocidad
    
    // **********************************
    rateX=((int)gyro.g.x-dc_offsetX)/100;  // Calculo de la velocidad X en °/s

    // Sumatoria (integración) de la velocidad para el calculo de grados X
    angleX += ((double)(prev_rateX + rateX) * sampleTime) / 2000;
    
    prev_rateX = rateX;  // Se debe recordar una muestra previa de velocidad
    // Se coloca el ángulo recorrido en una escala de 0 a 360 grados
    if (angleX < 0){
      angleX += 360;
    }
    else if (angleX >= 360)
    {
      angleX -= 360;
    }
    
    // **********************************
    rateY=((int)gyro.g.y-dc_offsetY)/100;  // Calculo de la velocidad Y en °/s

    // Sumatoria (integración) de la velocidad para el calculo de grados Y
    angleY += ((double)(prev_rateY + rateY) * sampleTime) / 2000;
    
    prev_rateY = rateY;  // Se debe recordar una muestra previa de velocidad
    // Se coloca el ángulo recorrido en una escala de 0 a 360 grados
    if (angleY < 0){
      angleY += 360;
    }
    else if (angleY >= 360)
    {
      angleY -= 360;
    }   
