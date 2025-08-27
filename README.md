# Disclaimer
Este codelab es una migración html a markdown de la guía de https://www.develou.com/room/

Todos los créditos por el contenido a sus creadores.

# Uso
## Instalar Go
En una ventana de powershell:

	winget install Golang.Go

O instala el ejecutable: https://go.dev/dl/

## Instalar Claat
1. Cierra y abre nueva ventana powershell para refrescar variables de entorno. Ahora ejecuta:

    ```
    go install github.com/googlecodelabs/tools/claat@latest
    ```

2. Genera los archivos html:

    ```
    claat export codelab.md
    ```

3. Para ver el contenido del codelab, inicia el servidor de claat desde la carpeta del proyecto:

    ```
    claat serve
    start http://localhost:9090/guia-room/
    ```