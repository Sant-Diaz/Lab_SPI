<h1 align="center">Laboratorio 3 — Cálculo Ambulatorio del Indice Pletismográfico Quirúrgico (SPI)</h1>

**Universidad Militar Nueva Granada**  | Programa: Ingeniería Biomédica · Semestre VII  | Asignatura: Instrumentación Biomédica y Biosensores


## 1. Introducción

### 1.1 Contexto teórico

Una gran proporción de procedimientos quirúrgicos genera daño tisular, razón por la cual entre el 20 % y el 80 % de los pacientes reportan dolor posoperatorio de moderado a intenso [1]. Esto hace fundamental evaluar con precisión la **nocicepción intraoperatoria** en pacientes bajo anestesia general y ajustar la analgesia oportunamente para reducir el dolor posoperatorio.

Entre los métodos desarrollados para la monitorización cuantitativa y objetiva de la nocicepción se destaca el **Índice Pletismográfico Quirúrgico** (SPI, *Surgical Pleth Index*, GE Healthcare). Este índice se basa en la señal **fotopletismográfica (PPG)** para estimar el equilibrio entre la activación nociceptiva y la acción analgésica durante la anestesia general [3].

El SPI puede variar entre **0 y 100**, donde los valores más altos indican mayor respuesta nociceptiva:

<div align="center">

| Rango SPI | Interpretación |
|-----------|---------------|
| 0 – 20 | Analgesia profunda / sin estrés nociceptivo |
| **20 – 50** | **Rango objetivo intraoperatorio óptimo** |
| > 50 | Respuesta nociceptiva elevada |

</div>

> Los valores de SPI deben mantenerse por debajo de 50, evitando incrementos superiores a 10 unidades durante la cirugía [5].

### 1.2 Características de la onda PPG relevantes para el SPI

La señal PPG refleja las variaciones del volumen sanguíneo periférico sincronizadas con el ciclo cardíaco. De cada pulsación se extraen dos características clave:

- **PPI** (*Pulse-to-Pulse Interval*, ms): tiempo entre dos máximos consecutivos, inversamente proporcional a la frecuencia cardíaca.
- **PPA** (*Pulse Pressure Amplitude*): diferencia entre el valor máximo y mínimo de cada ciclo, relacionada con el tono vasomotor periférico.

Ambas variables son moduladas por el sistema nervioso autónomo y, por tanto, reflejan el nivel de estrés fisiológico del paciente.

### 1.3 Importancia de la práctica

La relevancia de esta práctica reside en la extracción y cálculo de características derivadas de la onda de pulso para estimar el balance nocicepción-analgesia. Al replicar en condiciones ambulatorias el principio de funcionamiento del SPI clínico, se comprende cómo un índice utilizado en quirófano puede obtenerse con hardware de bajo costo.



## 2. Objetivos

**General:** Desarrollar un sistema de medición continua del índice pletismográfico quirúrgico (SPI) en condiciones ambulatorias.

**Específicos:**
- Reconocer las características fundamentales de la onda de pulso a partir de las cuales se obtiene el SPI.
- Construir un sistema que calcule el SPI en tiempo real bajo condiciones ambulatorias.
- Validar el funcionamiento del sistema mediante el *Cold Pressor Test* (CPT).



## 3. Definición matemática del SPI

El SPI se calcula a partir de las dos variables derivadas de la señal PPG. La aproximación académica de la fórmula propuesta por Huiku *et al.* (2007) es:

$$SPI = 100 - (0.33 \cdot \overline{PPA}_{norm} + 0.67 \cdot \overline{PPI}_{norm}) \times 100$$

Donde:
- $\overline{PPA}_{norm}$: amplitud de pulso normalizada respecto a un valor de referencia basal.
- $\overline{PPI}_{norm}$: intervalo entre pulsos normalizado respecto al mismo valor de referencia.
- Los coeficientes **0.33** y **0.67** reflejan la ponderación relativa de cada variable, dando mayor peso al intervalo entre pulsos.

> La fórmula exacta propietaria de GE Healthcare no está publicada en su totalidad. La expresión anterior es la aproximación académica más referenciada en la literatura [3][4].

Con cada nuevo máximo detectado en la señal PPG el sistema:
1. Calcula el **PPI** como la diferencia temporal respecto al máximo anterior.
2. Calcula el **PPA** como `valor_máximo − valor_mínimo` del ciclo.
3. Normaliza ambos valores respecto a la ventana basal (primeros 40 segundos).
4. Aplica la fórmula para obtener el SPI del latido actual.



## 4. Procedimiento

### Parte A — Sistema de adquisición

#### A.1 Hardware utilizado

El sistema de adquisición se construyó con los siguientes componentes principales:

- **Módulo MAX30102** — Sensor de oximetría y frecuencia cardíaca por PPG (comunicación I2C).
- **SP32** — Microcontrolador para leer el sensor y enviar datos por puerto serial.

El módulo MAX integra el LED infrarrojo, el fotodetector y el circuito de acondicionamiento de señal internamente, lo que simplifica el montaje respecto a una implementación discreta.

#### A.2 Conexión del módulo MAX a la SP32

<div align="center">

<table>
  <tr>
    <th>Pin módulo MAX</th>
    <th>Pin SP32</th>
  </tr>
  <tr>
    <td>VIN / VCC</td>
    <td>3.3V </td>
  </tr>
  <tr>
    <td>GND</td>
    <td>GND</td>
  </tr>
  <tr>
    <td>SDA</td>
    <td>D22</td>
  </tr>
  <tr>
    <td>SCL</td>
    <td>D21</td>
  </tr>  
</table>

</div>

> El módulo MAX30102 opera a 1.8V internamente pero acepta niveles lógicos de 3.3V en I2C

#### A.3 Cold Pressor Test (CPT)

El *Cold Pressor Test* es una técnica experimental que consiste en sumergir la mano en agua helada (0 – 4 °C) durante un tiempo determinado para generar una respuesta nociceptiva aguda, reproducible y controlada sin causar daño tisular permanente.

**Fundamento fisiológico:** El frío intenso activa los nociceptores tipo Aδ y C, generando activación simpática que se manifiesta como vasoconstricción periférica (↓ PPA) y aumento de la frecuencia cardíaca (↓ PPI), lo que eleva el SPI.

**Protocolo aplicado:**
<div align="center">

<table>
  <tr>
    <th>Tiempo (s)</th>
    <th>Fase</th>
    <th>Condición</th>
  </tr>
  <tr>
    <td>0 – 40</td>
    <td>Basal</td>
    <td>Reposo, dedo sobre sensor</td>
  </tr>
  <tr>
    <td>40 – 80</td>
    <td>CPT</td>
    <td>Mano contralateral en frío (0 – 4 °C)</td>
  </tr>
  <tr>
    <td>80 – 120</td>
    <td>Recuperación</td>
    <td>Retiro de la mano, vuelta al reposo</td>
  </tr>
</table>

</div>

El montaje experimental del *Cold Pressor Test* se realizará utilizando una bolsa de gel previamente congelada como estímulo térmico en lugar de inmersión en agua. El voluntario permanecerá en reposo en posición cómoda, con un sensor colocado en el dedo para el registro de variables fisiológicas. Durante la fase basal se adquirirán señales sin estímulo; posteriormente, en la fase de activación, el sujeto colocará la mano contralateral en contacto directo con la bolsa de gel frío para inducir la respuesta nociceptiva. Finalmente, en la fase de recuperación, se retirará la mano del estímulo frío y se continuará el registro hasta que las variables retornen a condiciones cercanas al basal. Este montaje permite un estímulo controlado, seguro y reproducible, manteniendo condiciones similares al protocolo estándar del test.
El voluntario puede retirar la mano si el dolor se vuelve insoportable. La maniobra no debe exceder 3 minutos en ningún caso.


<div align="center">

<table>
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/35d64fd4-6bb7-49dd-8622-e337eeae0a8e" width="300"><br>
      <sub><b>a. Reposo</b></sub>
    </td>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/a425c85f-198b-45d8-be8a-fc1607f68b49" width="300"><br>
      <sub><b>b. Mano contraria en frío</b></sub>
    </td>
  </tr>
</table>

</div>


### Parte B — Código MATLAB y resultados

#### B.1 Código de Arduino 
El siguiente código implementa la adquisición y procesamiento de la señal infrarroja (IR) proveniente de un sensor MAX30105. A partir de esta señal, se calcula el índice de perfusión (PI%), una medida relacionada con la variabilidad del flujo sanguíneo periférico.

```
#include <Wire.h>
#include "MAX30105.h"
#include "heartRate.h"

MAX30105 particleSensor;

const int WINDOW_SIZE = 100;
long irWindow[WINDOW_SIZE];
int  windowIndex = 0;
bool windowFull  = false;

void setup() {
  Serial.begin(115200);
  Wire.begin(21, 22);

  if (!particleSensor.begin(Wire, I2C_SPEED_FAST)) {
    Serial.println("ERROR");
    while (1);
  }

  particleSensor.setup();          // configuración base
  particleSensor.setPulseAmplitudeRed(0);   // apagado rojo
  particleSensor.setPulseAmplitudeIR(0x1F); // encender IR con buena amplitud
  particleSensor.setPulseAmplitudeGreen(0);  // apagado verde
}

void loop() {
  long irRaw = particleSensor.getIR();

  irWindow[windowIndex] = irRaw;
  windowIndex = (windowIndex + 1) % WINDOW_SIZE;
  if (windowIndex == 0) windowFull = true;

  float PI_percent = 0.0;

  if (windowFull) {
    long irMax = irWindow[0], irMin = irWindow[0];
    long irSum = 0;

    for (int i = 0; i < WINDOW_SIZE; i++) {
      if (irWindow[i] > irMax) irMax = irWindow[i];
      if (irWindow[i] < irMin) irMin = irWindow[i];
      irSum += irWindow[i];
    }

    float AC = (float)(irMax - irMin);
    float DC = (float)irSum / WINDOW_SIZE;

    if (DC > 0) PI_percent = (AC / DC) * 100.0;
  }

  // Enviar IR crudo + PI%
  Serial.print(irRaw);
  Serial.print(",");
  Serial.println(PI_percent, 4);

  delay(10); // ~100 Hz
}
```
Este algoritmo emplea una ventana deslizante de 100 muestras para estimar las componentes AC (pulsátil) y DC (continua) de la señal IR. A partir de estas, se calcula el índice de perfusión como la relación entre ambas, expresada en porcentaje. Este método permite obtener una señal más estable y representativa del comportamiento hemodinámico periférico.
#### B.2 Codigo de Matlab
El siguiente script en MATLAB permite adquirir, procesar y analizar una señal PPG en tiempo real proveniente de un microcontrolador. A partir de esta señal, se realiza el cálculo de la frecuencia cardíaca (BPM) y del índice de perfusión simplificado (SPI), útil para evaluar cambios hemodinámicos como los inducidos por el Cold Pressor Test.
```
clc;
clear;
close all;
```
Se define la frecuencia de muestreo y se solicita al usuario el tiempo total de adquisición.
```
Fs = 100;

answer = inputdlg("Ingrese el tiempo de adquisición (s):", ...
"Tiempo de adquisición", 1, {"20"});

if isempty(answer)
    error("Cancelado");
end

T = str2double(answer{1});
```
Se establece la comunicación con el microcontrolador para recibir los datos de la señal PPG.
```
s = serialport("COM8",115200);
configureTerminator(s,"LF");
flush(s);
pause(2);
```
Se configura la gráfica para mostrar la señal PPG y los picos detectados.
```
figure;
h1 = animatedline('LineWidth',1.5);
hPeak = animatedline('Color','g','Marker','o','LineStyle','none');

xlabel("Tiempo [s]");
ylabel("PPG");
title("Inicializando...");
grid on;
```
Se crean buffers y variables necesarias para el procesamiento, detección de picos y almacenamiento de resultados.
```
WINDOW_TIME = 5;
WINDOW_SAMPLES = round(WINDOW_TIME * Fs);

ppg_rt = [];
ts_rt  = [];
ppg_filt = [];

estado = 0;
last_peak_time = 0;
last_valley = 0;

peak_times = [];
pulse_amplitudes = [];

SPI_values = [];
SPI_times  = [];

all_peak_times  = [];
all_peak_values = [];
```
Se descartan muestras iniciales para evitar ruido del sensor al inicio.
```
disp("Estabilizando...");
tic;
while toc < 1.5
    readline(s);
end
```
Se leen los datos desde el puerto serial, se validan y se almacenan junto con su tiempo correspondiente.
```
linea = readline(s);
nums = regexp(linea,'[-+]?\d*\.?\d+','match');
y = str2double(nums{1});
```
Se elimina la componente DC, se suaviza la señal y se normaliza para mejorar la detección de picos.
```
dc = mean(ppg_rt);
y_dc = y - dc;
y_s = mean(ppg_rt(end-4:end)) - dc;
y_plot = y_s / max(abs(seg_dc));
```
Se detectan picos y valles analizando los cambios en la pendiente de la señal.
```
if ppg_filt(i) > ppg_filt(i-1)
    estado = 1;
elseif ppg_filt(i) < ppg_filt(i-1)
    estado = -1;
```
Se calcula la frecuencia cardíaca a partir del intervalo entre picos consecutivos.
```
dt = diff(peak_times);
bpm = 60 / mean(dt);
```
Se estima el índice de perfusión combinando la amplitud del pulso y la frecuencia cardíaca, con normalización y suavizado.
```
SPI_raw = 0.7 * amp_norm + 0.3 * hr_norm;
SPI_inst = 100 * SPI_raw;
```
Se actualiza la gráfica mostrando la señal y los valores de BPM y SPI.
```
addpoints(h1, t_now, y_plot);
title(sprintf("BPM: %.1f | SPI: %.1f", bpm, SPI));
```
Se divide la señal en tres segmentos temporales para analizar el comportamiento en diferentes fases del experimento.
```
seg_dur = T / 3;
```
Se calcula para cada segmento:
- BPM promedio
- SPI promedio
- Visualización de picos


Se muestra la evolución completa del SPI con su promedio y divisiones entre fases.
```
plot(SPI_times, SPI_values);
yline(mean(SPI_values));
```
Se cierra la comunicación serial y se finaliza el programa.
```
clear s
disp("Finalizado.");
```
Se implementa un sistema de procesamiento en tiempo real de la señal PPG, donde la detección de picos se realiza mediante el método conocido como “escalador”, basado en el análisis de los cambios de pendiente de la señal para identificar máximos (picos) y mínimos (valles). A partir de estos puntos, se calcula la frecuencia cardíaca (BPM) usando el intervalo entre picos consecutivos, y la amplitud del pulso como diferencia entre pico y valle. Posteriormente, estas variables se normalizan y combinan para estimar un índice de perfusión simplificado (SPI), el cual es suavizado para mayor estabilidad. En conjunto, el código permite adquirir datos desde un dispositivo externo, filtrar la señal, detectar eventos fisiológicos relevantes y visualizar en tiempo real la respuesta cardiovascular, siendo especialmente útil para analizar cambios inducidos por estímulos como el Cold Pressor Test.

#### B.3 Resultados
En la siguiente tabla se muestran los tiempos de las fases del Cold Pressor Test, utilizado en la práctica de laboratorio para medir el índice pletismográfico quirúrgico.
##### Tabla de valores SPI por fase
<div align="center">

<table style="border-collapse: collapse; text-align: center;">
  <tr>
    <th style="padding: 8px;">Tiempo (s)</th>
    <th style="padding: 8px;">Fase</th>
    <th style="padding: 8px;">Condición</th>
  </tr>
  <tr>
    <td style="padding: 8px;">0 – 40</td>
    <td style="padding: 8px;">Basal</td>
    <td style="padding: 8px;">Reposo, dedo sobre sensor</td>
  </tr>
  <tr>
    <td style="padding: 8px;">40 – 80</td>
    <td style="padding: 8px;">CPT</td>
    <td style="padding: 8px;">Mano contralateral en agua fría (0 – 4 °C)</td>
  </tr>
  <tr>
    <td style="padding: 8px;">80 – 120</td>
    <td style="padding: 8px;">Recuperación</td>
    <td style="padding: 8px;">Retiro de la mano, vuelta al reposo</td>
  </tr>
</table>

</div>


A continuación se presentan los resultados obtenidos a partir del procesamiento de la señal PPG durante el experimento. La señal fue dividida en tres intervalos de tiempo de 40 segundos cada uno, correspondientes a las diferentes fases del protocolo.



<div align="center">

<img src="https://github.com/user-attachments/assets/359edcb2-6410-47ba-a9f4-dd8ae9e447aa" width="900"><br>
<sub><b>a. Señal PPG </b></sub>

<br><br><br>

<img src="https://github.com/user-attachments/assets/d2d880bc-9c80-4bf0-8410-1c87546abeac" width="900"><br>
<sub><b>b. Detección de picos y unión entre latidos</b></sub>

<br><br><br>

<img src="https://github.com/user-attachments/assets/51e626de-0708-487f-a8e6-d22d34986219" width="900"><br>
<sub><b>c. Resultado de la detección de picos en el SPI</b></sub>

</div>
En la figura (a) se observa la señal PPG sin filtros recibida desde la ESP32, la señal procesada para calculo del SPI y la deteccion de picos a través del método del "escalador". En la figura (b) se muestran los picos detectados junto con su conexión temporal, lo cual facilita la identificación de la periodicidad de los latidos y el cálculo de la frecuencia cardíaca. En la figura (c) se muestra los resultados obtenidos en la prueba de CPT.

## 5. Análisis de resultados

A partir de las señales adquiridas mediante la ESP32, se evaluó el comportamiento de la señal PPG y del índice pletismográfico quirúrgico (SPI) en tres intervalos de 40 segundos: **basal (0–40 s), estímulo (Cold Pressor Test, 40–80 s) y recuperación (80–120 s)**.


### Comportamiento de la señal PPG

La señal sin filtrar presenta variaciones lentas asociadas a la componente DC y oscilaciones pulsátiles correspondientes al flujo sanguíneo. Tras el procesamiento, se obtiene una señal con mejor relación señal–ruido, permitiendo la detección robusta de picos y valles (**157 picos en 3001 muestras**), lo que indica una adecuada calidad de adquisición.

Durante el intervalo correspondiente al **Cold Pressor Test (CPT)**, se observa una **disminución significativa en la amplitud de la señal procesada**, especialmente al inicio del estímulo. Este comportamiento es consistente con un proceso de **vasoconstricción periférica inducida por el frío**, que reduce la componente pulsátil (AC) de la señal PPG.

En la fase de recuperación, la señal muestra una **tendencia a la estabilización**, aunque sin retornar completamente a las condiciones iniciales dentro del intervalo analizado, lo cual sugiere una recuperación hemodinámica progresiva.



### Comportamiento del SPI

Los valores promedio del SPI obtenidos fueron:

- **SPI basal (0–40 s):** 49.46  
- **SPI durante CPT (40–80 s):** 58.14  
- **SPI recuperación (80–120 s):** 48.96  

Se observa un **incremento claro del SPI durante el estímulo frío**, seguido de una **disminución en la fase de recuperación**, retornando a valores cercanos a la línea basal (~50).

### Relación entre PPG y SPI

Durante el CPT se observa una aparente contradicción: mientras la **amplitud de la señal PPG disminuye**, el **SPI aumenta**. Este comportamiento es fisiológicamente coherente debido a que:

- El SPI no depende únicamente de la amplitud de la señal  
- El estímulo frío activa el sistema nervioso simpático  
- Esto genera:
  - **Vasoconstricción periférica** → disminución de la amplitud PPG  
  - **Aumento del tono simpático** → incremento del SPI  

Por tanto, el SPI refleja principalmente la **respuesta autonómica global**, mientras que la PPG representa de forma más directa la **perfusión periférica local**.

### Interpretación fisiológica

El aumento del SPI durante el **Cold Pressor Test** se explica por la activación del sistema nervioso simpático ante un estímulo nociceptivo. Esta respuesta produce:

- Liberación de catecolaminas  
- Incremento del tono vascular  
- Modulación de la dinámica del pulso  

En la fase de recuperación, la disminución del SPI indica una **reducción progresiva de la activación simpática**, con retorno hacia condiciones basales. Los resultados obtenidos muestran que:

- El sistema permite una **detección confiable de la señal PPG**
- El SPI es sensible a cambios inducidos por el **Cold Pressor Test**
- Existe coherencia fisiológica entre:
  - **Disminución de la amplitud PPG** (vasoconstricción)
  - **Aumento del SPI** (activación simpática)

Esto valida el uso del SPI como un indicador indirecto de activación autonómica en sujetos conscientes, destacando que su interpretación depende del contexto experimental y no es directamente equivalente al uso clínico en pacientes anestesiados.

## 6. Conclusión

En el presente trabajo se desarrolló e implementó un sistema de adquisición y procesamiento de señales fotopletismográficas (PPG) basado en una plataforma ESP32, orientado a la estimación del índice pletismográfico quirúrgico (SPI) como indicador de la respuesta autonómica en sujetos conscientes.

Los resultados obtenidos demuestran que la señal PPG adquirida posee una calidad suficiente para la extracción de características temporales y morfológicas, evidenciada por la detección robusta de eventos pulsátiles. Durante la aplicación del Cold Pressor Test (CPT), se observó una disminución en la amplitud de la señal PPG, consistente con un proceso de vasoconstricción periférica mediado por la activación del sistema nervioso simpático.

De manera complementaria, el SPI presentó un incremento significativo durante el estímulo, seguido de un retorno progresivo hacia valores basales en la fase de recuperación. Este comportamiento confirma la sensibilidad del índice frente a cambios en la modulación autonómica, destacando su capacidad para capturar la respuesta fisiológica al estrés más allá de las variaciones puramente hemodinámicas locales.

La aparente divergencia entre la disminución de la amplitud de la PPG y el incremento del SPI resalta la naturaleza multifactorial de este índice, el cual integra información relacionada con la dinámica del pulso y la regulación simpática. En este sentido, los resultados evidencian que el SPI constituye una métrica más representativa de la respuesta nociceptiva/autonómica global, mientras que la PPG refleja principalmente cambios en la perfusión periférica.

En conjunto, los hallazgos validan la funcionalidad del sistema desarrollado para la detección de cambios fisiológicos inducidos por estímulos controlados, como el CPT. No obstante, se resalta que la interpretación del SPI en sujetos conscientes difiere de su uso clínico en contextos intraoperatorios, donde la respuesta autonómica se encuentra modulada farmacológicamente. 
## 7. Preguntas para la discusión

### Pregunta 1 — ¿Cómo se relacionan las variaciones del volumen sanguíneo periférico con el balance autonómico?

El volumen sanguíneo periférico está regulado por el sistema nervioso autónomo (SNA) a través del control del tono vasomotor de las arteriolas. Cuando predomina la **activación simpática** (estrés, dolor), se libera norepinefrina sobre los receptores α-adrenérgicos vasculares, produciendo vasoconstricción periférica. Esto se refleja en la señal PPG como una reducción de la amplitud del pulso (↓ PPA) y, frecuentemente, una disminución del intervalo entre pulsos (↓ PPI) por el aumento de la frecuencia cardíaca. Dado que ambas variables normalizadas disminuyen, el término sustraído en la fórmula del SPI se hace menor y el índice **aumenta**, reflejando mayor estrés nociceptivo.

Cuando predomina la **activación parasimpática** (reposo, analgesia adecuada), ocurre vasodilatación periférica y bradicardia relativa: PPA sube, PPI sube, y el SPI **disminuye**. De este modo, la onda PPG actúa como un espejo indirecto del balance simpático-parasimpático, lo que justifica su uso para monitorizar nocicepción incluso en pacientes inconscientes.

### Pregunta 2 — ¿Cómo se compara el SPI con el ANI y el índice de perfusión?
<div align="center">

<table style="text-align:center;">
  <tr>
    <th>Característica</th>
    <th>SPI</th>
    <th>ANI</th>
    <th>Índice de perfusión (PI)</th>
  </tr>
  <tr>
    <td>Señal de entrada</td>
    <td>PPG</td>
    <td>ECG (HRV)</td>
    <td>PPG</td>
  </tr>
  <tr>
    <td>Variable principal</td>
    <td>PPI + PPA ponderadas</td>
    <td>Variabilidad FC en banda parasimpática (0,15–0,5 Hz)</td>
    <td>Relación componente AC / DC de la PPG</td>
  </tr>
  <tr>
    <td>Rango</td>
    <td>0 – 100</td>
    <td>0 – 100</td>
    <td>0,02 % – 20 %</td>
  </tr>
  <tr>
    <td>Alerta nociceptiva</td>
    <td>SPI &gt; 50</td>
    <td>ANI &lt; 50</td>
    <td>No específico para nocicepción</td>
  </tr>
  <tr>
    <td>Fabricante</td>
    <td>GE Healthcare</td>
    <td>MDoloris Medical Systems</td>
    <td>Masimo</td>
  </tr>
</table>

</div>

El **ANI** evalúa la actividad parasimpática mediante análisis espectral de la variabilidad de la frecuencia cardíaca y tiende a ser más sensible a cambios rápidos de nocicepción, pero requiere ECG de alta calidad. El **SPI**, al basarse en la PPG, es más sencillo de adquirir y robusto en algunas condiciones. El **índice de perfusión** no fue diseñado para monitorizar dolor sino para evaluar el estado circulatorio periférico y la calidad de la señal PPG; sin embargo, dado que la vasoconstricción afecta tanto al PI como a la PPA del SPI, existe correlación entre ellos durante respuestas nociceptivas simpáticas. Los tres índices son complementarios y ninguno es superior en todas las condiciones clínicas.

---

## 8. Bibliografía

[1] J. L. Apfelbaum, C. Chen, S. S. Mehta y T. J. Gan, "Postoperative pain experience: results from a national survey suggest postoperative pain continues to be undermanaged," *Anesthesia & Analgesia*, vol. 97, no. 2, pp. 534–540, 2003. https://doi.org/10.1213/01.ANE.0000068822.10113.9E

[2] T. Ledowski, "Objective monitoring of nociception: a review of current commercial solutions," *British Journal of Anaesthesia*, vol. 123, no. 2, pp. e312–e321, 2019. https://doi.org/10.1016/j.bja.2019.03.024

[3] V. Bonhomme *et al.*, "Comparison of the Surgical Pleth Index™ with haemodynamic variables to assess nociception-anti-nociception balance during general anaesthesia," *British Journal of Anaesthesia*, vol. 106, no. 1, pp. 101–111, 2011. https://doi.org/10.1093/bja/aeq291

[4] M. Huiku *et al.*, "Assessment of surgical stress during general anaesthesia," *British Journal of Anaesthesia*, vol. 98, no. 4, pp. 447–455, 2007. https://doi.org/10.1093/bja/aem004

[5] S. Funcke *et al.*, "Validation of innovative techniques for monitoring nociception during general anesthesia," *Anesthesiology*, vol. 127, no. 2, pp. 272–283, 2017. https://doi.org/10.1097/ALN.0000000000001670

---

*Universidad Militar Nueva Granada — Ingeniería Biomédica — 2025*
