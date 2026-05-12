# Ciberseguridadv1
Repositorio hecho para entregar la evaluacion de ciberseguridad


# Lab 1: Integridad de Datos y Firma Digital
**Unidad 2 — Gestión Avanzada de Ciberseguridad: Actualización para la Prevención de Amenazas y Protección de Datos**

---

## 📄 Enunciado Original

>
> **Actividad:** Configuración de firma digital y verificación de integridad.
>
> **Comandos clave:** `openssl`, `sha256sum`, `gpg`.
>
> **Tarea:** Los alumnos deben generar un par de llaves (Pública/Privada), firmar un archivo de configuración crítico y verificar que si se cambia un solo bit del archivo, el hash de integridad falla.
>
> **Enfoque organizacional:** Simular la firma de un manifiesto de software antes de subirlo a producción.
>
> **Dinámica:**
> - **Generación de Hash:** Crear un archivo `config_bancaria.txt`. Usar `sha256sum` para generar su huella digital.
> - **Simulación de Ataque:** Un "atacante" modifica un solo carácter (ej: cambia un `0` por un `1` en un monto).
> - **Verificación:** El alumno vuelve a correr el hash y nota que la "huella" es totalmente distinta.
> - **Firma Digital (OpenSSL):**
>   - Generar llave privada: `openssl genrsa -out privada.pem 2048`
>   - Generar llave pública: `openssl rsa -in privada.pem -pubout -out publica.pem`
>   - Firmar archivo: `openssl dgst -sha256 -sign privada.pem -out firma.bin config_bancaria.txt`
>
> **Entorno:** Ubuntu Server (Víctima/Servidor) y Kali (Auditor).

---

## 📝 Descripción y Objetivos

Este laboratorio explora dos pilares fundamentales de la criptografía aplicada a la seguridad de la información: la **verificación de integridad mediante hashing** y el **no repudio mediante firma digital asimétrica**. Ambos conceptos son críticos en entornos organizacionales donde la autenticidad e integridad de los documentos —ya sean contratos, manifiestos de software o registros financieros— deben poder ser auditados y verificados sin ambigüedad.

El escenario planteado simula una situación de alto impacto en el sector financiero: un archivo de configuración con instrucciones de transferencia bancaria es generado y firmado por un operador legítimo (Kali Linux), para luego ser comprometido por un atacante que accede al servidor víctima (Ubuntu Server) mediante SSH y altera el monto de la transacción. El objetivo es demostrar, de forma empírica, que:

1. **El hashing (SHA-256) detecta cualquier alteración**, incluso de un solo carácter, gracias al *efecto avalancha*: una modificación mínima en los datos de entrada produce un hash de salida radicalmente distinto.

2. **La firma digital (RSA + SHA-256 mediante OpenSSL)** agrega una capa de autenticación que garantiza el *no repudio*: solo quien posee la llave privada puede generar una firma válida, y cualquier modificación posterior al archivo invalida la verificación criptográfica.

**Finalidad académica:** Internalizar el ciclo completo de protección de la integridad de datos — generación de hash → detección de manipulación → implementación de firma digital como mecanismo de no repudio —, aplicable directamente a pipelines de entrega de software, auditoría de configuraciones y custodia de documentos digitales en entornos corporativos.

---

## 🌐 Datos de Red

| Rol | Sistema Operativo | Dirección IP | Servicio expuesto |
|---|---|---|---|
| Auditor / Firmante | Kali Linux | `192.168.1.136` | — |
| Servidor / Víctima | Ubuntu Server | `192.168.1.115` | SSH (puerto 22) |

> ⚠️ Reemplazar las IPs con los valores reales del entorno de laboratorio antes de ejecutar cualquier comando.

---

## 🛠️ Herramientas Utilizadas

| Herramienta | Origen | Propósito |
|---|---|---|
| **sha256sum** | GNU Coreutils (nativo en Linux) | Generación y comparación de hashes SHA-256 |
| **OpenSSL** | Nativo en Kali Linux / Ubuntu | Generación del par de llaves RSA y firma digital |
| **SSH (OpenSSH)** | Nativo en ambas máquinas | Vector de acceso del atacante al servidor víctima |
| **echo / cat** | Shell POSIX | Creación y visualización del archivo objetivo |

---

## 🗺️ Hoja de Ruta

### Fase 1 — Generación de Hash y Verificación de Integridad

**Objetivo:** Demostrar que el algoritmo SHA-256 detecta cualquier alteración sobre un archivo, por mínima que sea, como consecuencia del efecto avalancha.

---

**Paso 1:** Crear el archivo de configuración crítica que simulará un manifiesto de transferencia bancaria:
```bash
echo "Monto:10000" > config_bancaria.txt
```

**Paso 2:** Generar la huella digital SHA-256 del archivo en su estado íntegro. Copiar y registrar este hash como valor de referencia:
```bash
sha256sum config_bancaria.txt
```
> Salida esperada (ejemplo): `a3f1c8...e7d2  config_bancaria.txt`

**Paso 3 — Simulación de ataque:** Desde la máquina Ubuntu Server (accedida mediante SSH), el atacante modifica un único carácter del archivo, alterando el monto de la transacción:
```bash
echo "Monto:10001" > config_bancaria.txt
```

**Paso 4:** Volver a generar el hash del archivo modificado y comparar con el valor registrado en el Paso 2:
```bash
sha256sum config_bancaria.txt
```
> **Resultado esperado:** El hash resultante es completamente distinto al original a pesar de que la modificación fue de un solo carácter, evidenciando el *efecto avalancha* inherente a las funciones de hash criptográficas.

---

### Fase 2 — Configuración de Firma Digital (OpenSSL)

**Objetivo:** Implementar un esquema de no repudio mediante criptografía asimétrica RSA combinada con SHA-256, de forma que cualquier alteración posterior al archivo invalide la firma digital.

---

**Paso 1:** Generar el par de llaves asimétricas RSA de 2048 bits. La llave privada es el secreto del firmante y nunca debe ser compartida ni subida al repositorio:
```bash
# Generar llave privada (mantener en custodia segura)
openssl genrsa -out privada.pem 2048

# Extraer la llave pública correspondiente
openssl rsa -in privada.pem -pubout -out publica.pem
```

**Paso 2:** Firmar el archivo con la llave privada. Este proceso genera un hash SHA-256 del contenido y lo cifra con la llave privada RSA, produciendo el archivo de firma binaria `firma.bin`:
```bash
openssl dgst -sha256 -sign privada.pem -out firma.bin config_bancaria.txt
```

**Paso 3:** Verificar la firma digital utilizando la llave pública. Este comando recalcula el hash del archivo, descifra la firma con la llave pública y compara ambos valores:
```bash
openssl dgst -sha256 -verify publica.pem -signature firma.bin config_bancaria.txt
```

| Resultado | Significado |
|---|---|
| `Verified OK` | El archivo es íntegro y fue firmado por quien posee la llave privada correspondiente |
| `Verification Failure` | El archivo fue alterado después de ser firmado, o la firma no corresponde a la llave pública utilizada |

---

## 📸 Evidencia Visual

### 🔴 Fase de Vulnerabilidad — Fallo de Integridad por Alteración del Archivo

<img width="975" height="295" alt="image" src="https://github.com/user-attachments/assets/de894d95-c43c-482e-91b1-341671640fae" />
<img width="975" height="123" alt="image" src="https://github.com/user-attachments/assets/15cf8570-e5de-4ccc-896c-7cf1e5ee6468" />

> **Observación:** Terminal de Kali mostrando el hash SHA-256 generado sobre `config_bancaria.txt` en su estado original (monto: 10000). Este valor hexadecimal de 64 caracteres constituye la huella digital de referencia para la posterior verificación de integridad.

---

<img width="975" height="170" alt="image" src="https://github.com/user-attachments/assets/f52e12c2-eefa-4fbf-874b-54fbb0d63a15" />

> **Observación:** Comparativa entre el hash original y el hash recalculado tras la alteración del monto (de 10000 a 10001) ejecutada por el atacante vía SSH. Se evidencia el *efecto avalancha*: a pesar de modificar un solo carácter, la totalidad de los 64 caracteres hexadecimales del hash resultante difieren del valor de referencia, detectando de forma inequívoca la manipulación del archivo.

---

### 🟢 Fase de Hardening — Firma Digital y Verificación Exitosa

<img width="975" height="419" alt="image" src="https://github.com/user-attachments/assets/8b9d5b32-3daf-47d6-9041-9fea2c311a89" />

> **Observación:** Salida de los comandos `openssl genrsa` y `openssl rsa -pubout`, confirmando la generación exitosa del par de llaves asimétricas RSA de 2048 bits. Los archivos `privada.pem` y `publica.pem` quedan disponibles para el proceso de firma y verificación respectivamente.

---

<img width="975" height="96" alt="image" src="https://github.com/user-attachments/assets/5b31d4ff-9aea-46cb-ab84-24572be651f0" />


> **Observación:** Resultado del comando `openssl dgst -sha256 -verify` ejecutado sobre el archivo en su estado íntegro, mostrando el mensaje **`Verified OK`**. Este output confirma que el archivo `config_bancaria.txt` no ha sido alterado desde el momento de la firma y que esta fue generada por el legítimo poseedor de la llave privada, garantizando simultáneamente integridad y no repudio.


## 📊 Comparativa de Resultados

| Escenario | Herramienta | Resultado | Conclusión |
|---|---|---|---|
| Archivo original → Hash | `sha256sum` | Hash A | Huella digital de referencia establecida |
| Archivo modificado → Hash | `sha256sum` | Hash B ≠ Hash A | Integridad comprometida — alteración detectada |
| Archivo íntegro → Verificar firma | `openssl dgst -verify` | `Verified OK` | Integridad y autenticidad confirmadas |

---

## 📚 Marco Conceptual

**Función de Hash Criptográfica (SHA-256):** Algoritmo de la familia SHA-2 (FIPS 180-4) que produce una salida de 256 bits (64 caracteres hexadecimales) de longitud fija a partir de una entrada de tamaño arbitrario. Sus propiedades fundamentales son: determinismo (mismo input → mismo output), resistencia a preimagen, resistencia a colisiones y el *efecto avalancha*. Es una función estrictamente unidireccional: no es computacionalmente viable reconstruir el input a partir del hash.

**Criptografía Asimétrica (RSA):** Esquema criptográfico basado en un par de llaves matemáticamente relacionadas — una pública (distribuible libremente) y una privada (custodiada por el propietario). En el contexto de firma digital, el firmante cifra el hash del documento con su llave privada; cualquier tercero puede verificar la autenticidad descifrando la firma con la llave pública correspondiente, sin necesidad de conocer la llave privada.

**Firma Digital:** Mecanismo criptográfico que combina hashing y criptografía asimétrica para garantizar simultáneamente la **integridad** del documento (cualquier alteración invalida la firma) y el **no repudio** del firmante (solo quien posee la llave privada pudo generar esa firma). Es el fundamento técnico de certificados X.509, TLS, firma de código y documentos legalmente vinculantes en formato digital.

**No Repudio:** Propiedad criptográfica que impide que el emisor de un mensaje o firmante de un documento niegue haberlo generado, dado que la firma solo pudo ser producida por quien posee la llave privada correspondiente al par de llaves utilizado.

---

### 📋 Reporte de Incidente Simulado

| Campo | Detalle |
|---|---|
| **Naturaleza de la anomalía** | Modificación no autorizada de un byte en archivo de configuración financiera (`config_bancaria.txt`) — alteración del monto de transferencia de 10000 a 10001 |
| **Sistema afectado** | Ubuntu Server — archivo `config_bancaria.txt` en directorio de trabajo |
| **Vector de acceso** | Protocolo SSH (puerto 22) desde máquina atacante |
| **Método de detección** | Discrepancia de hash SHA-256 entre valor de referencia y hash recalculado post-modificación |
| **Hora del hallazgo** | Registrar timestamp al momento de ejecución del laboratorio |
| **Impacto potencial** | En entorno productivo: fraude financiero por desvío de fondos sin detección inmediata si no se implementa verificación de integridad |
| **Contramedida aplicada** | Firma digital RSA-2048 + SHA-256 mediante OpenSSL — cualquier alteración posterior a la firma invalida la verificación criptográfica |

# Lab 2: Ataque y Defensa de Credenciales
**Unidad 2 — Gestión Avanzada de Ciberseguridad: Actualización para la Prevención de Amenazas y Protección de Datos**

---

## 📄 Enunciado Original

> **Actividad:** Realizar ataques de diccionario y fuerza bruta, para luego implementar MFA.
>
> **Herramientas:** Hydra, John the Ripper, Google Authenticator PAM module.
>
> **Tarea:** Romper una contraseña débil de SSH usando Hydra. Luego, instalar y configurar el módulo de Google Authenticator en el servidor Ubuntu para que, aunque el atacante tenga la clave, no pueda entrar sin el token.
>
> **Entorno:** Kali Linux (Atacante) vs Ubuntu (Víctima).
>
> **Dinámica:**
> - **Fase de Ataque:** Desde Kali, usar Hydra para realizar un ataque de diccionario contra el servicio SSH del Ubuntu:
>   ```bash
>   hydra -l usuario -P lista_passwords.txt ssh://[IP_Servidor]
>   ```
> - **Fase de Hardening (MFA):** En el Ubuntu Server, instalar el módulo PAM de Google Authenticator:
>   ```bash
>   sudo apt install libpam-google-authenticator
>   ```
>   Ejecutar `google-authenticator` y configurar la App en el celular.
> - **Configuración de SSH:** Modificar `/etc/pam.d/sshd` y `/etc/ssh/sshd_config` para exigir tanto la contraseña como el token (MFA).
> - **Prueba Final:** Volver a intentar el ataque con Hydra. Aunque Hydra encuentre la contraseña, la conexión fallará porque no puede saltarse el segundo factor.

---

## 📝 Descripción y Objetivos

Este laboratorio aborda uno de los vectores de ataque más frecuentes en entornos corporativos: la **explotación de credenciales débiles sobre el protocolo SSH mediante ataques de diccionario automatizados**. La premisa es simple pero contundente: si un servicio SSH expone autenticación solo por contraseña y esta es predecible, cualquier atacante con una lista de palabras comunes puede comprometer el servidor en cuestión de segundos utilizando herramientas de fuerza bruta como Hydra.

El laboratorio se estructura en cuatro fases progresivas:

1. **Fase de Reconocimiento y Preparación:** Se construye un diccionario de contraseñas simplificado que simula un wordlist real de ataque.

2. **Fase Ofensiva (Ataque de Diccionario):** Desde Kali Linux, se ejecuta Hydra apuntando al servicio SSH del servidor Ubuntu víctima. El objetivo es comprometer la cuenta mediante fuerza bruta de diccionario y obtener las credenciales en texto plano.

3. **Fase Defensiva (Hardening con MFA):** Se implementa autenticación multifactor (MFA) basada en TOTP (*Time-based One-Time Password*) mediante el módulo `libpam-google-authenticator`. Esta contramedida añade un segundo factor de autenticación que Hydra es incapaz de resolver, dado que el token es dinámico y cambia cada 30 segundos.

4. **Fase de Verificación:** Se reitera el ataque con Hydra para demostrar empíricamente que, aunque el atacante posea la contraseña correcta, el segundo factor neutraliza completamente el vector de intrusión.

**Finalidad académica:** Comprender que la seguridad de la autenticación no depende exclusivamente de la complejidad de la contraseña, sino de la arquitectura de verificación de identidad. Una contraseña robusta sin MFA sigue siendo un único punto de fallo; la autenticación multifactor convierte ese punto de fallo en una barrera compuesta que los ataques automatizados no pueden superar.

---

## 🌐 Datos de Red

| Rol | Sistema Operativo | Dirección IP | Servicio expuesto |
|---|---|---|---|
| Atacante / Auditor | Kali Linux | `10.24.7.24` | — |
| Servidor / Víctima | Ubuntu Server | `10.24.7.39` | SSH (puerto 22) |

> ⚠️ Las IPs corresponden al entorno de red del laboratorio ejecutado. Verificar y actualizar según el entorno de ejecución propio.

---

## 🛠️ Herramientas Utilizadas

| Herramienta | Origen | Propósito |
|---|---|---|
| **Hydra v9.5** | Incluido en Kali Linux | Ataque de diccionario automatizado contra SSH |
| **libpam-google-authenticator** | Repositorio oficial Ubuntu | Módulo PAM para MFA basado en TOTP |
| **Google Authenticator** | App móvil (Android/iOS) | Generación de tokens OTP de 6 dígitos cada 30s |
| **nano** | Nativo en Ubuntu | Edición de archivos de configuración del sistema |
| **OpenSSH** | Nativo en ambas máquinas | Protocolo de acceso remoto objetivo del ataque |

---

## 🗺️ Hoja de Ruta

### Fase 1 — Preparación del Diccionario de Ataque

**Objetivo:** Construir el wordlist que Hydra utilizará durante el ataque de diccionario.

**Paso 1:** Crear el archivo de diccionario con contraseñas comunes en Kali:
```bash
echo -e "123456\password\admin123\ubuntu\qwerty\mango2020" > lista_passwords.txt
```

> **Observación:** Comando `echo` generando el archivo `lista_passwords.txt` con contraseñas comunes. En un escenario real de auditoría, se utilizarían wordlists como `rockyou.txt` (14 millones de entradas) disponibles en `/usr/share/wordlists/` en Kali.

---

### Fase 2 — Ataque de Diccionario con Hydra

**Objetivo:** Comprometer la cuenta SSH del servidor Ubuntu explotando una contraseña débil mediante fuerza bruta de diccionario.

**Paso 3:** Ejecutar Hydra apuntando al servicio SSH con el usuario objetivo:
```bash
hydra -l mango -P lista_passwords.txt ssh://10.24.7.39
```

| Flag | Descripción |
|---|---|
| `-l mango` | Usuario objetivo (login fijo) |
| `-P lista_passwords.txt` | Wordlist con contraseñas a probar |
| `ssh://10.24.7.39` | Protocolo y dirección IP del objetivo |

<img width="980" height="438" alt="image" src="https://github.com/user-attachments/assets/bdf4c237-29bf-4504-8f47-cb8f7b807876" />

> **Observación:** Hydra v9.5 reporta `[22][ssh] host: 10.129.70.39 login: mango password: mango2020`. La línea en verde confirma que el ataque de diccionario tuvo éxito: el servicio SSH del servidor víctima fue comprometido en una sola iteración del wordlist. El tiempo total del ataque fue de 4 segundos, evidenciando la velocidad de este tipo de ataques contra credenciales débiles.

---

### Fase 3 — Hardening con MFA (En Ubuntu Server)

**Objetivo:** Implementar autenticación multifactor basada en TOTP para neutralizar el vector de ataque demostrado en la Fase 2.

**Paso 4:** Instalar el módulo PAM de Google Authenticator en el servidor Ubuntu:
```bash
sudo apt update && sudo apt install libpam-google-authenticator -y
```

<img width="980" height="201" alt="image" src="https://github.com/user-attachments/assets/4868fd70-33d5-49fb-aa9c-d1c7b943c7ea" />


> **Observación:** Terminal del Ubuntu Server mostrando la instalación exitosa del paquete `libpam-google-authenticator`. El módulo queda disponible para ser referenciado por el sistema PAM (*Pluggable Authentication Modules*) del sistema operativo.

---

**Paso 5:** Ejecutar el configurador **con el usuario a proteger** (no como root):
```bash
google-authenticator
```

<img width="980" height="747" alt="image" src="https://github.com/user-attachments/assets/8208a3c3-09a5-42d8-9c47-0c461a902b53" />

> **Observación:** El configurador solicita confirmación para usar tokens basados en tiempo (TOTP). Se responde `y` para activar el estándar RFC 6238, que genera códigos válidos por 30 segundos basados en la hora actual y un secreto compartido.

---

<img width="930" height="138" alt="image" src="https://github.com/user-attachments/assets/cf90d80b-28d2-47f9-88b4-54282003ffee" />

> **Observación:** El configurador genera un código QR y muestra la *secret key* (`ZHKE7VDYFZJNCF4CYCKZJ4F2WI`) en texto plano. El código QR debe ser escaneado con la app Google Authenticator en el dispositivo móvil para vincular el servidor. Esta clave es el secreto compartido sobre el que se derivarán todos los tokens TOTP futuros.

---

<img width="980" height="238" alt="image" src="https://github.com/user-attachments/assets/03b3d763-3734-47d2-af73-09f1d27a184d" />

> **Observación:** Tras ingresar el primer código generado por la app (`129897`), el configurador lo valida con `Code confirmed` y genera cinco *emergency scratch codes*. Estos códigos de un solo uso sirven como mecanismo de recuperación en caso de pérdida del dispositivo móvil y deben almacenarse de forma segura.

---

<img width="980" height="503" alt="image" src="https://github.com/user-attachments/assets/d8af4c14-69e5-48af-b137-44d697a66663" />

> **Observación:** Configuración de parámetros de seguridad adicionales: se deshabilita el reúso del mismo token en múltiples autenticaciones (previene ataques de replay), se habilita una ventana de tolerancia temporal de ±30 segundos para compensar desincronización de reloj entre cliente y servidor, y se activa el rate-limiting nativo del módulo (máximo 3 intentos cada 30 segundos).

---

<img width="294" height="392" alt="image" src="https://github.com/user-attachments/assets/3931d9a2-971b-450e-bc45-be287a93da64" />

> **Observación:** Captura del dispositivo móvil con la app Google Authenticator mostrando el token TOTP activo para la cuenta `mango@UbuntuServer` (código `679 004`). Otros tokens del dispositivo están ofuscados por privacidad. Esto confirma el vínculo exitoso entre el servidor y el segundo factor de autenticación físico.

---

### Fase 4 — Configuración Crítica del Sistema (Archivos PAM y SSH)

**Objetivo:** Modificar los archivos de configuración del sistema para que SSH exija obligatoriamente ambos factores de autenticación.

**Paso 6:** Editar el archivo de configuración PAM para SSH y añadir la directiva del módulo al final:
```bash
sudo nano /etc/pam.d/sshd
```
Línea añadida al final del archivo:
```
auth required libpam_google_authenticator.so
```

<img width="661" height="111" alt="image" src="https://github.com/user-attachments/assets/85f1d9a0-bbc2-41a0-9aa9-40049a82801b" />

> **Observación:** Contenido del archivo `/etc/pam.d/sshd` mostrando la línea `auth required libpam_google_authenticator.so` añadida al final. Esta directiva instruye al sistema PAM para que invoque el módulo de Google Authenticator como requisito obligatorio durante el proceso de autenticación SSH.

---

**Paso 7:** Editar la configuración del demonio SSH para habilitar autenticación interactiva:
```bash
sudo nano /etc/ssh/sshd_config
```

<img width="980" height="124" alt="image" src="https://github.com/user-attachments/assets/5a76b015-f13b-4422-a76e-4512172d64d0" />

> **Observación:** Parámetro `KbdInteractiveAuthentication yes` habilitado en `sshd_config`. Esta directiva permite que el servidor SSH interactúe con el usuario mediante prompts de teclado durante la autenticación, mecanismo necesario para solicitar el código TOTP.

---

<img width="550" height="131" alt="image" src="https://github.com/user-attachments/assets/8fe407bf-2b3b-40ea-97db-c7321ebc4fc7" />

> **Observación:** Parámetro `UsePAM yes` confirmado activo. Esta directiva delega el proceso de autenticación al stack PAM del sistema operativo, que a su vez invocará el módulo `libpam_google_authenticator.so` configurado en el paso anterior.

---

<img width="713" height="142" alt="image" src="https://github.com/user-attachments/assets/7ce9315f-75cc-4a29-8021-c14778fb6aae" />

> **Observación:** Directiva `AuthenticationMethods password,keyboard-interactive` añadida al final de `sshd_config`. Esta línea es el núcleo del MFA: instruye a OpenSSH para que exija ambos factores de forma secuencial — primero la contraseña y luego el token TOTP — antes de conceder acceso. Si cualquiera de los dos factores falla, la sesión es rechazada.

---

### Fase 5 — Verificación del Hardening

**Objetivo:** Confirmar que el MFA implementado neutraliza tanto el acceso manual sin token como el ataque automatizado con Hydra.

**Paso 8:** Intentar acceso SSH desde Kali con la contraseña previamente comprometida:
```bash
ssh mango@10.24.7.39
```

<img width="980" height="324" alt="image" src="https://github.com/user-attachments/assets/39319ec7-c37a-4e25-b9d4-ff6f064b9188" />

> **Observación:** Intento de acceso SSH desde Kali utilizando la contraseña `mango2020` descubierta por Hydra en la Fase 2. El servidor responde con `Permission denied, please try again`, bloqueando el acceso. Aunque la contraseña es correcta, la ausencia del segundo factor (código TOTP del dispositivo móvil) impide la autenticación exitosa.

---

**Paso 9:** Reiterar el ataque con Hydra para validar la efectividad del MFA:
```bash
hydra -l mango -P lista_passwords.txt ssh://10.24.7.39
```

<img width="980" height="417" alt="image" src="https://github.com/user-attachments/assets/9e538a0f-183d-4e40-86a9-dbb09c125eca" />

> **Observación:** Segunda ejecución de Hydra con el MFA activo. El resultado es `0 of 1 target successfully completed, 0 valid password found`. A pesar de que el wordlist contiene la contraseña correcta (`mango2020`), Hydra es incapaz de superar el segundo factor de autenticación. El protocolo `keyboard-interactive` requerido por el token TOTP interrumpe el flujo de autenticación automatizada, dado que Hydra no puede generar ni inferir un código válido que cambia cada 30 segundos.

---

## 📊 Comparativa de Resultados

| Escenario | Herramienta | Resultado | Conclusión |
|---|---|---|---|
| Ataque SSH — solo contraseña | Hydra | ✅ `password: mango2020` encontrada | Credencial comprometida en 4 segundos |
| Acceso manual — contraseña correcta, sin token | SSH client | ❌ `Permission denied` | MFA bloquea acceso sin segundo factor |
| Ataque SSH — con MFA activo | Hydra | ❌ `0 valid password found` | Ataque automatizado neutralizado completamente |

---

## 📚 Marco Conceptual

**Ataque de Diccionario:** Variante de fuerza bruta que prueba sistemáticamente palabras de un wordlist precompilado en lugar de todas las combinaciones posibles. Es significativamente más eficiente que la fuerza bruta pura porque explota el hecho de que los usuarios tienden a elegir contraseñas predecibles o derivadas de palabras del lenguaje natural. Herramientas como Hydra permiten paralelizar estos intentos a alta velocidad contra protocolos de red como SSH, FTP o HTTP.

**MFA / TOTP (RFC 6238):** La autenticación multifactor combina al menos dos factores de categorías distintas — algo que sabes (contraseña), algo que tienes (dispositivo móvil), algo que eres (biometría). TOTP (*Time-based One-Time Password*) es el estándar que sustenta Google Authenticator: genera un código de 6 dígitos derivado de un secreto compartido y la hora Unix actual (ventana de 30 segundos). El código es efímero e impredecible para un atacante sin acceso físico al dispositivo.

**PAM (Pluggable Authentication Modules):** Framework de autenticación modular de Linux (RFC no oficial, implementación original de Sun Microsystems) que permite agregar, modificar o encadenar mecanismos de autenticación sin modificar las aplicaciones que los usan. El archivo `/etc/pam.d/sshd` define la pila de módulos que se invocan secuencialmente cuando OpenSSH autentica una sesión entrante.

**keyboard-interactive (RFC 4256):** Método de autenticación SSH que establece un canal de diálogo interactivo entre el cliente y el servidor, permitiendo al servidor solicitar información adicional (como un código OTP) más allá de la contraseña estática. Es el mecanismo que hace posible el MFA sobre SSH sin modificar el protocolo base.


### 📋 Reporte de Incidente Simulado

| Campo | Detalle |
|---|---|
| **Naturaleza de la anomalía** | Compromiso de credenciales SSH mediante ataque de diccionario automatizado — contraseña débil (`mango2020`) expuesta en 4 segundos |
| **Sistema afectado** | Ubuntu Server — servicio OpenSSH (puerto 22), cuenta de usuario `mango` |
| **Vector de ataque** | Hydra v9.5 ejecutado desde Kali Linux — diccionario de 6 entradas |
| **Método de detección** | Ejecución controlada en entorno de laboratorio; en producción detectable mediante análisis de logs `/var/log/auth.log` o sistema IDS/SIEM |
| **Hora del hallazgo** | 2026-04-30 12:23:13 (timestamp registrado por Hydra en la captura) |
| **Impacto potencial** | En entorno productivo: acceso root escalable, exfiltración de datos, pivoting hacia otros sistemas de la red interna |
| **Contramedida aplicada** | MFA-TOTP mediante `libpam-google-authenticator` + modificación de `sshd_config` con `AuthenticationMethods password,keyboard-interactive` |
| **Resultado post-hardening** | Hydra reporta `0 valid password found` — vector de ataque completamente neutralizado |

---

# Laboratorio #3
# Lab 3: Protocolos de Comunicación Seguros
**Unidad 2 — Gestión Avanzada de Ciberseguridad: Actualización para la Prevención de Amenazas y Protección de Datos**

---

## 📄 Enunciado Original

> **Actividad:** Análisis de tráfico inseguro vs. cifrado.
>
> **Herramientas:** Wireshark, Certbot (Let's Encrypt), OpenVPN/WireGuard.
>
> **Tarea:** Capturar una sesión HTTP y ver la contraseña en texto plano. Luego, configurar un certificado SSL (HTTPS) y verificar que el tráfico ahora es ilegible. Configurar un túnel SSH con llaves deshabilitando el acceso por contraseña.
>
> **Entorno:** Kali (Wireshark) y un servidor Apache/Nginx.
>
> **Dinámica:**
> - **Sniffing de HTTP:** El alumno llena un formulario en una web HTTP mientras corre Wireshark. Verá su usuario y contraseña en texto plano (filtro `http.request.method == "POST"`).
> - **Hardening con SSH Tunneling:** Aprender a crear un túnel local para proteger tráfico inseguro: `ssh -L 8080:localhost:80 usuario@servidor`
> - **Implementación de VPN (Opcional/Teórico-Práctico):** Configurar un túnel simple con WireGuard o OpenVPN.
> - **Verificación:** Analizar el tráfico nuevamente en Wireshark. Ahora todo aparecerá como "TCP" o "WireGuard" cifrado, ilegible para el atacante.

---

## 📝 Descripción y Objetivos

Este laboratorio aborda una de las vulnerabilidades más prevalentes en entornos de red corporativos: la **transmisión de credenciales y datos sensibles en texto plano** mediante el protocolo HTTP. Desde la perspectiva de un auditor de seguridad, la ausencia de cifrado en tránsito constituye una superficie de ataque crítica, ya que cualquier agente con acceso a la red (o en posición de *man-in-the-middle*) puede capturar, leer y eventualmente reutilizar dichas credenciales sin necesidad de romper ningún mecanismo criptográfico.

El laboratorio se articula en dos fases claramente diferenciadas:

1. **Fase ofensiva (Sniffing):** Se demuestra de forma empírica cómo Wireshark permite interceptar y leer en texto plano las credenciales enviadas a través de una sesión HTTP, utilizando OWASP Juice Shop como aplicación web objetivo.

2. **Fase defensiva (Hardening):** Se implementa un **túnel SSH con reenvío de puertos local** (*Local Port Forwarding*), que encapsula el tráfico HTTP dentro de una sesión SSH cifrada. Esto permite proteger la comunicación sin necesidad de modificar ni reconfigurar la aplicación web en el servidor.

**Finalidad académica:** Comprender la diferencia fundamental entre un protocolo de comunicación en texto plano (HTTP) y uno que garantiza confidencialidad en tránsito (SSH/TLS), y desarrollar la capacidad de implementar contramedidas de *hardening* sobre sistemas ya desplegados.

---

## 🌐 Datos de Red

| Rol | Sistema Operativo | Dirección IP | Puerto relevante |
|---|---|---|---|
| Atacante / Auditor | Kali Linux | `192.168.1.136` | — |
| Servidor / Víctima | Ubuntu Server | `192.168.1.115` | `3000` (Juice Shop) |

> ⚠️ Reemplazar las IPs con los valores reales del entorno de laboratorio antes de ejecutar cualquier comando.

---

## 🛠️ Herramientas Utilizadas

| Herramienta | Origen | Propósito |
|---|---|---|
| **Wireshark** | Incluido en Kali Linux | Captura y análisis de tráfico de red |
| **OWASP Juice Shop** | Docker (imagen oficial) | Aplicación web vulnerable de práctica |
| **SSH Client (OpenSSH)** | Nativo en Kali Linux | Creación del túnel de *port forwarding* |
| **Docker** | Ubuntu Server | Contenerización y despliegue de Juice Shop |
| **Firefox / Navegador** | Kali Linux | Cliente web para generar tráfico HTTP |

---

## 🗺️ Hoja de Ruta

### Fase 1 — Sniffing de Tráfico HTTP (Escenario Inseguro)

**Objetivo:** Evidenciar la exposición de credenciales en una comunicación HTTP sin cifrar.

**Paso 1:** Iniciar Wireshark en Kali Linux y seleccionar la interfaz de red `eth0` como superficie de captura.

**Paso 2:** Aplicar el filtro de captura para aislar únicamente las solicitudes POST:
```
http.request.method == "POST"
```

**Paso 3:** Desde el navegador de Kali, acceder a la instancia de Juice Shop en el servidor Ubuntu:
```
http://192.168.1.115:3000
```

**Paso 4:** Ingresar credenciales ficticias en el formulario de login (ej. usuario `mango@test.com`, clave `mango123`) y enviar el formulario.

**Paso 5:** Observar en Wireshark el paquete POST capturado. Mediante la opción *"Follow HTTP Stream"*, es posible visualizar en texto plano el cuerpo de la solicitud JSON, incluyendo los campos `email` y `password` sin ningún tipo de ofuscación ni cifrado.

**Resultado esperado:** Las credenciales son completamente legibles para cualquier entidad con acceso pasivo a la red, demostrando el riesgo inherente del protocolo HTTP en ausencia de TLS.

---

### Fase 2 — Hardening mediante SSH Tunneling (Escenario Seguro)

**Objetivo:** Proteger la confidencialidad de los datos en tránsito mediante un túnel SSH de reenvío de puerto local, sin alterar la configuración de la aplicación web.

**Paso 1:** Detener la captura activa en Wireshark y cambiar el filtro de visualización a:
```
ssh
```

**Paso 2:** Desde la terminal de Kali, establecer el túnel SSH con *Local Port Forwarding*:
```bash
ssh -L 8080:localhost:3000 usuario_ubuntu@192.168.1.115
```

| Flag / Parámetro | Descripción técnica |
|---|---|
| `-L` | Indica *Local Port Forwarding* |
| `8080` | Puerto local en Kali que recibirá las conexiones del navegador |
| `localhost:3000` | Destino final del tráfico, evaluado desde la perspectiva del servidor SSH remoto |
| `usuario_ubuntu@192.168.1.115` | Usuario y dirección IP del servidor Ubuntu que actúa como endpoint del túnel |

**Paso 3:** Con el túnel establecido y activo, acceder a Juice Shop a través del extremo local del túnel:
```
http://localhost:8080
```

**Paso 4:** Repetir el intento de inicio de sesión con las mismas credenciales ficticias utilizadas en la Fase 1.

**Paso 5:** Analizar el tráfico capturado en Wireshark.

**Resultado esperado:** Wireshark registra únicamente paquetes con protocolo SSH. El payload de los paquetes aparece completamente cifrado, sin ninguna información legible sobre las credenciales o la estructura de la solicitud HTTP subyacente.

---

## 📸 Evidencia Visual

### 🔴 Fase de Vulnerabilidad — Credenciales en Texto Plano

<img width="980" height="738" alt="image" src="https://github.com/user-attachments/assets/50c128a0-31ff-4f7a-ae44-1833e56e144f" />

> **Observación:** Wireshark con el filtro `http.request.method == "POST"` activo. En el panel de detalle inferior se aprecia el cuerpo de la solicitud HTTP, donde los campos `email` y `password` son completamente legibles en texto plano, confirmando la ausencia de cifrado en la capa de transporte.

---

<img width="980" height="678" alt="image" src="https://github.com/user-attachments/assets/0ead97ff-f50b-4ef3-9616-214cf4ab204f" />

> **Observación:** Vista del stream HTTP reconstruido por Wireshark mediante la opción *"Follow HTTP Stream"*. El payload JSON expone las credenciales de forma íntegra, tal como circularían por la red en un entorno de producción sin HTTPS habilitado.

---

### 🟢 Fase de Hardening — Tráfico Cifrado mediante Túnel SSH

<img width="950" height="223" alt="image" src="https://github.com/user-attachments/assets/da9fcdb9-36d0-44de-b4df-49363368aa02" />

> **Observación:** Terminal de Kali mostrando la ejecución exitosa del comando `ssh -L 8080:localhost:3000`. La sesión SSH queda establecida, indicando que el túnel de reenvío de puertos está activo y listo para encapsular el tráfico de la aplicación web.

---
<img width="980" height="713" alt="image" src="https://github.com/user-attachments/assets/d16be136-0916-4aa5-b220-5fcf5ab6d956" />

<img width="980" height="437" alt="image" src="https://github.com/user-attachments/assets/2fb699cd-c787-4e53-bc1f-1bad57ba19a5" />


> **Observación:** Wireshark con filtro `ssh` activo tras acceder a Juice Shop a través del túnel. Se evidencia que todo el tráfico generado por la sesión de login está encapsulado en paquetes SSH. El payload es completamente ilegible, confirmando que las credenciales viajan cifradas de extremo a extremo entre Kali y el servidor Ubuntu, neutralizando el vector de ataque por sniffing pasivo.

---

## 📊 Comparativa de Resultados

| Indicador | Sin Protección (HTTP directo) | Con Hardening (SSH Tunnel) |
|---|---|---|
| Protocolo visible en Wireshark | HTTP | SSH |
| Credenciales visibles en payload | ✅ Sí — texto plano (JSON) | ❌ No — datos cifrados |
| Riesgo de interceptación | **CRÍTICO** | **MITIGADO** |
| Modificación en la aplicación web | No aplica | No requerida |
| Costo de implementación | — | Bajo (comando SSH nativo) |

---

## 📚 Marco Conceptual

**HTTP vs. HTTPS/TLS:** El protocolo HTTP opera en la capa de aplicación sin ningún mecanismo de cifrado, exponiendo la totalidad del tráfico a cualquier nodo con capacidad de captura pasiva. HTTPS resuelve esta deficiencia añadiendo una capa TLS/SSL que garantiza confidencialidad e integridad mediante cifrado asimétrico para el intercambio de claves y simétrico para el cifrado del canal. En escenarios donde modificar la configuración del servidor no es viable operacionalmente, el SSH tunneling ofrece una alternativa pragmática de bajo costo.

**SSH Local Port Forwarding:** Mecanismo del protocolo SSH (RFC 4254) que permite redirigir un puerto local del cliente hacia un destino accesible desde el servidor SSH remoto. El tráfico que transita por este túnel está protegido por el algoritmo de cifrado negociado durante el handshake SSH (típicamente AES-256-CTR o ChaCha20-Poly1305), garantizando confidencialidad e integridad de los datos transmitidos.

**Sniffing pasivo:** Técnica de intercepción de paquetes en la que el atacante no interfiere con el flujo de comunicación, sino que captura y analiza los paquetes que circulan por la red de forma no intrusiva. Es especialmente efectivo contra protocolos sin cifrado (HTTP, Telnet, FTP) y en entornos de red compartida o redes Wi-Fi abiertas.

---
