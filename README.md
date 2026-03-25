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

#### A.3 Sketch de Arduino

#### A.4 Verificación con Serial Plotter

#### A.5 Cold Pressor Test (CPT)

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
s = serialport("COM12",115200);
configureTerminator(s,"LF");
flush(s);
pause(2);
```
Se crean las líneas animadas para visualizar la señal y los picos detectados.
```
figure;
h1 = animatedline('LineWidth',1.5);
hPeak = animatedline('Color','g','Marker','o','LineStyle','none');

xlabel("Tiempo [s]");
ylabel("PPG");
title("Inicializando...");
grid on;
```
Se definen buffers y variables necesarias para el procesamiento, detección de picos y cálculo de métricas.
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
```
Se descartan muestras iniciales para evitar ruido o datos inestables.
```
disp("Estabilizando...");
tic;
while toc < 1.5
    readline(s);
end
```
Se leen los datos desde el puerto serial y se convierten a valores numéricos.
```
while toc <= T

    linea = readline(s);
    nums = regexp(linea,'[-+]?\d*\.?\d+','match');

    if isempty(nums)
        continue;
    end

    y = str2double(nums{1});
    if isnan(y)
        continue;
    end
```
Se elimina la componente DC, se suaviza la señal y se normaliza para facilitar el análisis.
```
dc = mean(ppg_rt(end-win_dc:end));
y_dc = y - dc;
y_s = mean(ppg_rt(end-4:end)) - dc;
y_plot = y_s / max(abs(seg_dc));
```
Se identifican picos y valles analizando cambios en la pendiente de la señal.
```
if ppg_filt(i) > ppg_filt(i-1)
    estado = 1;
elseif ppg_filt(i) < ppg_filt(i-1)
    estado = -1;
```
Se calcula la frecuencia cardíaca a partir del tiempo entre picos consecutivos.
```
dt = diff(peak_times);
bpm = 60 / mean(dt);
```
Se obtiene un índice basado en la amplitud del pulso y la frecuencia cardíaca, con normalización y suavizado.
```
SPI_raw = 0.7 * amp_norm + 0.3 * hr_norm;
SPI_inst = 100 * SPI_raw;
```
Se actualiza la gráfica y se muestran los valores calculados.
```
addpoints(h1, t_now, y_plot);
title(sprintf("BPM: %.1f | SPI: %.1f", bpm, SPI));
drawnow limitrate;
```
Se cierra la comunicación serial y se finaliza el programa.
```
clear s
disp("Finalizado");
```
Se implementa un sistema de procesamiento en tiempo real de la señal PPG, donde la detección de picos se realiza mediante el método conocido como “escalador”, basado en el análisis de los cambios de pendiente de la señal para identificar máximos (picos) y mínimos (valles). A partir de estos puntos, se calcula la frecuencia cardíaca (BPM) usando el intervalo entre picos consecutivos, y la amplitud del pulso como diferencia entre pico y valle. Posteriormente, estas variables se normalizan y combinan para estimar un índice de perfusión simplificado (SPI), el cual es suavizado para mayor estabilidad. En conjunto, el código permite adquirir datos desde un dispositivo externo, filtrar la señal, detectar eventos fisiológicos relevantes y visualizar en tiempo real la respuesta cardiovascular, siendo especialmente útil para analizar cambios inducidos por estímulos como el Cold Pressor Test.

#### B.3 Resultados

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




##### Señal PPG sin procesar



##### PPI y PPA en el tiempo


## 5. Análisis de resultados

### Análisis 1 — Comparación con valores clínicos intraoperatorios

Durante una cirugía bajo anestesia general con analgesia adecuada, el SPI típicamente se mantiene en el rango **20 – 50**. Valores superiores a 50 indican respuesta nociceptiva insuficientemente controlada y suelen requerir ajuste de la dosis analgésica.

| Condición | SPI práctica | SPI clínico de referencia |
|-----------|-------------|---------------------------|
| Sin estímulo / reposo | [COMPLETAR] | 20 – 50 (analgesia adecuada) |
| Estímulo nociceptivo (CPT) | [COMPLETAR] | > 50 (requiere analgesia adicional) |
| Recuperación | [COMPLETAR] | < 50 (post-analgesia) |

> [COMPLETAR con análisis propio: ¿los valores obtenidos son consistentes con el rango clínico esperado? ¿Por qué sí o por qué no?]

### Análisis 2 — Alcance y limitaciones del sistema

**Fortalezas:**
- El módulo MAX integra el circuito de acondicionamiento, reduciendo la complejidad del hardware y los artefactos por ruido externo respecto a una implementación discreta.
- La señal PPG obtenida permite calcular PPI y PPA con suficiente resolución para estimar el SPI.
- El sistema captura la respuesta fisiológica al CPT de forma cuantitativa.

**Limitaciones:**

| Limitación | Descripción |
|-----------|-------------|
| Artefactos por movimiento | El sensor es muy susceptible a movimiento del dedo durante la captura. |
| Normalización subjetiva | Los valores basales de PPI y PPA dependen del estado inicial del voluntario, introduciendo variabilidad entre sujetos. |
| Fórmula aproximada | La expresión propietaria de GE Healthcare no está publicada; se usa la aproximación académica de Huiku *et al.* |
| Contexto no clínico | La validación del SPI fue desarrollada para pacientes bajo anestesia general; extrapolar a individuos conscientes tiene limitaciones interpretativas. |

---

## 6. Conclusiones

**Puntos clave a incluir:**
- La **nocicepción** es el proceso neurofisiológico de detección de estímulos dañinos; el **dolor** es la experiencia subjetiva consciente que resulta de él. Un paciente bajo anestesia experimenta nocicepción sin dolor consciente — el SPI mide la primera.
- El aumento del SPI durante el CPT confirma que el sistema es sensible a la activación simpática producida por un estímulo nociceptivo.
- [Reflexión propia sobre el funcionamiento del sistema, dificultades encontradas y posibles mejoras.]


## 7. Preguntas para la discusión

### Pregunta 1 — ¿Cómo se relacionan las variaciones del volumen sanguíneo periférico con el balance autonómico?

El volumen sanguíneo periférico está regulado por el sistema nervioso autónomo (SNA) a través del control del tono vasomotor de las arteriolas. Cuando predomina la **activación simpática** (estrés, dolor), se libera norepinefrina sobre los receptores α-adrenérgicos vasculares, produciendo vasoconstricción periférica. Esto se refleja en la señal PPG como una reducción de la amplitud del pulso (↓ PPA) y, frecuentemente, una disminución del intervalo entre pulsos (↓ PPI) por el aumento de la frecuencia cardíaca. Dado que ambas variables normalizadas disminuyen, el término sustraído en la fórmula del SPI se hace menor y el índice **aumenta**, reflejando mayor estrés nociceptivo.

Cuando predomina la **activación parasimpática** (reposo, analgesia adecuada), ocurre vasodilatación periférica y bradicardia relativa: PPA sube, PPI sube, y el SPI **disminuye**. De este modo, la onda PPG actúa como un espejo indirecto del balance simpático-parasimpático, lo que justifica su uso para monitorizar nocicepción incluso en pacientes inconscientes.

### Pregunta 2 — ¿Cómo se compara el SPI con el ANI y el índice de perfusión?
<div align="center">
| Característica | SPI | ANI | Índice de perfusión (PI) |
|----------------|-----|-----|--------------------------|
| Señal de entrada | PPG | ECG (HRV) | PPG |
| Variable principal | PPI + PPA ponderadas | Variabilidad FC en banda parasimpática (0,15–0,5 Hz) | Relación componente AC / DC de la PPG |
| Rango | 0 – 100 | 0 – 100 | 0,02 % – 20 % |
| Alerta nociceptiva | SPI > 50 | ANI < 50 | No específico para nocicepción |
| Fabricante | GE Healthcare | MDoloris Medical Systems | Masimo |
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
