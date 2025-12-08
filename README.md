# Generador de Etiquetas para Herbario

Esta es una aplicación web para generar etiquetas para especímenes de herbario.
Permite importar datos desde un archivo CSV o pegarlos directamente desde una
hoja de cálculo, y luego genera etiquetas con un formato personalizable para
imprimir.

## Características

-   Importación de datos desde CSV o pegando desde una hoja de cálculo.
-   Configuración de las etiquetas:
    -   Dimensiones (ancho, alto, relleno).
    -   Estilo del borde (grosor, estilo).
    -   Tipografía y tamaño de fuente.
-   Previsualización en tiempo real de los cambios de configuración.
-   Función "¡Sorpréndeme!" para generar un diseño de etiqueta aleatorio.
-   Generación de una lista de "Orden de entrega" para imprimir junto con las etiquetas.
-   Interfaz de arrastrar y soltar para archivos CSV.

## Cómo Colaborar

1.  Clona el repositorio:
    ```bash
    git clone https://github.com/peregrinogris/etiquetas-herbario.git
    ```
2.  Instala las dependencias:
    ```bash
    npm install
    ```
3.  Inicia el servidor de desarrollo:
    ```bash
    npm run dev
    ```
4.  Abre tu navegador en la URL que se indica en la terminal (normalmente `http://localhost:5173`).

## Licencia

Este proyecto está bajo la Licencia ISC. Ver el archivo `LICENSE.MD` para más detalles.
