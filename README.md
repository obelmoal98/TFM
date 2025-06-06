# Automatización Inteligente de Formularios Web: Un Enfoque Basado en VLMs y LLMs (Código Fuente del TFM)

Este repositorio contiene el código fuente desarrollado para el Trabajo Final de Máster (TFM) titulado "Automatización Inteligente de Formularios Web: Un Enfoque Basado en VLMs y LLMs", presentado por Oriol Belmonte Alcalà en la Universitat Oberta de Catalunya (UOC) para el Máster Universitario en Ciencia de Datos.

**Autor:** Oriol Belmonte Alcalà
**Tutor:** Diego Calvo Barreno
**Profesor:** Josep Anton Mir Tutusaus
**Fecha de Entrega del TFM:** Junio 2025

## Resumen del Proyecto

La cumplimentación manual de formularios web es una tarea que consume mucho tiempo y es propensa a errores. Los métodos de automatización tradicionales a menudo fallan debido a la naturaleza dinámica de las interfaces web. Este proyecto aborda la necesidad crítica de un agente robusto para la automatización de formularios web que opere únicamente con información visual, independientemente de las estructuras HTML o DOM.

Se desarrolló un agente autónomo que interpreta y completa inteligentemente formularios web utilizando solo capturas de pantalla. El enfoque utiliza una arquitectura modular que integra:
* **OmniParser** para la identificación de elementos visuales a partir de capturas de pantalla.
* El modelo multimodal **GPT-4o** para el razonamiento y la planificación de acciones.
* **LangChain/LangGraph** con **Playwright** para la orquestación del flujo de trabajo y la interacción con el navegador.

El rendimiento del agente se probó en tres formularios web de diversa complejidad, logrando una tasa de envío del 100% en todos ellos. Para una aplicación sencilla, el 100% de los campos se rellenaron correctamente. Los errores observados generalmente involucraron interacciones con selectores de fecha o menús desplegables.

## Arquitectura y Funcionamiento

El sistema se divide en dos componentes principales, implementados en distintos cuadernos de Jupyter:

1.  **Servicio API con OmniParser (`Omniparser_API.ipynb`):**
    * Recibe una imagen (captura de pantalla de la web) mediante una petición HTTP POST.
    * Utiliza OmniParser (que internamente usa los modelos YOLOv8 para detección de iconos y Florence2 para descripciones y OCR para identificar elementos interactivos y texto).
    * Devuelve una imagen base64 con los elementos detectados y etiquetados numéricamente, junto con una lista JSON de bounding boxes (coordenadas, contenido, tipo de interactividad).
    * Se utiliza `ngrok` para exponer este servidor FastAPI local a internet, haciéndolo accesible para el cuaderno del agente.

2.  **Agente GPT-4o con LangGraph (`OmniParser_+_GPT_4o.ipynb`):**
    * El agente se inicializa con una instancia del navegador Playwright y una URL de destino.
    * Se proporciona una tarea de alto nivel al agente (ej. "Rellena este formulario con estos datos").
    * **Percepción:**
        * Playwright toma una captura de la página actual.
        * La captura se envía al servicio API de OmniParser.
        * La API devuelve la imagen anotada y los datos de los bounding boxes.
    * **Razonamiento:**
        * La imagen anotada, las descripciones de los bounding boxes, la tarea original y el historial de acciones/observaciones previas se formatean en un prompt multimodal para GPT-4o. (El prompt está inspirado en ReAct y la propuesta de WebVoyager).
        * GPT-4o procesa esta entrada y decide la siguiente acción (ej. `Click [etiqueta]`, `Type [etiqueta]; [texto]`, `Scroll WINDOW; down`, `ANSWER; [resultado]`).
    * **Interacción:**
        * La acción predicha se analiza y ejecuta mediante la herramienta correspondiente (ej. Playwright hace clic en un elemento).
    * **Observación y Bucle (Memoria y Orquestación):**
        * El resultado de la acción se convierte en una observación, que se añade al `scratchpad` (memoria).
        * El proceso (anotación, decisión, acción) se repite, gestionado por LangGraph, hasta que GPT-4o determina que la tarea está completa y emite una `ANSWER`.

## Tecnologías Utilizadas

* **Entorno de Ejecución:** Google Colab con Python (v3.11.12 por defecto) y GPU NVIDIA Tesla T4 (16 GB VRAM).
* **Modelo Visual/Percepción:** OmniParser (v2).
* **Modelo de Razonamiento:** GPT-4o accedido vía Azure OpenAI Service.
* **Framework del Agente:** LangChain y LangGraph para la definición de prompts, cadenas, memoria y orquestación del grafo de ejecución.
* **Interacción Web:** Playwright para controlar el navegador y automatizar acciones.
* **Servicio API:** FastAPI, Uvicorn.
* **Exposición del Servicio:** PyNgrok.

## Prerrequisitos

* Python 3.9+
* Git
* Acceso a Azure OpenAI Service con un despliegue del modelo GPT-4o.
* Cuenta de `ngrok` y authtoken para exponer el servicio API de OmniParser.
* (Recomendado) GPU compatible con CUDA para el servicio OmniParser si se ejecuta localmente fuera de Colab. En Colab, se utiliza la GPU proporcionada.

## Instalación y Configuración

Configurar los dos componentes (API y Agente) de forma secuencial, preferiblemente en el entorno de Google Colab como se describe.
