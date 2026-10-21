
# Bitácora de Requerimientos y Auditoría IA — Sesión 01

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Encuadre, Diagnóstico de Punteros y Metodología IA  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Josue Efrain Martin Pool  
**Matrícula:** 24531353  
**Track:** C  

---

## 1. Fase 1: El Artesano (Dificultad Manual)

*Reto:* Inversión de buffer in-place (char*)

*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*

> Durante la fase manual identifiqué que la principal dificultad era invertir los caracteres de una cadena utilizando su mismo espacio de memoria, sin crear arreglos auxiliares ni utilizar memoria dinámica.
>
> Para resolverlo era necesario controlar dos posiciones, una al inicio y otra al final de la cadena, e intercambiar sus caracteres hasta llegar al centro.
>
> También debía evitar acceder a posiciones inválidas de memoria, especialmente al recibir un puntero NULL o una cadena vacía.
>
> En la primera ejecución dejé pendiente la implementación de la función `invertir()`, por lo que el Test Harness obtuvo únicamente 3 de 6 pruebas correctas. Los casos de cadenas de longitud mayor que uno fallaron porque la función todavía no realizaba los intercambios.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Lenguaje C99, compilador GCC 13.3.0 y bibliotecas estándar. Entorno de Google Colab. |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores. |
| **Especificación (E)** | Implementar `void invertir(char *s)` utilizando punteros o índices, O(n) tiempo, O(1) memoria, sin malloc y con manejo de NULL. |
| **Output (O)** | Solo código modular en C compatible con `api_sesion01.h` y análisis de complejidad. |

### 2.2 Prompt Estructurado

```text
Actúa como un ingeniero de software de sistemas
y compiladores experto en lenguaje C99.

Necesito desarrollar una función llamada invertir()
que permita invertir una cadena de caracteres (char*)
directamente en su misma ubicación de memoria.

La implementación debe cumplir con los siguientes
requerimientos:

1. Utilizar lenguaje C99 y compilador GCC.
2. Respetar exactamente la firma:
   void invertir(char *s);
3. Incluir el archivo de cabecera "api_sesion01.h".
4. Invertir la cadena en su mismo espacio de memoria.
5. Utilizar dos índices y una variable temporal.
6. No utilizar malloc ni arreglos auxiliares.
7. Cumplir con complejidad temporal O(n).
8. Cumplir con complejidad espacial O(1).
9. Evitar fallos de segmentación al recibir NULL.
10. Manejar cadenas vacías y de un solo carácter.
11. Conservar el carácter de terminación '\0'.
12. No incluir una función main(), porque el
    Test Harness contiene su propio main().
13. Compilar sin advertencias utilizando:
    -Wall -Wextra -Werror.

Entrega únicamente el código modular de la función
y un análisis breve de su complejidad.
```

---

## 3. Código Devuelto por el Modelo

El siguiente código corresponde a la solución corregida
para cumplir con el contrato establecido en
`api_sesion01.h`.

```c
#include "api_sesion01.h"
#include <string.h>

// Funcion para invertir una cadena in-place.
// Complejidad temporal: O(n).
// Complejidad espacial: O(1).

void invertir(char *s) {

    // Validar el puntero NULL.
    if (s == NULL) {
        return;
    }

    // Obtener la longitud de la cadena.
    size_t longitud = strlen(s);

    // Evitar intercambios innecesarios.
    if (longitud < 2) {
        return;
    }

    // Definir las posiciones inicial y final.
    size_t izquierda = 0;
    size_t derecha = longitud - 1;

    // Intercambiar caracteres hasta llegar al centro.
    while (izquierda < derecha) {

        char temporal = s[izquierda];

        s[izquierda] = s[derecha];
        s[derecha] = temporal;

        izquierda++;
        derecha--;
    }
}
```

**Análisis de complejidad:**

**Complejidad temporal O(n):**

La función `strlen()` recorre la cadena para determinar
su longitud. Posteriormente, el ciclo `while` realiza
aproximadamente n/2 intercambios.

Por lo tanto, el tiempo total es O(n).

**Complejidad espacial O(1):**

El algoritmo utiliza únicamente dos variables para
controlar las posiciones y una variable temporal
para realizar los intercambios.

No utiliza memoria dinámica ni crea arreglos auxiliares.

**Modificación in-place:**

Los caracteres se intercambian directamente en la
cadena original sin modificar el carácter '\0'.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

*Pruebas ejecutadas:* Valores NULL, buffers vacíos, longitudes impares, longitudes pares y cadenas de un carácter.

**Resultados de la ejecución inicial en Google Colab:**

```text
=== AUDITORIA SESION 01 - Inversion de buffer in-place ===
Caso                               esperado:              obtenido:              resultado
[ 1] invertir("HOLA")                   esperado: ALOH         obtenido: HOLA         FALLA
[ 2] invertir("abc")                    esperado: cba          obtenido: abc          FALLA
[ 3] invertir("")                       esperado:              obtenido:              OK
[ 4] invertir("AB")                     esperado: BA           obtenido: AB           FALLA
[ 5] invertir("a")                      esperado: a            obtenido: a            OK
[ 6] invertir(NULL)                     esperado: sin crash    obtenido: sin crash    OK

RESULTADO: 3/6 pruebas superadas
Aun hay 3 caso(s) sin resolver. Revisa tu implementacion.
```

**Resultado de la prueba inicial del archivo de IA:**

```text
undefined reference to `invertir'
collect2: error: ld returned 1 exit status
```

Este error ocurrió porque el archivo `ia_sesion01_c.c`
todavía no contenía una implementación de la función
`invertir()` requerida por el contrato.

**Verificación de requisitos:**

* [ ] Caso base y límites superados sin *Crash* en los seis casos. La ejecución inicial obtuvo 3/6; falta ejecutar nuevamente el código corregido.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*. No se detectaron fallos en los casos iniciales, pero falta una comprobación adicional de memoria.
* [x] La complejidad teórica de la implementación corregida coincide con O(n) tiempo y O(1) memoria.

### Reporte Crítico

*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

> Durante la auditoría identifiqué que el principal problema era que el archivo correspondiente a la inteligencia artificial no contenía la implementación de la función `invertir()`, por lo que el compilador no podía enlazar correctamente el programa con el Test Harness.
>
> Además, la implementación manual inicial no realizaba ningún intercambio, lo que provocó que las cadenas "HOLA", "abc" y "AB" permanecieran sin modificaciones. Por esta razón, únicamente se superaron 3 de las 6 pruebas.
>
> Para solucionar estos problemas, preparé una implementación que respeta la firma `void invertir(char *s)` definida en `api_sesion01.h`, utiliza dos índices para recorrer la cadena y una variable temporal para intercambiar sus caracteres.
>
> También agregué una validación para evitar desreferenciar punteros NULL y una condición para manejar correctamente cadenas vacías o de un solo carácter.
>
> La solución corregida mantiene una complejidad temporal O(n) y espacial O(1), sin utilizar memoria dinámica. Queda pendiente ejecutar nuevamente el Test Harness para confirmar que los seis casos se superan correctamente.

---

**Firma digital:** josue6martin
