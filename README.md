# Modificación de Pitch haciendo uso de líneas de retardo
Una alternativa muy empleada para realizar la operación de `Pitch shifting` es usar líneas de retardo. 
Al retardar la secuencia $x(n)$ haciendo uso de una función lineal de la forma:

$$d(n) = m(n)\cdot n,$$ 

siendo $n$ la variable independiente o índice temporal discreto, $m(n)$ la pendiente de la función retardo (con $m(n)<1$) y $d(n)$ el retardo (que puede ser fraccionario) a aplicar a la secuencia, la salida obtenida

$$y(n)=x(n-d(n))$$ 

variará su frecuencia instantánea según la relación: 

$$f_i^y(n) = f_i^x(n)(1-m),$$ 

siendo $f_i^y(n)$ la frecuencia instantánea de la secuencia retardada y $f_i^x(n)$ la de la secuencia original.

Es decir, si la pendiente $m(n)$ de la función de retardo es cero, no se cambia la frecuencia instantánea, si $m(n)<0$ la frecuencia instantánea aumenta respecto de la de la secuencia original, y si $1> m(n) >0$ disminuye.

Para realizar un cambio de la frecuencia instantánea de una secuencia por un factor $k>0$, simplemente hay que usar una pendiente constante $m=1-k$. 

Sin embargo, usar una línea de retardo con pendiente lineal constante, además de modificar la frecuencia instantánea de la secuencia introduce los siguientes problemas:
* En caso de pendientes $m$ negativas, implicaría retardos positivos (tendríamos que buscar muestras en instantes posteriores al actual). Esto implica que el procesado no es causal y no podría implementarse en tiempo real.
* Como el retardo se incrementa o decrementa linealmente con $n$ no tendríamos un valor de los retados máximos y mínimos a considerar a no ser que conozcamos la duración de la secuencia. Por tanto, no resulta un método acotado en cuanto a la gestión de la memoria que debemos considerar ya que si se aplica a secuencias de larga duración, la memoria a considerar puede ser muy grande. Si se aplicara en procesado en tiempo real, deberíamos reservar un tamaño máximo de memoria asociado al retardo máximo, y este se podría ver desbordado alcanzado un determinado tiempo de procesado.
* En realidad, este procesado modifica también la duración de la secuencia. 

Todos estos inconvenientes se resuelven imponiendo un retardo máximo a considerar en la línea de retardo, de forma que al alcanzarlo, la línea de retardo vuelve a cero. Es decir, la función de retardo no sería una recta sino una función en diente de sierra.  

La variación del Pitch usando lineas de retardo se basa en esta idea, usando retardos máximos en torno a 30ms, donde las señales de audio se consideran más o menos estacionarias.

Tenga en cuenta que el usar una función de retardo en diente de sierra introduce discontinuidades en la pendiente ($m$) de la función lo que se traduce en saltos en el retardo a aplicar, que a su vez implica la aparición de artefactos auditivos. Estos problemas se resuelven usando dos líneas de retardo desfasadas que se activan de forma alternativa y que solapan adecuadamente las muestras retardadas por cada una de ellas en las transiciones donde aparecen las discontinuidades. 

Veamos paso a paso, como se crea este efecto de pitch shifting usando líneas de retardo.

Necesitamos procesar de forma particular las muestras correspondientes a las transiciones abruptas en la función del retardo $d(n)$.

Esto lo hacemos definiendo dos líneas de retardo desfasadas que se activan de forma alternativa excepto en el margen en el que se solapan donde ambas están activas.

Cada línea de retardo la asociaremos a un canal que tendrá una versión retardada a tramos (incompleta) de la secuencia a procesar, para posteriormente solapar ambos canales adecuadamente. Para ello aplicamos una ganancia complementaria a cada canal (por ejemplo, usando curvas de igual potencia)

Por ejemplo, definimos un par de líneas de retardo con un cierto solape temporal entre ellas (en este caso del 10%) y la misma pendiente y retardo máximo en ambas, y calculamos las ganancias a aplicar a cada canal de manera que la suma de ambos canales conserve la potencia de la señal original:

![image](https://github.com/user-attachments/assets/8e04aea4-cf0b-4761-9507-16a825478939)
![image](https://github.com/user-attachments/assets/ac519b21-7b54-4009-b651-4b1871da3311).

Aplicamos estas líneas de retardo a la señal tonal de antes:

![image](https://github.com/user-attachments/assets/0f804eac-64a9-4a2f-be7a-c64d178449c5)
![image](https://github.com/user-attachments/assets/7eeca2f2-ae43-456f-8cfc-2082c1ab42a4)
![image](https://github.com/user-attachments/assets/67cd7dfb-c50b-4182-a939-34e6bc90dd7b)

Puesto que el % del solape y el retardo máximo han sido fijados sin tener en cuenta la naturaleza de la secuencia a procesar, la secuencia procesada no es en este caso un tono perfecto pero las imperfecciones que aparecen no se aprecian auditivamente hablando tanto como las discontinuidades anteriores provocadas por los saltos de fase de una única línea de retardo

Ahora si podemos escuchar la señal con el cambio de Pitch deseado y sin artefactos auditivos. La función `pitchshifting_tr` integra todo este procesado permitiendo ejecutarlo en tiempo real. Además, permite modificar durante la ejecución la relación de cambio de Pitch variando el valor de $k$ entre los límites $$-1 \leq k \leq 0.5 $$ (es decir permite modificar el Pitch entre la octava superior y la inferior y lo puede hacer en pasos de medio semitono).



