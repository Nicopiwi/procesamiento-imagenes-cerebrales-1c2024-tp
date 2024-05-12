# Detección de sujetos con desórdenes neuropsiquiátricos usando marcadores extraídos de fMRI en estado de reposo y Aprendizaje Automático


--------------

## Integrantes
Pablo Groisman, Nicolás Yanovsky, Nicolás Rozenberg
## Introducción

Los trastornos neuropsiquiátricos, como el Trastorno por Déficit de Atención e Hiperactividad (TDAH), el Trastorno Bipolar (TB) y la Esquizofrenia, representan una carga significativa para la salud pública en todo el mundo debido a su impacto en el funcionamiento cognitivo, emocional y social de los individuos afectados. Estos trastornos son complejos y multifacéticos, lo que dificulta su diagnóstico y tratamiento efectivo.

En este estudio, inspirado en ideas de un trabajo similar [^1] , nos proponemos explorar si es plausible distinguir de manera binaria entre sujetos de control y aquellos diagnosticados con TDAH, Trastorno Bipolar y Esquizofrenia con técnicas de aprendizaje automático con dos enfoques distintos: Utilizando conectividad funcional, y utilizando el marcador Amplitud de Fluctuaciones de Baja Frecuencia (conocido como ALFF, acrónimo de Amplitude of Low Frequency Fluctuations) extraídos de señales BOLD (Blood Oxygenation Level Dependent) tomadas mediante fMRI, en estado de reposo. Se han realizado trabajos intentando relacionar la conectividad funcional con estas patologías [^2] [^3].  Además, se han hecho estudios con los mismos propósitos, utilizando el marcador ALFF, que por ejemplo puede estar correlacionado con patologías como esquizofrenia o AHDH, especialmente en estado de reposo [^4] [^5].


## Métodos

### Datos
Los datos son extraídos de la fuente de datos pública UCLA Consortium for Neuropsychiatric Phenomics LA5c Study (Poldrack et al., 2020)[^6], disponible en el sitio OpenNeuro, en formato BIDS (Brain Imaging Data Structure).

La muestra está compuesta por 272 sujetos, con edades del rango de entre 21 y 50 años, de los cuales 130 son sujetos de control y 142 son sujetos con trastornos neuropsiquiátricos. 50 fueron diagnosticados con esquizofrenia, 49 con trastorno bipolar y 43 con TDAH. Contiene imágenes cerebrales, entre ellas de señales BOLD de fMRI para cada sujeto en cada tarea, como también información demográfica. Cada sujeto fue sometido a diversas tareas, entre ellas un período de reposo de 304 segundos, y tareas de memoria y de atención.

Además, se encuentran disponibles las imágenes preprocesadas. Se encuentran disponibles n=265 sujetos con datos preprocesados, ya que a siete les faltaba el scan de T1-w. Entre las transformaciones que se realizaron, se aplicó la transformación del volumen de T1-w al espacio de MNI152 y se realizó corrección de movimientos a las imágenes de fMRI.

### Extracción de features

En primer lugar, separamos un 20% de los sujetos de manera equitativa que luego utilizaremos al final para validar los modelos que obtengamos y continuamos el procesamiento de los datos con el otro 80% de los sujetos, correspondientes a las señales BOLD en reposo.

En segundo lugar resulta necesario aclarar que utilizamos la herramienta Open Source Junifer[^7], una herramienta end-to-end para proyectos de aprendizaje automático aplicados a neuroimágenes, para la obtención y transformación de los datos, y luego aplicar modelos aprendizaje automático. A partir de las imágenes de fMRI preprocesadas provistas por el dataset, parcelamos el cerebro de acuerdo a Schaefer et al., 2018. [^8]

A partir de acá tomaremos dos enfoques distintos para probar los modelos. En ambos casos haremos confound removal. Para el primero, calcularemos la conectividad funcional entre las regiones obteniendo así una matriz de 100*100. Para el segundo utilizaremos como features al ALFF de cada parcela.



### ML Pipeline

Utilizando el enfoque de conectividad funcional, primero armamos la matriz de features poniendo en cada fila el vector conseguido producto de desenrollar la matriz triangular superior de conectividad funcional de cada sujeto. Al ser 100 regiones de interés, terminamos con una matriz de $N\times\frac{100^2}{2}$. 
Teniendo en cuenta que $N << \frac{100^2}{2}$ y los problemas asociados que trae esto, decidimos usar PCA para reducir las dimensiones del problema.

Probamos distintos clasificadores (Random Forest, LDA, SVM, XGBoost) y realizamos repeated (10 veces) Nested Cross Validation para estimar el error de generalización de la selección del modelo final con cross-validation en conjunto con grid search (5-fold, estratificado). Para SVM realizamos el proceso de selección de hiperparámetros para cada tipo de kernel (lineal, polinomial, rbf). A lo largo del proceso la métrica de evaluación usada siempre es el AUC-ROC. 

El modelo final seleccionado será el que mayor AUC obtuvo, pero teniendo en cuenta la consistencia de los resultados del repeated nested cross-validation. 
Finalmente reportamos su performance estimada en el set de testeo previamente separado.

Por último, repetiremos el proceso desde aplicar PCA hasta obtener métricas utilizando como features el marcador ALFF de cada parcela.


[^1]: Extraído de https://school.brainhackmtl.org/project/brotherwood_connectivity/
[^2]: Sudre G, Szekely E, Sharp W, Kasparek S, Shaw P. Multimodal mapping of the brain's functional connectivity and the adult outcome of attention deficit hyperactivity disorder. Proc Natl Acad Sci U S A. 2017 Oct 31;114(44):11787-11792. doi: 10.1073/pnas.1705229114. Epub 2017 Oct 16. PMID: 29078281; PMCID: PMC5676882.
[^3]: Syan, S., Smith, M., Frey, B., Remtulla, R., Kapczinski, F., Hall, G. and Minuzzi, L., 2018. Resting-state functional connectivity in individuals with bipolar disorder during clinical remission: a systematic review. Journal of Psychiatry and Neuroscience, 43(5), pp.298-316.
[^4]: Wang, P., Yang, J., Yin, Z. et al. Amplitude of low-frequency fluctuation (ALFF) may be associated with cognitive impairment in schizophrenia: a correlation study. BMC Psychiatry 19, 30 (2019). https://doi.org/10.1186/s12888-018-1992-4
[^5]: Zang YF, He Y, Zhu CZ, Cao QJ, Sui MQ, Liang M, Tian LX, Jiang TZ, Wang YF. Altered baseline brain activity in children with ADHD revealed by resting-state functional MRI. Brain Dev. 2007 Mar;29(2):83-91. doi: 10.1016/j.braindev.2006.07.002. Epub 2006 Aug 17. Erratum in: Brain Dev. 2012 Apr;34(4):336. PMID: 16919409.
[^6]: Bilder, R and Poldrack, R and Cannon, T and London, E and Freimer, N and Congdon, E and Karlsgodt, K and Sabb, F (2020). UCLA Consortium for Neuropsychiatric Phenomics LA5c Study. OpenNeuro. [Dataset] doi: 10.18112/openneuro.ds000030.v1.0.0. **Para más información sobre la extracción y preprocesado de los datos, consultar en https://f1000research.com/articles/6-1262/v2**
[^7]: El repositorio se encuentra en https://github.com/juaml/junifer. Desarrollado y mantenido por el grupo de Machine Learning Aplicado del Forschungszentrum Juelich.
[^8]: Schaefer et al. Local-Global Parcellation of the Human Cerebral Cortex from Intrinsic Functional Connectivity MRI (2018): Ver https://pubmed.ncbi.nlm.nih.gov/28981612/