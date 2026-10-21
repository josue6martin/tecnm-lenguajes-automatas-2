
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

> Durante la fase manual identifiqué que el principal desafío era manipular correctamente los punteros y los límites de memoria para invertir los caracteres de un buffer sin utilizar memoria adicional. Era necesario controlar las posiciones inicial y final para evitar accesos fuera de los límites y conservar el carácter de terminación de la cadena. En esta fase únicamente se ejecutó el programa base que mostraba el mensaje del reto, por lo que la inversión del buffer quedó pendiente de implementar.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Lenguaje C99, compilador GCC y bibliotecas estándar. Entorno Google Colab con GCC 13.3.0. |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores. |
| **Especificación (E)** | Invertir un buffer de caracteres (`char*`) mediante punteros, con complejidad temporal O(n), complejidad espacial O(1) y validación básica de entradas. |
| **Output (O)** | Código modular en lenguaje C99 y análisis de complejidad temporal y espacial. |

### 2.2 Prompt Estructurado

```text
Actúa como un ingeniero de software de sistemas y compiladores
experto en lenguaje C99.

Necesito desarrollar una función modular llamada
invertir_buffer_ia que reciba un puntero char* y una longitud
de tipo int para invertir los caracteres de un buffer
directamente en su ubicación original (in-place).

La implementación debe cumplir con los siguientes requisitos:

1. Utilizar punteros para recorrer el buffer desde ambos extremos.
2. Realizar la inversión sin crear otro arreglo auxiliar.
3. Cumplir con una complejidad temporal O(n).
4. Cumplir con una complejidad espacial O(1).
5. Validar punteros NULL y longitudes menores o iguales a cero.
6. Mantener la compatibilidad con el compilador GCC y C99.
7. Utilizar únicamente bibliotecas estándar de C.
8. Evitar modificar el carácter de terminación de las cadenas,
   siempre que la longitud proporcionada lo excluya.

Entrega únicamente el código modular en C99 y un análisis
breve de su complejidad temporal y espacial.
```

---

## 3. Código Devuelto por el Modelo

El siguiente código corresponde a la implementación optimizada
utilizada durante la Fase 2 en Google Colab, almacenada en el
archivo `ia_sesion01_c.c`.

```c
#include <stdio.h>
#include <string.h>

// Implementación optimizada aplicando contrato C-R-E-O
// Complejidad: O(n) tiempo y O(1) memoria.

void invertir_buffer_ia(char *buffer, int len) {
    if (buffer == NULL || len <= 0) return;

    char *left = buffer;
    char *right = buffer + len - 1;

    while (left < right) {
        char temp = *left;
        *left = *right;
        *right = temp;
        left++;
        right--;
    }
}
```

**Análisis de complejidad:**

- **Complejidad temporal O(n):** El algoritmo recorre el buffer desde ambos extremos y realiza aproximadamente n/2 intercambios, por lo que el tiempo de ejecución aumenta linealmente respecto a la longitud del buffer.
- **Complejidad espacial O(1):** Solamente utiliza dos punteros y una variable temporal para intercambiar caracteres, sin crear arreglos adicionales ni reservar memoria dinámica.
- **Modificación in-place:** Los caracteres se intercambian directamente dentro del buffer original.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

*Pruebas ejecutadas:* Valores NULL, buffers vacíos, longitudes impares y longitudes pares.

El Test Harness fue implementado en el archivo
`harness_sesion01_c.c` y compilado junto con
`ia_sesion01_c.c`, utilizando el siguiente comando:

```bash
gcc -Wall -Wextra ia_sesion01_c.c harness_sesion01_c.c -o test_bin && ./test_bin
```

**Resultados obtenidos en Google Colab:**

```text
=== AUDITORIA CRITICA: SESION 01 ===
Prueba NULL: PASO ✅
Prueba Buffer Vacío: PASO ✅
Prueba Longitud Impar ('abcde'): PASO ✅
Prueba Longitud Par ('abcd'): PASO ✅
Ejecutando pruebas de estrés... PASO ✅
```

**Verificación de requisitos:**

* [x] Caso base y límites probados superados sin *Crash*.
* [x] No se observaron accesos fuera de límites ni *Memory Leaks* en las pruebas ejecutadas; no se realizó una verificación exhaustiva con sanitizadores.
* [x] La complejidad teórica coincide con la implementada: O(n) tiempo y O(1) memoria.

### Reporte Crítico

*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

> Durante la auditoría identifiqué que la IA incluyó correctamente una validación inicial para evitar fallos al recibir punteros NULL o longitudes menores o iguales a cero. Sin embargo, la función no comprueba si la longitud proporcionada supera el tamaño real del buffer, lo que podría provocar accesos fuera de los límites de memoria.
>
> Para verificar el funcionamiento de la implementación, utilicé un Test Harness con casos de prueba que incluyeron un puntero NULL, un buffer vacío, una cadena de longitud impar ("abcde") y una cadena de longitud par ("abcd"). Todas las pruebas ejecutadas finalizaron correctamente, sin fallos de segmentación.
>
> Como medida de mejora, se identificó la necesidad de garantizar que la longitud recibida sea válida y que el buffer apunte a una región de memoria accesible y modificable. También sería recomendable utilizar `size_t` para representar la longitud en una implementación futura.
>
> La auditoría permitió comprobar que la solución cumple con los requisitos de inversión in-place y complejidad O(n) en tiempo y O(1) en memoria, aunque su seguridad depende de que el programa que llama a la función proporcione parámetros válidos.

---

**Firma digital:** josue6martin
