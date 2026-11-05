
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

> Durante la implementación manual identifiqué que la principal dificultad era invertir una cadena de caracteres directamente en su espacio de memoria, sin utilizar arreglos auxiliares ni reservar memoria adicional.
>
> Para resolver el problema fue necesario utilizar dos índices, uno al inicio y otro al final de la cadena, e intercambiar sus caracteres mediante una variable temporal hasta llegar al centro.
>
> También fue importante controlar los límites de la cadena para evitar accesos inválidos y conservar el carácter de terminación `\0`.
>
> Otro aspecto que consideré fue la validación de punteros `NULL` y cadenas vacías, ya que intentar acceder a memoria mediante un puntero nulo podría provocar un fallo de segmentación.
>
> Después de completar la implementación manual en `artesano_sesion01_c.c`, ejecuté el Test Harness en Google Colab y obtuve 6 de 6 pruebas superadas, comprobando el correcto funcionamiento de la inversión in-place.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Lenguaje C99, compilador GCC 13.3.0 y bibliotecas estándar de C. Entorno Google Colab. |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores. |
| **Especificación (E)** | Implementar `void invertir(char *s)` utilizando dos índices y una variable temporal, con O(n) tiempo, O(1) memoria y validación de punteros NULL. |
| **Output (O)** | Código modular en C99, sin función `main()`, compatible con `api_sesion01.h` y acompañado de un análisis de complejidad. |

### 2.2 Prompt Estructurado

```text
Actúa como un ingeniero de software de sistemas y
compiladores experto en lenguaje C99 y manipulación
de cadenas mediante punteros.

Necesito desarrollar una función modular llamada
invertir() que permita invertir una cadena de
caracteres directamente en su misma ubicación
de memoria (in-place).

La función debe respetar exactamente la siguiente firma:

void invertir(char *s);

La implementación debe cumplir con los siguientes
requisitos:

1. Utilizar lenguaje C99 y bibliotecas estándar.
2. Incluir el archivo "api_sesion01.h".
3. Invertir los caracteres de la cadena original.
4. Utilizar dos índices y una variable temporal.
5. No utilizar malloc ni arreglos auxiliares.
6. Cumplir con complejidad temporal O(n).
7. Cumplir con complejidad espacial O(1).
8. Validar punteros NULL para evitar fallos.
9. Manejar correctamente cadenas vacías.
10. Manejar cadenas de un solo carácter.
11. Conservar el carácter de terminación '\0'.
12. No incluir la función main(), ya que el
    Test Harness contiene su propio main().
13. Compilar sin advertencias utilizando:
    -Wall -Wextra -Werror.
14. Mantener compatibilidad con GCC y C99.

Entrega únicamente el código modular de la función
y un análisis breve de complejidad temporal y espacial.
```

---

## 3. Código Devuelto por el Modelo

El siguiente código corresponde a la implementación
propuesta por la inteligencia artificial durante
la Fase 2 de Google Colab.

**Archivo:** `ia_sesion01_c.c`

```c
/* ia_sesion01_c.c - Fase 2: El Cientifico
 *
 * Implementacion de inversion de buffer in-place.
 * Compatible con api_sesion01.h.
 */

#include "api_sesion01.h"
#include <string.h>
#include <stddef.h>

void invertir(char *s) {
    if (s == NULL) {
        return;
    }

    size_t longitud = strlen(s);

    if (longitud == 0) {
        return;
    }

    size_t i = 0;
    size_t j = longitud - 1;
    char temp;

    while (i < j) {
        temp = s[i];
        s[i] = s[j];
        s[j] = temp;
        i++;
        j--;
    }
}
```

**Análisis de complejidad:**

**Complejidad temporal O(n):**

La función `strlen()` recorre la cadena para obtener
su longitud. Posteriormente, el ciclo `while` realiza
aproximadamente n/2 intercambios.

Por lo tanto, el algoritmo mantiene una complejidad
temporal O(n).

**Complejidad espacial O(1):**

La implementación utiliza dos índices de tipo
`size_t` y una variable temporal de tipo `char`.

No crea arreglos adicionales ni reserva memoria
dinámica, por lo que su complejidad espacial es O(1).

**Modificación in-place:**

Los caracteres se intercambian directamente dentro
del buffer original, sin crear copias y conservando
el carácter de terminación `\0`.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

*Pruebas ejecutadas:* Valores NULL, buffers vacíos, longitudes impares, longitudes pares y cadenas de un carácter.

**Comandos utilizados en Google Colab:**

Compilación y ejecución de la implementación manual:

```bash
gcc -std=c11 -Wall -Wextra -Werror harness_sesion01_c.c artesano_sesion01_c.c -o test_artesano && ./test_artesano
```

Compilación y ejecución de la implementación generada por IA:

```bash
gcc -std=c11 -Wall -Wextra -Werror harness_sesion01_c.c ia_sesion01_c.c -o test_ia && ./test_ia
```

**Resultados obtenidos en ambas implementaciones:**

```text
=== AUDITORIA SESION 01 - Inversion de buffer in-place ===
Caso                               esperado:              obtenido:              resultado
[ 1] invertir("HOLA")                   esperado: ALOH         obtenido: ALOH         OK
[ 2] invertir("abc")                    esperado: cba          obtenido: cba          OK
[ 3] invertir("")                       esperado:              obtenido:              OK
[ 4] invertir("AB")                     esperado: BA           obtenido: BA           OK
[ 5] invertir("a")                      esperado: a            obtenido: a            OK
[ 6] invertir(NULL)                     esperado: sin crash    obtenido: sin crash    OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion01.md
```

**Verificación de requisitos:**

* [x] Caso base y límites evaluados superados sin *Crash*.
* [x] Los seis casos ejecutados no presentaron accesos inválidos observables. La implementación no utiliza `malloc`, por lo que no reserva memoria dinámica que deba liberarse. No se ejecutaron sanitizadores para comprobar exhaustivamente los límites de memoria.
* [x] La complejidad teórica coincide con la implementada: O(n) tiempo y O(1) memoria.

### Reporte Crítico

*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

> Durante la auditoría comprobé que la inteligencia artificial implementó correctamente la inversión de cadenas, incluyendo la validación de punteros NULL, el manejo de cadenas vacías y el intercambio de caracteres sin utilizar memoria adicional.
>
> Sin embargo, identifiqué que la implementación utiliza `strlen()` para calcular la longitud de la cadena, por lo que depende de que el puntero recibido haga referencia a una cadena válida y terminada correctamente con el carácter `\0`.
>
> Si se proporcionara un buffer sin terminación nula o un puntero inválido distinto de NULL, la función podría acceder a posiciones de memoria no permitidas. Esta situación no fue evaluada por el Test Harness.
>
> Para evitar problemas durante las pruebas, trabajé con arreglos de caracteres correctamente inicializados y cadenas terminadas en `\0`. También verifiqué que la función incluyera una condición de salida al recibir un puntero NULL y que los índices utilizados para los intercambios permanecieran dentro de los límites de la cadena.
>
> Como mejora adicional, sería conveniente documentar explícitamente que la función requiere una cadena válida, modificable y terminada en `\0`, así como incorporar pruebas adicionales mediante herramientas de análisis de memoria.
>
> Finalmente, tanto la implementación manual como la generada por IA superaron las seis pruebas del Test Harness, obteniendo un resultado de 6/6 y demostrando que cumplen con los requisitos funcionales establecidos para la sesión.

---

**Firma digital:** josue6martin
