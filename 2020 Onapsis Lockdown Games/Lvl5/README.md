# Writeup Challenge2 Onapsis Lockdown Games
>Author: Tomàs Tropea

# Analisis & Resoluciòn

Tenemos un archivo binario a disposiciòn, llamado sudoku. Lo ejecutamos para ver de que se trata, y vemos que es una version del sudoku pero de 16x16:

![alt text][ejecucion]

Una posible forma de resolverlo que pensè fue la de jugarlo y ganarlo normalmente. Para esto, utilice un solver de sudoku de 16x16 en esta pagina https://www.dcode.fr/hexadoku-sudoku-16-solver.

Al ganar el sudoku, obtuve una parte de la flag, y luego complete el segundo nivel de manera analoga, y obtuve el resto de la flag:

> ONA{sUd0_5uDO_5Ud0KUuu!!!}

[ejecucion]: ejecucion.png
