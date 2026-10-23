# RESPUESTAS · SI3003 Workshop RAG


## T1 · Capas HNSW

| Capa   |   Teórico |   Simulado |
|:-------|----------:|-----------:|
| >= 1   | 62500     |      62675 |
| >= 2   |  3906.25  |       4005 |
| >= 3   |   244.141 |        270 |

En las capas altas hay menos nodos, lo que permite dar saltos amplios sin explorar una gran cantidad de candidatos. Después se refina la búsqueda en las capas inferiores.

## T2 · ef y k

Con k=10 y ef=5 no hay capacidad para mantener diez candidatos: ef debe ser como mínimo k. Una elección inicial razonable es ef=max(10, k) o algo mayor según el recall requerido; se aumenta si falta calidad de recuperación, a costa de latencia. ef se puede ajustar en consultas sin reconstruir el índice; ef_construction se usa cuando se construye el grafo, por lo que modificarlo requiere reconstrucción para cambiar las conexiones ya creadas.

## T3 · Inserción y búsqueda

| Aspecto              | Inserción                             | Búsqueda                              | Por qué                                         |
|:---------------------|:--------------------------------------|:--------------------------------------|:------------------------------------------------|
| ¿Se sortea capa?     | Sí, asigna nivel al nodo nuevo        | No, recorre capas existentes          | Solo la inserción crea nodos y niveles          |
| ef en capas altas    | Normalmente 1 para navegación inicial | Normalmente 1 para navegación inicial | En la parte alta interesa acercarse rápidamente |
| Fase principal       | efConstruction, conexiones y vecinos  | efSearch (ef), k resultados           | Una construye el grafo; otra consulta           |
| ¿Modifica el índice? | Sí                                    | No                                    | Consultar no añade vectores                     |

`vectorstore.add_documents(...)` inserta y `retriever.invoke(...)` busca.

## T4 · Invariante FAISS

|   Posición FAISS | Antes   | Después de borrar B sin actualizar FAISS   |
|-----------------:|:--------|:-------------------------------------------|
|                0 | A       | A                                          |
|                1 | B       | C                                          |
|                2 | C       | D                                          |
|                3 | D       | IndexError                                 |

La búsqueda original [1,2,3] pasa a devolver [C,D,IndexError] al consultar la lista. Las posiciones 1 y 2 generan errores silenciosos de correspondencia; la posición 3 produce una excepción por índice fuera de rango. Un store vectorial administrado integra IDs y payloads para mantener la correspondencia entre vector y documento (sin eliminar la necesidad de manejar borrados correctamente).

## T5 · Fuerza bruta

768 000 000 multiplicaciones/consulta × 100 consultas/s = 76 800 000 000 multiplicaciones/s = 7.68 × 10^10 multiplicaciones/s, sin incluir otras operaciones o costos de memoria. La búsqueda exacta puede volverse demasiado costosa en grandes colecciones y altos QPS. ANN sacrifica exactitud/recall para ganar velocidad y escalabilidad.
