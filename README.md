# Ciberseguridadv1
Repositorio hecho para entregar la evaluacion de ciberseguridad


#Laboratorio #1
Para el siguiente laboratorio se van a utilizar dos entornos. Uno de ellos utiliza Kali Linux, el cual será el encargado de generar el hash y manejar las llaves y la generación de estas. El otro entorno va a ser el encargado de realizar un ataque informático y cambiar parte de la información contenida en el archivo de claves bancarias mediante ssh.

Procedimiento
Utilizando el entorno de Kali, se crea un documento de texto con un monto de 10000 el cual será encriptado más adelante.
 <img width="975" height="295" alt="image" src="https://github.com/user-attachments/assets/87ccaaeb-b678-4f3e-a526-37264e7bf0dc" />

Firma original usando sha256sum.
 
Se da inicio al servicio ssh en maquina victima
 
Se modifica el archivo configuración_bancaria.txt
 
Se vuelve a generar el hash desde la maquina victima
 
Ilustración 1- Se puede ver que el hash cambio completamente
Fase 2: Configuración de Firma Digital (OpenSSL)
En esta fase, se generarán claves privadas y públicas para firmar un certificado.
Creación de llave privada y llave publica respectivamente:
 
Creación de la firma digital del archivo:
 
Verificación de la firma digital y de la encriptación del archivo:
 





# Laboratorio #3
