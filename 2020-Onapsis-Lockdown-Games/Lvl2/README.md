# Writeup Challenge2 Onapsis Lockdown Games
>Author: Tomàs Tropea

# Analisis & Resoluciòn

En este challenge tenemos un archivo llamado *knocking.py* y otro *.music*. Primero que todo, ejecutamos el .py y observamos que nos muestra lo siguiente:

![alt text][ejecucion1-py]

A su vez, podemos observar que generò a traves del archivo *.music*, una canciòn llamada *.KnockinOnHeavensDoor.mp3*. Dejamos en pausa por un segundo la ejecuciòn de este programa, y analizamos si hay algo interesante en el nuevo archivo generado. Para esto vamos a revisar la metadata utilizando *exiftool*, con el comando *exiftool .KnockinOnHeavensDoor.mp3*. Con esto podemos ver algunas partes de la metadata relevantes:

![alt text][cancion-mp3]

Dentro de los campos que aparecen, uno interesante es el **Comment**. Parece estar codificado en base64, entonces lo podemos intentar decodificar y asi obtenemos:

```
 Hello Guys, here are the rules for the game. Remember to place them where everyone can see them... Also, i send you the flag, please hidde it in a safe place, not like last time... Flags are not supposed to be hidden in secret tags!

RULES:

1) To run the game, use:

"Python2 Knockin.py"

2) When the game is ready, press Enter to start the song.

3) Hit INTRO (new line) each time you hear the word "Knock" or "Knocking" in the chorus. Dont hit INTRO ("knock") in the rest of the song, just the chorus. Example:
   
 INTRO  INTRO  INTRO
"Knock, knock, knockin' on heaven's door"

4) Dont knock to soon or too late, follow the song or you will fail to open heavens doors!

Good Luck...

```

Esto nos esta diciendo como jugar al juego para poder ganarlo. Despues de haber leido esto intente jugar siguiendo los pasos y fallando multiples veces. Finalmente pude seguir el ritmo y asi ganar el juego, obteniendo la flag:

> ONA{Kn0(kin_IntR0_He4veN}

Nota: La metadata que mostraba la herramienta exiftool no era toda la que habia presente. Si se utilizaba otra herramienta se podia ver que habia otro campo llamado *FLAG*, el cual tenia la flag codificada en base64. Era un camino mas directo, para no estar 5 minutos (o potencialmente horas) siguiendo el ritmo de la canciòn. :D











[ejecucion1-py]: ejecucion1.png
[cancion-mp3]: cancion.png
