
# Bitácora de Requerimientos y Auditoría IA — Sesión 03
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Google Colab, Mágicos y Flujo de Entrega  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Josue Efrain Martin Pool  
**Matrícula:** 24531353       
**Track:** C  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Manipulación de archivos y llamadas OS  

*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Durante la implementación manual, una de las dificultades que encontré fue contar correctamente las líneas de un archivo, especialmente cuando la última línea no terminaba con un salto de línea (`\n`). También tuve que verificar que las rutas fueran válidas y cerrar correctamente los archivos después de utilizarlos. Para solucionarlo, utilicé las funciones `fopen()`, `fgetc()`, `fputs()` y `fclose()`, agregando condiciones para evitar errores con archivos inexistentes o parámetros nulos.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC en Google Colab, utilizando bibliotecas estándar y compilación con `-Wall -Wextra -Werror`. |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores, especializado en programación C y manejo de archivos. |
| **Especificación (E)** | Implementar `escribir_texto()` y `contar_lineas()` respetando las firmas de `api_sesion03.h`. Devolver 0 al escribir correctamente, -1 ante errores y el número de líneas al leer un archivo, incluyendo la última sin salto de línea. |
| **Output (O)** | Código modular en C, sin `main()`, compatible con el harness, utilizando funciones estándar y complejidad temporal O(n) y espacial O(1). |

### 2.2 Prompt Estructurado
```text
Contexto (C):
Estoy trabajando en la Sesión 03 de Lenguajes y
Autómatas II, utilizando C99 y GCC en Google Colab.
El código debe compilar con -Wall -Wextra -Werror.

Rol (R):
Actúa como un ingeniero de software especializado
en programación C, compiladores y manejo de archivos.

Especificación (E):
Implementa las siguientes funciones respetando
exactamente sus firmas:

int escribir_texto(const char *ruta, const char *texto);
int contar_lineas(const char *ruta);

Requisitos:
1. escribir_texto debe crear un archivo y escribir
   el texto recibido.
2. Debe devolver 0 si la escritura fue exitosa
   y -1 si ocurre algún error.
3. contar_lineas debe contar todas las líneas
   de un archivo.
4. Debe contar la última línea aunque no termine
   con un salto de línea.
5. Si el archivo está vacío, debe devolver 0.
6. Si el archivo no existe, debe devolver -1.
7. Validar parámetros NULL y cerrar los archivos.
8. Utilizar únicamente bibliotecas estándar.
9. No incluir la función main().

Output (O):
Devuelve únicamente el código fuente en C para
ia_sesion03_c.c y un análisis breve de complejidad.
El código debe ser compatible con api_sesion03.h
y harness_sesion03_c.c.
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion03_c.c - Fase 2: El Cientifico */

#include "api_sesion03.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

/**
 * @brief Escribe una cadena de texto en un archivo.
 * @return 0 si tiene exito, -1 si ocurre un error.
 */
int escribir_texto(const char *ruta, const char *texto) {
    if (!ruta || !texto) {
        return -1;
    }

    FILE *file = fopen(ruta, "w");
    if (!file) {
        return -1;
    }

    size_t long_texto = strlen(texto);

    if (long_texto > 0) {
        size_t escritos = fwrite(
            texto, sizeof(char), long_texto, file
        );

        if (escritos < long_texto) {
            fclose(file);
            return -1;
        }
    }

    if (fclose(file) != 0) {
        return -1;
    }

    return 0;
}

/**
 * @brief Cuenta las lineas de un archivo de texto.
 * @return Numero de lineas o -1 si ocurre un error.
 */
int contar_lineas(const char *ruta) {
    if (!ruta) {
        return -1;
    }

    FILE *file = fopen(ruta, "r");
    if (!file) {
        return -1;
    }

    int lineas = 0;
    int ch;
    int last_ch = EOF;

    while ((ch = fgetc(file)) != EOF) {
        if (ch == '\n') {
            lineas++;
        }
        last_ch = ch;
    }

    fclose(file);

    // Contar la ultima linea sin salto final
    if (last_ch != EOF && last_ch != '\n') {
        lineas++;
    }

    return lineas;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Detección de errores de compilación GCC

* [x] Caso base y límites superados sin *Crash*.
* [x] No se observaron accesos fuera de límites ni *Memory Leaks* en las pruebas realizadas.
* [x] La complejidad teórica coincide con la implementada.

**Resultados del Test Harness:**

```text
=== AUDITORIA SESION 03 - Archivos: escribir y contar lineas ===
Caso                               esperado:              obtenido:              resultado
[ 1] escribir_texto(...)==0             esperado: 0            obtenido: 0            OK
[ 2] contar_lineas(3 lineas)            esperado: 3            obtenido: 3            OK
[ 3] contar_lineas(sin salto final)     esperado: 2            obtenido: 2            OK
[ 4] contar_lineas(archivo vacio)       esperado: 0            obtenido: 0            OK
[ 5] contar_lineas(inexistente)         esperado: -1           obtenido: -1           OK
[ 6] escribir_texto(ruta invalida)      esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion03.md
```

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Al revisar el código generado por la IA, observé que sí contemplaba los casos principales, como archivos vacíos, rutas inexistentes y líneas sin salto final. Sin embargo, identifiqué que no verificaba posibles errores durante la lectura mediante `ferror()`. Para corregir esta limitación, se tendría que comprobar `ferror()` antes de cerrar el archivo y devolver -1 si ocurre un fallo. También revisé que las funciones respetaran las firmas establecidas y que el código compilara sin advertencias. Finalmente, ejecuté las seis pruebas del harness en Google Colab y obtuve 6/6 resultados correctos tanto en mi implementación manual como en la generada por IA. Aprendí que es importante revisar el código y comprobar su funcionamiento mediante pruebas automatizadas.

---
**Firma digital:** Josue Martin
