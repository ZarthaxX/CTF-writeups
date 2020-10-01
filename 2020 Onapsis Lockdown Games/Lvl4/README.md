# Writeup Challenge2 Onapsis Lockdown Games
>Author: Tomàs Tropea

# Analisis & Resoluciòn

Tenemos un solo archivo, que es un binario llamado *MazeRuner*. Al ejecutarlo nos presenta un juego de un laberinto, donde podemos movernos hacia arriba, abajo, izquierda o derecha. El objetivo es escapar del laberinto, pero la dificultad que presenta es que no podemos ver nada, como se puede ver a continuacion:

![alt text][ejecucion]

Una idea seria ver como podemos hacer para ver el laberinto, y asi hacer mas facil ganarlo. Entonces, veamos el binario del juego en busca de algo interesante:

![alt text][asm-idea]

Esta parte del codigo asm es el comienzo del main, donde marque las partes relevantes para la resolucion. Con flechas negras marque el camino que queremos lograr. Ese camino termina ejecutando la instruccion:

```nasm
mov cs:_SYS_DEV_SIGN, 1
```
Al setear este valor podremos ver el laberinto desde el comienzo del juego y ganarlo facilmente, aunque falta un detalle que veremos al final.
Entonces, para lograr esto, vemos primero lo que esta marcado con rojo, el primer salto. Este depende del contenido que se encuentra en **[rbp+var_24]**, que si vemos al principio de todo vemos que se le asigna a esta posicion:

```nasm
mov [rbp+var_24], edi
```
En este caso edi tiene el valor de argc, entonces nos interesa pasarle algun argumento al programa, para que argc > 1, y asi pasar este primer jump.

El segundo salto, marcado en verde, compara 2 strings para decidir a donde saltar. En nuestro caso, nos interesa saltar si la comparacion es cero, es decir, si ambos strings son iguales. Lo que esta pasando en esta parte es que compara el primer argumento que le enviamos al programa, con el que se encuentra en *cs:OPT*, que tiene el valor "-Debugg". Entonces, basta con llamar al programa de la siguiente forma para lograr el salto deseado:
> ./MazeRuner -Debugg

Falta el ultimo salto, marcado con violeta. En este se compara el byte de la posicion mas baja en *eax* con el valor *0x54*, que se corresponde con el caracter 'T'. El valor de eax dependerà de lo que haya en *[rbp+dest], que si observamos el principio del main, vemos lo siguiente:

```nasm
mov rdx, cs:_STR_DEV
lea rax, [rbp+dest]
mov rsi, rdx        ; src
mov rdi, rax        ; dest
call _strcpy
```
Se esta copiando en la posicion de memoria que nos interesa el string que se encuentra en *cs:_STR_DEV*, el cual es "FALSE". Entonces lo que podemos hacer simplemente, es editar el string en el binario, con algun hex editor, y cambiamos el string FALSE por TALSE, para que la comparaciòn con el caracter 'T' sea verdadera.

Con esto, logramos visualizar el laberinto desde el comienzo:

![alt text][maze-solved1]

El unico problema con esto, es que si intentamos llegar al final, no nos darà la flag como queremos. Veamos el motivo de esto con el codigo que imprime la flag:

![alt text][asm-idea2]

Lo que pasa con esto, es que nos imprime el string "<place_info_in_product_version>", porque la comparaciòn que esta marcada en rojo no dio cero. Previamente habiamos hecho que *cs:_SYS_DEV_SIGN* valiera 1, para poder ver el laberinto, pero ahora no nos permite obtener la flag. Una posible solucion a esto, es aplicar un parche al binario, y modificar el salto para que haga lo contrario, es decir, cambiamos:
```nasm
jz short loc_4012AE
```
por
```nasm
jnz short loc_4012AE
```
Con esto logramos que se imprima la flag al final, la cual termina siendo:

>ONA{1_tr0pez0n_!=_ca1d4?}

[ejecucion]: ejecucion.png
[asm-idea]: asm-idea.png
[maze-solved1]: maze-solved1.png
[asm-idea2]: asm-idea2.png
