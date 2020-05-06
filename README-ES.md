# **Sietch**

## **¿Qué es Sietch?**

Sietch es el nombre código para el nuevo conjunto de funcionalidades RPC y reglas de consenso relacionadas con el uso de fondos blindados. La creación de transacciones blindadas es modificada por encima del nivel del Zcash Protocolo mismo, ningún cambio en esa capa fueron realizados. Adicionalmente, no hubo cambios en como la capa RPC es utilizada, esto es, Sietch es implementado en los hushd internos y software existente que utilizan `hush-cli` y/o la interfaz RPC no necesita ser cambiada en ninguna manera. Esto preserva la compatibilidad con todo el software existente del ecosistema Zcash Protocolo a un nivel muy bajo.

Sietch define ciertas condiciones de cartera y transacción que deben ser confirmadas para que una transacción sea aceptada por la red de trabajo así como hacer que todos los clientes tengan un nuevo comportamiento por defecto que es más preservador de la privacidad. Esto incluye ambos, comportamiento de nueva cartera que es no-consenso (actualmente implementado) y también reglas de consenso futuro para hacer cumplir este comportamiento por defecto.

Sietch no modifica nada acerca de los internos de Zcash como zk-SNARKs, criptográficos primitivos o cambiar algo de la profunda matemática conocimiento cero involucrada, como los parámetros de Curvas Elípticas, tipos de datos u otra maquinaria criptográfica.

Cualquier moneda del Zcash Protocol puede utilizar esta “actualización” hacia el Hush Protocolo Sietch-habilitado, aunque debería ser notado que Hush fue la primer moneda puramente Sapling del Zcash Protocol y por lo tanto nuestra implementación ignora las zaddrs Sprout. Esto es porque agradecidamente no tenemos zaddrs o transacciones Sprout en nuestra red principal V3. Sietch es compatible con antiguas zaddrs sprout y darle soporte quedaría solamente como un ejercicio para la moneda de privacidad interesada.

## **¿Cómo es implementado Sietch?**

Sietch está escrito en C++ y dentro del código de nodo completo hushd, lo cual significa que todo el código existente que interactúa con Hush puede beneficiarse sin cambios. Algunas características opcionales/avanzadas en el futuro podrían ser implementadas al nivel de billetera GUI. La meta es que el usuario no necesite pensar o hacer algo diferente para tener más privacidad cuando haga transacciones.

Ya que Sietch trabaja al nivel de hushd, todo el código existente, así como billeteras GUI, pools de minado e intercambios, no requieren de cambios de código. Ellos pueden hacer transacciones blindadas como siempre lo han hecho, a través de los mismos métodos y parámetros RPC.

Desde la perspectiva del usuario promedio de billetera GUI, Esto Simplemente Funciona. No hay pasos adicionales obligatorios cuando envíen cada transacción. No mantenimiento obligatorio de billetera será necesario para mantener su privacidad. No opciones confusas para “estar correctos” en cada transacción. Lo mismo será con los intercambios, pozos de minado y todo usuario de Hush. Ningún cambio será requerido por el usuario final para habilitar Sietch, otro más que actualizar al software Hush compatible.

Podría haber algunas opciones para usuarios avanzados, para ajustar las configuraciones, pero usarlas no serán requeridas para incrementar la privacidad. Hush está dedicado a que todas las futuras características de privacidad estén ENCENDIDAS por defecto en lugar de preguntar a nuestros usuarios por optar-por privacidad, que como sabemos es un gran error.

Los usuarios no tendrán que optar-por para Sietch y no habrá manera de apagar esta funcionalidad. Adicionalmente, Sietch será habilitada en la red principal como característica optar-por, al principio, como primer ola de adopción y para prueba. Cuando todos los usuarios hayan actualizado al software Hush Protocolo con Sietch-habilitado, una nueva regla de consenso entrará en efecto, la cual prevendrá viejos clientes no-Sietch.

Adicionalmente, Hush podría incrementar su PROTOCOL_VERSION con la característica Sietch, la cual permitiría a los nodos Hush Sietch-habilitado bloquear eficazmente potenciales clientes maliciosos hushd no-Sietch a nivel igual-a-igual.

## **¿Cúales son las metas de Sietch?**

* Hacer los análisis de enlazabilidad de las transacciones totalmente blindadas (z2z) drásticamente más caras.
* Permitir a la gente realizar x transacciones zaddrs con seguridad/privacidad sin tener que pensar en filtrado de metadatos o técnicas avanzadas.
* Prevenir que el usuario promedio realice algunas operaciones blockchain que puedan repartir demasiados metadatos.
* Romper con los supuestos en software analistas de blockchain
* Analistas de blockchain requieran escribir nuevo software
* Incrementar el costo computacional para el análisis de la actual blockchain
* Incrementar la privacidad de todos los fondos en el pozo Hush blindado.
* Introducir no-determinismo para contrarrestar el ataque estilo ITM/Metaverse a los metadatos.

## **¿Cuáles son las limitaciones/desventajas de Sietch?**

La meta de Sietch es incrementar la privacidad a costa de incrementar el uso de espacio-bloque por transacción, incrementar el tiempo de CPU y RAM para validar transacciones y como efecto secundario, incrementar tiempos de sincronización.

Sietch potencialmente agrega entradas y salidas a las transacciones para incrementar su privacidad, y como el número máximo actual de receptores es bajo, cuando se habilite Sietch en la práctica, las transacciones pueden ser enviadas sobre 1000 receptores incluso con las protecciones Sietch, así que esto no afecta el uso normal.

¿Qué tan largo puede ser el promedio de zxtn? Investiga: ¿Cuántas zouts mientras se esté bajo <10s?

Potencialmente más comisiones por transacción si la transacción se vuelve muy grande para requerir más de las comisiones por defecto.

No se incrementa el uso RAM, ya que crear JoinSplits es un proceso serial, pero las transacciones blindadas pueden tomar más segundos CPU, ya que habría más receptores por default.

## **Detalles de Implementación**

Estos RPCs son modificados directamente:
```
z_sendmany
```
Estos RPCs actualmente no son modificados, pero podrían serlo en un futuro:
```
z_shieldcoinbase
z_mergetoaddress
```
Estos RPCs interactúan con zaddr xtns y pueden reportar diferente o adicional información después de estos cambios:
```
z_viewtransaction
z_listtransactions
listunspent
```
Estos nuevos RPCs fueron añadidos en Sietch:
```
z_listnullifiers
```
## **z_sendmany Regla de Siete**

Nuestra meta es ser no-determinista mientras también se insertan suficientes zouts de tal manera que tengamos al menos N=7 zouts.

Ya que el caso normal es tener 2 zouts (un receotor para z2z y una zout para el cambio de regreso a la dirección emisora), el caso normal podría ser añadir 5 zouts a las z2z xtn normales.

* Para z=>t, debemos añadir 7 zouts
* Para t=>z, debemos añadir 6 zouts
*	Para t=>(z, z), debemos añadir 5 zouts
*	Para z=>z, debemos añadir 6 zouts

La razón por la cual N=7 fue elegido es por el simple hecho de `6!=720` mientras que `7!=5040`. Este parámetro fue elegido en respuesta al ataque ITM, el cual depende de un número pequeño de zouts y hace un algoritmo combinacional en todas las posibilidades. Estos algoritmos combinacionales incrementan en espacio de estado para cada enlace en una larga cadena de transacciones. Tradicionalmente cada xtn solo tiene unas pocas zouts y como `2!=2` y `3!=6`, la explosión combinacional no tiene oportunidad de ralentizar el ataque ITM.

Ahora, considerando 5040 elecciones para cada enlace en una cadena, y digamos que estamos estudiando una cadena de longitud L=10. Comparando con pre-Sietch, podríamos tener `2^10=1024` posibilidades contra
```
5040^10 = 10575608481180064985917685760000000000
```
posibilidades. Incluso para cadenas cortas de xtns, la explosión combinacional de posibilidades vuelve el ataque ITM extremadamente caro. Esto limita a la búsqueda por enlazabilidad de metadatos colo en cadenas cortas con mucho soporte de metadatos adicionales, elevando efectivamente la barra para los ataques y removiendo muchos atacantes potenciales que no tienen los suficientes recursos. En código pre-Sietch, el ataque ITM podría potencialmente investigar cadenas muy largas, docenas, cientos y tal vez miles de transacciones en longitud, con un hardware de mercadería. Sietch incrementa exponencialmente el costo de hacer esto, en RAM, CPU y tiempo-corriendo. Un atacante necesitaría botnet y recursos de nivel-supercomputadora para atacar la misma longitud de cadena de código pre-Sietch, o más como, atacantes de-anonimizadores se enfocarían en estudiar cadenas de corta enlazabilidad y con muchos adicionales metadatos para análisis de sincronización, análisis de cantidad y potencialmente pasivos o activos ataques de polvo.

## **No-determinismo**

La explosión combinacional sola puede protegernos mucho. Pero eso solo es una capa de defensa y seguramente no nos salvará de las inevitables computadoras cuánticas las cuales están siendo optimizadas cada día.

Cuando se agregan zutxos, para romper los ataques de metadatos ITM/Metaverse a nivel profundo, debemos romper la profunda suposición que está arraigada profundamente dentro del comportamiento de la billetera Bitcoin: determinismo.

Dado exactamente la misma wallet.dat, Bitcoin, Zcash y Hush podrían actuar en exactamente la misma manera cuando envían transacciones, cada vez. Dados todos los dato de entrada, uno puede predecir el comportamiento de lo que sucederá. Obviamente, esto es una muy buena idea en Bitcoin, para el suministro total y responsabilidad de emisión. Pero esto también contribuye a que Bitcoin se convierta en una “moneda de vigilancia”, que perfectamente conserva los metadatos hasta el final de los tiempos.

Zcash no tiene razón para cambiar esto y no lo han hecho, así que este comportamiento está arraigado profundamente en todas las bifurcaciones de Bitcoin y Zcash. A la luz de los ataques ITM/Metaverse, este determinismo es considerado peligroso por el autor. La razón es que el ataque ITM/Metaverse utiliza el hecho de que las operaciones de la billetera son predecibles, para extraer más metadatos de lo que se pensaba posible anteriormente en las xtns z2z y t=>z. La única manera de prevenir esto es romper la suposición de un comportamiento predecible de la billetera.

## **Desventajas/Problemas/Ataques**

El código que usa puramente transacciones crudas y las difunde a la red de trabajo necesita realizar trabajo extra para soportar Sietch. Esto es por lo que SDL tiene su propia implementación de Sietch, y cualquier tipo de billetera hardware que utiliza transacciones crudas, en el futuro podría necesitar aprender acerca de Sietch.

## **Ataques de Metadatos Contra Sietch**

Si, Sietch en sí misma puede ser atacada en los metadatos! TLDR: El peor de los casos es que un atacante robe el archivo wallet.dat y realice una inmensa cantidad de trabajo para reducir la privacidad a niveles pre-Sietch, usando el ataque ITM.

Al principio había una sola implementación Sietch de 200 zaddrs que fueron fijas. Si la wallet.dat que posee esas zaddrs fuera robada, podría ser usada para borrar toda la “privacidad polvo” Sietch desde el gráfico de transacción, haciendo el trabajo de analistas de blockchain y/o atacantes ITM más sencillo. Este ataque teórico fue lo que estimuló las direcciones dinámicas Sietch y también una mejor manera de generar zaddrs dentro de SDL: usando frase-semilla BIP39 para generar una simple zaddr y después borrar la frase-semilla. Este método no deja wallet.dat en el dico para ser robada y el material de llave privada para la zaddr solo existe en la memoria de corto tiempo.

Actualmente en producción hay 200 zaddrs estáticas en `hushd` y 10,000 zaddrs estáticas (BIP39-derivadas) en `SDL`. El código dinámico Sietch zaddr para `hushd` está completo y puede ser visto en: https://github.com/MyHush/hush3/tree/sietch_dynamic Está siendo probado en su desempeño ya que realiza algunas cosas exóticas.

No hay wallet.dat que se pueda robar para recuperar datos acerca de Sietch zoutputs para 10,000 de las 10,200 zaddrs actualmente en el pozo Sietch zaddr, así que este ataque ya no es viable. Las Sietch zaddrs dinámicas podrían hacer el proceso entero mucho más seguro previniendo que analistas/atacantes incluso de saber las zaddrs que podrían ser potencialmente una producción de Sietch. Estas Sietch zaddrs dinámicas podrían ser generadas en el tiempo-de-ejecución y las claves privadas incluso nunca ser escritas en el disco, ni ser parte de la `hdseed` de ninguna wallet.dat en el caso de `SDL`.

## **Conclusiones**

El Protocolo Hush Sietch-habilitado puede ser pensado como el uso de las ideas de explosión combinacional y no-determinismo para frustrar flamantes técnicas de análisis de blockchain. No-determinismo es el arma más fuerte, pero no añade suficiente privacidad a menos que se adicione la apropiada cantidad de explosión combinacional para el análisis de enlazabilidad. Juntas son una potente arma que también pueden darnos el mando para recurrir a un incremento futuro de seguridad, como el número mínimo de salidas zaddr permitidas en una transacción.

Por esto es por lo que ambas técnicas se complementan una a la otra y tienen una grandiosa mejora a la privacidad cuando se utilizan juntas

## **Implementaciones Actuales**

Actualmente hay 4(!) implementaciones de Sietch en el mundo Hush, 2 dentro de los interiores de `hushd` para `SilentDragonLite` que usa transacciones crudas y no la interfaz RPC de `z_sendmany`. Cada una de estas 2 implementaciones tiene su versión estática (pasando por un pozo de zaddrs Sietch) y dinámica (zaddrs Sietch generadas dinámicamente en el tiempo-de-ejecución). Actualmente las implementaciones estáticas se encuentran en producción a partir de `Hush 3.3.0` y `SilentDragonLite 1.1.3` y las versiones dinámicas están casi completas y sometidas pruebas de desempeño.
