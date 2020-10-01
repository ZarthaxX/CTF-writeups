# Writeup Challenge2 Onapsis Lockdown Games
>Author: Tomàs Tropea

# Analisis & Resoluciòn

Tenemos un unico archivo, que se llama *Minesweeper.jar*. Intentamos ejecutarlo y aparece un juego, que resulta ser el buscaminas:

![alt text][ejecucion-jar]


Si bien considere la opciòn de ganarlo "limpiamente", intente hacer un poco de trampa. Primero utilice la herramienta *jd-gui* para examinar el codigo del *Minesweeper.jar*. Para esto, corrì el comando:

> java -jar jd-gui-1.6.6-min.jar MineSweeper.jar

Luego de examinar un poco el codigo del juego, me concentre en buscar la parte que dibujara las minas del juego. Logre encontrar la siguiente funciòn en el *Board.class*:

![alt text][jd-gui]

Lo importante de esta funciòn, es que es ejecutada al comienzo del programa. Entonces podemos tratar de modificarla para que dibuje las minas al comienzo del juego, y asi ganar facilmente. La imagen de las minas esta asociado con el numero 9 dentro del arreglo *this.img[i]*, que es el que se utiliza para dibujar las distintas celdas. Si vemos el codigo, ese numero solo puede ser seleccionado una vez que la variable *this.inGame* es falsa, es decir, cuando haya finalizado el juego. Entonces, primero que todo vamos a querer modificar el codigo para poder pasar ese chequeo. La segunda cosa que hay que hacer es saltar, en el caso de caer en el ultimo caso donde se le asigna a la variable i el valor 10, a la linea que incrementa b1. Esto ultimo hay que hacerlo para mantener la logica que tenia el codigo previo a modificar lo primero que se mencionò. Entonces, estas dos modificaciones se pueden visualizar a continuacion:

![alt text][jd-gui-modif]

La flecha roja representa la primera, y la azul la segunda modificacion. Para hacer estas modificaciones, primero tenemos que guardar con el *jd-gui* el source file *Board.class* para poder modificarlo. Luego, basta con reemplazar el byte code de esta funciòn dentro del  *Board.class*, utilizando un editor de java byte code, por ejemplo *JBE*, que se corre asi:
>java -cp bin ee.ioc.cs.jbe.browser.BrowserApplication

Vamos a modificar entonces las partes indicadas por las *-->* como vemos aqui:

```java
iconst_0	
istore_2	
iconst_0	
istore_3	
goto 85	
iconst_0	
istore 4	
goto 81	
aload_0	
getfield game/Board/field [I	
iload_3	
bipush 23	
imul	
iload 4	
iadd	
iaload	
istore 5	
aload_0	                     --> nop
getfield game/Board/inGame Z --> goto 30	
ifeq 27	
iload 5	
bipush 9	
if_icmpne 27	
aload_0	
iconst_0	
putfield game/Board/inGame Z	
aload_0	
getfield game/Board/inGame Z	
ifne 54	
iload 5	
bipush 19	
if_icmpne 36	
bipush 9	
istore 5	
goto 66	
iload 5	
bipush 29	
if_icmpne 42	
bipush 11	
istore 5	
goto 66	
iload 5	
bipush 19	
if_icmple 48	
bipush 12	
istore 5	
goto 66	
iload 5	
bipush 9	
if_icmple 66	
bipush 10	
istore 5	
goto 66	                     -->  goto 65
iload 5	
bipush 19	
if_icmple 60	
bipush 11	
istore 5	
goto 66	
iload 5	
bipush 9	
if_icmple 66	
bipush 10	
istore 5	
iinc 2 1 <--- linea 65: b1++	
aload_1

```

Con las primeras 2 lineas modificadas, logramos omitir el primer chequeo. Luego, con la tercer linea modificada logramos saltar a *b1++*, que esta indicado mas abajo, como la linea 65. 
Una vez hechas estas modificaciones, las incorporamos al *MineSweeper.jar*, con esta linea:
> jar uf MineSweeper.jar Board.class

Finalmente iniciamos el juego y podemos jugar viendo todas las minas desde el inicio:

![alt text][ejecucion-modif-jar]

Basta ahora con tocar todas las celdas que no sean minas para obtener el flag:

![alt text][flag]

La ventana que aparece nos dice la flag:

>ONA{m1n3$...theR3_@re_MiNe5_Ev3ryw4ere!}

[ejecucion-jar]: ejecucion.png
[jd-gui]: jd-gui.png
[jd-gui-modif]: jd-gui-modif.png
[ejecucion-modif-jar]: ejecucion-modif.png
[flag]: flag.png
