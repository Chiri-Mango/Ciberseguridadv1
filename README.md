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
