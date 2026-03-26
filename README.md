# Laboratorio 3 — Cálculo ambulatorio del índice pletismográfico quirúrgico (SPI)

**Universidad Militar Nueva Granada**  
Programa: Ingeniería Biomédica · Semestre VII  
Asignatura: Instrumentación Biomédica y Biosensores


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

<img src="https://github.com/user-attachments/assets/24266495-abf9-419c-972a-fa5fbe58e5fd" width="1100"><br>
<sub><b>a. Señal PPG segmentada en tres intervalos (40 s cada uno)</b></sub>

<br><br>

<img src="https://github.com/user-attachments/assets/a78bfb63-b7d4-4395-9d28-e5d97f47ac63" width="1100"><br>
<sub><b>b. Detección de picos y unión entre latidos</b></sub>

</div>
En la figura (a) se observa la señal PPG filtrada dividida en tres segmentos temporales, lo que permite visualizar el comportamiento de la señal en cada fase del experimento. En la figura (b) se muestran los picos detectados junto con su conexión temporal, lo cual facilita la identificación de la periodicidad de los latidos y el cálculo de la frecuencia cardíaca.


## 5. Análisis de resultados

A partir de las señales obtenidas, se evaluó el comportamiento de la señal PPG y del índice de perfusión (SPI) en tres intervalos de 40 segundos, correspondientes a las fases basal, estímulo (Cold Pressor Test) y recuperación.

### Comportamiento de la señal PPG

La señal PPG filtrada presenta una morfología periódica estable en los tres segmentos, con detección consistente de picos, lo que indica una adecuada calidad de adquisición. La frecuencia cardíaca promedio se mantiene relativamente constante entre segmentos (≈ 91–96 BPM), lo cual es característico de un sujeto en estado de reposo.

Durante el segundo intervalo (CPT), se observa una ligera disminución en la amplitud de la señal, lo que sugiere la presencia de vasoconstricción periférica inducida por el estímulo frío. Posteriormente, en la fase de recuperación, la amplitud tiende a aumentar nuevamente, evidenciando un proceso de reperfusión.

### Comportamiento del SPI

Los valores promedio del SPI obtenidos fueron:

- **Segmento 1 (Basal):** 86.5  
- **Segmento 2 (CPT):** 80.2  
- **Segmento 3 (Recuperación):** 89.2  

Se observa una **disminución del SPI durante el estímulo frío**, seguida de un **aumento en la fase de recuperación**. Este comportamiento es consistente con la fisiología esperada en sujetos conscientes, donde el frío induce vasoconstricción periférica, reduciendo la amplitud de la señal PPG y, por ende, el valor del índice.


### Comparación con valores clínicos intraoperatorios

En el contexto clínico, el SPI se utiliza para evaluar el balance nocicepción–analgesia en pacientes bajo anestesia general. En estos casos, el comportamiento esperado es diferente al observado en este experimento.

<div align="center">

<table>
  <tr>
    <th>Fase</th>
    <th>Paciente anestesiado (clínico)</th>
    <th>Sujeto consciente (experimental)</th>
  </tr>
  <tr>
    <td>Basal</td>
    <td>SPI bajo (20–50)</td>
    <td>SPI alto (≈80–90)</td>
  </tr>
  <tr>
    <td>Estímulo nociceptivo</td>
    <td>Aumento del SPI</td>
    <td>Disminución del SPI</td>
  </tr>
  <tr>
    <td>Recuperación / analgesia</td>
    <td>Disminución del SPI</td>
    <td>Aumento del SPI</td>
  </tr>
</table>

</div>

Esta diferencia se debe a que en el entorno clínico:
- El paciente está bajo efecto de anestésicos y opioides  
- El SPI está calibrado para detectar cambios en la nocicepción  
- La respuesta simpática está modulada farmacológicamente  

Mientras que en este experimento:
- El sujeto está consciente y en reposo  
- No hay intervención farmacológica  
- El SPI refleja principalmente cambios hemodinámicos periféricos  


### Interpretación fisiológica

La disminución del SPI durante el Cold Pressor Test se explica por la activación del sistema nervioso simpático, que genera vasoconstricción periférica. Esto reduce la componente pulsátil (AC) de la señal PPG, afectando directamente el cálculo del índice. Aunque la frecuencia cardíaca puede aumentar ligeramente, su contribución al SPI es menor en comparación con la amplitud del pulso.

Los resultados obtenidos son coherentes con la fisiología de un sujeto consciente sometido a un estímulo frío. La disminución del SPI durante el CPT y su posterior recuperación confirman que el sistema implementado es capaz de detectar cambios en la perfusión periférica. Sin embargo, se evidencia que la interpretación del SPI depende fuertemente del contexto, ya que su significado clínico en anestesia no es directamente extrapolable a condiciones experimentales sin intervención farmacológica.

## 6. Conclusione

El presente trabajo permitió implementar y validar un sistema de adquisición y procesamiento de la señal fotopletismográfica (PPG) orientado al análisis del índice de perfusión (SPI) como indicador indirecto de la actividad autonómica. A partir de los resultados obtenidos, se evidenció la capacidad del sistema para detectar variaciones fisiológicas asociadas a la activación simpática inducida por el Cold Pressor Test, particularmente a través de cambios en la amplitud de la señal y en la dinámica temporal de los latidos.

Es importante resaltar que la nocicepción, entendida como el proceso neurofisiológico de detección de estímulos potencialmente dañinos, difiere del dolor como experiencia subjetiva consciente. En este sentido, aunque el SPI es utilizado clínicamente en pacientes bajo anestesia para evaluar el balance nocicepción–analgesia, en el contexto experimental desarrollado —con un sujeto consciente— el índice refleja principalmente cambios hemodinámicos periféricos más que una medida directa de dolor.

Los resultados mostraron una disminución del SPI durante la fase de estímulo frío, seguida de una recuperación posterior, comportamiento que es consistente con la vasoconstricción periférica inducida por la activación del sistema nervioso simpático. Este hallazgo confirma que, aun cuando el estímulo se aplicó en la extremidad contralateral a la medición, la respuesta observada es de carácter sistémico y fisiológicamente coherente.

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
