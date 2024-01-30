## Introducción general:

- Importancia de la visión por computador en ramas como la astrofísica: observación de elementos celestes, agujeros negros,...

- Sistemas y algoritmos actuales muy limitados: aparición de ruido y no se captan otros elementos (¿materia oscura, ....?)

- Se han propuesto algoritmos novedosos para captar lo invisible. Reconstrucciones de imagen y vídeo a partir de lo captado por telescopios gigantes.

- Relación inversa entre resolución angular (buena resolución) y diámetro de los telescopios. Caso del agujero central de la Vía Láctea: emisión predicha de 2.5*10^(-10) radianes, quiero captar 10^(-10) radianes, con longitud de onda 1.3mm ---> necesito un telescopio de 13000 km de diámetro (buscar fórmula e investigar para una longitud de onda fija).

- De las muchas técnicas que se pueden proponer, nos centraremos en la reconstrucción de imágenes con datos obtenidos por VLBI (soluciona el problema anterior).

- VLBI: Interferometría de línea de base muy larga. Usar muchos telescopios pequeños, dispuestos alrededor de la Tierra, para conseguir imágenes de baja resolución, reunirlas, y reconstruirlas. Una especie de "divide y vencerás". Poner imagen de la Tierra con varias líneas unidas, simulando un VLBI.

- Problema actual del VLBI: Faltan muchos datos, hay que hacer muchas suposiciones. Actualmente, la demanda de alta resolución y de nuevas formas de desafiar a la relatividad general hace que esto empiece a no ser viable (no fiable).

- Más problemas sobre la resolución: Si quiero mejorar la resolución, o separo dos telescopios del VLBI o bajo la longitud de onda. Si bajo longitud de onda(?????) y si separo telescopios, el tamaño de la Tierra limitado y muy costoso.

- Aplicación específica en el EHT (Event Horizon Telescope) para agujeros negros, pero podría usarse en otros proyectos como SKAO.

## Intro al VLBI

- Soluciones viables a los problemas de los algoritmos actuales para reconstruir imágenes con VLBI--->diseñar algoritmos más robustos y fiables a la disposición de pocos datos. Se propone CHIRP (Continuous High-Resolution Image Reconstruction using Path Priors).

- Idea: Parte de que el problema "reconstruir imágenes haciendo muchas suposiciones con pocos datos" está mal planteado. Lidia con el ruido atmosférico.

- Clave en el VLBI la distancia, B, entre telescopios. Llegada tardía de la señal electromagnética. Teorema de Cittert-Zernike (relaciona la función de correlación espacial en un campo de ondas electromagnéticas con la transformada de Fourier de la distribución de la intensidad). Si aplicamos la inversa de esta transformada, se podría reconstruir la imagen de la fuente.

- Debido a la rotación terrestre, se usan los cierres de fase (permite recuperar información útil perdidad por estas alteraciones) para reconstruir bien las imágenes.

- Muchas fórmulas y términos de óptica y física que pueden no venir a cuento (aunque si veo que se usan en CHIRP, habrá que mencionarlos).

## CHIRP

- Algoritmos importantes antes a destacar:
    1) **CLEAN**: Para reconstruir imágenes. El proceso sería algo como: suponemos imagen con un número de fuentes brillantes--> iterativamente, localizo localmente el punto más brillante y deconvoluciono su vecindario para eliminar ruido-->finalmente se desenfoca la imagen para obtener reconstrucción realista. PROBLEMA: ¿Un agujero negro tiene fuentes brillantes?

    2) **Interferometría óptica**: *Algoritmo BSMEM*, que usa aproximación bayesiana y gradiente descendente con suposición de entropía máxima. PROBLEMA: La imagen de un planeta puede ser simple y uniforme. ¿La Nebulosa de Orión sigue un patrón simple? *Algoritmo SQUEEZE*, usa cadenas de Markov Monte Carlo. Toma varios inputs y, promediándolo, da la salida. Robusto. PROBLEMA: Muy difícil obtener los parámetros necesarios para un óptimo funcionamiento, converge lentamente, alto costo computacional.


Respecto a CHIRP, tenemos los siguientes puntos:
1) **La imagen es una función continua**:  Queremos recuperar I: \Omega \subset R^2-->R^2, I=I(l,m). En vez de discretizar (da errores en el proceso de optimización), parametrizamos la función (**la idea es reducir los errores en optimización**). Al ser continua al menos, aplicaremos la transformada de Fourier con una parametrización que use funciones pulso continuas desplazadas y escaladas (DEFINIR). Si $l \in [-F_l/2,F_l/2]$, $m \in [-F_m/2,F_m/2]$, una parametrización sería en un espacio N_lxN_m, con expresiones de m y l las ecs (7) y (8) del artículo. 

    1.1) Dada esa parametrización, por teorema desplazamiento + teorema Cittert-Zernike, obtenemos soluciones de visibilidad en términos de la transformada de h, pasando de una integración compleja a un cálculo matricial.

    1.2)Esta forma de proceder se observa que es mejor (da menos errores) que si discretizamos la imagen (figura 3, importante explicarla).

2) **Modelando la energía**: Con lo anterior, vamos a buscar una estimación MAP (enfoque estadístico que busca la solución más probable a partir de evidencias y suposiciones) de los coeficientes x, dada M (pensar vector M-dim de emv) las mediciones del bispectro complejo (llamadas y). Usamos la función de máxima verosimilitud "logaritmo de verosimilitud de parche esperado" -EPLL_r(x), y -D(y|x). Se explican a continuación:

    2.1) -D(y|x): **ES LA DISCREPANCIA ENTRE MEDICIONES OBSERVADAS Y PREDICCIONES HECHAS POR EL MODELO BASADE EN LA IMAGEN ACTUAL x**. Su expresión es la sumatorio (11), donde supuestos 3 telescopios, $k_{1,2}$,$k_{2,3}$,$k_{1,3}$ las visibilidades, e $y_k$ el ruido asociado, calculamos epsilon a partir de los vectores A etc..., otros parámetros en el artículo y supuesto ruido gaussiano $\Sigma_k > 0$, luego $\Sigma_k^{-1} > 0$. Se deduce que la expresión -D(y|x) es siempre negativa. **Nota**: Para M observaciones, el sumatorio va de k=1 a k=M, sumando para todas las componentes del vector. Se puede ponderar (11) si hubiese haz primario y esparcimiento interestelar.

    2.2) -EPLL_r(x):**APORTA RESTRICCIONES DE CÓMO DEBERÍA SER LA IMAGEN**. Es un regularizador. Ya es conocido y se apoya en que $EPLL_r(x) = \sum_{n=1}^{N} log(p(P_n \cdot x)$, con N patches pulso en la imagen, $P_n$ la matriz que extrae el n-ésimo patch de x, y p la función probablidad. Como la función probabilidad está en [0,1], el logaritmo es menor igual a 0 siempre, y -EPLL_r(x) va a ser positivo siempre. **nota**: P_n extrae el parche n de la imagen x, y se evalua. Como queremos maximizar la probabilidad, aplicando logaritmos, **esto se traduce en aproximar la suma a 0**

3) Nuestra ecuación principal es $$f_r(x|y) = -D(y|x) - EPLL_r(x)$$, que queremos minimizar (pues por lo expuesto antes, esto se traduce en que tanto -D tiende a 0, i.e., la imagen predicha se ajusta a las mediciones, y -EPLL_r converge a 0, esto es, las predicciones tienden a ser menos ruidosas). En resumen, encontrar mínimo de (9) IMPLICA que la imagen x, es lo menos ruidosa, y se acerca a las observaciones lo máximo posible.

4) **PROCESO DE OPTIMIZACIÓN**: Usamos "Half Quadratic Splitting", HQS, que usará N parches, ${z_i} i=1,..N$, tantos como los parches $P_i*x$. El proceso es iterativo y es el siguiente:
    - Paso 1: Calculamos {z_i} tal que sean los parches MÁS PROBABLES a ser los del input x, conocidos los parches corruptos {P_i*x} de x, y un parámetro $\beta$

    - Paso 2: Tras obtener las aproximaciones {z_i} a partir de x y sus parches corruptos, podemos tener dos situaciones. Si se trabaja con mediciones de la visibilidad, tendríamos una ecuación cuadrática, de solución fácil. Si no fuese el caso, que suele ser lo general, recurrimos a métodos numéricos para su cálculo: o aproximamos por polinomio de Taylor de orden 2 en torno al PVI y resolvemos, o usamos por ejemplo gradiente descendente (aunque es más lento). **Nota**: se recomienda usar para $\beta$ potencias de 2 (los autores usan \beta=2^m m=0,2,3,4,5,6,7,8,9).

    - Framework multiescala: Los autores comienzan con la primera imagen x_0, que tendrá ruido en torno a la densidad de flujo medio (intensidad media de la imagen). Se procede a hacer el algoritmo iterativo (Paso 1+Paso 2, iterativamente). Se tiene que incrementar el número de pulsos (vecindarios) en cada iteración, que describan mejor la imagen, para obtener finalmente la mejor imagen a baja resolución, y después se aumentaría la resolución de la misma. 

    ## Dataset

    Lo crean los autores. Reunen una serie de imágenes asociadas a objetos observados con VLBI en función de la longitud de onda, distancia,... 5000 mediciones simuladas (artifiales, no reales, pero se intentan acercar a lo que captaría el EHT) de objetos celestes simulados con el MIT Array Performance Simulator. Obtenidos con observaciones reales de agujeros negros, ... Hay 33 conjuntos de medidas reales usando VLBI de 43 GHz. Se proponen métricas para la evaluación.

    ## Resultados y discusión

    Para el dataset propuesto, aunque no sean observaciones del telescopio final (EHT) se arrojan buenos resultados.

        - Para las imágenes simuladas con parámetros realistas del EHT, hay buenos resultados. 

        - Comparación de los resultados de CHIRP con CLEAN, SQUEEZE y BSMEM. Para empezar, CLEAN necesita usar CASA, que necesita datos calibrados (no ruido atmosférico). Los otros dos, aunque no hace falta calibración previa, no arrojan mejores resultados que CHIRP (tablas página 7, **ponerlas y comentarlas**), aunque puede dar mejores haciendo un estudio más profundo y complejo de la situación (punto para CHIRP jeje).

        - **Sensibilidad al ruido**: Tener cuidado con el ruido introducido por los propios telescopios, puede esconder info importante si esta llega débilmente al telescopio.

        - **Efecto del Patch Prior**: Por la elección hecha del prior patch, es muy flexible, pudiendo hacer otras suposiciones sin que el resultado varíe mucho (robustez a suposiciones). Figura (8), cambian las suposiciones, en general no altera mucho el resultado.

        - **Medidas reales**: Aunque no se dispone de las imágenes reales, se compara el resultado con otras reconstrucciones de otros algoritmos, dando comparativas altas.

## Conclusiones

Se propone un algoritmo que arroja mejores resultados que los actuales (2012), a partir de observaciones con VLBI. Se introduce un dataset para interferometría. En la web, hay una comparación entre algoritmos.