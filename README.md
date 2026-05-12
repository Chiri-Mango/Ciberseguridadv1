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
