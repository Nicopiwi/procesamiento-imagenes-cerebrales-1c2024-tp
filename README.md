# Trabajo final de asignatura Procesamiento de Imágenes Cerebrales con aplicaciones a Machine Learning

## Título

Detección de sujetos con desórdenes neuropsiquiátricos usando marcadores extraídos de fMRI en estado de reposo y Aprendizaje Automático

## Integrantes 

Pablo Groisman, Nicolás Yanovsky, Nicolás Rozenberg

## Informes

- [Preentrega](https://github.com/Nicopiwi/picml-1c2024-tp/blob/main/doc/pre.md)
- [Post-análisis](https://github.com/Nicopiwi/picml-1c2024-tp/blob/main/doc/post.md)

## Reproducción


### Configurar environment

- Crear environment de Conda

```
conda create -n picml_env python=3.11
conda activate picml_env
```

- Instalar [Datalad](https://www.datalad.org/)

```
conda install -c conda-forge datalad
```

- Instalar dependencias

```
pip install -r requirements.txt
```

- Instalar ANTs

Los pasos de instalación se pueden ver en https://github.com/ANTsX/ANTs?tab=readme-ov-file


### Visualizar exploración de modelos

La exploración de los modelos se encuentra en el archivo `./src/notebook.ipynb`
