# Batería UD3 + UD4 · Explícalo con tus palabras

> No se entrega. Son preguntas como las de la defensa de la Práctica 2: si sabes contestarlas, vas bien.

1. ¿Qué pasa si quitas `ct state established,related accept` de forward? ¿Por qué no sale ningún error?

Qué pasa: Se cortan las conexiones activas o de respuesta (las respuestas de servicios o del exterior hacia la LAN no pueden volver).
Por qué no hay error: El analizador de sintaxis de nft comprueba que la regla esté bien escrita, no evalúa la lógica de red. Por eso la carga sin errores, pero fallará el tráfico en ejecución.

2. Tienes un `accept` de la LAN a la DMZ y un `drop` de la DMZ a la LAN. ¿Importa el orden entre ellos? ¿Y si los dos hablaran
   del mismo tráfico?

Tráficos distintos: El orden importa si las reglas se solapan, ya que nftables procesa secuencialmente.
Mismo tráfico: Manda la primera regla que haga match (first match), porque la evaluación se detiene en la primera coincidencia.


3. ¿Por qué con el DNAT no basta y hace falta también una regla en forward?

Porque el DNAT solo modifica la dirección IP de destino en prerouting, pero el paquete debe atravesar la cadena forward. Si forward no permite explícitamente ese tráfico, la política por defecto (drop) lo descartará.


4. En tu nmap, el 443 sale `closed` y el 8080 `filtered`. ¿Qué diferencia hay?

closed: El puerto es alcanzable y responde (normalmente con un paquete RST), pero no hay ningún servicio escuchando en él.
filtered: El puerto no responde porque un cortafuegos está bloqueando los paquetes antes de que lleguen al destino o impidiendo que la respuesta salga.

5. En el `wg0.conf` del remoto, ¿qué llave y qué candado hay? ¿De quién es cada uno?

Llave (PrivateKey): Es la clave privada del equipo remoto para firmar y descifrar su propio tráfico.
Candado (PublicKey en [Peer]): Es la clave pública del firewall, usada por el remoto para cifrar los paquetes de modo que solo el firewall pueda abrirlos.
