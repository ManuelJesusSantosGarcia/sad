# Respuestas · Batería UD5 + UD6

> Esto es para ti: no se entrega ni cuenta. Pero si sabes contestar aquí, en la defensa de la Práctica 2 vas sobrado.

## UD5 · IDS/IPS y WAF

1. Un IDS detecta un ataque. ¿Lo para? ¿Por qué? ¿Qué tendrías que cambiar para que lo cortara?

¿Lo para? No. Un IDS (Intrusion Detection System) es puramente pasivo: analiza el tráfico, detecta patrones maliciosos y genera una alerta, pero no interrumpe ni bloquea las conexiones.

¿Por qué? Su diseño está pensado para monitorizar y registrar eventos sin alterar el flujo normal de la red, evitando cortes accidentales (falsos positivos) que tumben servicios legítimos.

¿Qué tendrías que cambiar? Tendrías que pasar de un IDS a un IPS (Intrusion Prevention System), o configurar el sistema en modo activo/inline para que, al detectar la firma del ataque, actúe bloqueando el paquete (por ejemplo, mediante reglas de iptables/nftables o integrándolo con Fail2ban).

2. En tu regla de la inyección SQL pusiste el destino `any` y no `$HOME_NET`. ¿Por qué? (pista: el DNAT del firewall).

Motivo: Debido a que el servidor web está protegido detrás de un mecanismo de redirección de puertos o DNAT (Destination Network Address Translation) en el firewall perimetral.

Cuando las peticiones de los atacantes llegan desde el exterior (Internet), el firewall traduce la IP pública de destino a la IP privada de la red interna donde reside el servidor web. Si hubieras limitado el destino estrictamente a $HOME_NET antes de que se aplique el DNAT, las reglas de inspección de paquetes (como las de Snort) podrían no coincidir correctamente con el direccionamiento esperado o ignorar el tráfico entrante antes de la traducción, por lo que usar any garantiza que se capturen las peticiones dirigidas al servicio web independientemente de la fase de traducción de direcciones.

3. El WAF está en `DetectionOnly` y llega `?id=1' OR '1'='1`. ¿Qué código recibe el atacante? ¿Dónde queda apuntado?

Código que recibe el atacante: Recibe un código HTTP 200 OK (o el código normal que devuelva la aplicación web ante esa consulta), ya que el WAF está en modo DetectionOnly (solo detección) y no bloquea la petición; simplemente permite que llegue al servidor backend y este la procesa.

Dónde queda apuntado: Queda registrado en los ficheros de log del WAF (por ejemplo, en ModSecurity bajo /var/log/modsec_audit.log o el syslog configurado), detallando la regla de la OWASP Core Rule Set (CRS) que ha coincidido con el ataque de inyección SQL.

4. El WAF bloquea el comentario de una clienta. ¿Qué nombre tiene ese problema? ¿Cómo lo arreglaste **sin** abrir la
   puerta a toda la web?

Nombre del problema: Falso positivo (False Positive).

Cómo se arregló: En lugar de desactivar todo el módulo de seguridad o relajar las reglas generales de la aplicación, se localizó en los logs del WAF el ID exacto de la regla (Rule ID) que había saltado por error ante un texto legítimo de la clienta. Posteriormente, se aplicó una excepción específica (mediante una directiva de exclusión como SecRuleRemoveById o una exclusión por variable/argumento en ModSecurity) únicamente para esa regla en la ruta o parámetro concreto del formulario de comentarios, manteniendo el resto de la protección intacta.

## UD6 · Bastionado

5. ¿Por qué `PermitRootLogin no`, si root es el administrador? ¿Cómo entra entonces el administrador?

Por qué se deshabilita: Porque el usuario root es un objetivo primario y universal para ataques de fuerza bruta en SSH desde Internet. Si un atacante adivina o revienta la contraseña de root, obtiene control absoluto e inmediato de la máquina.

Cómo entra el administrador: El administrador accede mediante su propio usuario sin privilegios (por ejemplo, admin o su nombre de usuario personal) y, una vez dentro de la sesión SSH, escala privilegios de forma segura ejecutando comandos con sudo -i o su -, o bien utilizando autenticación por clave pública SSH (deshabilitando contraseñas).

6. Explica, con tus palabras y paso a paso, cómo Fail2ban deja fuera a una IP que prueba contraseñas.

Monitoreo de logs: Fail2ban lee continuamente los archivos de registro del sistema (como /var/log/auth.log en servicios SSH).

Detección de patrones: El demonio compara las líneas nuevas con expresiones regulares (regex) diseñadas para identificar fallos de autenticación (por ejemplo, "Failed password for invalid user...").

Conteo de intentos: Si una misma dirección IP acumula un número de fallos superior al límite configurado (maxretry = 3) dentro de la ventana de tiempo establecida (findtime = 10m), se activa la alerta.

Aplicación del baneo: Fail2ban ejecuta la acción configurada (banaction = nftables-multiport), interactuando directamente con el cortafuegos (nftables) para añadir dinámicamente una regla que bloquee todo el tráfico proveniente de esa IP.

Liberación temporal: Tras transcurrir el tiempo de castigo (bantime = 1h), Fail2ban retira la regla del cortafuegos y la IP recupera el acceso (a menos que vuelva a fallar)

7. Lynis te daba 58 y ahora da 63. ¿Por qué no llega a 100? ¿Significa que el servidor sigue inseguro?

Por qué no llega a 100: Lynis es una herramienta de auditoría muy estricta que evalúa cientos de parámetros de buenas prácticas (políticas de contraseñas corporativas complejas, parches de kernels específicos, herramientas de borrado seguro, cifrados muy estrictos o configuraciones de correo y DNS que quizás ni se usan en tu máquina). Alcanzar un 100 es prácticamente imposible en entornos reales de producción porque muchas advertencias dependen del contexto de negocio o de limitaciones del sistema operativo.

¿Significa que sigue inseguro? No necesariamente. Un incremento en la puntuación indica que el proceso de endurecimiento (hardening) ha mejorado notablemente el nivel de seguridad. Las advertencias restantes (warnings o suggestions) deben revisarse una a una para determinar si aplican a tu infraestructura o si se trata de elementos asumidos y controlados.
