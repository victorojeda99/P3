PAV - P3: detección de pitch
============================

Esta práctica se distribuye a través del repositorio GitHub [Práctica 3](https://github.com/albino-pav/P3).
Siga las instrucciones de la [Práctica 2](https://github.com/albino-pav/P2) para realizar un `fork` de la
misma y distribuir copias locales (*clones*) del mismo a los distintos integrantes del grupo de prácticas.

Recuerde realizar el *pull request* al repositorio original una vez completada la práctica.

Ejercicios básicos
------------------

- Complete el código de los ficheros necesarios para realizar la detección de pitch usando el programa
  `get_pitch`.

   * Complete el cálculo de la autocorrelación e inserte a continuación el código correspondiente.

  La función de autocorrelación se define como la correlación cruzada de la señal consigo misma.

  ```cpp
  void PitchAnalyzer::autocorrelation(const vector<float> &x, vector<float> &r) const {

    for (unsigned int l = 0; l < r.size(); ++l) {
      r[l]=0;
      for(unsigned int n=l;n<x.size(); n++){
        r[l]+=x[n]*x[n-l];
      }
      r[l]=r[l]/x.size();
    }

    if (r[0] == 0.0F) //to avoid log() and divide zero 
      r[0] = 1e-10; 
  }
  ```
  La hemos calculado según la definición:
  <img src="/img/img1.png" width="300" align="center">


   * Inserte una gŕafica donde, en un *subplot*, se vea con claridad la señal temporal de un segmento de
     unos 30 ms de un fonema sonoro y su periodo de pitch; y, en otro *subplot*, se vea con claridad la
	 autocorrelación de la señal y la posición del primer máximo secundario.

	 NOTA: es más que probable que tenga que usar Python, Octave/MATLAB u otro programa semejante para
	 hacerlo. Se valorará la utilización de la librería matplotlib de Python.

   Finalmente Hemos usado Matlab para generar la gráfica y lo hemos hecho con el siguiente código:
   <img src="/img/img2.png" width="300" align="center">

   Esto nos generó la siguiente gráfica:
   <img src="/img/img3a.png" width="1200" align="center">

  En la gráfica superior obtenemos el periodo de pitch:
  T_pitch = 0.01394-0.006562 = 0.007378 = 7 ms

  Por tanto el valor del pitch es:
  pitch = 1/ T_pitch = 1/(7*10^-3) = 142.86 Hz

  Podemos ver que este valor tiene sentido ya que lo ha grabado un chico y el valor de pitch para estos suele ser al rededor de 130 Hz (de media) en cambio en las mujeres es alrededor de 220 Hz.

   * Determine el mejor candidato para el periodo de pitch localizando el primer máximo secundario de la
     autocorrelación. Inserte a continuación el código correspondiente.

  Con el código siguiente, recorremos todos los valores para encontrar el primer máximo secundario de la correlacion. Después, dentro de el bucle, comprobamos si el valor almacenado es mayor al valor actual y si esto se cumple, guardamos la posición actual como máxima y calculamos la potencia de la señal.

```cpp
vector<float>::const_iterator iR = r.begin(), iRMax = iR + npitch_min;

  for(iR = r.begin() + npitch_min; iR < r.begin() + npitch_max; iR++){
    if(*iR>*iRMax) {
      iRMax = iR;
    }
  }

    unsigned int lag = iRMax - r.begin();

    float pot = 10 * log10(r[0]);
```

  La potencia la calculamos sabiendo que la pótencia màxima coincide con la posición 0 de la autocorrelación.

   * Implemente la regla de decisión sonoro o sordo e inserte el código correspondiente.

  Para determinar si un sonido es sonoro o sordo primeramente comprobamos la poténcia de la señal, ya que si el sonido es sonoro tendrá una potencia significativamente mayor a si es sordo. Por lo que identificamos que la función de autocorrelación tiene dos máximos superiores a unos umbrales predeterminados. Esto nos ayuda a encontrar el pitch ya que solo tiene sentido calcularlo en los tramos sonoros.

```cpp
  bool PitchAnalyzer::unvoiced(float pot, float r1norm, float rmaxnorm) const {
    if ((rmaxnorm > UMBRAL_RMAXNORM || r1norm > UMBRAL_R1NORM) && pot > -UMBRAL_POT){
      return false; //voice
    }else{
      return true;  //silence
    }
  }
```

- Una vez completados los puntos anteriores, dispondrá de una primera versión del detector de pitch. El 
  resto del trabajo consiste, básicamente, en obtener las mejores prestaciones posibles con él.

  * Utilice el programa `wavesurfer` para analizar las condiciones apropiadas para determinar si un
    segmento es sonoro o sordo. 
	
	  - Inserte una gráfica con la detección de pitch incorporada a `wavesurfer` y, junto a ella, los 
	    principales candidatos para determinar la sonoridad de la voz: el nivel de potencia de la señal
		(r[0]), la autocorrelación normalizada de uno (r1norm = r[1] / r[0]) y el valor de la
		autocorrelación en su máximo secundario (rmaxnorm = r[lag] / r[0]).

		Puede considerar, también, la conveniencia de usar la tasa de cruces por cero.

	    Recuerde configurar los paneles de datos para que el desplazamiento de ventana sea el adecuado, que
		en esta práctica es de 15 ms.

    La gráfica que hemos obtenido con WaveSurfer es la siguiente
    <img src="/img/img4.png" width="1200" align="center">

    Las gráficas de la imagen superior están en el siguiente orden (de arriba a abajo): 1. Tasa de cruces por cero (ZCR) 2. El valor de la autocorrelación en su máximo secundario 3. Autocorrelación normalizada de uno 4. El nivel de potencia de la señal 5. Detector de Pitch 6. Waveform de la señal.

    Vemos que detectamos bien el tramo sonoro y la detección del pitch es correcta.

      - Use el detector de pitch implementado en el programa `wavesurfer` en una señal de prueba y compare
	    su resultado con el obtenido por la mejor versión de su propio sistema.  Inserte una gráfica
		ilustrativa del resultado de ambos detectores.
  
    La gráfica que hemos obtenido con WaveSurfer es la siguiente
    <img src="/img/img5.png" width="1200" align="center">

    Comparando las dos gráficas vemos que son bastante parecidas.

  * Optimice los parámetros de su sistema de detección de pitch e inserte una tabla con las tasas de error
    y el *score* TOTAL proporcionados por `pitch_evaluate` en la evaluación de la base de datos 
	`pitch_db/train`..

  <img src="/img/img6.png" width="1200" align="center">

   * Inserte una gráfica en la que se vea con claridad el resultado de su detector de pitch junto al del
     detector de Wavesurfer. Aunque puede usarse Wavesurfer para obtener la representación, se valorará
	 el uso de alternativas de mayor calidad (particularmente Python).
   

Ejercicios de ampliación
------------------------

- Usando la librería `docopt_cpp`, modifique el fichero `get_pitch.cpp` para incorporar los parámetros del
  detector a los argumentos de la línea de comandos.
  
  Esta técnica le resultará especialmente útil para optimizar los parámetros del detector. Recuerde que
  una parte importante de la evaluación recaerá en el resultado obtenido en la detección de pitch en la
  base de datos.

  * Inserte un *pantallazo* en el que se vea el mensaje de ayuda del programa y un ejemplo de utilización
    con los argumentos añadidos.

- Implemente las técnicas que considere oportunas para optimizar las prestaciones del sistema de detección
  de pitch.

  Entre las posibles mejoras, puede escoger una o más de las siguientes:

  * Técnicas de preprocesado: filtrado paso bajo, *center clipping*, etc.
  * Técnicas de postprocesado: filtro de mediana, *dynamic time warping*, etc.
  * Métodos alternativos a la autocorrelación: procesado cepstral, *average magnitude difference function*
    (AMDF), etc.
  * Optimización **demostrable** de los parámetros que gobiernan el detector, en concreto, de los que
    gobiernan la decisión sonoro/sordo.
  * Cualquier otra técnica que se le pueda ocurrir o encuentre en la literatura.

  Encontrará más información acerca de estas técnicas en las [Transparencias del Curso](https://atenea.upc.edu/pluginfile.php/2908770/mod_resource/content/3/2b_PS%20Techniques.pdf)
  y en [Spoken Language Processing](https://discovery.upc.edu/iii/encore/record/C__Rb1233593?lang=cat).
  También encontrará más información en los anexos del enunciado de esta práctica.

  Incluya, a continuación, una explicación de las técnicas incorporadas al detector. Se valorará la
  inclusión de gráficas, tablas, código o cualquier otra cosa que ayude a comprender el trabajo realizado.

  También se valorará la realización de un estudio de los parámetros involucrados. Por ejemplo, si se opta
  por implementar el filtro de mediana, se valorará el análisis de los resultados obtenidos en función de
  la longitud del filtro.
   

Evaluación *ciega* del detector
-------------------------------

Antes de realizar el *pull request* debe asegurarse de que su repositorio contiene los ficheros necesarios
para compilar los programas correctamente ejecutando `make release`.

Con los ejecutables construidos de esta manera, los profesores de la asignatura procederán a evaluar el
detector con la parte de test de la base de datos (desconocida para los alumnos). Una parte importante de
la nota de la práctica recaerá en el resultado de esta evaluación.
