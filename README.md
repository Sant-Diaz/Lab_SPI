# Laboratorio 3 — Cálculo ambulatorio del índice pletismográfico quirúrgico (SPI)

**Universidad Militar Nueva Granada**  
Programa: Ingeniería Biomédica · Semestre VII  
Asignatura: Instrumentación Biomédica y Biosensores

---

## Integrantes

| Nombre | GitHub |
|--------|--------|
| [Nombre 1] | [@usuario1] |
| [Nombre 2] | [@usuario2] |



## 1. Introducción

### 1.1 Contexto teórico

Una gran proporción de procedimientos quirúrgicos genera daño tisular, razón por la cual entre el 20 % y el 80 % de los pacientes reportan dolor posoperatorio de moderado a intenso [1]. Esto hace fundamental evaluar con precisión la **nocicepción intraoperatoria** en pacientes bajo anestesia general y ajustar la analgesia oportunamente para reducir el dolor posoperatorio.

Entre los métodos desarrollados para la monitorización cuantitativa y objetiva de la nocicepción se destaca el **Índice Pletismográfico Quirúrgico** (SPI, *Surgical Pleth Index*, GE Healthcare). Este índice se basa en la señal **fotopletismográfica (PPG)** para estimar el equilibrio entre la activación nociceptiva y la acción analgésica durante la anestesia general [3].

El SPI puede variar entre **0 y 100**, donde los valores más altos indican mayor respuesta nociceptiva:

| Rango SPI | Interpretación |
|-----------|---------------|
| 0 – 20 | Analgesia profunda / sin estrés nociceptivo |
| **20 – 50** | **Rango objetivo intraoperatorio óptimo** |
| > 50 | Respuesta nociceptiva elevada |

> Los valores de SPI deben mantenerse por debajo de 50, evitando incrementos superiores a 10 unidades durante la cirugía [5].

### 1.2 Características de la onda PPG relevantes para el SPI

La señal PPG refleja las variaciones del volumen sanguíneo periférico sincronizadas con el ciclo cardíaco. De cada pulsación se extraen dos características clave:

- **PPI** (*Pulse-to-Pulse Interval*, ms): tiempo entre dos máximos consecutivos, inversamente proporcional a la frecuencia cardíaca.
- **PPA** (*Pulse Pressure Amplitude*): diferencia entre el valor máximo y mínimo de cada ciclo, relacionada con el tono vasomotor periférico.

Ambas variables son moduladas por el sistema nervioso autónomo y, por tanto, reflejan el nivel de estrés fisiológico del paciente.

### 1.3 Importancia de la práctica

La relevancia de esta práctica reside en la extracción y cálculo de características derivadas de la onda de pulso para estimar el balance nocicepción-analgesia. Al replicar en condiciones ambulatorias el principio de funcionamiento del SPI clínico, se comprende cómo un índice utilizado en quirófano puede obtenerse con hardware de bajo costo.

---

## 2. Objetivos

**General:** Desarrollar un sistema de medición continua del índice pletismográfico quirúrgico (SPI) en condiciones ambulatorias.

**Específicos:**
- Reconocer las características fundamentales de la onda de pulso a partir de las cuales se obtiene el SPI.
- Construir un sistema que calcule el SPI en tiempo real bajo condiciones ambulatorias.
- Validar el funcionamiento del sistema mediante el *Cold Pressor Test* (CPT).

---

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

---

## 4. Procedimiento

### Parte A — Sistema de adquisición

#### A.1 Hardware utilizado

El sistema de adquisición se construyó con los siguientes componentes principales:

- **Módulo MAX30100 / MAX30102** — Sensor de oximetría y frecuencia cardíaca por PPG (comunicación I2C).
- **Arduino UNO / Nano** — Microcontrolador para leer el sensor y enviar datos por puerto serial.

El módulo MAX integra el LED infrarrojo, el fotodetector y el circuito de acondicionamiento de señal internamente, lo que simplifica el montaje respecto a una implementación discreta.

#### A.2 Conexión del módulo MAX al Arduino

| Pin módulo MAX | Pin Arduino |
|---------------|------------|
| VIN / VCC | 3.3V ó 5V (ver datasheet del módulo) |
| GND | GND |
| SDA | A4 (UNO) / SDA (Nano) |
| SCL | A5 (UNO) / SCL (Nano) |
| INT | D2 (opcional, para interrupciones) |

> El módulo MAX30102 opera a 1.8V internamente pero acepta niveles lógicos de 3.3V en I2C. Verificar el módulo específico utilizado respecto a compatibilidad con 5V.

#### A.3 Sketch de Arduino

```cpp
// =====================================================
// Laboratorio 3 — Adquisición PPG con módulo MAX
// Envía valores de la señal IR por puerto serial
// para ser leídos desde MATLAB
// =====================================================

#include <Wire.h>
#include "MAX30105.h"   // Librería SparkFun MAX3010x

MAX30105 sensor;

void setup() {
  Serial.begin(115200);
  Wire.begin();

  if (!sensor.begin(Wire, I2C_SPEED_FAST)) {
    Serial.println("ERROR: Sensor no detectado. Verificar conexiones.");
    while (true);
  }

  // Configuración del sensor
  sensor.setup();
  sensor.setPulseAmplitudeRed(0);    // Apagar LED rojo (no se usa para PPG)
  sensor.setPulseAmplitudeIR(30);    // Intensidad LED IR (ajustar 0-255)
  sensor.setSampleRate(200);         // 200 muestras/segundo
}

void loop() {
  long valorIR = sensor.getIR();
  Serial.println(valorIR);
  delay(5);   // ~200 Hz
}
```

> Librería requerida: **SparkFun MAX3010x** (instalar desde el gestor de librerías del IDE de Arduino buscando "MAX3010x").

#### A.4 Verificación con Serial Plotter

Con el sketch cargado, abrir **Herramientas → Serial Plotter** (115200 baud) y colocar el dedo sobre el sensor. Verificar que la señal PPG sea visible como una onda periódica. La amplitud y forma de la onda dependen de la presión del dedo sobre el sensor — ajustar hasta obtener una señal clara.

> 📷 **[IMAGEN] — Insertar aquí foto del montaje (módulo MAX + Arduino + dedo)**
> `parte_A/imagenes/montaje_sensor.jpg`

> 📷 **[IMAGEN] — Insertar aquí captura del Serial Plotter mostrando la señal PPG**
> `parte_A/imagenes/serial_plotter_ppg.png`

#### A.5 Cold Pressor Test (CPT)

El *Cold Pressor Test* es una técnica experimental que consiste en sumergir la mano en agua helada (0 – 4 °C) durante un tiempo determinado para generar una respuesta nociceptiva aguda, reproducible y controlada sin causar daño tisular permanente.

**Fundamento fisiológico:** El frío intenso activa los nociceptores tipo Aδ y C, generando activación simpática que se manifiesta como vasoconstricción periférica (↓ PPA) y aumento de la frecuencia cardíaca (↓ PPI), lo que eleva el SPI.

**Protocolo aplicado:**

| Tiempo (s) | Fase | Condición |
|-----------|------|-----------|
| 0 – 40 | Basal | Reposo, dedo sobre sensor |
| 40 – 80 | CPT | Mano contralateral en agua fría (0 – 4 °C) |
| 80 – 120 | Recuperación | Retiro de la mano, vuelta al reposo |

> El voluntario puede retirar la mano si el dolor se vuelve insoportable. La maniobra no debe exceder 3 minutos en ningún caso.

---

### Parte B — Código MATLAB y resultados

#### B.1 Código de adquisición y cálculo del SPI

Ver: [`parte_B/codigo/adquisicion_SPI.m`](parte_B/codigo/adquisicion_SPI.m)

```matlab
%% =========================================================
%  Laboratorio 3 — Cálculo ambulatorio del SPI
%  Instrumentación Biomédica y Biosensores — UMNG
%
%  adquisicion_SPI.m
%  Captura señal PPG desde Arduino (módulo MAX) por serial,
%  detecta máximos/mínimos, calcula PPI y PPA, y computa
%  el SPI con cada pulsación.
% =========================================================

clc; clear; close all;

%% --- PARÁMETROS (ajustar según equipo) ---
COM_PORT   = 'COM3';    % Windows: 'COM3', 'COM4'... | Linux/Mac: '/dev/ttyUSB0'
BAUD_RATE  = 115200;
FS         = 200;       % Frecuencia de muestreo (Hz) — debe coincidir con Arduino
BASAL_TIME = 40;        % Segundos de fase basal para normalización

MIN_PEAK_DISTANCE = round(FS * 0.4);  % Mín. 0.4 s entre picos (~150 bpm máx.)
MIN_PEAK_HEIGHT   = 1000;             % Umbral mínimo — ajustar según señal del MAX

%% --- SOLICITAR DURACIÓN ---
duracion  = input('Duración de la captura en segundos (ej: 120): ');
N_muestras = duracion * FS;

fprintf('\n>>> Capturando %d segundos en %s a %d baud...\n', duracion, COM_PORT, BAUD_RATE);

%% --- CONEXIÓN SERIAL ---
% MATLAB R2019b o posterior:
puerto = serialport(COM_PORT, BAUD_RATE);
configureTerminator(puerto, "LF");
flush(puerto);

% MATLAB R2018a o anterior (descomentar si aplica):
% puerto = serial(COM_PORT, 'BaudRate', BAUD_RATE);
% fopen(puerto);

%% --- ADQUISICIÓN ---
señal_ppg = zeros(1, N_muestras);
tiempo    = (0:N_muestras-1) / FS;

for i = 1:N_muestras
    linea = readline(puerto);                    % R2019b+
    % linea = fgetl(puerto);                     % R2018a-
    señal_ppg(i) = str2double(strtrim(linea));
    if mod(i, FS) == 0
        fprintf('  %d / %d s completados\n', i/FS, duracion);
    end
end
fprintf('\n>>> Captura finalizada.\n');
clear puerto

%% --- DETECCIÓN DE PICOS ---
fprintf('>>> Procesando señal...\n');
[picos_val, picos_idx] = findpeaks(señal_ppg, ...
    'MinPeakDistance', MIN_PEAK_DISTANCE, ...
    'MinPeakHeight',   MIN_PEAK_HEIGHT);

N_picos = length(picos_idx);
fprintf('    Picos detectados: %d\n', N_picos);

if N_picos < 5
    warning('Muy pocos picos detectados. Ajustar MIN_PEAK_HEIGHT.');
end

%% --- CÁLCULO DE PPI Y PPA ---
PPI = zeros(1, N_picos - 1);
PPA = zeros(1, N_picos - 1);

for k = 1:(N_picos - 1)
    PPI(k) = (picos_idx(k+1) - picos_idx(k)) / FS * 1000;  % en ms
    segmento = señal_ppg(picos_idx(k):picos_idx(k+1));
    PPA(k)   = picos_val(k) - min(segmento);
end

tiempo_SPI = tiempo(picos_idx(2:end));

%% --- NORMALIZACIÓN (respecto a fase basal) ---
idx_basal = tiempo_SPI <= BASAL_TIME;
if sum(idx_basal) < 3
    warning('Pocos pulsos en basal. Usando todos para normalizar.');
    idx_basal = true(size(tiempo_SPI));
end

PPI_ref  = mean(PPI(idx_basal));
PPA_ref  = mean(PPA(idx_basal));
PPI_norm = min(max(PPI / PPI_ref, 0), 1);
PPA_norm = min(max(PPA / PPA_ref, 0), 1);

fprintf('    PPI basal: %.1f ms | PPA basal: %.1f\n', PPI_ref, PPA_ref);

%% --- CÁLCULO DEL SPI ---
% Huiku et al., BJA 2007 | Bonhomme et al., BJA 2011
SPI = 100 - (0.33 .* PPA_norm + 0.67 .* PPI_norm) .* 100;
SPI = max(0, min(100, SPI));

%% --- RESULTADOS EN CONSOLA ---
fprintf('\n==============================================\n');
fprintf('  RESULTADOS SPI — Captura de %d s\n', duracion);
fprintf('==============================================\n');
fprintf('  %-18s %8s %8s %8s\n', 'Fase', 'Prom.', 'Min.', 'Max.');
fprintf('  %-18s %8s %8s %8s\n', '----', '-----', '----', '----');

fases    = {'Basal (0-40s)', 'CPT (40-80s)', 'Recup. (80-120s)'};
limites  = [0 40; 40 80; 80 120];

for f = 1:size(limites,1)
    idx = tiempo_SPI >= limites(f,1) & tiempo_SPI < limites(f,2);
    if sum(idx) > 0
        fprintf('  %-18s %8.1f %8.1f %8.1f\n', fases{f}, ...
            mean(SPI(idx)), min(SPI(idx)), max(SPI(idx)));
    end
end
fprintf('==============================================\n\n');

%% --- GUARDAR ---
save('parte_B/resultados/workspace_SPI.mat', ...
    'señal_ppg','tiempo','tiempo_SPI','PPI','PPA','SPI', ...
    'picos_idx','picos_val','FS','duracion');
fprintf('>>> Workspace guardado. Ejecute grafica_SPI.m\n');
```

#### B.2 Código de visualización del SPI

Ver: [`parte_B/codigo/grafica_SPI.m`](parte_B/codigo/grafica_SPI.m)

```matlab
%% =========================================================
%  grafica_SPI.m — Ejecutar después de adquisicion_SPI.m
%  Genera figuras de alta resolución para el informe.
% =========================================================

clc; clear; close all;

load('parte_B/resultados/workspace_SPI.mat');

C_BASAL = [0.20, 0.60, 0.86];
C_CPT   = [0.86, 0.27, 0.22];
C_REC   = [0.20, 0.75, 0.47];
C_SPI   = [0.30, 0.30, 0.70];
ALPHA   = 0.12;

%% --- FIGURA 1: Señal PPG cruda ---
fig1 = figure('Position', [100 100 1000 350]);
plot(tiempo, señal_ppg, 'Color', [0.4 0.4 0.4], 'LineWidth', 0.8);
hold on;
scatter(tiempo(picos_idx), picos_val, 25, C_CPT, 'filled');
ylims = ylim;
patch([0 40 40 0],     repmat(ylims,1,1), C_BASAL,'FaceAlpha',ALPHA,'EdgeColor','none');
patch([40 80 80 40],   repmat(ylims,1,1), C_CPT,  'FaceAlpha',ALPHA,'EdgeColor','none');
patch([80 120 120 80], repmat(ylims,1,1), C_REC,  'FaceAlpha',ALPHA,'EdgeColor','none');
xline(40,'--k','Inicio CPT','LabelVerticalAlignment','bottom','FontSize',9);
xline(80,'--k','Fin CPT',   'LabelVerticalAlignment','bottom','FontSize',9);
xlabel('Tiempo (s)'); ylabel('Amplitud (sensor MAX)');
title('Señal PPG cruda — Módulo MAX');
legend('PPG','Picos','Basal','CPT','Recuperación','Location','best');
grid on; xlim([0 duracion]);
exportgraphics(fig1,'parte_B/resultados/señal_ppg_cruda.png','Resolution',300);

%% --- FIGURA 2: SPI vs. tiempo ---
fig2 = figure('Position', [100 500 1000 450]);
hold on;
patch([0 40 40 0],     [0 0 100 100], C_BASAL,'FaceAlpha',ALPHA,'EdgeColor','none','HandleVisibility','off');
patch([40 80 80 40],   [0 0 100 100], C_CPT,  'FaceAlpha',ALPHA,'EdgeColor','none','HandleVisibility','off');
patch([80 120 120 80], [0 0 100 100], C_REC,  'FaceAlpha',ALPHA,'EdgeColor','none','HandleVisibility','off');
patch([0 duracion duracion 0],[20 20 50 50],[0.9 0.9 0.3],'FaceAlpha',0.08,'EdgeColor','none','DisplayName','Rango óptimo (20–50)');
yline(50,'--r','SPI = 50','FontSize',9,'HandleVisibility','off');
yline(20,'--g','SPI = 20','FontSize',9,'HandleVisibility','off');
plot(tiempo_SPI, SPI, '-o', 'Color', C_SPI, 'LineWidth', 1.8, ...
    'MarkerSize', 3, 'MarkerFaceColor', C_SPI, 'DisplayName', 'SPI calculado');

fases_lim = [0 40; 40 80; 80 120];
fases_nom = {'Basal','CPT','Recup.'};
fases_col = {C_BASAL, C_CPT, C_REC};
for f = 1:3
    idx = tiempo_SPI >= fases_lim(f,1) & tiempo_SPI < fases_lim(f,2);
    if sum(idx) > 0
        mu = mean(SPI(idx));
        plot(fases_lim(f,:),[mu mu],'--','Color',fases_col{f},'LineWidth',2,'HandleVisibility','off');
        text(mean(fases_lim(f,:)), mu+3, sprintf('%s \\mu=%.1f', fases_nom{f}, mu), ...
            'HorizontalAlignment','center','FontSize',9,'Color',fases_col{f},'FontWeight','bold');
    end
end
xline(40,'--k','HandleVisibility','off'); xline(80,'--k','HandleVisibility','off');
text(20,97,'Basal',        'HorizontalAlignment','center','FontSize',11,'Color',C_BASAL,'FontWeight','bold');
text(60,97,'Cold Pressor', 'HorizontalAlignment','center','FontSize',11,'Color',C_CPT,  'FontWeight','bold');
text(100,97,'Recuperación','HorizontalAlignment','center','FontSize',11,'Color',C_REC,  'FontWeight','bold');
xlabel('Tiempo (s)'); ylabel('SPI (0–100)');
title('Índice Pletismográfico Quirúrgico (SPI) vs. tiempo');
legend('Location','southeast'); ylim([0 100]); xlim([0 duracion]);
grid on; box on;
exportgraphics(fig2,'parte_B/resultados/grafica_SPI_vs_tiempo.png','Resolution',300);

%% --- FIGURA 3: PPI y PPA ---
fig3 = figure('Position',[100 100 1000 500]);
subplot(2,1,1);
plot(tiempo_SPI, PPI, '-o','Color',[0.1 0.5 0.8],'LineWidth',1.5,'MarkerSize',3);
hold on; xline(40,'--k'); xline(80,'--k');
ylabel('PPI (ms)'); title('Intervalo entre pulsos (PPI)'); grid on; xlim([0 duracion]);

subplot(2,1,2);
plot(tiempo_SPI, PPA, '-o','Color',[0.8 0.3 0.1],'LineWidth',1.5,'MarkerSize',3);
hold on; xline(40,'--k','Inicio CPT'); xline(80,'--k','Fin CPT');
xlabel('Tiempo (s)'); ylabel('PPA (unid. sensor)');
title('Amplitud de pulso (PPA)'); grid on; xlim([0 duracion]);

sgtitle('Características extraídas de la onda PPG','FontWeight','bold');
exportgraphics(fig3,'parte_B/resultados/PPI_PPA_vs_tiempo.png','Resolution',300);

fprintf('>>> Figuras guardadas en parte_B/resultados/\n');
```

#### B.3 Resultados

##### Tabla de valores SPI por fase

| Fase | Tiempo (s) | SPI promedio | SPI mínimo | SPI máximo |
|------|-----------|-------------|-----------|-----------|
| Basal | 0 – 40 | [COMPLETAR] | [COMPLETAR] | [COMPLETAR] |
| CPT | 40 – 80 | [COMPLETAR] | [COMPLETAR] | [COMPLETAR] |
| Recuperación | 80 – 120 | [COMPLETAR] | [COMPLETAR] | [COMPLETAR] |

##### Gráfica SPI vs. tiempo

> 📷 **[IMAGEN] — `parte_B/resultados/grafica_SPI_vs_tiempo.png`**

##### Señal PPG cruda

> 📷 **[IMAGEN] — `parte_B/resultados/señal_ppg_cruda.png`**

##### PPI y PPA en el tiempo

> 📷 **[IMAGEN] — `parte_B/resultados/PPI_PPA_vs_tiempo.png`**

---

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

> [COMPLETAR — Orientación: reflexionar sobre la diferencia entre dolor y nocicepción, lo que implica para la práctica clínica, y sobre el desempeño del sistema construido]

**Puntos clave a incluir:**
- La **nocicepción** es el proceso neurofisiológico de detección de estímulos dañinos; el **dolor** es la experiencia subjetiva consciente que resulta de él. Un paciente bajo anestesia experimenta nocicepción sin dolor consciente — el SPI mide la primera.
- El aumento del SPI durante el CPT confirma que el sistema es sensible a la activación simpática producida por un estímulo nociceptivo.
- [Reflexión propia sobre el funcionamiento del sistema, dificultades encontradas y posibles mejoras.]

---

## 7. Preguntas para la discusión

### Pregunta 1 — ¿Cómo se relacionan las variaciones del volumen sanguíneo periférico con el balance autonómico?

El volumen sanguíneo periférico está regulado por el sistema nervioso autónomo (SNA) a través del control del tono vasomotor de las arteriolas. Cuando predomina la **activación simpática** (estrés, dolor), se libera norepinefrina sobre los receptores α-adrenérgicos vasculares, produciendo vasoconstricción periférica. Esto se refleja en la señal PPG como una reducción de la amplitud del pulso (↓ PPA) y, frecuentemente, una disminución del intervalo entre pulsos (↓ PPI) por el aumento de la frecuencia cardíaca. Dado que ambas variables normalizadas disminuyen, el término sustraído en la fórmula del SPI se hace menor y el índice **aumenta**, reflejando mayor estrés nociceptivo.

Cuando predomina la **activación parasimpática** (reposo, analgesia adecuada), ocurre vasodilatación periférica y bradicardia relativa: PPA sube, PPI sube, y el SPI **disminuye**. De este modo, la onda PPG actúa como un espejo indirecto del balance simpático-parasimpático, lo que justifica su uso para monitorizar nocicepción incluso en pacientes inconscientes.

### Pregunta 2 — ¿Cómo se compara el SPI con el ANI y el índice de perfusión?

| Característica | SPI | ANI | Índice de perfusión (PI) |
|----------------|-----|-----|--------------------------|
| Señal de entrada | PPG | ECG (HRV) | PPG |
| Variable principal | PPI + PPA ponderadas | Variabilidad FC en banda parasimpática (0,15–0,5 Hz) | Relación componente AC / DC de la PPG |
| Rango | 0 – 100 | 0 – 100 | 0,02 % – 20 % |
| Alerta nociceptiva | SPI > 50 | ANI < 50 | No específico para nocicepción |
| Fabricante | GE Healthcare | MDoloris Medical Systems | Masimo |

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
