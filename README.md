# Proyecto: Modelo cliente-servidor
### Integrantes
- Camargo Loaiza Miguel Angel
- Fuentes Padilla Aarón Esteban 
- Pacheco Olivas Juan Pablo Valentin
---
### Declaración de Puntaje
Este proyecto se realizó con el objetivo de alcanzar un puntaje de los 100 puntos.

###Declaración de uso de IA
En la elaboración de este proyecto hemos usado la ayuda de inteligencias artificlaes como Microsoft Copilot y ChatGPT.

# Descripción del funcionamiento del programa

1. *El servidor se pone en funcionamiento*  
   El servidor inicia su ejecución y queda en espera activa, preparado para aceptar conexiones entrantes.
   ![Ejemplo de imagen](Servidor iniciado.png)

2. *El cliente establece la conexión con el servidor*  
   El cliente se conecta al servidor a través de un canal de comunicación previamente definido, como un socket o una dirección IP específica.
   ![Ejemplo de imagen](Cliente conectado.png)
3. *El cliente cifra un folder con archivos*  
   Antes de enviar el archivo, el cliente aplica un algoritmo de cifrado (en formato: cifrar_folder:<inputFolder>;<outputFolder>) para proteger su contenido, asegurando la confidencialidad de los datos.
Ejemplo: 
cifrar_folder: C:\Users\YUGEN\Documents\NetBeansProjects\Cliente_chat_archivo\hola;C:\Users\YUGEN\Documents\NetBeansProjects\Cliente_chat_archivo\hola_des
 ![Ejemplo de imagen](Cliente conectado.png)

4. *El cliente transfiere el archivo cifrado al servidor*  
   El cliente envía el archivo ya cifrado a través de la conexión establecida, garantizando que el servidor reciba la información en un formato seguro.
   
5. *El cliente envía una carpeta con archivos cifrados al servidor*  
   El cliente selecciona una carpeta que contiene varios archivos, los cifra utilizando un algoritmo de seguridad (como AES o RSA) y luego transfiere la carpeta completa al servidor mediante la conexión establecida. El proceso asegura que los datos sensibles estén protegidos durante toda la transmisión.

6. *El servidor transfiere archivos al cliente*  
   El servidor selecciona los archivos que deben enviarse al cliente y establece un protocolo de transferencia seguro para entregarlos. Los archivos pueden ser cifrados antes de la transmisión si es necesario, garantizando la integridad y confidencialidad de los datos en tránsito.
    ![Ejemplo de imagen](servidor envia archivo a cliente.png)
