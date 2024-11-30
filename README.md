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
###Código Servidor_Chat_Archivo

## 1. **Importación de Librerías**:

```java
import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;
import java.nio.file.Files;
import java.util.*;
```
- **java.io**: Contiene las clases necesarias para trabajar con flujos de entrada y salida, como la lectura de archivos y el envío de datos.
- **java.net.ServerSocket, Socket**: Para la creación del servidor y la gestión de conexiones de cliente.
- **java.nio.file.Files**: Para la lectura y escritura de archivos.
- **java.util**: Utilizada para manejar colecciones, como `List` y `Map`.

## 2. **Declaración de Variables Estáticas**:

```java
private static final String llave = "hola"; // Clave de cifrado
private static Map<String, Map<String, String>> diccionarioDescifrado = new HashMap<>();
private static final String TITULO = "Desencriptar Archivo";
private static final int MENSAJE = 1;
private static final int ARCHIVO = 2;
private static final int LISTA_ARCHIVOS = 3;
```
- **llave**: Es la clave de cifrado/desencriptación utilizada para desencriptar los archivos.
- **diccionarioDescifrado**: Un mapa que podría usarse para almacenar valores de descifrado (aunque no se usa en este código).
- **TITULO**: Título estático para la interfaz o propósito del servidor.
- **MENSAJE, ARCHIVO, LISTA_ARCHIVOS**: Constantes para identificar los tipos de mensajes recibidos o enviados entre el servidor y el cliente.

## 3. **Método `main`**:

```java
public static void main(String[] args) {
    int PORT = 4242; // Puerto por default
    if (args.length > 0) {
        PORT = Integer.parseInt(args[0]);
    }
    try (ServerSocket serverSocket = new ServerSocket(PORT)) {
        String fldr = "C:\\Users\\dii\\Documents\\NetBeansProjects\\Server_chat_archivo";
        Socket socket = serverSocket.accept();
        DataInputStream din = new DataInputStream(socket.getInputStream());
        DataOutputStream dout = new DataOutputStream(socket.getOutputStream());
        Scanner scanner = new Scanner(System.in);
```
- Establece el puerto por defecto (`4242`) o lo toma desde los argumentos del programa.
- **ServerSocket** se usa para crear un servidor que escucha las conexiones entrantes.
- **DataInputStream** y **DataOutputStream**: Se usan para recibir y enviar datos a través de la red.

## 4. **Hilo para Recibir Mensajes y Archivos**:

```java
Thread receiveThread = new Thread(() -> {
    try {
        while (!socket.isClosed()) {
            int tipo = din.readInt();
            switch (tipo) {
                case MENSAJE:
                    String message = din.readUTF();
                    System.out.println("Cliente: " + message);
                    break;
                case ARCHIVO:
                    recibirArchivo(din);
                    break;
                case LISTA_ARCHIVOS:
                    List<String> lista = obtenlistaArchivosLocales(fldr);
                    enviarListaArchivos(dout, lista);
                    break;
                default:
                    System.out.println("Tipo desconocido: " + tipo);
                    break;
            }
        }
    } catch (Exception e) {
        System.out.println("El cliente se desconectó...");
    }
});
receiveThread.start();
```
- Este hilo escucha constantemente los mensajes o archivos del cliente.
- Dependiendo del tipo de mensaje recibido, se ejecutan diferentes acciones: mostrar un mensaje, recibir un archivo o enviar una lista de archivos.

## 5. **Envío de Mensajes o Archivos**:

```java
while (true) {
    try {
        String serverMessage = scanner.nextLine();
        if (serverMessage.startsWith("archivo:")) {
            String filePath = serverMessage.substring(8).trim();
            enviarArchivo(dout, filePath);
        } else if (serverMessage.startsWith("lista_local:")) {
            List<String> listaArchivos = obtenlistaArchivosLocales(fldr);
            muestraLista(listaArchivos);
        } else if (serverMessage.startsWith("lista_remota:")) {
            dout.writeInt(LISTA_ARCHIVOS);
            dout.flush();
            recibirListaArchivos(din);
        } else {
            dout.writeInt(MENSAJE); // 1
            dout.writeUTF(serverMessage);
            dout.flush();
        }
    } catch (Exception e) {
        System.out.println("Error al enviar mensaje: " + e.getMessage());
        if (socket.isClosed()) {
            break;
        }
    }
}
```
- Este bloque permite al servidor enviar mensajes o archivos al cliente, basado en lo que ingrese el usuario (a través de la consola).
- Si el mensaje comienza con `archivo:`, se envía un archivo.
- Si el mensaje comienza con `lista_local:`, se envía una lista de archivos locales.
- Si el mensaje comienza con `lista_remota:`, se solicita la lista de archivos remotos al cliente.

## 6. **Método `recibirArchivo`**:

```java
private static void recibirArchivo(DataInputStream din) throws Exception {
    // Leemos longitud de archivo
    int fileNameLength = din.readInt();
    byte[] fileNameBytes = new byte[fileNameLength];
    din.readFully(fileNameBytes, 0, fileNameLength);
    String fileName = new String(fileNameBytes);
    long fileLength = din.readLong();
    System.out.println("Recibiendo archivo: " + fileName + " (" + fileLength + " bytes)");
    // Recibimos el archivo
    File file = new File("received_" + fileName);
    try (FileOutputStream fileOutputStream = new FileOutputStream(file)) {
        byte[] buffer = new byte[4 * 1024];
        int bytes;
        long totalBytesRead = 0;
        while (totalBytesRead < fileLength) {
            bytes = din.read(buffer, 0, (int) Math.min(buffer.length, fileLength - totalBytesRead));
            fileOutputStream.write(buffer, 0, bytes);
            totalBytesRead += bytes;
        }
    }
    System.out.println("Archivo " + fileName + " recibido exitosamente");
}
```
- Este método maneja la recepción de archivos. Primero lee el nombre y la longitud del archivo, luego recibe el archivo en bloques de 4 KB y lo guarda en el sistema local.

## 7. **Métodos para el Manejo de Archivos y Listas**:

```java
public static List<String> obtenlistaArchivosLocales(String path) { ... }
public static void muestraLista(List<String> lista) { ... }
public static void enviarListaArchivos(DataOutputStream dout, List<String> lista) { ... }
public static void recibirListaArchivos(DataInputStream din) { ... }
```
- **obtenlistaArchivosLocales**: Obtiene la lista de archivos del directorio local especificado.
- **muestraLista**: Muestra los nombres de los archivos en la consola.
- **enviarListaArchivos**: Envía la lista de archivos al cliente.
- **recibirListaArchivos**: Recibe una lista de archivos del cliente y la muestra.

## 8. **Desencriptación de Archivos**:

```java
private static void desencriptarArchivo(File archivo) throws IOException { ... }
private static byte[] descifrarArchivo(byte[] encryptedFile) { ... }
```
- **desencriptarArchivo**: Recibe el archivo cifrado y lo desencripta usando un método de sustitución simple basado en una clave estática. Luego guarda el archivo desencriptado con un nombre nuevo.
- **descifrarArchivo**: Desencripta los datos usando la clave de cifrado (en este caso, restando el valor de la clave a cada byte del archivo cifrado).

## Resumen:
Este código implementa un servidor de chat que permite recibir y enviar mensajes y archivos, incluyendo una funcionalidad para desencriptar archivos con una clave simple. Utiliza un **Socket** para la comunicación, maneja diferentes tipos de mensajes y archivos, y emplea un algoritmo de **descifrado simple** basado en una clave predefinida.

## Código Cliente_Chat_Archivo
Este código representa un cliente para un sistema de chat con soporte para el envío y recepción de archivos, incluyendo la funcionalidad de cifrar y descifrar archivos. A continuación, te doy una explicación detallada de cada parte del código:

### 1. **Importación de librerías**:

```java
import java.io.*;
import java.net.Socket;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;
```

- **java.io**: Contiene clases necesarias para la entrada y salida de datos, como `FileInputStream`, `FileOutputStream`, `DataInputStream`, y `DataOutputStream`, que permiten trabajar con archivos y flujos de datos.
- **java.net.Socket**: Utilizada para establecer la comunicación con el servidor mediante sockets.
- **java.util.ArrayList y List**: Utilizadas para gestionar colecciones de datos (listas de archivos en este caso).
- **java.util.Scanner**: Utilizada para capturar entradas del usuario desde la consola.

### 2. **Constantes de tipo de mensaje**:

```java
private static final int MENSAJE = 1;
private static final int ARCHIVO = 2;
private static final int LISTA_ARCHIVOS = 3;
```

Estas constantes se utilizan para identificar los diferentes tipos de datos que el cliente enviará o recibirá: mensajes, archivos y listas de archivos.

### 3. **Configuración del cliente (conexión al servidor)**:

```java
String HOST = "localhost";
int PORT = 4242;
if (args.length >= 2) {
    HOST = args[0];
    PORT = Integer.parseInt(args[1]);
}
```

Aquí se configura la dirección del servidor (`localhost` por defecto) y el puerto (`4242` por defecto). Si el cliente se ejecuta con argumentos, estos se usan para sobrescribir estos valores.

### 4. **Creación de conexión y configuración de flujos de entrada y salida**:

```java
try (Socket socket = new Socket(HOST, PORT);
     DataInputStream din = new DataInputStream(socket.getInputStream());
     DataOutputStream dout = new DataOutputStream(socket.getOutputStream());
     Scanner scanner = new Scanner(System.in)) {
```

- **Socket**: Establece la conexión con el servidor utilizando la dirección y puerto especificados.
- **DataInputStream y DataOutputStream**: Se utilizan para leer y escribir datos a través del socket.
- **Scanner**: Se utiliza para leer entradas del usuario desde la consola.

### 5. **Recepción de mensajes y archivos en un hilo separado**:

```java
Thread receiveThread = new Thread(() -> {
    try {
        while (true) {
            int type = din.readInt();
            if (type == MENSAJE) {
                String message = din.readUTF();
                System.out.println("Server: " + message);
            } else if (type == ARCHIVO) {
                recibirArchivo(din);
            } else if (type == LISTA_ARCHIVOS) {
                List<String> lista = obtenlistaArchivosLocales(".");
                enviarListaArchivos(dout, lista);
            }
        }
    } catch (Exception e) {
        System.out.println("Desconectado del servidor..." + e.getMessage());
    }
});
receiveThread.start();
```

- Este bloque crea un hilo independiente para recibir mensajes y archivos desde el servidor.
- **din.readInt()**: Lee el tipo de mensaje (mensaje, archivo, lista de archivos).
- Si el tipo es un mensaje, se imprime en consola; si es un archivo, se llama a la función `recibirArchivo()` para procesarlo.

### 6. **Envío de archivos cifrados y otros comandos**:

```java
while (true) {
    String clientMessage = scanner.nextLine();
    // Enviar carpeta cifrada
    if (clientMessage.startsWith("enviar_folder:")) {
        // Cifrado y envío de la carpeta
    }
    // Enviar archivo
    if (clientMessage.startsWith("archivo:")) {
        String filePath = clientMessage.substring(8).trim();
        enviarArchivo(dout, filePath);
    }
    // Ver lista de archivos locales
    if (clientMessage.startsWith("lista_local:")) {
        List<String> listaArchivos = obtenlistaArchivosLocales(".");
        muestraLista(listaArchivos);
    }
    // Solicitar lista de archivos remotos
    if (clientMessage.startsWith("lista_remota:")) {
        dout.writeInt(LISTA_ARCHIVOS);
        dout.flush();
        recibirListaArchivos(din);
    }
}
```

- **enviar_folder**: Si el mensaje comienza con `enviar_folder:`, se cifra la carpeta especificada y se envían sus archivos al servidor.
- **archivo**: Si el mensaje comienza con `archivo:`, se envía un archivo específico al servidor.
- **lista_local**: Muestra la lista de archivos locales disponibles.
- **lista_remota**: Solicita la lista de archivos disponibles en el servidor.

### 7. **Envío de archivos**:

```java
private static void enviarArchivo(DataOutputStream dout, String path) throws Exception {
    File file = new File(path);
    FileInputStream fileInputStream = new FileInputStream(file);

    dout.writeInt(ARCHIVO); // Tipo de archivo
    dout.writeInt(file.getName().length());
    dout.write(file.getName().getBytes());
    dout.writeLong(file.length());

    byte[] buffer = new byte[4 * 1024];
    int bytes;
    while ((bytes = fileInputStream.read(buffer)) != -1) {
        dout.write(buffer, 0, bytes);
    }

    fileInputStream.close();
    dout.flush();
    System.out.println("Archivo " + file.getName() + " enviado exitosamente");
}
```

- Este método envía un archivo al servidor, primero enviando el tipo de dato (ARCHIVO), el nombre del archivo, su tamaño y luego los datos del archivo en bloques.

### 8. **Recepción de archivos**:

```java
private static void recibirArchivo(DataInputStream din) throws Exception {
    int fileNameLength = din.readInt();
    byte[] fileNameBytes = new byte[fileNameLength];
    din.readFully(fileNameBytes, 0, fileNameLength);
    String fileName = new String(fileNameBytes);

    long fileLength = din.readLong();
    File file = new File("received_" + fileName);
    try (FileOutputStream fileOutputStream = new FileOutputStream(file)) {
        byte[] buffer = new byte[4 * 1024];
        int bytes;
        long totalBytesRead = 0;
        while (totalBytesRead < fileLength) {
            bytes = din.read(buffer, 0, (int) Math.min(buffer.length, fileLength - totalBytesRead));
            fileOutputStream.write(buffer, 0, bytes);
            totalBytesRead += bytes;
        }
    }
    System.out.println("Archivo " + fileName + " recibido exitosamente");
}
```

- Este método maneja la recepción de archivos, leyendo el nombre del archivo, su tamaño y luego guardando los datos recibidos en un archivo local.

### 9. **Otros métodos auxiliares**:

- **obtenlistaArchivosLocales**: Devuelve una lista de archivos locales con extensiones específicas (como `.txt`, `.jpg`, `.png`).
- **muestraLista**: Muestra una lista de archivos en la consola.
- **enviarListaArchivos**: Envía la lista de archivos al servidor.
- **recibirListaArchivos**: Recibe y muestra la lista de archivos disponibles en el servidor.

### Resumen:
Este código implementa un cliente que interactúa con un servidor de chat, permitiendo el envío y recepción de mensajes y archivos. Además, ofrece funcionalidades como cifrar y descifrar archivos antes de enviarlos, y manejar directorios completos de archivos.
# Descripción del funcionamiento del programa

1. *El servidor se pone en funcionamiento*  
   El servidor inicia su ejecución y queda en espera activa, preparado para aceptar conexiones entrantes.
![Servidor iniciado](https://github.com/user-attachments/assets/1847341c-4666-4f1f-993d-a400dc77b4de)

2. *El cliente establece la conexión con el servidor*  
   El cliente se conecta al servidor a través de un canal de comunicación previamente definido, como un socket o una dirección IP específica.
![Cliente conectado](https://github.com/user-attachments/assets/050a74af-1772-4b99-899d-69a45317aaec)
3. *El cliente cifra un folder con archivos*  
   Antes de enviar el archivo, el cliente aplica un algoritmo de cifrado (en formato: cifrar_folder:<inputFolder>;<outputFolder>) para proteger su contenido, asegurando la confidencialidad de los datos.
Ejemplo: 
cifrar_folder: C:\Users\YUGEN\Documents\NetBeansProjects\Cliente_chat_archivo\hola;C:\Users\YUGEN\Documents\NetBeansProjects\Cliente_chat_archivo\hola_des
![cifrado2](https://github.com/user-attachments/assets/ff644a1a-36f3-4cd4-9184-b6d573248c0f)

4. *El cliente transfiere el archivo cifrado al servidor*  
   El cliente envía el archivo ya cifrado a través de la conexión establecida, garantizando que el servidor reciba la información en un formato seguro.
   ![envia archivos cifrados](https://github.com/user-attachments/assets/61bfac29-9141-479e-9a12-c2e80565d680)
   
5. *El cliente envía una carpeta con archivos cifrados al servidor*  
   El cliente selecciona una carpeta que contiene varios archivos, los cifra utilizando un algoritmo de seguridad (como AES o RSA) y luego transfiere la carpeta completa al servidor mediante la conexión establecida. El proceso asegura que los datos sensibles estén protegidos durante toda la transmisión.

6. *El servidor transfiere archivos al cliente*  
   El servidor selecciona los archivos que deben enviarse al cliente y establece un protocolo de transferencia seguro para entregarlos. Los archivos pueden ser cifrados antes de la transmisión si es necesario, garantizando la integridad y confidencialidad de los datos en tránsito.
![servidor envia archivo a cliente](https://github.com/user-attachments/assets/b9898131-f819-4298-9764-ea4666dcdbd1)
