T L S  H A N D S H A K E 
________________________


El saludo TLS 1.2
TLS 1.2 se puede configurar para usar varios algoritmos de intercambio de claves, y entre ellos, el más conocido y ampliamente utilizado es el algoritmo de intercambio de claves RSA. Veamos cómo funciona aproximadamente el algoritmo RSA en TLS 1.2 y luego hablemos de cómo difiere el proceso en TLS 1.3 con el algoritmo de intercambio de claves Diffie-Hellman (DH).

El intercambio de claves RSA funciona más o menos de la siguiente manera en TLS 1.2:

El servidor escucha nuevas conexiones en el puerto 443.

El cliente se conecta al puerto 443 e inicia el proceso de saludo con un mensaje ClientHello al servidor. Este mensaje contiene las versiones de TLS y las suites de cifrado que el cliente admite, y un número aleatorio de 32 bytes conocido como Cliente Aleatorio.

El servidor responde con un mensaje ServerHello. El servidor selecciona una cifra y la versión de TLS de la lista proporcionada por el cliente, luego genera un número aleatorio diferente de 32 bytes conocido como Servidor Aleatorio y lo envía junto con el certificado SSL del servidor.

El cliente utiliza el certificado SSL para verificar la identidad del servidor. Los certificados SSL contienen una clave pública generada por el servidor y una firma digital firmada con la clave privada de una tercera parte de confianza conocida como autoridad de certificación (CA). La mayoría de los navegadores web y sistemas operativos vienen con claves públicas de CAs de confianza, que se utilizan para verificar que la CA emitió el certificado.

El cliente verifica que el servidor tiene la clave privada coincidente con el certificado a través de ClientKeyExchange. El cliente genera el secreto de premaestro (un número aleatorio de 48 bytes) y lo cifra con la clave pública del servidor obtenida del certificado. Si el servidor es el verdadero propietario del certificado, debe poder descifrar el mensaje y obtener el secreto de premaestro original.

clientKeySchange detallado: 
Dentro del handshake de TLS (Transport Layer Security), el paso ClientKeyExchange es una parte crucial en la que el cliente envía información necesaria para establecer la clave de sesión compartida con el servidor. Aquí hay una descripción más detallada de este paso:

Después de que el servidor envía su certificado al cliente durante el handshake de TLS, el cliente responde con el mensaje ClientKeyExchange. Este mensaje contiene la información necesaria para que ambas partes (cliente y servidor) acuerden una clave de sesión.


Explico detalladamente clientKeyExchange:

Generación del Secreto de Premaestro (Pre-Master Secret): El cliente genera un secreto de premaestro, que es un número aleatorio de 48 bytes. Este secreto de premaestro se utiliza para derivar la clave de sesión compartida.

Cifrado con la Clave Pública del Servidor: El cliente cifra el secreto de premaestro con la clave pública del servidor, que se encuentra en el certificado SSL del servidor. Este paso garantiza que solo el servidor, poseedor de la clave privada correspondiente, pueda descifrar este mensaje y obtener el secreto de premaestro original.

Envío al Servidor: El cliente envía el mensaje ClientKeyExchange al servidor, que contiene el secreto de premaestro cifrado.

El servidor, al recibir este mensaje, utiliza su clave privada para descifrar el secreto de premaestro. Ahora, ambas partes tienen el secreto de premaestro, así como los valores del ClientRandom y ServerRandom generados durante el inicio del handshake. Estos elementos se utilizan para derivar la clave de sesión compartida mediante algoritmos acordados previamente.

Es importante destacar que este proceso específico de ClientKeyExchange es específico de la configuración de TLS 1.2 con el intercambio de claves RSA. En TLS 1.3 y otros esquemas de intercambio de claves, como el Diffie-Hellman, el proceso puede variar, pero la esencia es facilitar el intercambio de información para la generación de una clave de sesión compartida segura.


---

continua con el resto de handshake







El servidor descifra el secreto de premaestro. Ahora tanto el servidor como el cliente tienen el secreto de premaestro, el Servidor Aleatorio y el Cliente Aleatorio. Pueden usar eso para derivar el secreto maestro utilizando el cifrado acordado previamente y llegar a la misma clave (simétrica), que se convertirá en la clave de sesión una vez que ambas partes verifiquen que sus claves coinciden.
El cliente envía un mensaje ChangeCipherSpec. Esto indica que el cliente está listo para crear la clave de sesión y comenzar a comunicarse a través de un túnel cifrado con esa clave.
El cliente demuestra al servidor que tiene las claves correctas. El cliente hashea todos los registros de saludo hasta ese punto, los cifra con la clave de sesión y los envía al servidor. Si el servidor genera la clave correcta, podrá descifrar ese mensaje y verificar los hasheos de registros (que el servidor puede generar de forma independiente).
El servidor demuestra al cliente que tiene las claves correctas. El servidor hace lo mismo y lo envía al cliente.
Se establece una conexión segura. Si ambas claves coinciden, el saludo TLS está completo.
Problemas con TLS 1.2
TLS 1.2 dejó muchos de sus parámetros de configuración en manos de los usuarios, lo que a menudo resultó en malas elecciones (el viejo adagio de seguridad de "entre demasiadas opciones, se elegirá la peor").

Además, en nombre de la compatibilidad hacia atrás, TLS 1.2 también permitía el uso de cifrados antiguos e inseguros. (TLS 1.2 en sí mismo se lanzó en 2008). Con el tiempo, se volvió más propenso a ataques a medida que se descubrían vulnerabilidades en cifrados antiguos o versiones de TLS.

Además de todo eso, el intercambio de claves RSA tiene algunas debilidades:

Se utiliza el mismo par de claves público-privado tanto para autenticar al servidor como para cifrar el secreto de premaestro (punto #5 arriba). Si un atacante obtiene la clave privada, podrá regenerar las claves de sesión y descifrar toda la sesión.
No hay Secreto Perfecto Hacia Adelante (PFS). Dado que la clave privada del servidor no cambia, un atacante podría grabar y almacenar datos cifrados durante años, esperando descifrarlos años después si la clave privada del servidor se filtra o se compromete.
Rendimiento. Es necesario realizar dos rondas de comunicación entre el cliente y el servidor hasta que se establece la clave de sesión.
Para abordar estos problemas, el IETF comenzó a desarrollar TLS 1.3 en 2013 y finalmente lo lanzó al público en agosto de 2018.

¿Qué cambió en TLS 1.3?
En la práctica, las principales diferencias entre TLS 1.3 y 1.2 son:

Saludo más corto
Solo admite algoritmos de intercambio de claves con Secreto Perfecto Hacia Adelante (PFS), como el efímero Diffie-Hellman
Soporte para reanudación de tiempo de viaje cero (0-RTT)
Esto representa un cambio de paradigma en TLS 1.3: cambiar el enfoque de la configurabilidad del usuario y la compatibilidad hacia atrás a la seguridad al ser más opinativo.

Esto permitió a los diseñadores crear un saludo más corto (al restringir DH como el único algoritmo de intercambio de claves) pero también evitó que los desarrolladores eligieran opciones inseguras al restringir el número de opciones disponibles en primer lugar.

Por ejemplo, aunque técnicamente era posible usar DH como el método de intercambio de claves en TLS 1.2, los participantes eran responsables de establecer los parámetros DH. Elegir los parámetros correctos es crucial para asegurarse de que el intercambio DH sea seguro, ya que algunos parámetros pueden romperse trivialmente.

En 2015, los investigadores de seguridad publicaron un documento describiendo cómo el ataque WeakDH, que afectaba al 8.4% de los principales 1 millón de dominios en ese momento, podía engañar a los servidores para que usaran parámetros DH pequeños (inseguros), lo que permitía a los adversarios descifrar datos transmitidos entre el cliente y el servidor.